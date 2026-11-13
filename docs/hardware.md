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

- Motor A a B pohonu hlavy (H-bot, v Klipperu `stepper_x` a `stepper_y` kinematiky `corexy`) a motor osy Z: po jednom TMC5160T (osa Z má jeden motor – zdroj: uživatel, stroj).
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
- Kinematika pohybu hlavy: H-bot – oba motory pohonu hlavy (vzadu po stranách) pohánějí jeden společný řemen spojený s vozíkem (zdroj: uživatel, 2026-11-13). Klipper H-bot obsluhuje kinematikou `corexy` (zdroj: `~/klipper/config/example-corexy.cfg` a `docs/Config_Reference.md` na Pi, Klipper v0.13.0-816).
- Motory pohonu hlavy (A, B) – ražba přečtená z fotek uživatele (2026-11-13), číslice s malou nejistotou:
  - motor A: „MOONS' STEPPING MOTOR / TYPE 17HD6445-10N / 6035220 14.05.18“
  - motor B: „MOONS' STEPPING MOTOR / TYPE 17HD6446-02N / 6008220 14/06/18“
  - Oba typy v katalozích Moons' chybí (zakázkové varianty). Řada 17HD6 je dvoustupňová (rotor 82 g·cm², délka kolem 48 mm); katalogový sourozenec 17HD6017N: odpor 7,5 Ω, indukčnost 14 mH, jmenovitý proud 0,8 A, přídržný moment 0,49 Nm (zdroj: [katalog Moons' hybrid steppers](https://cdn.shopify.com/s/files/1/0180/7671/files/Moons-hybrid-steppermotors.pdf), tabulka 17HD6) – shoda s naměřenými 7,5 Ω. Pracovní předpoklad: jmenovitý proud kolem 0,8 A (neověřeno).
  - Dříve uvedený MS17HD2P4100-05 (1 A, 3,9 Ω) pochází z internetu, ne ze stroje – pro tuto Z18 neplatí.
- Krokový motor Z: typ neznámý.
- Odpor cívky 7,5 Ω u obou cívek všech tří motorů (měřeno multimetrem na každé dvojici vodičů, uživatel, 2026-11-11). Testovací `run_current` 0,35 A; strop 0,8 A podle katalogového sourozence.
- Z teardown guide MakerBotu (2016, [alloys.com.au](https://www.alloys.com.au/documents/Z18%20Teardown%20Guide.pdf); lokální kopie `docs/zdroje/Z18-Teardown-Guide.pdf`):
  - Pohon hlavy: dva „Gantry Motors“ vzadu po stranách gantry; kabel k vozíku plochým kabelem (FFC) přes Rikers PCB (horní část zadního pravého sloupku).
  - Z: motor v zásuvkové části dole („Drawer Bay“), pohání řemenem („Z-Stage Belt“) závitové tyče stolu – `rotation_distance` osy Z tedy závisí na převodu řemenu i stoupání tyčí.
  - Vyhřívaná komora: dvě topné spirály („Heating Coil Assembly“) vzadu, termistor spirály, ventilátor topení („Heater Fan“) na Gowanus PCB; okolní termistor („Ambient Thermistor“) na gantry.
  - Desky: Brooklyn PCB (hlavní, 20pinové napájení ze zdroje), Gowanus PCB (zadní elektronika), Rikers PCB, Red Hook PCB (přední levý sloupek, spínač dveří – „Interlock Door Cable“).
  - Zdroj v zásuvkové části, zezadu 4 bezpečnostní šrouby Torx T10.
  - Typ ani parametry motorů návod neuvádí.
- Pro srovnání (jiný stroj): Replicator 5. generace má na X/Y motor SM42HT38C-1004AF s cívkou 4,1 Ω (zdroj: [makerbot-users](https://groups.google.com/g/makerbot-users/c/7FOeKLHlUgw), neověřeno).

Doplnit: krokové motory (typ, proud, krok), endstopy, extruder / tisková hlava, vyhřívaná komora, napájecí zdroj, ventilátory, displej.
