# Perfilado del Prototipo en C — MobileNetV2

Guía de reproducibilidad para medir el desempeño del prototipo en CPU
(entrega "Prototipo en C", semana 7 de EL5859). Este documento describe
**cómo** se mide el tiempo, en qué orden, y qué se espera obtener en cada
paso. Se actualiza a medida que se va ejecutando cada etapa.

El perfilado se enfoca únicamente en las capas implementadas en **C++**
(la inferencia de MobileNetV2 en sí). El preprocesamiento de la imagen
(decodificar, redimensionar, normalizar) corre en Python/PyTorch y
queda fuera del alcance de esta medición.

## Estado general

| Paso | Descripción | Estado |
|------|-------------|--------|
| 1 | Instrumentación por capa C++ (`std::chrono`) | Hecho |
| 2 | Muestreo (>100 corridas, warm-up) | Hecho |
| 3 | CSV detallado por capa | Hecho |
| 4 | Overhead de la instrumentación | Hecho |
| 5 | Validación externa (`perf`) | Hecho |

---

## 1. Instrumentación manual por capa (`std::chrono`)

Se mide el tiempo de cada capa C++ individualmente:

- Conv2D (capa inicial)
- Depthwise Conv2D
- Pointwise Conv2D
- BatchNorm2D
- Activación (ReLU6)
- Pooling (Global Average Pool)
- Suma residual (Layer Add)
- Salida (Linear / clasificador)

**Cómo:** cada llamada a un kernel, dentro de `bindings/pybind_module.cpp`,
queda envuelta en un `ScopedTimer` (`kernels/Profiler.hpp`) que usa
`std::chrono::high_resolution_clock`. Cada registro guarda: tipo de
operación (`etapa`), forma del tensor de entrada, y duración en ms. El
buffer se expone a Python vía `cpp_kernels.get_and_clear_profile_records()`
y `cpp_kernels.clear_profile_records()`.

**Estado:** implementado y verificado — una inferencia completa produce
151 registros, que coincide exactamente con el conteo teórico de
operaciones del modelo (17 bloques + capa inicial + capa final + pooling
+ clasificador). El preprocesamiento en Python no pasa por este buffer,
así que no contamina la medición.

## 2. Muestreo

- Se corre la inferencia **más de 100 veces** sobre el mismo tensor de
  entrada (o ciclando sobre un set chico de tensores reales, ej. 2–3
  imágenes distintas).
- Warm-up: 5–10 corridas completas descartadas antes de empezar a medir,
  para evitar medir efectos de arranque en frío (cachés vacías,
  frecuencia de CPU aún no estabilizada).
- No hace falta un dataset de ~100 imágenes distintas: las capas C++ son
  loops sin ramificación dependiente del contenido (el tiempo depende de
  la forma del tensor, no de los valores), así que repetir el mismo
  tensor mide correctamente el desempeño de cómputo. La única razón para
  usar un set chico de tensores reales en vez de uno solo es cubrir el
  caso de valores "raros" (por ejemplo, muchos ceros/subnormales después
  de ReLU6) que en teoría podrían cambiar ligeramente el tiempo de
  aritmética de punto flotante.
- Herramienta: `app/profile_run.py` (ver más abajo).

**Criterio de aceptación:** al menos 100 iteraciones medidas registradas
en el CSV, además de las corridas de warm-up (descartadas, no aparecen
en el CSV).

## 3. CSV con detalle por capa

Salida esperada: `resultados/perfilado.csv`, con una fila por cada
operación ejecutada:

| Columna | Descripción |
|---|---|
| `etapa` | tipo de operación (`conv2d`, `depthwise_conv`, `pointwise_conv`, `batchnorm`, `activacion`, `pooling`, `layer_add`, `linear`) |
| `indice_llamada` | posición secuencial de la llamada dentro de esa corrida (0..150) — como el forward es determinístico, esta posición identifica de forma consistente a qué capa/bloque corresponde |
| `forma_tensor` | forma del tensor de entrada a esa operación, ej. `1x32x112x112` |
| `iteracion` | número de corrida (1..N) |
| `imagen` | qué imagen/tensor de entrada se usó en esa corrida |
| `tiempo_ms` | duración medida en milisegundos |

Esto permite luego agregar por tipo de operación (¿cuánto del tiempo
total es depthwise conv?) o ver la variación entre corridas para una
misma capa (ej. detectar el ruido de sistema que ya vimos entre dos
`batchnorm` de la misma forma: 1.10 ms vs 9.47 ms en una sola corrida).

Se genera corriendo:

```bash
PYTHONPATH="$(pwd)" venv/bin/python app/profile_run.py \
    --images test_images/imagen.jpeg \
    --iterations 120 \
    --warmup 8 \
    --output resultados/perfilado.csv
```

