# VGA Digital Clock FPGA **PEGAR EL LINK PÚBLICO AQUÍ**

## Integrantes

- Mauro Navarro Acuña
- Josué Arce Cruz
- Gabriel Arguedas Guzmán

---

# Primer avance (Semana 9)
## 8/Abr/2026 - 10/Abr/2026
Se conversó entre los integrantes del grupo sobre el diagrama de bloques del proyecto y se investigó con ayuda de inteligencia artificial utilizando referencias de la web como apoyo para el funcionamiento del proyecto del reloj en VGA. Una vez realizada la investigación y se comprendió el flujo se optó por utilizar la pagina web Lucidchart.com para realizar la imagen del diagrama de flujo, la cual se presentó el día 10 de abril del 2026.

# Creación del proyecto en Vivado
## 17/Abr/2026 - 19/Abr/2026
**Gabriel Arguedas:**
Estuve trabajando en vivado y realizando algunos módulos, basandome en el libro de teoría EL3313__Book.pdf y en algunos laboratorios anteriores como los de la calculadora o el semaforo, ademas de que me puse a buscar informacion sobre los debouncer, divisores de reloj, displays de 7 segmentos ya que estos módulos son parte fundamental del funcionamiento del proyecto. Una vez obtuve esa información del libro utilicé la inteligencia artificial para lograr plasmar correctamente la síntesis de vivado y poder crear de una correcta manera (quizá no la mas eficiente) los siguientes módulos:
- clock_divider.v (Corresponde al divisor de reloj, creado para el conteo de segundos)
- debouncer.v (Evita el rebote en los botones y switches de la FPGA)
- display_7seg.v (Muestra la hora y el conteo de los ss, mm y hh en los displays correspondientes)
- parte del top_reloj.v (La parte que se encarga de llevar a cabo el conteo, debouncer, divisor de reloj y mostrar en los displays de 7 segmentos)
- control_hora_manual.v (Se encarga de llevar el conteo en segundos y hacer funcional el modo editor de hora).

También realicé pruebas y simulaciones en vivado para ver como avanzaba el conteo de los ss, mm y hh, esto para verificar que funcionara correctamente, lo cual afortunadamente si funcionó, como ya la simulacion me servía le pasé lo que tenía a los compañeros por el grupo de whatsapp y ahí el compañero Josué hizo la prueba del código en la FPGA y funcionó correctamente.

---

# Simulación y banco de pruebas (Testbench)
## 20/Abr/2026 - 30/Abr/2026

Las pruebas de simulación se realizan con **Icarus Verilog** (`iverilog`) y su ejecutor `vvp`, herramientas libres disponibles para Linux.

## Instalación de Icarus Verilog

Primero verificá si ya lo tenés instalado:

```bash
iverilog -V
```

Si el comando no se reconoce, instalalo con:

```bash
sudo apt update
sudo apt install iverilog
```

Una vez instalado, podés confirmar con:

```bash
iverilog -V
vvp -V
```

## Ejecutar todos los testbenches

Desde la raíz del proyecto, entrá a la carpeta `scripts/` y ejecutá el script general:

```bash
cd scripts
bash run_tests.sh
```

Este script corre todos los módulos en orden (primero los módulos base, luego el sistema VGA completo) y muestra un resumen final de `PASS` / `FAIL`. Los archivos compilados y logs quedan guardados en `sim/out/`, que se crea automáticamente si no existe.

La salida se verá similar a esto:

```
==============================================
 Ejecutando testbenches - VGA Digital Clock
==============================================

── Etapa 1: Módulos base ──────────────────────
► tb_clock_divider ... PASS (3 checks)
► tb_clk_25MHz ... PASS (2 checks)
► tb_debouncer ... PASS (4 checks)
► tb_control_hora_manual ... PASS (5 checks)
► tb_vga_sync ... PASS (3 checks)
► tb_vram ... PASS (2 checks)
► tb_font_rom ... PASS (1 checks)
► tb_display_7seg ... PASS (3 checks)

── Etapa 2: Sistema funcional VGA ────────────
► tb_img_gen ... PASS (2 checks)
► tb_top_reloj ... PASS (6 checks)

==============================================
 RESUMEN
==============================================
 Total:  10
 PASS:   10
 FAIL:   0
==============================================
```

## Ejecutar un testbench individual

Si solo querés probar un módulo en particular, podés llamar directamente a su script desde la carpeta `scripts/`:

```bash
cd scripts
bash run_tb_<nombre_modulo>.sh
```

Los scripts disponibles son:

| Script | Módulo que prueba |
|---|---|
| `run_tb_clk_25MHz.sh` | Generador de reloj 25 MHz |
| `run_tb_clock_divider.sh` | Divisor de reloj (1 Hz) |
| `run_tb_debouncer.sh` | Debouncer de botones |
| `run_tb_control_hora_manual.sh` | Control y edición de hora |
| `run_tb_vga_sync.sh` | Sincronización VGA |
| `run_tb_vram.sh` | Memoria de video (VRAM) |
| `run_tb_font_rom.sh` | ROM de fuente/dígitos |
| `run_tb_display_7seg.sh` | Display de 7 segmentos |
| `run_tb_img_gen.sh` | Generador de imagen VGA |
| `run_tb_top_reloj.sh` | Sistema completo (top level) |

## ¿Qué significan los resultados?

| Resultado | Significado |
|---|---|
| `PASS (N checks)` | El módulo pasó N verificaciones sin errores |
| `FAIL (N fallos, M OK)` | Hubo N errores en las verificaciones del testbench |
| `INFO (sin assertions)` | El testbench corrió bien pero no tiene verificaciones definidas |
| `ERROR DE COMPILACIÓN` | El código no compiló; revisar el log en `sim/out/` |

Si hay un `FAIL`, el script muestra directamente en pantalla los mensajes de error para facilitar el diagnóstico.

---

# Declaratoria de uso de inteligencia artificial

Durante el desarrollo de este proyecto se utilizó inteligencia artificial como herramienta de apoyo en prácticamente todas las etapas del trabajo.

En una primera instancia, la IA fue útil para entender conceptos que al comienzo no eran del todo claros para el equipo, como el estándar VGA, la sincronización de señales, el manejo de memorias en FPGA o el comportamiento de un debouncer a nivel de hardware. En esos casos se usó de forma similar a como se consulta documentación técnica, siempre contrastando con el libro del curso y los materiales de laboratorio.

También se utilizó para la generación de código Verilog. Esto no fue simplemente copiar respuestas, sino un proceso donde el equipo describía el problema, revisaba lo que la IA proponía, lo ajustaba al contexto del proyecto y lo verificaba mediante simulación. En varios módulos el código generado necesitó correcciones antes de funcionar bien.

En resumen, la IA estuvo presente a lo largo de todo el proyecto, tanto para aprender como para construir, pero las decisiones de diseño, la verificación del funcionamiento y la comprensión del sistema fueron trabajo del equipo.
