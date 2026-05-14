[README (2).md](https://github.com/user-attachments/files/27743116/README.2.md)
# 🕐 Controlador VGA con Reloj Digital

> Implementación en FPGA de un reloj digital con salida VGA, desarrollado en Verilog para la tarjeta **Nexys A7 (Artix-7)**. Permite visualizar y ajustar horas, minutos y segundos en pantalla a través de botones y switches físicos.

---

## 📋 Tabla de Contenidos

- [Descripción General](#descripción-general)
- [Diagrama de Bloques](#diagrama-de-bloques)
- [Análisis de Timing](#análisis-de-timing)
- [Consumo de Recursos](#consumo-de-recursos)
- [Consumo de Potencia](#consumo-de-potencia)
- [Requisitos](#requisitos)
- [Instrucciones de Reproducción](#instrucciones-de-reproducción)
- [Estructura del Proyecto](#estructura-del-proyecto)

---

## 📖 Descripción General

Este proyecto implementa un **reloj digital de tiempo real** que se visualiza mediante el protocolo VGA a una resolución de **640×480 @ 60 Hz**. El diseño está compuesto por módulos independientes que gestionan la sincronización VGA, la lógica del reloj, la memoria de video (VRAM), el generador de imagen, la fuente de caracteres numéricos (font ROM) y la decodificación para displays de 7 segmentos.

Los displays de 7 segmentos de la tarjeta se utilizan como salida auxiliar para verificar el correcto funcionamiento del reloj y el control de hora en tiempo real.

### Control de hora

El modo de operación se selecciona mediante dos switches:

| Switch J15 (RST) | Switch L16 | Switch M13 | Modo                        |
|:----------------:|:----------:|:----------:|-----------------------------|
| 1                | X          | X          | Reset → 00:00:00            |
| 0                | 0          | 0          | Flujo normal (reloj corre)  |
| 0                | 0          | 1          | Modificar **segundos**      |
| 0                | 1          | 0          | Modificar **minutos**       |
| 0                | 1          | 1          | Modificar **horas**         |

Una vez seleccionado el campo a editar, se usan los botones:

| Botón       | Pin  | Función       |
|-------------|------|---------------|
| BTNU        | M18  | Incrementar   |
| BTND        | P18  | Decrementar   |

---

## 🧱 Diagrama de Bloques

```
                         ┌──────────┐
                         │  Clock   │
                         │ (100 MHz)│
                         └────┬─────┘
                              │ 100 MHz
                         ┌────▼──────┐
                         │ Divisor   │──── 1 Hz ────────────────────────┐
                         │   Clk     │                                  │
                         └────┬──────┘                                  │
                              │ 25 MHz (a todos los módulos inferiores)  │
    ┌──────────┐              │                                          │
    │ Botones  │         ┌────▼──────────────────────────────┐          │
    │(BTNU/D)  │──────►  │                                   │          │
    └──────────┘         │          Debouncer                │          │
    ┌──────────┐         │                                   │          │
    │ Switches │──────►  │                                   │          │
    │(J15,L16, │         └───────────────┬───────────────────┘          │
    │  M13)    │                         │ señales limpias               │
    └──────────┘                         │                               │
                                  ┌──────▼──────┐                        │
           Reset ────────────────►│  Control    │◄───────────────────────┘
                                  │  de hora    │ 1 Hz
                                  └──────┬──────┘
                                         │ Hora Actual (hh, mm, ss)
                    Reset ──────────────►│
                                  ┌──────▼──────┐   addr / pixel_data / we
                                  │ Generador   ├─────────────────────────►┐
                                  │ de imagen   │◄── font_rom (consulta)   │
                                  └─────────────┘                          │
                                                                            │
                    ┌───────────────────────────────────────────────────────┤
                    │                                                        │
             ┌──────▼──────┐   addr (read)   ┌────────────────────┐        │
             │    VRAM     │◄────────────────│  Controlador VGA   │        │
             │ (Dual Port) │                 │ (HSYNC, VSYNC, RGB)│──► Pantalla
             └──────┬──────┘  Pixel_data     └────────────────────┘
                    └─────────────────────────────────────►(write)
```

### Descripción de Módulos

| Módulo                  | Instancia          | Función                                                        |
|-------------------------|--------------------|----------------------------------------------------------------|
| `debouncer`             | `db_up/down/rst/sw`| Elimina rebotes en botones y switches de control               |
| `clock_divider`         | `div_1hz`          | Divide el reloj de 100 MHz a 1 Hz para el conteo del reloj    |
| `clk_25MHz`             | `div_25`           | Genera el pixel clock de 25 MHz para VGA                      |
| `control_hora_manual`   | `reloj`            | Lógica del reloj HH:MM:SS con control manual vía switches      |
| `display_7seg`          | `disp`             | Codificación BCD a 7 segmentos (verificación en tarjeta)      |
| `font_rom`              | —                  | ROM con los bitmaps de los dígitos 0–9 para renderizado VGA   |
| `img_gen`               | `imagen`           | Genera los píxeles del reloj consultando font_rom y escribiendo en VRAM |
| `vram`                  | `memoria`          | Memoria de video dual-port (BRAM) para el framebuffer         |
| `vga_sync`              | `sync`             | Generación de señales Hsync, Vsync y coordenadas de píxel     |

---

## ⏱️ Análisis de Timing

Reporte obtenido con **Vivado Timing Summary** tras la implementación (reloj de 100 MHz):

### Setup

| Métrica                        | Valor           |
|-------------------------------|-----------------|
| Worst Negative Slack (WNS)    | **-5.088 ns**   |
| Total Negative Slack (TNS)    | -1827.970 ns    |
| Endpoints fallidos            | 1423 / 4813     |

### Hold

| Métrica                        | Valor           |
|-------------------------------|-----------------|
| Worst Hold Slack (WHS)        | 0.067 ns ✅     |
| Total Hold Slack (THS)        | 0.000 ns        |
| Endpoints fallidos            | 0 / 4813        |

### Pulse Width

| Métrica                        | Valor           |
|-------------------------------|-----------------|
| Worst Pulse Width Slack (WPWS)| 4.500 ns ✅     |
| Endpoints fallidos            | 0 / 522         |

> El diseño presenta violaciones de timing en setup, lo que significa que algunas señales no alcanzan a estabilizarse a tiempo dentro del ciclo de reloj de 100 MHz. Hold y Pulse Width no tienen problemas. El proyecto funciona en la práctica, pero sería necesario optimizar el diseño para cumplir todos los requisitos de timing.

---

## 📊 Consumo de Recursos

Datos obtenidos del **Vivado Implementation Report** para la Nexys A7 (Artix-7 XC7A100T):

| Módulo                    | Slice LUTs | Slice Registers | F7 Muxes | F8 Muxes | Block RAM | DSPs |
|---------------------------|-----------|-----------------|----------|----------|-----------|------|
| **top_reloj** (total)     | 42,922    | 281             | 9,459    | 842      | 120       | 2    |
| `imagen` (img_gen)        | 42,617    | 105             | 9,457    | 842      | 1         | 0    |
| `memoria` (vram)          | 126       | 8               | 0        | 0        | 120       | 0    |
| `reloj` (control_hora)    | 59        | 20              | 2        | 0        | 0         | 0    |
| `sync` (vga_sync)         | 23        | 22              | 0        | 0        | 0         | 0    |
| `disp` (display_7seg)     | 13        | 17              | 0        | 0        | 0         | 0    |
| `div_1hz` (clock_divider) | 21        | 28              | 0        | 0        | 0         | 0    |
| `div_25` (clk_25MHz)      | 3         | 3               | 0        | 0        | 0         | 0    |
| `db_*` (debouncers ×3)    | 13 c/u    | 17 c/u          | 0        | 0        | 0         | 0    |

### Utilización respecto al dispositivo (XC7A100T)

| Recurso          | Usado   | Disponible | Utilización |
|------------------|---------|------------|-------------|
| Slice LUTs       | 42,922  | 63,400     | **67.7 %**  |
| Slice Registers  | 281     | 126,800    | **0.2 %**   |
| F7 Muxes         | 9,459   | 31,700     | **29.8 %**  |
| F8 Muxes         | 842     | 15,850     | **5.3 %**   |
| Block RAM Tiles  | 120     | 135        | **88.9 %**  |
| DSPs             | 2       | 240        | **< 1 %**   |
| Bonded IOB       | 35      | 210        | **16.7 %**  |

> ⚠️ El módulo `imagen` (img_gen) domina el uso de LUTs (~99 % del total). La BRAM está casi al límite debido al framebuffer en VRAM. El módulo `display_7seg` añade recursos adicionales al diseño al proveer una salida auxiliar de verificación.

---

## ⚡ Consumo de Potencia

Análisis realizado con **Vivado Power Analysis** sobre el netlist implementado:

| Componente        | Potencia  | Porcentaje |
|-------------------|-----------|------------|
| **Total On-Chip** | **0.56 W**| —          |
| Dinámica total    | 0.456 W   | 81 %       |
| → Logic           | 0.197 W   | 43 %       |
| → Signals         | 0.192 W   | 42 %       |
| → BRAM            | 0.049 W   | 11 %       |
| → Clocks          | 0.009 W   | 2 %        |
| → I/O             | 0.008 W   | 1 %        |
| → DSP             | < 0.001 W | < 1 %      |
| Device Static     | 0.104 W   | 19 %       |

> 🌡️ Temperatura de juntura estimada: **27.6 °C** (ambiente 25 °C, θJA = 4.6 °C/W).  
> Nivel de confianza del análisis: **Low** (sin vectores de simulación detallados).

---

## 🛠️ Requisitos

### Hardware
- Tarjeta **Nexys A7** (Artix-7 XC7A100T o XC7A50T)
- Monitor con entrada **VGA**
- Cable VGA
- Cable USB para programación

### Software
- **Vivado Design Suite 2024.1** o superior

---

## 🚀 Instrucciones de Reproducción

### 1. Clonar el repositorio

```bash
git clone https://github.com/tu-usuario/vga-reloj-digital.git
cd vga-reloj-digital
```

### 2. Abrir el proyecto en Vivado

```
1. Abrir Vivado
2. File → Open Project → seleccionar vga_reloj.xpr
```

### 3. Agregar fuentes (si es proyecto nuevo)

```
Project Manager → Add Sources → Add or Create Design Sources
Agregar todos los archivos .v de la carpeta /src/
Archivo top-level: top_reloj.v
```

### 4. Síntesis, implementación y programación

```
1. Flow Navigator → Run Synthesis
2. Flow Navigator → Run Implementation
3. Flow Navigator → Generate Bitstream
4. Open Hardware Manager → Open Target → Auto Connect
   → Program Device → seleccionar el .bit generado → Program
```

### 5. Verificar en pantalla

Conectar el monitor VGA a la Nexys A7. Al programar correctamente se mostrará **00:00:00** en pantalla. Los displays de 7 segmentos de la tarjeta mostrarán la misma hora como verificación adicional.

Para ajustar la hora, usar los switches y botones según la tabla de la sección [Control de hora](#control-de-hora).

---

## 📁 Estructura del Proyecto

```
vga-reloj-digital/
├── src/
│   ├── top_reloj.v           # Top-level
│   ├── vga_sync.v            # Sincronización VGA
│   ├── img_gen.v             # Generador de imagen
│   ├── font_rom.v            # ROM de bitmaps de dígitos 0–9
│   ├── vram.v                # Memoria de video dual-port (BRAM)
│   ├── control_hora_manual.v # Lógica del reloj con control manual
│   ├── clock_divider.v       # Divisor 100 MHz → 1 Hz
│   ├── clk_25MHz.v           # Divisor 100 MHz → 25 MHz
│   ├── display_7seg.v        # Decodificador BCD a 7 segmentos
│   └── debouncer.v           # Debouncer para botones y switches
├── constraints/
│   └── nexys_a7.xdc          # Constraints de pines
├── sim/
│   ├── tb_top_reloj.v         # Testbench top-level
│   ├── tb_vga_sync.v          # Testbench sincronización VGA
│   ├── tb_img_gen.v           # Testbench generador de imagen
│   ├── tb_font_rom.v          # Testbench ROM de caracteres
│   ├── tb_vram.v              # Testbench memoria de video
│   ├── tb_control_hora.v      # Testbench lógica del reloj
│   ├── tb_clock_divider.v     # Testbench divisor 1 Hz
│   ├── tb_clk_25MHz.v         # Testbench divisor 25 MHz
│   ├── tb_display_7seg.v      # Testbench decodificador 7 segmentos
│   └── tb_debouncer.v         # Testbench debouncer
└── README.md
```


