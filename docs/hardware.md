# Hardware

## Nová elektronika

| Položka | Model | Stav | Poznámka |
|---|---|---|---|
| Řídicí deska | BTT Octopus Pro V1.1, MCU STM32H723 | objednáno 2026-10-17 | |
| Drivery krokových motorů | 2× TMC5160T PRO (k Octopus Pro) | objednáno 2026-10-17 | SPI |
| Drivery krokových motorů | 2× TMC5160T PRO (k Octopus Pro) | objednáno 2026-11-03 | celkem 4× TMC5160T ve slotech Octopus Pro, přiřazení viz níže |
| Hostitel Klipperu | Raspberry Pi 3 | k dispozici | |
| Deska tiskové hlavy | BTT EBB36 CAN, varianta s MAX31865 | objednáno 2026-10-24 | revize desky doplnit po dodání; má vlastní driver extruderu |
| Extruder + hotend | BIQU H2 V2S (do 270 °C) | objednáno 2026-10-24 | |

## Napájení

- Nový zdroj 24 V / 500 W, ne původní ze Z18 (zdroj: uživatel; výrobce a model – doplnit). Proud 500 W / 24 V ≈ 20,8 A.
- Polaritu a napětí výstupu před prvním připojením k Octopus Pro změřit multimetrem.
- Původní elektronika Z18 byla napájená 12 V (zdroj: uživatel, 2026-11-11). Původní 12V spotřebiče (ventilátory, topení komory, případně další) nesmí na 24 V – u každého před připojením ověřit napětí.

## Přiřazení driverů (rozhodnuto 2026-11-03)

- X, Y, Z: po jednom TMC5160T (osa Z má jeden motor – zdroj: uživatel, stroj).
- 4. TMC5160T: rezerva.
- Extruder: driver na EBB36.
- Konkrétní sloty na Octopus Pro určit při zapojení.

## Plán podložky (rozhodnuto 2026-11-03)

- Teď: původní podložka Z18 z PC-ABS, nevodivá (zdroj: 3d-edu.ch, neověřeno) – vylučuje indukční a vířivoproudé sondy.
- Později (bez termínu): vyhřívaná kovová podložka. Pak zvážit výměnu sondy za vířivoproudou (např. Cartographer V3 CAN).

## Otevřené otázky

- Dotyková sonda: navrženo BLTouch V3.1 (Antclabs) na port sondy EBB36 – funguje na plastové i budoucí kovové podložce. Před nákupem ověřit teplotu vyhřívané komory Z18 proti teplotní odolnosti sondy (obojí neověřeno).
- Propojení CAN mezi Raspberry Pi a EBB36: Octopus Pro jako most USB→CAN (firmware Klipperu v režimu „USB to CAN bridge“), nebo samostatný adaptér (např. BTT U2C).
- Uchycení EBB36 na H2 V2S a uchycení H2 V2S na vozík Z18 (pravděpodobně vlastní držák).
- Použít MAX31865 (čidlo PT100/PT1000), nebo standardní termistor dodávaný s H2 V2S.

## Původní MakerBot Z18

- Osa Z: jeden krokový motor (zdroj: uživatel, stroj).
- Krokové motory X, Y, Z: typ a jmenovitý proud neznámé (štítky nepřístupné, na internetu nenalezeno). Odpor cívky 7,5 Ω u obou cívek všech tří motorů (měřeno multimetrem na každé dvojici vodičů, uživatel, 2026-11-11). Odhad jmenovitého proudu 0,4–0,7 A z typického jmenovitého napětí NEMA 17 2,5–5 V – neověřeno. Testovací `run_current` proto 0,35 A.
- Pro srovnání (jiný stroj): Replicator 5. generace má na X/Y motor SM42HT38C-1004AF s cívkou 4,1 Ω (zdroj: [makerbot-users](https://groups.google.com/g/makerbot-users/c/7FOeKLHlUgw), neověřeno).

Doplnit: krokové motory (typ, proud, krok), endstopy, extruder / tisková hlava, vyhřívaná komora, napájecí zdroj, ventilátory, displej.
