# Test krokových motorů

Konfigurace: `klipper/printer.cfg` (fáze 1 – jen motory, bez topení a koncových spínačů, `run_current` 0,35 A). Předpoklad: firmware nahraný do Octopus Pro (viz [raspberry-pi.md](raspberry-pi.md)), rozhraní `can0` na Pi nastavené, `canbus_uuid` doplněné.

## Bezpečnost

- Motor nikdy nepřipojovat ani neodpojovat pod napětím – napěťová špička zničí driver.
- Propojky na desce měnit jen bez napájení (zdroj: [BTT wiki – Octopus Pro](https://global.bttwiki.com/Octopus%20Pro.html)).
- ⚠️ Napájení: propojka napětí motorů u každého slotu TMC5160 – jedna poloha bere napětí desky, druhá samostatný vstup motorů (až 60 V). Máme jeden zdroj 24 V → poloha „napětí desky“. Polohu na V1.1 ověřit podle desky před prvním zapnutím (online wiki popisuje V1.0 a ukazuje ji jen obrázkem).
- Propojky pod driverem pro režim SPI ověřit podle desky (wiki je ukazuje jen obrázkem).
- Žádné původní 12V díly Z18 nepřipojovat (zdroj je 24 V).

## Zapojení

1. Na každém motoru označit dvojice vodičů (cívky, 7,5 Ω uvnitř dvojice, mezi dvojicemi nekonečno), např. páskou 1 a 2.
2. Na konektor Octopusu jedna dvojice na piny 1–2, druhá na piny 3–4. Pořadí pinů ověřit podle potisku na spodní straně desky (obvykle `2B 2A 1A 1B`, každá cívka na sousedních pinech – neověřeno).
3. Sloty: motor A pohonu hlavy → Driver0 (`stepper_x`), motor B → Driver1 (`stepper_y`), Z → Driver2 (Driver3 = rezervní TMC5160T). Z18 má kinematiku H-bot: jeden motor sám hlavou pohne šikmo nebo po jedné ose podle toho, jak je řemen vedený, ne čistě v X nebo Y – při `STEPPER_BUZZ` je to v pořádku.
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

Doplnit po testu (datum, motor, projev, úpravy).
