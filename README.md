# RealtekAirPortRTWFix

An **AirPortRTW 1.2.0** build patched to keep **Realtek Bluetooth coexistence** working on
RTL8821CE-class combo cards under macOS.

Verified on: HP 15s-dy0xxx / Intel i7-8565U / RTL8821CE (PCI `10EC:C821`, subsystem `103C:831A`) / macOS Tahoe 26.7.2.

---

## Why this patch exists

`RTL8821CE` is a **Wi-Fi + Bluetooth combo** device: both halves share the same RF front-end and
antenna. The stock AirPortRTW driver does not always honour the coexistence (coex) handshake with
the Bluetooth core, which shows up as:

- Bluetooth audio / BLE connections dropping while Wi-Fi is busy
- Wi-Fi scan requests starving the Bluetooth link
- After suspend/resume, one half stops working

This build keeps the original AirPortRTW 1.2.0 feature set (including WPA3-Personal support) and
restores the coexistence behaviour needed when the Realtek Bluetooth side is driven by
`RealtekBluetoothFirmware.kext` + `BlueToolFixup.kext`.

---

## Status

**In active use** on the machine above together with:

| Component | Version |
|---|---|
| `AirPortRTW.kext` | 1.2.0 (this build) |
| `RealtekBluetoothFirmware.kext` | 1.0.2 (official release binary) |
| `BlueToolFixup.kext` | 2.7.3 |
| `RealtekBTPatcher.kext` | BLE HID fix for Tahoe |

> Note: `RealtekBTPatcher` (the BLE HID fix) lives in a separate repository.

---

## Install

1. `Kernel → Add`: `AirPortRTW.kext` (Enabled).
2. Make sure `AirPort_RTW88.kext` is **disabled** — do not run two Realtek Wi-Fi drivers.
3. Reboot.

---

## Credits

- **[JoMei9019-real/AirPort-RTW](https://github.com/JoMei9019-real/AirPort-RTW)** — upstream AirPortRTW driver (v1.2.0), the base of this build.
- **[torvalds/linux](https://github.com/torvalds/linux)** — `drivers/net/wireless/realtek/rtw88`, reference for the coexistence / RF arbitration logic.
- **[OpenIntelWireless](https://github.com/OpenIntelWireless)** — community reference for native-style macOS Wi-Fi driver structure.
- **[acidanthera/Lilu](https://github.com/acidanthera/Lilu)** — kernel patching framework used by the ecosystem this driver lives in.
- **[acidanthera/WhateverGreen](https://github.com/acidanthera/WhateverGreen)** and **[acidanthera/BrcmPatchRAM](https://github.com/acidanthera/BrcmPatchRAM)** — for the general kext packaging conventions.

If your project should be credited and is missing, please open an issue.

---

## Disclaimer

Provided as-is, no warranty. Keep a bootable backup EFI before swapping Wi-Fi drivers.
