# AGENTS.md – makerbot-Z18-replace

Osobní projekt: přestavba MakerBot Replicator Z18 na Klipper.

## Konvence

- Dokumentace (README, `docs/`) česky s diakritikou; komentáře v konfiguraci Klipperu, commit messages a identifikátory anglicky (Conventional Commits, `type(scope): subject`).
- Hardware a jeho parametry evidovat v `docs/hardware.md`; neověřené údaje označit „neověřeno“ a uvést zdroj.
- Hodnoty v `printer.cfg` (piny, rotation_distance, proudy driverů, limity os) vždy s odkazem na zdroj (pinout desky, datasheet, měření).
- Lokální osobní poznámky do neverzovaného `AGENTS.local.md`.

## Bezpečnost

Úpravy vyhřívání (heater, termistory, `max_temp`, `verify_heater`) a napájení vždy výslovně označit a nechat uživatele odsouhlasit – chyba může způsobit požár.
