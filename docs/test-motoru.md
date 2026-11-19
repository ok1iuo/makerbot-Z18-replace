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
- Přeměřeno delší dráhou s `rotation_distance` 34,8: `G1 X200` → vozík ~208 mm doprava, `G1 Y200` (2×) → druhé měření ~207 mm dozadu; pohyb čistě v jedné ose. → 34,8 × 207,5 / 200 ≈ 36,1, nejblíže GT2 18 zubů = 36,0 (posuv by byl 206,9 mm). Nastaveno 36; první měření na 100 mm (87 mm) považováno za nepřesné. Počet zubů neověřen.
- Kontrola s `rotation_distance` 36: `G1 X200` → 200 mm, `G1 Y200` → 200 mm (pravítko, uživatel, 2026-11-14).
- Teplota: H a V 5 min pod proudem 0,35 A v klidu (`SET_STEPPER_ENABLE`, `ihold` = `irun` = 28). `DRV_STATUS` bez `otpw`/`ot`, `GSTAT` 0. Dotykem (uživatel): motory studené, chladiče driverů nanejvýš lehce vlažné.
- Zátěžový test s pohybem: vozík ručně uprostřed a označený, `SET_KINEMATIC_POSITION X=150 Y=150`, 5 min (34 cyklů) `G1` ±50 mm střídavě v X a Y rychlostí 50 mm/s; po každém testu `DUMP_TMC` a `M84`, vozík zkontrolován proti značce.
  - 0,6 A: `GSTAT` 0, bez `otpw`/`ot`; vozík na značce (bez ztráty kroků); motory studené, drivery vlažné; bez neobvyklých zvuků.
  - 0,8 A (`irun=31`): `GSTAT` 0, bez `otpw`/`ot`; vozík na značce; motory vlažné, odhadem ~29 °C (dotykem), drivery vlažné; bez neobvyklých zvuků.
  - Pět minut na ustálení teploty motoru nestačí (tepelná setrvačnost motoru je řádově desítky minut) – teplotu při 0,8 A znovu zkontrolovat po delším provozu nebo prvním tisku.
- Zkouška rychlosti při 0,8 A: limity zvednuté jen za běhu (`SET_VELOCITY_LIMIT`, po restartu Klipperu zpět na hodnoty z `printer.cfg`), zrychlení 1500 mm/s², vozík od značky uprostřed, 10 cyklů ±50 mm střídavě v X a Y pro každou rychlost, po každé rychlosti `M84` a kontrola značky.
  - 50, 100, 150, 200, 250 mm/s: Klipper bez chyb, vozík vždy na značce (bez ztráty kroků), bez neobvyklých zvuků.
  - Při 1500 mm/s² vozík na 250 mm/s potřebuje 21 mm na rozjezd i brzdění – na úseku 50 mm horní mez testu.
  - Zrychlení při 250 mm/s: 3000, 5000, 7000 mm/s² – Klipper bez chyb, vozík vždy na značce, bez neobvyklých zvuků.
  - Rychlost při 7000 mm/s²: 300, 400, 500 mm/s – Klipper bez chyb, vozík vždy na značce, bez neobvyklých zvuků. Při 500 mm/s vozík potřebuje ~18 mm na rozjezd i brzdění, na úseku 50 mm blízko horní meze testu. Hranice ztráty kroků nenalezena.
  - Osmička (oba motory naráz, plynulé změny směru): dvě kružnice r = 25 mm dotýkající se ve značce (X ±50 mm, Y ±25 mm), každá z 72 úseček po 5°, 10 průchodů, limity 500 mm/s a 7000 mm/s². Klipper bez chyb, vozík na značce, bez neobvyklých zvuků. Trvání ~25 s (časové značky Moonrakeru) na ~3,1 m dráhy → průměr ~130 mm/s; Klipper na krátkých úsečkách rychlost omezil víc než odhad z odstředivého zrychlení (~420 mm/s), příčina neověřena.
  - Vozík při testech bez tiskové hlavy (jen původní držák, fotka uživatele 2026-11-14) – výrazně lehčí než s H2 V2S + EBB36. Limity s novou hlavou a pro kvalitu tisku znovu ověřit (input shaper).

### 2026-11-14 – najíždění bez koncových spínačů (sensorless homing)

Z18 nemá koncové spínače a původně najížděla vpravo vzadu (zdroj: uživatel). Postup podle `~/klipper/docs/TMC_Drivers.md` (oddíl „Sensorless Homing“, Klipper v0.13.0-816).

