mauro@HP240:~/vga-digital-clock-fpga/scripts$ cat ~/vga-digital-clock-fpga/README.md
# VGA Digital Clock FPGA **PEGAR EL LINK PÚBLICO AQUÍ**

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
mauro@HP240:~/vga-digital-clock-fpga/scripts$ cat ~/vga-digital-clock-fpga/scripts/run_tests.sh
#!/bin/bash
# ============================================================
# Script: run_tests.sh
# Descripción: Ejecuta todos los scripts de testbench
#              del proyecto VGA Digital Clock.
#              Llama a cada script individual y consolida
#              el reporte final de PASS/FAIL.
# Uso: cd sim/ && ./run_tests.sh
# ============================================================

GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m'

PASS=0
FAIL=0
TOTAL=0

SCRIPT_DIR="$(dirname "$0")"

echo "=============================================="
echo " Ejecutando testbenches - VGA Digital Clock"
echo "=============================================="
echo ""

run_script() {
    local script=$1
    local name=$(basename "$script" .sh | sed 's/run_//')

    echo -n "► $name ... "

    output=$(bash "$script" 2>&1)
    exit_code=$?

    passes=$(echo "$output" | grep -c "^PASS")
    fails=$(echo "$output" | grep -c "^FAIL")

    if [ $exit_code -eq 0 ] && [ "$fails" -eq 0 ] && [ "$passes" -gt 0 ]; then
        echo -e "${GREEN}PASS ($passes checks)${NC}"
        PASS=$((PASS + 1))
    elif [ "$fails" -gt 0 ] || [ $exit_code -ne 0 ]; then
        echo -e "${RED}FAIL ($fails fallos, $passes OK)${NC}"
        echo "$output" | grep "^FAIL\|ERROR" | sed 's/^/  /'
        FAIL=$((FAIL + 1))
    else
        echo -e "${YELLOW}INFO (sin assertions)${NC}"
        PASS=$((PASS + 1))
    fi

    TOTAL=$((TOTAL + 1))
}

echo "── Etapa 1: Módulos base ──────────────────────"
run_script "$SCRIPT_DIR/run_tb_clock_divider.sh"
run_script "$SCRIPT_DIR/run_tb_clk_25MHz.sh"
run_script "$SCRIPT_DIR/run_tb_debouncer.sh"
run_script "$SCRIPT_DIR/run_tb_control_hora_manual.sh"
run_script "$SCRIPT_DIR/run_tb_vga_sync.sh"
run_script "$SCRIPT_DIR/run_tb_vram.sh"
run_script "$SCRIPT_DIR/run_tb_font_rom.sh"
run_script "$SCRIPT_DIR/run_tb_display_7seg.sh"

echo ""
echo "── Etapa 2: Sistema funcional VGA ────────────"
run_script "$SCRIPT_DIR/run_tb_img_gen.sh"
run_script "$SCRIPT_DIR/run_tb_top_reloj.sh"

echo ""
echo "=============================================="
echo " RESUMEN"
echo "=============================================="
echo -e " Total:  $TOTAL"
echo -e " ${GREEN}PASS:   $PASS${NC}"
echo -e " ${RED}FAIL:   $FAIL${NC}"
echo "=============================================="

if [ "$FAIL" -gt 0 ]; then
    exit 1
else
    exit 0
fi
mauro@HP240:~/vga-digital-clock-fpga/scripts$ cat ~/vga-digital-clock-fpga/scripts/run_tb_clk_25MHz.sh
#!/bin/bash
# ============================================================
# Script: run_tb_clk_25MHz.sh
# Descripción: Compila y ejecuta el testbench de ../sim/tb_clk_25MHz
# Uso: cd sim/ && ./run_tb_clk_25MHz.sh
# ============================================================
GREEN='\033[0;32m'
RED='\033[0;31m'
NC='\033[0m'
RTL_DIR="$(dirname "$0")/../rtl"
OUT_DIR="$(dirname "$0")/../sim/out"
mkdir -p "$OUT_DIR"

TB_NAME="tb_clk_25MHz"
echo "=============================================="
echo " Test: $TB_NAME"
echo "=============================================="

iverilog -o "$OUT_DIR/${TB_NAME}.out" \
         $RTL_DIR/clk_25MHz.v \
         "$(dirname "$0")/../sim/tb_clk_25MHz.v" 2>"$OUT_DIR/${TB_NAME}_compile.log"

if [ $? -ne 0 ]; then
    echo -e "${RED}ERROR DE COMPILACIÓN${NC}"
    cat "$OUT_DIR/${TB_NAME}_compile.log"
    exit 1
fi

vvp "$OUT_DIR/${TB_NAME}.out" | tee "$OUT_DIR/${TB_NAME}_result.log"

passes=$(grep -c "^PASS" "$OUT_DIR/${TB_NAME}_result.log")
fails=$(grep -c "^FAIL" "$OUT_DIR/${TB_NAME}_result.log")

echo "=============================================="
if [ "$fails" -eq 0 ] && [ "$passes" -gt 0 ]; then
    echo -e " ${GREEN}PASS ($passes checks)${NC}"
    exit 0
elif [ "$fails" -gt 0 ]; then
    echo -e " ${RED}FAIL ($fails fallos, $passes OK)${NC}"
    exit 1
else
    echo -e " INFO (sin assertions)"
    exit 0
fi
echo "=============================================="
mauro@HP240:~/vga-digital-clock-fpga/scripts$ 
