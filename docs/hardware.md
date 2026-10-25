# Hardware

## Nová elektronika

| Položka | Model | Stav | Poznámka |
|---|---|---|---|
| Řídicí deska | BTT Octopus Pro V1.1, MCU STM32H723 | objednáno 2026-10-17 | |
| Drivery krokových motorů | 2× TMC5160T PRO (k Octopus Pro) | objednáno 2026-10-17 | SPI; na které osy – rozhodnout |
| Hostitel Klipperu | Raspberry Pi 3 | k dispozici | |
| Deska tiskové hlavy | BTT EBB36 CAN, varianta s MAX31865 | objednáno 2026-10-24 | revize desky doplnit po dodání; má vlastní driver extruderu |
| Extruder + hotend | BIQU H2 V2S (do 270 °C) | objednáno 2026-10-24 | |

## Otevřené otázky

- Drivery pro osy, na které nezbude TMC5160T (Z18 má osy X, Y, Z; extruder řídí EBB36).
- Propojení CAN mezi Raspberry Pi a EBB36: Octopus Pro jako most USB→CAN (firmware Klipperu v režimu „USB to CAN bridge“), nebo samostatný adaptér (např. BTT U2C).
- Uchycení EBB36 na H2 V2S a uchycení H2 V2S na vozík Z18 (pravděpodobně vlastní držák).
- Použít MAX31865 (čidlo PT100/PT1000), nebo standardní termistor dodávaný s H2 V2S.

## Původní MakerBot Z18

Doplnit: krokové motory (typ, proud, krok), endstopy, extruder / tisková hlava, vyhřívaná komora, napájecí zdroj, ventilátory, displej.
