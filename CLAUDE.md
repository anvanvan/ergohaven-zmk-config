# ergohaven-zmk-config

Personal ZMK keymap config for Ergohaven boards (Velvet V3 / V3 UI, op36, k03, imperial44).
`build.yaml` drives the GitHub Actions build matrix; `config/` holds the `.keymap` / `.conf` / `.json` files.

## Deploy: `bin/deploy`

Fetches the CI firmware built for **HEAD** (push first — no local build) and flashes it via the UF2 bootloader volume. `firmware/` is gitignored and replaced on every fetch.

```
bin/deploy                     # status: commit, build, firmware/, bootloader volume — read-only, never waits
bin/deploy flash [target...]   # fetch + copy onto the bootloader volume once it mounts (--timeout <s>, default 120)
bin/deploy fetch [target...]   # only replace firmware/ with the wanted targets
bin/deploy targets             # list all targets in the build
```

- Default target `velvet_v3_ui_right` = the central half (holds the keymap, carries Studio) — keymap changes only need this one. The left half only needs reflashing for changes that affect the peripheral firmware.
- `flash` blocks until a human puts the board into the bootloader (BOOT key: adj layer, right extra key pos 45; or double-tap reset) — don't run it unattended; use `bin/deploy` / `fetch` instead.
- Output: stdout = `key: value` data, stderr = progress; errors come as `error:` + `fix:`; exit 0 ok · 1 failed · 2 usage. `gh` resolves this fork to its parent, so the script always passes `-R <origin>`.
- Layers unchanged after flashing → a keymap saved in ZMK Studio overrides the flashed one: Studio → *Restore Stock Settings*.

## Firmware: ZMK, not Vial

- This board runs **ZMK** — ZMK has no Vial support (upstream request closed "not planned"), so keymap editing goes through **ZMK Studio** (`studio-rpc-usb-uart` snippet + `CONFIG_ZMK_STUDIO=y`, already set in `build.yaml`).
- **Vial** is only reachable by reflashing **RMK** (`ergohaven/rmk`, a fork of `HaoboGu/rmk`; the older `ergohaven/rmk-eh` is now marked legacy and points here) — that wipes the ZMK config, and the Vial GUI is USB-only anyway, so it buys nothing for wireless editing.
- Full research incl. sources: [`docs/velvet-vial-rmk-compatibility.md`](docs/velvet-vial-rmk-compatibility.md).
