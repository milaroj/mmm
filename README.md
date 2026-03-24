# Laboratorio 4

**Semana #5: 18/Marzo/2026**  
**Taller de Diseño Digital - EL3313: I Semestre 2026**  
**Profesor: Luis G. León-Vega, Ph.D**

---

### Integrantes

| Nombre | Carné |
|---|---|
| Angie Hernández Mairena | 2021093932 |
| Brayan Solís Rojas | 2020168427 |
| Milagro Rojas Sánchez | 2020412342 |

---

## Objetivo

Ejercitar los conceptos vistos durante la parte teórica del taller mediante la exposición del estudiante ante proyectos prácticos sencillos, con la finalidad de asimilar el flujo de trabajo del diseño digital con EDA.

---

## Preguntas de Seguimiento del Aprendizaje

### 1. Complete la Tabla 1

| Reporte | Uniciclo | Multiciclo | Segmentado |
|---|---|---|---|
| Consumo CLBs (%) | 1.11% | 0.29% | 1.10% |
| Consumo DSPs (%) | 0% | 0% | 0% |
| Retraso crítico | 2.682 ns | 9.929 ns | 8.282 ns |
| Frecuencia máxima | 372.9 MHz | 100.7 MHz | 120.7 MHz |

> **Nota metodológica — cómo obtener estos datos en Vivado:**
>
> **Consumo de CLBs y DSPs** → luego de correr *Run Implementation*, ir a:
> `Reports → Report Utilization`
> En la tabla resultante buscar:
> - **Slice LUTs** bajo la sección *CLB Logic*: `LUTs usados / 63400 × 100`
> - **DSPs** bajo la sección *DSP*: `DSPs usados / 240 × 100`
>
> **Retraso crítico y Frecuencia máxima** → luego de correr *Run Implementation*, ir a:
> `Reports → Report Timing Summary`
> Buscar el valor de **WNS (Worst Negative Slack)** y aplicar:
> ```
> Retraso crítico = Período de reloj − WNS
>                = 10 ns − WNS
>
> Frecuencia máxima = 1 / Retraso crítico
> ```
> Por ejemplo, con WNS = 7.318 ns:
> ```
> Retraso crítico = 10 − 7.318 = 2.682 ns
> Frecuencia máxima = 1 / 2.682 ns ≈ 372.9 MHz
> ```

---

### 2. Basado en las observaciones en los diagramas de elaboración, ¿por qué las distintas microarquitecturas consumen menos o más recursos? ¿Cómo puede asociarlo a lo visto en las microarquitecturas de RISC-V?

> La microarquitectura uniciclo instancia 8 multiplicadores y un árbol de sumas combinacional simultáneamente, lo que se traduce en mayor uso de LUTs. La segmentada tiene un consumo similar porque aunque divide el cómputo en etapas, sigue instanciando los mismos 8 multiplicadores en paralelo más registros de pipeline adicionales. La multiciclo consume significativamente menos recursos (0.29% vs ~1.1%) porque reutiliza un único multiplicador y un acumulador, a costa de requerir más ciclos por resultado.
>
> Esto es análogo a las microarquitecturas de RISC-V: el procesador uniciclo instancia toda la ruta de datos de forma paralela (ALU, memorias, sumadores de PC) sin compartir hardware, mientras que el multiciclo reutiliza la misma ALU en diferentes etapas de la instrucción. El segmentado de RISC-V, al igual que en este lab, mantiene múltiples unidades activas simultáneamente pero agrega registros entre etapas, por lo que su uso de recursos es comparable al uniciclo.

---

### 3. En sus propias palabras, ¿cuándo se debe usar una microarquitectura u otra? Ejemplifique en casos como: bajo consumo de potencia con alta duración de batería, computación y medicina.

> La microarquitectura **uniciclo** es adecuada cuando se necesita latencia mínima y el diseño es simple. En aplicaciones de **bajo consumo con alta duración de batería** (ej. sensores IoT, wearables), sin embargo, no es la mejor opción porque mantiene toda la lógica activa aunque no se use, disipando más energía estática.
>
> La microarquitectura **multiciclo** es ideal para sistemas con **restricciones de área y potencia**, como dispositivos médicos implantables (marcapasos, monitores de glucosa) o nodos de sensores con batería. Al reutilizar hardware, reduce el área del chip y el consumo, aceptando una latencia mayor como compromiso.
>
> La microarquitectura **segmentada** es la elección en aplicaciones de **alto rendimiento sostenido**, como procesamiento de señales médicas en tiempo real (imágenes de resonancia magnética, ultrasonido) o aceleradores de cómputo científico. Aunque tiene mayor latencia inicial, su throughput de un resultado por ciclo la hace superior cuando se procesan grandes volúmenes de datos continuamente.

---

### 4. De acuerdo con su implementación del banco de registros, ¿qué tanto difiere su implementación del banco de registros de un procesador?

> _Una implementación de RISC-V por ejemplo, tiene 32 registros de 32 bits que ofrecen bastante espacio y capacidad para realizar operaciones. Mientras esta implementación 'reg_bank' solo tiene 16 registros de 8 bits. Por otro lado, 'reg_bank' tiene la habilidad de que cualquier registro puede tener cualquier valor pero las implementaciones RISC-V tienen el registro 0 en ese mismo valor por hardware lo que vuelve al primero ligeramente mas flexible. La lectura de RISC-V y de 'reg_bank' tienen igual cantidad de puertos de lectura y de escritura, pero en RISC-V la dirección viene del decodificador de instrucciones mientras en el caso de la implementación de 'reg_bank' viene de switches físicos. 

---

### 5. Si le solicitan cuál microarquitectura ofrece el mejor balance entre rendimiento y consumo de recursos, ¿cuál elegiría y por qué?

> La microarquitectura **multiciclo** ofrece el mejor balance para este diseño específico. Con solo 0.29% de CLBs frente al ~1.1% de las otras dos, reduce drásticamente el uso de recursos sin sacrificar correctitud. Si bien su frecuencia máxima (100.7 MHz) es la más baja de las tres, sigue siendo más que suficiente para operar a 100 MHz en la Nexys A7. El costo de 9 ciclos por resultado es aceptable en aplicaciones donde los vectores no cambian continuamente. La arquitectura segmentada sería preferible únicamente en escenarios de procesamiento en flujo continuo donde el throughput sostenido sea crítico.

---
