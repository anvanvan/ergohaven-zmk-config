# ergohaven-zmk-config

Personal ZMK keymap config for Ergohaven boards (Velvet V3 / V3 UI, op36, k03, imperial44).
`build.yaml` drives the GitHub Actions build matrix; `config/` holds the `.keymap` / `.conf` / `.json` files.

## Firmware: ZMK, not Vial

- This board runs **ZMK** — ZMK has no Vial support (upstream request closed "not planned"), so keymap editing goes through **ZMK Studio** (`studio-rpc-usb-uart` snippet + `CONFIG_ZMK_STUDIO=y`, already set in `build.yaml`).
- **Vial** is only reachable by reflashing **RMK** (`ergohaven/rmk`, a fork of `HaoboGu/rmk`; the older `ergohaven/rmk-eh` is now marked legacy and points here) — that wipes the ZMK config, and the Vial GUI is USB-only anyway, so it buys nothing for wireless editing.
- Full research incl. sources: [`docs/velvet-vial-rmk-compatibility.md`](docs/velvet-vial-rmk-compatibility.md).
