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
- Od 2026-11-14 napájené měničem 24 V → 5 V ze zdroje tiskárny přes piny GPIO (zdroj: fotka uživatele; výstup 5,1 V, proud přes 2 A podle uživatele, 2026-11-14; typ měniče neověřen). Napájení přes GPIO obchází vratnou pojistku Pi. `vcgencmd get_throttled` = `0x0` při čtyřech startech bez zátěže (2026-11-14, měřeno přes SSH); zátěžový test zatím neproběhl.

## Firmware pro Octopus Pro (přeloženo 2026-11-11)

Volby `make menuconfig` (zdroj: hlavička `~/klipper/config/generic-bigtreetech-octopus-pro-v1.1.cfg`, volby CAN dle [Voron docs](https://docs.vorondesign.com/build/software/octopus_klipper.html)):

- Enable low-level configuration options, STM32H723, 128KiB bootloader, 25 MHz crystal
- Communication interface: USB to CAN bus bridge (USB on PA11/PA12)
- CAN bus interface: CAN bus (on PD0/PD1), CAN bus speed 1000000

Úplná konfigurace je v `klipper/firmware/octopus-pro-h723.config`; na Pi ji obnovíš zkopírováním do `~/klipper/.config` a `make olddefconfig`. Výsledek z Klipperu v0.13.0-816-gee2c6fb6: `~/klipper/out/klipper.bin`, 49 024 B, SHA-256 `be699dc590062a82a617a00f184bbfd8a7ebc3e0738fffe198447e7b6ea0fe92`.

### Nahrání přes USB z Pi (hlavní cesta)

Režim DFU je zavaděč v pevné paměti STM32. `make flash` volá `scripts/flash_usb.py`, který pro STM32H7 spustí `dfu-util` se startovací adresou z `.config` (`0x8020000`), takže zavaděč BTT zůstane netknutý (ověřeno čtením `scripts/flash_usb.py` na Pi, 2026-11-11). `dfu-util` 0.11 je na Pi nainstalovaný.

1. Bez napájení nasadit na Octopus propojku BOOT0. Pi je napájené ze stejného zdroje 24 V – před vypnutím zdroje ho vypnout `sudo systemctl poweroff`.
2. Propojit Octopus s Pi kabelem USB-C, zapnout napájení.
3. `lsusb` – musí ukázat `0483:df11 ... STM Device in DFU Mode`.
4. Nahrát:
   ```
   cd ~/klipper
   sudo service klipper stop
   make flash FLASH_DEVICE=0483:df11
   sudo service klipper start
   ```
5. Vypnout napájení, sundat propojku BOOT0, zapnout.

Ověřeno 2026-11-14: deska z výroby s firmwarem Marlin (`MARLIN_STM32H723ZX`), s BOOT0 hlášena jako `0483:df11`, nahráno `make flash` (49 024 B na `0x08020000`). `dfu-util` na konci vypíše `Error during download get_status` a `make` skončí chybou – to je reset desky po opuštění DFU, zápis proběhl celý („File downloaded successfully“) a deska hned naběhla jako `1d50:606f` (výrobce Klipper, `stm32h723xx`).

V režimu „USB to CAN bridge“ se deska nehlásí jako sériový port, takže každá další aktualizace firmwaru znovu potřebuje propojku BOOT0 (nebo doinstalovat zavaděč Katapult pro aktualizace přes CAN). Přerušené nahrávání (např. restart Pi při podpětí) desku nezničí – postup stačí zopakovat.

### Nahrání z microSD (záloha)

`klipper.bin` přejmenovat na `firmware.bin`, nahrát na microSD (FAT32), vložit do Octopusu a resetovat; po úspěchu se soubor přejmenuje na `FIRMWARE.CUR` (zdroj: [BTT wiki – Octopus Pro](https://global.bttwiki.com/Octopus%20Pro.html)).

## Rozhraní can0 (nastaveno 2026-11-14)

Dokumentace Klipperu (`docs/CANBUS.md`) počítá s `/etc/network/interfaces.d/can0` (ifupdown), Pi ale používá NetworkManager, který `can0` nespravuje (`nmcli device status`: `unmanaged`). Stejné parametry proto nastavuje pravidlo udev `/etc/udev/rules.d/80-can0.rules`:

```
SUBSYSTEM=="net", ACTION=="add", KERNEL=="can0", RUN+="/usr/bin/ip link set can0 type can bitrate 1000000", RUN+="/usr/bin/ip link set can0 txqueuelen 128", RUN+="/usr/bin/ip link set can0 up"
```

Pravidlo se uplatní při každém připojení desky, tedy i po `FIRMWARE_RESTART`. Ověřeno po restartu Pi: `can0` UP, bitrate 1000000, qlen 128.

UUID Octopus Pro: `479eb403415a` (`~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0`).

Dokumentace Klipperu upozorňuje, že režim USB to CAN bridge bez funkční sběrnice (zakončovací odpory 120 Ω a aspoň jeden další uzel) může způsobovat občasné chyby i při komunikaci se samotnou deskou – do připojení EBB36 s tím počítat.

## Další kroky

1. Zátěžový test napájení z měniče (viz výše).
2. Test motorů s `klipper/printer.cfg` (firmware nahraný, `can0` nastavené).
3. EBB36: firmware Klipperu pro CAN, napojení na `can0`.
4. `printer.cfg` pro Z18 (v repu `klipper/`).
