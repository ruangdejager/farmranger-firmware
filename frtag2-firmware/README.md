# frtag2-firmware

OTA firmware images for the **frtag2** device (STM32WLE5CC).

Companion to the fr9 farmranger; downloaded over HTTPS by the fr9's Quectel
EG91 modem and then forwarded to the frtag2 primary over UART for on-device
storage and distribution to LoRa secondaries.

## URLs

```
https://ruangdejager.github.io/farmranger-firmware/frtag2-firmware/version.json
https://ruangdejager.github.io/farmranger-firmware/frtag2-firmware/latest.bin
https://ruangdejager.github.io/farmranger-firmware/frtag2-firmware/latest.hex
```

`version.json` is the OTA manifest — the device downloads it first, compares
the version, and only pulls `latest.bin` if it's newer. `latest.hex` is
kept for manual programming via ST-LINK and is never used by OTA.

## Contents

- `version.json` — the manifest (`version`, `file`, `size`, `crc32`)
- `latest.bin` / `latest.hex` — current release, both formats
- `v<M>.<m>.<p>.bin` / `v<M>.<m>.<p>.hex` — versioned copies retained for rollback

## Publishing

Written to by frtag2's post-build script
(`frtag2/build_scripts/create_release_hex_files.ps1`) — it copies the just-built
artifacts here after every build. To publish, `git push` this repo.
