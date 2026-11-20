# PrusaSlicer pro Z18 s Klipperem

Nastavení vlastní tiskárny v PrusaSliceru (Linux, uživatel zvolil PrusaSlicer 2026-11-20). Makra `PRINT_START` a `PRINT_END` jsou v `klipper/printer.cfg`.

## Printer Settings

- General → Firmware → G-code flavor: `Klipper`
- General → Bed shape: 260 × 290 mm, počátek −40 / −10 – souřadnice sliceru pak odpovídají tiskové ploše X 40–300, Y 10–300 (viz [test-motoru.md](test-motoru.md), oddíl rozsah os a tisková plocha)
- General → Max print height: 400 mm
- Custom G-code → Start G-code:
  ```
  PRINT_START EXTRUDER=[first_layer_temperature[initial_extruder]]
  ```
- Custom G-code → End G-code:
  ```
  PRINT_END
  ```

## Print Settings

- Output options → Label objects: `Firmware-specific` – slicer zapíše `EXCLUDE_OBJECT_DEFINE`, `PRINT_START` pak měří mapu podložky jen pod objekty (`BED_MESH_CALIBRATE ADAPTIVE=1`) a v Mainsailu jde zrušit jednotlivý objekt.

## Co dělají makra

- `PRINT_START`: tryska 150 °C (ještě neteče), `G28`, mapa podložky pod objekty (bez označených objektů celá), ohřev na teplotu ze sliceru, čisticí čára 100 mm vpředu na Y 8, Z 0,3.
- `PRINT_END`: zatažení 2 mm, podložka o 10 mm dolů, hlava na X 290 Y 290, topení a ventilátor vypnout, smazat mapu, vypnout motory.

Ventilátor chlazení výtisku zatím není zapojený; `[fan]` na FAN1 je definovaný jen proto, aby příkazy M106/M107 ze sliceru nezastavily tisk.
