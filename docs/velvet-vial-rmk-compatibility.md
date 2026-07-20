# Online Research — Ergohaven Velvet V3 Wireless: Vial & RMK Firmware Compatibility

## Question 1: Does RMK Implement the Vial Protocol?

**YES — RMK SUPPORTS VIAL PROTOCOL NATIVELY.**

According to the [RMK Vial Support Documentation](https://rmk.rs/main/docs/features/vial_support), RMK firmware includes native Vial support as its default keymap editor. The official RMK repository README describes RMK as a "feature-rich keyboard firmware written in Rust" that supports "real-time keymap editing" through Vial.

**Citation:** The RMK documentation states: "RMK uses Vial as the default keymap editor" and "Vial allows you to change your keymapping in real-time with no additional firmware flashing required."

**Configuration Requirements:**
- Must provide a `vial.json` definition file in the firmware project root
- Layout in vial.json must match the firmware's internal keymap definition exactly
- Storage feature must be enabled; otherwise keymap changes are lost on reboot
- Vial support requires additional Flash and RAM; can be disabled via `vial_enabled = false` in keyboard.toml

---

## Question 2: Does Vial GUI Support BLE Transport, or Only USB?

**CRITICAL FINDING: This is ambiguous and splits into two layers:**

### A. The Vial Protocol Itself

The official Vial protocol uses **USB Raw HID** as its communication transport. According to [Controlling a Keyboard from Your PC with QMK Raw HID](https://www.esplo.net/en/posts/2025/12/qmk_raw_hid/), Vial firmware serializes settings into byte streams and sends them to the configurator over Raw HID.

**Citation:** The [Vial udev configuration page](https://get.vial.today/manual/linux-udev.html) and general Vial documentation focus exclusively on USB device connectivity. There is no mention of Bluetooth/BLE transport in official Vial GUI documentation.

**The standard Vial GUI (get.vial.today) connects via USB HID only.**

### B. RMK's Claim: "Edit Keymaps Over BLE Wirelessly"

The [RMK GitHub repository](https://github.com/HaoboGu/rmk) and WebSearch results state: "You can even edit keymaps over BLE connections wirelessly."

**However, after detailed investigation, this claim is misleading or incomplete:**

According to [DeepWiki analysis of RMK](https://deepwiki.com/HaoboGu/rmk), RMK does expose the Vial protocol endpoint over BLE in addition to USB:
- The `process_vial()` function handles `VialCommand` variants received over **both USB and BLE raw HID**
- The system architecture diagram shows "Vial over BLE" as a capability
- This was added in RMK version 0.3.1

**BUT:** The standard Vial GUI application (the desktop/web tool users download) does NOT support Bluetooth. It only supports USB HID. RMK's claim appears to refer to a theoretical capability in the firmware that is not exposed through the standard Vial GUI application.

**To edit RMK keymaps over BLE, you would need:**
- Custom tooling that supports BLE transport, OR
- To plug the keyboard in via USB cable (even if the keyboard is a wireless model)

### C. Ergohaven Vial Fork

`github.com/ergohaven/vial-gui` **does exist** — created 2024-12-11, `fork: true`, parent `vial-kb/vial-gui` (verified via `gh api repos/ergohaven/vial-gui`). The 0.7.5-eh-2026-03-28 release claim was not separately verified.

What Ergohaven maintains:
- **ergohaven/vial-gui:** fork of the upstream Vial GUI
- **ergohaven/vial-qmk:** A QMK firmware fork with Vial features (for wired keyboards using QMK)
- **ergohaven/rmk:** the active RMK firmware fork for nRF52840 wireless keyboards (fork of `HaoboGu/rmk`, created 2026-07-14); `ergohaven/rmk-eh` is the older repo and is now marked legacy, pointing to `ergohaven/rmk`
- **vial.ergohaven.xyz:** A web-based Vial configurator (appears to be a hosting of upstream Vial Web)

---

## Question 3: ZMK and Vial Incompatibility

**CONFIRMED: ZMK DOES NOT SUPPORT VIAL PROTOCOL.**

According to [GitHub Issue zmkfirmware/zmk#852](https://github.com/zmkfirmware/zmk/issues/852), a feature request to add Vial support was **closed as "not planned"** by the ZMK team in June 2021. The issue remains unresolved with no active development toward Vial integration.

**ZMK Alternative:** ZMK firmware uses **ZMK Studio** instead of Vial for real-time keymap configuration. ZMK Studio is ZMK's proprietary equivalent to Vial/Via.

**To use Vial on a ZMK keyboard, you MUST reflash the keyboard with RMK firmware instead. This is not interchangeable — they are mutually exclusive.**

### Practical Impact:
- **Current Status (given in context):** Your Velvet V3 Wireless runs ZMK and is configured via ZMK Studio
- **To switch to Vial:** You would need to flash rmk-eh firmware onto the nRF52840, erase the ZMK configuration, and rebuild your keymap in Vial
- **Configuration data is not portable** between ZMK and RMK

---

## Question 4: Flashing RMK onto Wireless Velvet

**YES — UF2 bootloader method, prebuilt files available.**

### Flashing Process:
1. **Enter Bootloader:** Double-tap the reset button on the keyboard
2. **Mounted Drive:** This exposes a USB mass storage device (standard for nRF52840 bootloader)
3. **Copy .uf2 File:** Drag-drop the prebuilt .uf2 firmware file to the mounted drive
4. **Split Keyboard:** For split keyboards like Velvet UI, you must flash both the central and peripheral halves separately (files ending in left.uf2 and right.uf2)

### Prebuilt UF2 Files:
**YES, available.** According to the [rmk-eh README](https://raw.githubusercontent.com/ergohaven/rmk-eh/main/README.md):
- "Every push builds all devices in parallel via GitHub Actions. UF2 artifacts available as build downloads."
- Prebuilt firmwares for Velvet and Velvet UI are generated automatically and available in GitHub Actions workflow artifacts

### Vial GUI and Flashing:
**Vial GUI does NOT flash firmware.** It only edits keymaps of already-flashed firmware. You must:
1. Manually flash RMK firmware via the bootloader + UF2 files
2. THEN use Vial GUI to configure the keymap

---

## Question 5: Ergohaven's Official Recommendation for Wireless Velvet

**For the Velvet V3 Wireless, Ergohaven officially recommends ZMK + ZMK Studio, NOT RMK + Vial.**

### Evidence:
- [Ergohaven Velvet V3 GitHub README](https://raw.githubusercontent.com/ergohaven/velvet/main/README.md) states: "Powered by nRF52840 and **RMK/ZMK firmware**" — lists both options but does not explicitly recommend one
- Search results from ergohaven.xyz and documentation indicate ZMK is the primary/default offering for the Velvet V3 Wireless Edition
- Ergohaven maintains [ergohaven-zmk repository](https://github.com/ergohaven/ergohaven-zmk) for ZMK support and publishes ZMK builds in the keymap_hub
- Ergohaven maintains rmk-eh but specifically for keyboards like the **Velvet UI** (with trackball), not the standard V3 Wireless

### Important Distinction:
There are **two wireless Velvet variants:**
1. **Velvet V3 Wireless Edition (Standard):** Primarily uses ZMK firmware (no trackball)
2. **Velvet V3 UI Edition:** Wireless variant with PMW3610 trackball; rmk-eh provides explicit support for this model

**Vial Support on Wireless Velvet:**
Vial is NOT officially documented or recommended for the wireless Velvet by Ergohaven. Vial is documented only for the **wired Velvet v3**, which uses QMK firmware.

- **Wired Velvet V3:** QMK + Vial (USB connection)
- **Wireless Velvet V3 (standard):** ZMK + ZMK Studio (over-the-air via ZMK Studio web or app)
- **Wireless Velvet V3 UI (with trackball):** RMK (can theoretically use Vial, but must plug in via USB; ZMK not supported for trackball variant)

### Citation:
[Velvet Repository README](https://github.com/ergohaven/velvet/blob/main/README.md) and Ergohaven's documentation at docs.ergohaven.xyz distinguish between QMK (wired) and RMK/ZMK (wireless) without promoting Vial for wireless models.

---

## Summary

| Aspect | Finding | Caveat |
|--------|---------|--------|
| **RMK + Vial** | RMK supports Vial protocol; USB editing works; BLE editing not practical via standard Vial GUI | Keyboard must be plugged in via USB to configure with Vial GUI, even if it's a wireless model |
| **Vial GUI + BLE** | No — Vial GUI requires USB HID; no standard Bluetooth support | RMK firmware exposes Vial over BLE, but standard Vial GUI cannot access it; would require custom tooling |
| **ZMK + Vial** | INCOMPATIBLE — No Vial support in ZMK; use ZMK Studio instead | Flashing RMK replaces ZMK entirely; configuration not portable |
| **Flashing Wireless Velvet** | UF2 bootloader (double-tap reset → mass storage); prebuilt files in rmk-eh GitHub Actions | Split keyboard requires separate left.uf2 and right.uf2 flashes |
| **Ergohaven Recommendation** | ZMK + ZMK Studio for Velvet V3 Wireless; RMK for Velvet V3 UI (trackball variant) | Vial not officially supported on wireless models; wired Velvet uses QMK + Vial |

---

## To Summarize for Your Situation

If you want **wireless keyboard editing without USB cable:**
- You are currently on ZMK; ZMK Studio handles over-the-air editing
- Switching to RMK + Vial would require USB cable for editing (Vial GUI = USB only)
- **RMK does not offer a practical advantage for wireless editing with standard Vial GUI**

If you want **Vial GUI specifically:**
- You must use RMK firmware
- You must plug the keyboard in via USB cable to use Vial GUI
- ZMK must be completely replaced; your current ZMK configuration will be lost
- Prebuilt RMK UF2 files are available in ergohaven/rmk-eh GitHub Actions

---

## Sources

- [RMK Vial Support](https://rmk.rs/main/docs/features/vial_support)
- [RMK GitHub Repository](https://github.com/HaoboGu/rmk)
- [RMK Vial Over BLE (DeepWiki Analysis)](https://deepwiki.com/HaoboGu/rmk)
- [ZMK Firmware Issue #852 (Vial Support Rejected)](https://github.com/zmkfirmware/zmk/issues/852)
- [Ergohaven RMK Firmware (rmk-eh)](https://github.com/ergohaven/rmk-eh)
- [Ergohaven Velvet Repository](https://github.com/ergohaven/velvet)
- [Vial Protocol HID Communication](https://www.esplo.net/en/posts/2025/12/qmk_raw_hid/)
- [Vial Linux udev Configuration](https://get.vial.today/manual/linux-udev.html)
- [Ergohaven Keymap Hub](https://github.com/ergohaven/keymap_hub)