- Zapojení: propojky M0DIAG (→ `PG6`) a M1DIAG (→ `PG9`) na Octopus Pro (pinout `BIGTREETECH Octopus Pro V1.1-Pin.jpg`, manuál kap. 4.3; deska dodaná bez propojek, použité z volných slotů driverů). `diag1_pin: ^!PG6` / `^!PG9`, `endstop_pin: tmc5160_stepper_x/y:virtual_endstop`, `homing_retract_dist: 0`, `homing_speed: 18` (`rotation_distance` / 2).
- Dráha změřená ručně (uživatel): X 308 mm, Y 325 mm. Najíždění v kladném směru (X doprava, Y dozadu), `position_endstop` = `position_max` = 305 / 322 – 3 mm odstup od protějšího dorazu.
- Ladění při proudu 0,4 A (oba motory), start vozíku ~50 mm od dorazu. Pojistka: dočasně `position_min` 40 → 35 mm pod `position_endstop` – Klipper ujede nejvýš 1,5 × (`position_endstop` − `position_min`) (`klippy/kinematics/corexy.py`), pak hlásí „No trigger … after full movement“.

| `sgt` | X | Y |
|---|---|---|
| 0–1 | falešný náraz po ~0,5 s | 1: falešný náraz po ~0,5 s (0 nezkoušeno) |
| 2 | na doraz jedním dotykem | na doraz jedním dotykem |
| 3 | 5× za sebou OK (3,13 s ±5 ms) | 5× za sebou OK (3,13 s ±2 ms) |
| 4 | na doraz jedním dotykem | náraz nepozná |
| 5, 6, 8, 16, 32 | náraz nepozná (X) | 5: náraz nepozná |

- Výsledek podle vzorce Klipperu: X i Y `driver_SGT: 3`. Rozmezí (X 2–4, Y 2–3) je užší než doporučených 5 – po změně proudu, rychlosti najíždění nebo hmotnosti vozíku (nová hlava) ladit znovu; případně zkusit vyšší `homing_speed`.
- Při nerozpoznaném nárazu řemen přeskakuje i při 0,4 A – motor se dál točí a driver zastavení nevidí; proto vyšší citlivost a krátká pojistka.
- Incident: první pokus (0,8 A, `sgt` 63, pojistka jen skript na Pi) – skript nouzový stop neposlal (dotazy bez časového limitu), vozík narážel a řemen přeskakoval, uživatel vypnul zdroj. Mechanika bez poškození, Pi po tvrdém vypnutí v pořádku. Pojistky pro další testy: limit dráhy v Klipperu, nezávislý časovač s `emergency_stop`.
- `[homing_override]`: proud 0,4 A, pauza 2 s, `G28 X`, odjezd 15 mm, pauza 2 s, `G28 Y`, odjezd 15 mm, návrat proudu (`run_current` z konfigurace). `G28 Z` ohlásí chybu (osa Z zatím bez najíždění).
- Ověřeno s plnou dráhou: `G28 X Y` z ~50 mm od dorazů za 14,4 s a ze středu za 25,1 s; vždy jedním dotykem, bez přeskočení; konečná poloha X 290, Y 307; proud po najetí zpět `irun=31` (0,8 A).

## Výsledek 2026-11-19 – driver osy Z

- TMC5160T osazený v Driver2, `[tmc5160 stepper_z]` zapnutý (`cs_pin` PC6, `run_current` 0,35 A). Klipper `ready`.
- `DUMP_TMC STEPPER=stepper_z`: `IOIN version=0x30` (driver odpovídá), `GSTAT reset=1 uv_cp=1` – příznaky ze startu napájení; po prvním pohybu motoru `GSTAT 00000000`.

## Výsledek 2026-11-19 – motor Z

- Disk z podložky sundaný, podložka uprostřed výšky, `run_current` 0,35 A, `FORCE_MOVE` rychlostí 2 mm/s (skutečnou). Poloha podložky měřená uživatelem měřidlem shora (rozsah do 150 mm, čtení na setiny s nejistotou desetin).
- Směr: `FORCE_MOVE` +10 mm zvedl podložku (~2 mm) – Klipper má u pohyblivé podložky +Z směrem od trysky, proto `dir_pin: !PG3`.
- `rotation_distance` (původní zástupná hodnota 8):

| Pohyb | Zadáno | Měřidlo | Ujeto | Vychází `rotation_distance` |
|---|---|---|---|---|
| dolů | −250 mm při 8 (před otočením směru) | 61,51 → 99,2 | 37,69 mm | 1,206 |
| dolů | +30 mm při 1,206 | 99,2 → 128,91 | 29,71 mm | 1,194 |
| nahoru (vymezení vůle) | −5 mm při 1,2 | 128,91 → 123,82 | 5,09 mm | – |
| nahoru (ověření) | −50 mm při 1,2 | 123,82 → 73,91 | 49,91 mm | 1,198 |

- Nastaveno `rotation_distance: 1.2` (průměr prvních dvou měření); ověření na 50 mm s odchylkou 0,2 % – v přesnosti měřidla. Vůle při změně směru se neprojevila.
- Mechanický původ neověřen: 1,2 by odpovídalo např. závitovým tyčím se stoupáním 2 mm a převodu řemenu 3 : 5 – spočítat zuby řemenic a stoupání, až bude spodek přístupný.
- Nic neobvyklého neslyšet (uživatel). Najíždění osy Z zatím chybí (plán: BLTouch), `position_max` zůstává zástupná.
