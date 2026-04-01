# Cables y materiales — Plataforma 1 (Gabinete PLC)

---

## Calibres de cable interno

| Circuito | Calibre | Color (ver código) | Tipo |
|----------|---------|--------------------|------|
| Entradas digitales I1–I8 | AWG 22 / 0.5mm² | Verde | Flexible THHN o control |
| Entradas analógicas AI1–AI6 | AWG 22 / 0.5mm² | Azul claro | Flexible THHN o control |
| Salidas analógicas AO1–AO2 | AWG 22 / 0.5mm² | Naranja | Flexible THHN o control |
| Salidas PWM P1–P4 | AWG 22 / 0.5mm² | Blanco | Flexible THHN o control |
| Alimentación 24V a módulos | AWG 18 / 0.75mm² | Rojo (+) / Negro (−) | Flexible THHN o control |
| Salidas relé O1–O4 | AWG 16 / 1.5mm² | Amarillo | Flexible THHN o control |
| **Termopar CELSIUS** | — | Gris | **Cable de termopar tipo K compensado** ⚠️ |

> ⚠️ **Importante — Termopar:** No usar cable de cobre normal. El cable de termopar tipo K tiene conductores de aleación específica (Chromel/Alumel). Usar cobre genera una unión parásita que introduce error en la medición de temperatura.

---

## Lista completa de materiales — Plataforma 1

### Ya en la cotización
| Ítem | Cant. cotizada | Estado |
|------|---------------|--------|
| Riel DIN perforado CP644 1MT | 1 | ⚠️ Insuficiente — ver nota |
| Ducto ranurado 20×25mm | 1 | ⚠️ Revisar cantidad |
| WAGO 2202 2.5mm² gris | 100 | ✓ |
| WAGO 2202 2.5mm² verde/tierra | 10 | ✓ |

> ⚠️ **Riel DIN:** El gabinete tiene 3 rieles de ~450mm cada uno = ~1.35m mínimo. Con 1 metro no alcanza. Pedir **2 metros** para tener margen de corte.
>
> ⚠️ **Ducto:** El diseño tiene canaleta superior, media 1, media 2 y laterales. Verificar cuántos metros lineales se necesitan según las dimensiones del gabinete.

---

### Faltantes — agregar a la cotización

#### Accesorios de borneras
| Ítem | Cant. estimada | Nota |
|------|---------------|------|
| Borneras rojo/negro 2.5mm² (alimentación Riel 2) | 8 pares (16 uds) | Equivalente Dinkle DK4N o similar |
| End stops / topes finales de riel | 12 uds | 2 por extremo de riel × 3 rieles |
| Divisores de grupo | 16 uds | Para separar grupos de señales visualmente |
| Puentes de cortocircuito (jumper bar) | 4 tiras | Para unir borneras de +24V o GND comunes |
| Marcadores de cable numerados (clip) | 1 juego | Para identificar I1–I8, O1–O4, AI1–AI6, etc. |

#### Cable interno
| Color | Calibre | Circuito | Metros estimados* |
|-------|---------|----------|-----------------|
| Rojo | 0.75mm² | +24V alimentación | ~3m |
| Negro | 0.75mm² | GND alimentación | ~3m |
| Verde | 0.5mm² | Entradas digitales × 8 | ~6m |
| Azul claro | 0.5mm² | Entradas analógicas × 6 | ~5m |
| Naranja | 0.5mm² | Salidas analógicas × 2 | ~2m |
| Blanco | 0.5mm² | Salidas PWM × 4 | ~3m |
| Amarillo | 1.5mm² | Salidas relé × 4 | ~4m |
| Gris | Tipo K compensado | Termopar | ~1m |

*Estimados para gabinete 500×600mm con canaletas. Pedir 20–30% extra por errores de corte.

#### Entradas/salidas al gabinete
| Ítem | Cant. | Nota |
|------|-------|------|
| Prensaestopa para cable 110VAC | 1 | Entrada de alimentación desde tomacorriente |
| Extensor USB-C con panel mount | 1 | Para programar el Opta sin abrir el gabinete |