**Criterio de aceptación:** el CSV se puede cargar y agregar (ej. con
pandas) sin filas corruptas o vacías.

## 4. Overhead de la instrumentación

Se corre la inferencia dos veces bajo las mismas condiciones:

1. **Sin instrumentación** — build de `pybind_module.cpp` sin los
   `ScopedTimer` (o comentando temporalmente las llamadas al profiler).
2. **Con instrumentación** — build actual, con las mediciones activas.

Se compara el tiempo total de ambas corridas para reportar cuánto tiempo
extra introduce el medir. Se espera que sea pequeño (unos pocos % del
tiempo total); si no lo es, hay que revisar si la instrumentación está
mal ubicada.

**Criterio de aceptación:** overhead reportado en % y en ms absolutos.

**Observación (resultado obtenido):** comparando `make build` (sin
`PROFILE_KERNELS`) vs `make build PROFILE=1`, con 200 corridas
(`--overhead-only`) cada uno: 286.40 ms/corrida sin instrumentar vs
281.45 ms/corrida instrumentado. La instrumentada salió "más rápida",
lo cual no es un efecto real — indica que el overhead introducido por
`std::chrono` es **menor que el ruido normal entre corridas** (~1-2%),
no medible con una sola comparación de 200 corridas. Conclusión para el
reporte: overhead despreciable / indistinguible del ruido de medición.

## 5. Validación con herramienta externa (`perf`)

Se corre `perf stat` y `perf record` sobre el binario ya compilado, para
confirmar que el "hotspot" que reporta la instrumentación manual
coincide con el hotspot real a nivel de instrucciones de CPU.

```bash
perf stat -- venv/bin/python app/profile_run.py --iterations 30
perf record -- venv/bin/python app/profile_run.py --iterations 30
perf report
```

(Puede requerir `sudo apt install linux-tools-common linux-tools-$(uname -r)`
y permisos de `perf_event_paranoid` según la máquina.)

**Criterio de aceptación:** la función que más tiempo consume según
`perf report` coincide con la etapa que más tiempo acumula en el CSV
(sección 3).

**Observación (resultado obtenido, máquina Dell G16, ver "Entorno de
prueba" abajo):** `perf report` da como hotspot principal
`pointwise_conv2d_forward` con **59.45%** de los ciclos de CPU, seguido
de `depthwise_conv2d_forward` (8.74%), `batchnorm2d_forward` (5.71%),
`conv2d_forward` (4.51%) y `relu6_forward` (4.25%). El orden coincide
exactamente con el de la instrumentación manual (paso 3): pointwise
domina, seguido de depthwise. El porcentaje relativo difiere un poco
entre ambas técnicas (71% vs 59% para pointwise) porque `perf` cuenta
ciclos de CPU reales (incluye stalls de memoria) mientras `std::chrono`
mide tiempo de reloj de pared (incluye scheduling del SO) — pero ambas
técnicas validan al mismo cuello de botella.

Otro dato relevante de `perf stat` sobre la misma corrida: IPC
(instrucciones por ciclo) de solo ~1.9-2.4, y ~33-37% de los ciclos en
`tma_backend_bound` — evidencia de que el código no está vectorizado y
de que el acceso a memoria es un factor limitante, justo lo que hay que
atacar en la etapa de optimización con paralelismo.

---

## Entorno de prueba

| Característica | Valor |
|---|---|
| Equipo | Dell G16-7620 |
| Procesador | Intel Core i7-12700H (12ª gen) |
| Arquitectura | x86-64 |
| Set de instrucciones vectoriales | AVX2 |
| Núcleos / hilos | 14 núcleos / 20 hilos |
| Frecuencia base / turbo | 2.3 GHz / 4.7 GHz |
| RAM | 16 GiB (2×8 GiB), SODIMM 4800 MHz |

Falta agregar aquí la segunda computadora personal y el sistema
empotrado (requisito del enunciado, sección "Prototipo en C").

---

## Reproducibilidad (entorno base)

```bash
cd Proyecto-ICH
make setup
make build
make run
```

Debe terminar con `RESULTADO: PASS` antes de empezar cualquier
perfilado — si no pasa, el kernel no es numéricamente correcto y
cualquier medición de tiempo no tiene sentido todavía.

## Notas

- La resolución de entrada (224, 192, 160, ...) es un eje de experimento
  aparte al muestreo de la sección 2: ya se confirmó que el tiempo
  escala con el tamaño espacial (∼cuadrático), no con el contenido de la
  imagen. Ver `app/run_cpp_mobilenetv2.py`, que ya acepta una resolución
  opcional como segundo argumento.
- Máquinas de prueba: se necesita correr esto en al menos dos
  computadoras personales y un sistema empotrado (requisito del
  enunciado), así que el script de perfilado debe poder correr igual en
  las tres sin cambios manuales.
