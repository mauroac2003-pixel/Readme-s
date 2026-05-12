# VGA Digital Clock — FPGA Nexys A7

**Curso:** EL3313 — Taller de Diseño Digital  
**Semestre:** I Semestre 2026  
**Profesor:** Luis G. León-Vega, Ph.D  
**Repositorio:** https://github.com/mauroac2003-pixel/vga-digital-clock-fpga

---

## Descripción

Sistema digital implementado en FPGA (Nexys A7-100T) que visualiza un reloj digital en una pantalla VGA a resolución 640×480 @ 60 Hz. El sistema muestra la hora en formato `HH:MM:SS` sobre una imagen de fondo personalizada y permite ajustar la hora mediante switches y botones de la tarjeta.

El diseño está implementado en Verilog HDL con arquitectura modular jerárquica y sigue buenas prácticas de diseño digital adoptadas en academia e industria.

---

## Diagrama de bloques

```
                        NEXYS A7 — top_reloj
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  clk (100MHz) ──┬──► clk_25MHz ──► clk_25_en ──► vga_sync         │
│                 │                                    │              │
│                 └──► clock_divider ──► clk_1hz       │hcount/vcount│
│                                          │            │             │
│  rst_sw ──► debouncer ──► rst_clean      │            ▼             │
│  btn_up ──► debouncer ──► btn_up_clean   │         ┌──────┐        │
│  btn_down ► debouncer ──► btn_down_clean │         │ VRAM │──────► RGB
│                                          │         │(BRAM)│  pixel │
│  sw[1:0] ──────────────────────────────►│         └──────┘  color │
│                                          ▼            ▲             │
│                                  control_hora_manual  │             │
│                                    │  hh / mm / ss    │             │
│                                    └──────────────►img_gen          │
│                                                   (escribe VRAM)    │
│                                    │                                │
│                                    └──► display_7seg ──► seg / an   │
│                                                                     │
│  blink (~2Hz) ─────────────────────────►img_gen / display_7seg     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘

Salidas VGA:  hsync, vsync, vga_r[3:0], vga_g[3:0], vga_b[3:0]
Salidas 7seg: seg[6:0], an[7:0]
```

---

## Módulos del sistema

| Módulo | Archivo | Descripción |
|--------|---------|-------------|
| `top_reloj` | `rtl/top_reloj.v` | Módulo top — integra todos los subsistemas |
| `vga_sync` | `rtl/vga_sync.v` | Generador de sincronización VGA 640×480@60Hz |
| `vram` | `rtl/BRAM.v` | Memoria de video BRAM doble puerto (307200 píxeles × 12 bits) |
| `img_gen` | `rtl/img_gen.v` | Generador de imagen — escribe fondo y dígitos en VRAM |
| `font_rom` | `rtl/font_rom.v` | ROM de caracteres (dígitos 0-9 y ':') en formato 8×16 |
| `clock_divider` | `rtl/clock_divider.v` | Divisor de 100 MHz a 1 Hz |
| `clk_25MHz` | `rtl/clk_25MHz.v` | Generador de enable de 25 MHz para VGA |
| `control_hora_manual` | `rtl/control_hora_manual.v` | Contador de tiempo con modo edición manual |
| `debouncer` | `rtl/debouncer.v` | Módulo antirrebote para botones y reset |
| `display_7seg` | `rtl/display_7seg.v` | Controlador de display 7 segmentos multiplexado (debug) |

---

## Estructura del repositorio

```
vga-digital-clock-fpga/
├── rtl/                        # Código HDL sintetizable
│   ├── top_reloj.v             # Módulo top
│   ├── vga_sync.v              # Controlador VGA
│   ├── BRAM.v                  # Memoria de video (VRAM)
│   ├── img_gen.v               # Generador de imagen
│   ├── font_rom.v              # ROM de caracteres
│   ├── clock_divider.v         # Divisor 100MHz → 1Hz
│   ├── clk_25MHz.v             # Enable 25MHz para VGA
│   ├── control_hora_manual.v   # Control de hora
│   ├── debouncer.v             # Antirrebote
│   ├── display_7seg.v          # Display 7 segmentos
│   └── fondo_profe.coe         # Imagen de fondo (formato hex)
├── sim/                        # Simulación y testbenches
│   ├── tb_vga_sync.v
│   ├── tb_vram.v
│   ├── tb_img_gen.v
│   ├── tb_font_rom.v
│   ├── tb_clock_divider.v
│   ├── tb_clk_25MHz.v
│   ├── tb_control_hora_manual.v
│   ├── tb_debouncer.v
│   ├── tb_display_7seg.v
│   ├── tb_top_reloj.v
│   └── run_tests.sh            # Script de automatización de pruebas
├── constraints/
│   └── artix7.xdc              # Constraints para Nexys A7-100T
├── .gitignore
└── README.md
```

---

## Uso de la tarjeta

| Entrada | Descripción |
|---------|-------------|
| `SW0` | Reset del sistema |
| `SW1` | Modo edición: `00`=automático, `01`=editar segundos |
| `SW2` | Modo edición: `10`=editar minutos, `11`=editar horas |
| `BTNU` | Incrementar valor en modo edición |
| `BTND` | Decrementar valor en modo edición |

---

## Dependencias y herramientas

- **Vivado Design Suite** 2024.x — síntesis e implementación
- **Icarus Verilog (iverilog)** — simulación de testbenches
- **Tarjeta FPGA:** Nexys A7-100T (Artix-7 XC7A100T)
- **Monitor VGA** con resolución mínima 640×480

---

## Simulación de testbenches

### Requisitos

```bash
sudo apt install iverilog
```

### Ejecutar todas las pruebas

```bash
cd sim/
chmod +x run_tests.sh
./run_tests.sh
```

### Ejecutar un testbench individual

```bash
cd sim/
iverilog -o out/tb_vga_sync.out ../rtl/vga_sync.v tb_vga_sync.v
vvp out/tb_vga_sync.out
```

---

## Flujo GitFlow

```
feature/vga_digital_clock ──┐
                             ├──► develop ──► main (v1.0.0)
feature/testbenches ─────────┤
feature/vga_functional ──────┘
```

- `main` — versión estable de producción (entrega final, tag `v1.0.0`)
- `develop` — integración del desarrollo
- `feature/*` — desarrollo de funcionalidades individuales

---

## Integrantes

- Arce Cruz Josué
- Navarro Acuña Mauro Agustín
- Arguedas Guzmán Gabriel

---

## Licencia

Proyecto académico — EL3313 Taller de Diseño Digital, I Semestre 2026.  
Instituto Tecnológico de Costa Rica.
