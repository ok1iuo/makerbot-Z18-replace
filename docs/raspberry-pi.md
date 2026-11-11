# Raspberry Pi 3 – hostitel Klipperu

## Systém

- Raspberry Pi OS Lite 64-bit, obraz `2026-10-06-raspios-trixie-arm64-lite.img.xz` (Debian 13 Trixie), SHA-256 ověřen proti oficiálnímu `.sha256`.
- Karta microSD 32 GB, zapsáno `dd` (2026-10-25).

## První start (cloud-init)

Konfigurace v `bootfs/user-data` (zdroj formátu: https://raspberrypi.com/news/cloud-init-on-raspberry-pi-os):

- hostname `makerbot-z18` (dostupné jako `makerbot-z18.local`), časové pásmo Europe/Prague, klávesnice cz
- uživatel `max` bez hesla (`lock_passwd: true`), přihlášení jen SSH klíčem, `sudo` bez hesla
- SSH zapnuté, přihlášení heslem přes SSH vypnuté
- síť: ethernet, DHCP (`network-config` ponechán ve výchozím stavu)

- trvalý systémový log: `/etc/systemd/journald.conf.d/50-persistent.conf` (`Storage=persistent`); Raspberry Pi OS jinak drží log jen v RAM (`/usr/lib/systemd/journald.conf.d/40-rpi-volatile-storage.conf`)

## Instalace Klipperu (KIAUH)

KIAUH je naklonovaný v `~/kiauh` a má neinteraktivní CLI:

```
cd ~/kiauh && export TERM=xterm
./kiauh.sh install klipper --count 1 --create-example-cfg </dev/null
./kiauh.sh install moonraker --create-example-cfg </dev/null
./kiauh.sh install mainsail --install-config </dev/null
```

## Stav (2026-10-25)

Nainstalováno a běží (ověřeno `systemctl is-active` a dotazem na API Moonrakeru):

- Klipper v0.13.0-816-gee2c6fb6 (`~/klipper`, služba `klipper`)
- Moonraker v0.11.0-4-gb559b9a (`~/moonraker`, služba `moonraker`)
- Mainsail na portu 80 (nginx): http://makerbot-z18.local/
- konfigurace v `~/printer_data/config/` – zatím jen ukázkový `printer.cfg` (MCU není připojené, Klipper proto hlásí „not ready“)

## Napájení

- Původní zdroj: Pi se 2× restartovalo při instalaci balíčků (rozbalování `gcc-avr`).
- USB nabíječka 60 W, měřeno `vcgencmd get_throttled` a `journalctl -k`: podpětí krátce při startu (`0x50000`); při 60 s plné zátěže 4 jader na 1,4 GHz trvalé podpětí (`0x50005`) a pokles CPU na 600 MHz.
- S procesorem omezeným na 600 MHz (`scaling_max_freq`, platí jen do restartu) instalace doběhla bez pádu a bez nového podpětí (`~/throttle.log`, 872 vzorků po 1 s, vše `0x50000`).
- Doporučení: originální zdroj Raspberry Pi 5,1 V / 2,5 A, nebo ve finále měnič 24 V → 5 V ze zdroje tiskárny.

## Firmware pro Octopus Pro (přeloženo 2026-11-11)

Volby `make menuconfig` (zdroj: hlavička `~/klipper/config/generic-bigtreetech-octopus-pro-v1.1.cfg`, volby CAN dle [Voron docs](https://docs.vorondesign.com/build/software/octopus_klipper.html)):

- Enable low-level configuration options, STM32H723, 128KiB bootloader, 25 MHz crystal
- Communication interface: USB to CAN bus bridge (USB on PA11/PA12)
- CAN bus interface: CAN bus (on PD0/PD1), CAN bus speed 1000000

Úplná konfigurace je v `klipper/firmware/octopus-pro-h723.config`; na Pi ji obnovíš zkopírováním do `~/klipper/.config` a `make olddefconfig`. Výsledek z Klipperu v0.13.0-816-gee2c6fb6: `~/klipper/out/klipper.bin`, 49 024 B, SHA-256 `be699dc590062a82a617a00f184bbfd8a7ebc3e0738fffe198447e7b6ea0fe92`.

Nahrání (neověřeno na této desce): `klipper.bin` přejmenovat na `firmware.bin`, nahrát na microSD (FAT32), vložit do Octopusu a zapnout; po úspěchu se soubor přejmenuje na `FIRMWARE.CUR`.

Rozhraní `can0` na Pi: dokumentace Klipperu (`docs/CANBUS.md`) počítá s `/etc/network/interfaces.d/can0` (ifupdown), Pi ale používá NetworkManager (`systemctl is-enabled` 2026-11-11: NetworkManager enabled, systemd-networkd disabled) – způsob nastavení vyřešit po připojení desky.

## Další kroky

1. Vyřešit napájení (viz výše).
2. Po dodání Octopus Pro: nahrát přeložený firmware (viz výše), nastavit rozhraní `can0` na Pi, test motorů s `klipper/printer.cfg`.
3. EBB36: firmware Klipperu pro CAN, napojení na `can0`.
4. `printer.cfg` pro Z18 (v repu `klipper/`).
