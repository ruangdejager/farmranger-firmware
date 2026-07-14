# farmranger-firmware

Public HTTPS host for Farmranger OTA firmware images, served via GitHub Pages.

Embedded devices (a Quectel EG91 LTE modem on the fr9 farmranger) fetch
firmware from this repo without any authentication, cookies, or user
interaction — the modem just does `HTTPS GET` on the URLs below.

## URLs

Base:

```
https://ruangdejager.github.io/farmranger-firmware/
```

Per-product firmware lives under its own subdirectory. Today: one
product, `frtag2-firmware/`. The OTA flow always starts with
`version.json`, then downloads the file it names.

```
https://ruangdejager.github.io/farmranger-firmware/frtag2-firmware/version.json
https://ruangdejager.github.io/farmranger-firmware/frtag2-firmware/latest.bin
https://ruangdejager.github.io/farmranger-firmware/frtag2-firmware/latest.hex
```

Older, versioned artifacts (`v1.0.0.bin`, `v1.0.0.hex`, ...) live alongside
so you can pin a rollback if you need to.

## Layout

```
farmranger-firmware/
├── frtag2-firmware/       Per-product OTA folder
│   ├── version.json       Manifest — version / file / size / crc32
│   ├── latest.bin         App image, raw binary (what the modem downloads)
│   ├── latest.hex         Same, as Intel HEX — for manual programming
│   ├── v1.0.0.bin         Versioned copies retained for rollback
│   ├── v1.0.0.hex
│   └── README.md
├── README.md
└── .gitignore
```

## Publishing a new firmware

The frtag2 build (`C:\Code\frtag2`) produces versioned artifacts in its
`Debug/` folder (`frtag2_<M>_<m>_<p>.{bin,hex}`,
`frtag2_<M>_<m>_<p>_ota.{bin,hex}`, `frtag2_<M>_<m>_<p>_bl_<b>.{bin,hex}`),
then copies/derives the `latest_*` files below into `frtag2-firmware/` via
its post-build script (`build_scripts/create_release_hex_files.ps1`):
`latest.{bin,hex}` (the OTA endpoint), `latest_ota.{bin,hex}` (same bytes,
kept under its own name), `latest_bl.{bin,hex}` (bootloader alone), and
`latest_app_bl.{bin,hex}` (combined, for initial programming). After a
build:

1. Confirm `frtag2-firmware/version.json`, `latest.bin`, `latest.hex`, and
   the versioned pair (`v<M>.<m>.<p>.{bin,hex}`) are updated.
2. `git add . && git commit -m "release v<M>.<m>.<p>" && git push`.
3. GitHub Pages redeploys automatically — typical latency is under a minute
   after push.

Bump the version in `frtag2/fr_app/inc/config/version_config.h` (the fields
`VERSION_SW_MAJOR`/`MINOR`/`PATCH`) before building; the post-build script
reads that file to name the artifacts and regenerate `version.json`.

## GitHub Pages configuration

- Source: **Deploy from a branch**
- Branch: `master`
- Folder: `/` (root)

## Notes

- `version.json` contains only the filename (`"file": "latest.bin"`), not a
  full URL — the device holds the base URL and appends the filename, so the
  base can move without a firmware change.
- CRC32 in `version.json` is uppercase hex; the device verifies against it
  after download.
- The bootloader-combined image (`latest_app_bl.{bin,hex}` on the build
  host) is **not** published here — it is only used for initial programming
  via a debugger, never sent over OTA. OTA only ships the application image.

## Future

Design-space kept intentionally open for later additions to `version.json`
without breaking the existing contract:

- `sha256` alongside `crc32` for integrity
- `min_bootloader_version` for compatibility gates
- `channel` (stable / beta / dev) — split into sibling manifests
- `notes_url` — release notes
- Digital signature
- Delta updates
