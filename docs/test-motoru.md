# Test krokových motorů

Konfigurace: `klipper/printer.cfg` (fáze 1 – jen motory, bez topení a koncových spínačů, `run_current` 0,35 A). Firmware v Octopus Pro, rozhraní `can0` na Pi a `canbus_uuid` jsou hotové (2026-11-14, viz [raspberry-pi.md](raspberry-pi.md)).

## Bezpečnost

- Motor nikdy nepřipojovat ani neodpojovat pod napětím – napěťová špička zničí driver.
- Propojky na desce měnit jen bez napájení (zdroj: [BTT wiki – Octopus Pro](https://global.bttwiki.com/Octopus%20Pro.html)).
- ⚠️ Napájení: propojka napětí motorů u každého slotu TMC5160 – jedna poloha bere napětí desky, druhá samostatný vstup motorů (až 60 V). Máme jeden zdroj 24 V → poloha „napětí desky“. Polohu na V1.1 ověřit podle desky před prvním zapnutím (online wiki popisuje V1.0 a ukazuje ji jen obrázkem).
- Propojky pod driverem pro režim SPI ověřit podle desky (wiki je ukazuje jen obrázkem).
- Žádné původní 12V díly Z18 nepřipojovat (zdroj je 24 V).

## Zapojení

1. Na každém motoru označit dvojice vodičů (cívky, 7,5 Ω uvnitř dvojice, mezi dvojicemi nekonečno), např. páskou 1 a 2. Původní kabely jsou ploché 4žilové (popsané H, V, Z) v jednom 16pinovém konektoru IDC – měřit po žilách plochého kabelu; pokud jsou páry žíly 1–2 a 3–4, stačí zachovat jejich pořadí v novém 4pinovém konektoru.
2. Na konektor Octopusu jedna dvojice na piny 1–2, druhá na piny 3–4. Pořadí pinů ověřit podle potisku na spodní straně desky (obvykle `2B 2A 1A 1B`, každá cívka na sousedních pinech – neověřeno).
3. Sloty: motor H pohonu hlavy → Driver0 (`stepper_x`), motor V → Driver1 (`stepper_y`), motor Z → Driver2 (Driver3 = rezervní TMC5160T). Z18 má kinematiku H-bot: jeden motor sám hlavou pohne šikmo nebo po jedné ose podle toho, jak je řemen vedený, ne čistě v X nebo Y – při `STEPPER_BUZZ` je to v pořádku.
4. Bez napájení posunout vozík a stůl rukou doprostřed.

## Test

Postupně pro `stepper_x`, `stepper_y`, `stepper_z`:

1. `DUMP_TMC STEPPER=stepper_x` – ověří komunikaci s driverem po SPI.
2. `STEPPER_BUZZ STEPPER=stepper_x` – motor 10× popojede o 1 mm tam a zpět.

| Projev | Příčina | Náprava |
|---|---|---|
| Plynulý pohyb tam a zpět | zapojení v pořádku | – |
| Bzučí nebo škube na místě | smíchané vodiče z různých dvojic | vypnout napájení, opravit rozdělení na piny 1–2 / 3–4 |
| Otáčí se opačně | prohozená dvojice | při bzučení nevadí; směr opraví `dir_pin` při nastavení koncových spínačů |

Po testu `M84` (vypnutí motorů) a rukou zkontrolovat teplotu motorů – teplé v pořádku, horké (neudržíš ruku) ne.

## Výsledky

### 2026-11-14 – drivery bez motorů

- Osazené jen 2× TMC5160T: Driver0 (H, `stepper_x`), Driver1 (V, `stepper_y`); `[tmc5160 stepper_z]` v `printer.cfg` zakomentovaný do osazení Driver2.
- Propojky podle manuálu BTT (`BTT_Octopus_pro_EN.pdf`, kap. 3.3 a 4.1) a pinoutu `BIGTREETECH Octopus Pro V1.1-Pin.jpg`: SPI – 4 propojky mezi řadou pinů MCU (u driveru) a řadou pinů driveru, řada SLP/5V volná; napětí motorů – propojka na VM–V_FUSED (napětí desky 24 V), pin HV u svorkovnice MOTOR_POWER volný. Zkontrolováno z fotek, SPI propojky po opravě jen podle uživatele (pod driverem nejdou vyfotit).
- Konfigurace nasazená na Pi (původní ukázková zazálohovaná jako `~/printer_data/config/printer.cfg.kiauh-example`), Klipper „ready“.
- `DUMP_TMC` u obou: `IOIN version=0x30` (TMC5160), `GLOBALSCALER=32`, `irun=28`, `mres=4` (16 mikrokroků). `GSTAT` po zapnutí `reset=1 uv_cp=1`; po smazání (`SET_TMC_FIELD ... FIELD=reset/uv_cp VALUE=1`) znovu přečteno `00000000`.
- Statistiky Klipperu: `canstat_mcu bus_state=passive`, rostoucí `tx_error` – na sběrnici CAN chybí druhý uzel (viz [raspberry-pi.md](raspberry-pi.md)); USB bez opakovaných přenosů (`bytes_retransmit=0`).

### 2026-11-14 – motory H a V

- Motory připojené před zapnutím, vozík ručně uprostřed. Osy: při pohledu zepředu (dvířky) X zleva doprava, Y zepředu dozadu.
- `STEPPER_BUZZ` u `stepper_x` (H) a `stepper_y` (V) bez chyby; `DRV_STATUS` po testu bez příznaků zkratu, přehřátí a přerušené cívky.
- Test kinematiky (`[force_move]` povolený v testovací konfiguraci): druhý motor držený `SET_STEPPER_ENABLE`, `FORCE_MOVE ... DISTANCE=50 VELOCITY=10` tam a zpět.
  - H (`stepper_x`) +50: vozík šikmo doprava a dozadu (+X, +Y).
  - V (`stepper_y`) +50: vozík šikmo doprava a dopředu (+X, −Y).
  - Odpovídá kinematice `corexy` (x = (a + b) / 2, y = (a − b) / 2) – H-bot potvrzený, směry obou motorů správné, `dir_pin` beze změny.
- `rotation_distance` X/Y: s hodnotou 40 vozík při `SET_KINEMATIC_POSITION` a `G1 X100` (oba motory naráz) jel čistě doprava o ~87 mm (měřeno pravítkem, nepřesně) → 40 × 87 / 100 ≈ 34,8. Ověřit počtem zubů řemenice × rozteč řemenu (GT2 17 z. = 34,0; MXL 17 z. = 34,54; GT2 18 z. = 36,0).
