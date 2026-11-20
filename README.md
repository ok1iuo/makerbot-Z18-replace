# MakerBot Replicator Z18 → Klipper

Přestavba tiskárny MakerBot Replicator Z18 z původní elektroniky a firmwaru MakerBot na otevřenou elektroniku s firmwarem [Klipper](https://www.klipper3d.org/).

## Struktura

- `docs/` – dokumentace: původní hardware, nový hardware, zapojení, postup přestavby
- `docs/raspberry-pi.md` – instalace hostitele Klipperu, překlad a nahrání firmwaru Octopus Pro
- `docs/test-motoru.md` – postup testu krokových motorů
- `klipper/` – konfigurace Klipperu (`printer.cfg` a související soubory); zatím jen testovací konfigurace pro motory
- `klipper/firmware/` – konfigurace pro překlad firmwaru desek (`make menuconfig`)

## Stav

Octopus Pro a EBB36 s firmwarem Klipper propojené po CAN; pohon hlavy (H/V) otestovaný včetně najíždění bez spínačů; motor osy Z otestovaný (směr, `rotation_distance` 1,2); na EBB36 extruder H2 V2S včetně topení s PID a kalibrace vytlačování; osa Z najíždí na BLTouch (`z_offset` změřený). Tisková plocha X 40–300, Y 10–300, Z do 400 mm. Mapa podložky změřená (rozdíl 0,38 mm). Další kroky: startovací makro (mapa podložky, čisticí čára), vedení filamentu k hlavě, první tisk. Podrobnosti v [docs/hardware.md](docs/hardware.md).
