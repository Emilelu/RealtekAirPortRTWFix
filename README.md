# RealtekAirPortRTWFix

An **AirPortRTW 1.2.1** build patched to keep **Realtek Bluetooth coexistence** working on
RTL8821CE-class combo cards under macOS.

Verified on: HP 15s-dy0xxx / Intel i7-8565U / RTL8821CE (PCI `10EC:C821`, subsystem `103C:831A`) / macOS Tahoe 26.7.2.

---

## Why this patch exists

`RTL8821CE` is a **Wi-Fi + Bluetooth combo** device: both halves share the same RF front-end and
antenna. The stock AirPortRTW driver calls `rtw88_force_wifi_only()` during bring-up, which tells
the rtw88 core to run Wi-Fi alone and skip the Bluetooth coexistence (BT-coex) handshake. The
symptom on this machine was:

- Bluetooth audio / BLE connections dropping while Wi-Fi is busy
- Wi-Fi scan requests starving the Bluetooth link
- After suspend/resume, one half stops working

This build keeps the upstream AirPortRTW feature set and **only** disables that one call, so the
coexistence path stays enabled when the Realtek Bluetooth side is driven by
`RealtekBluetoothFirmware.kext` + `BlueToolFixup.kext`.

## What changed relative to upstream

| Change | Detail |
|---|---|
| `src/kext/AirPortRTW.cpp:231` | `rtw88_force_wifi_only();` commented out |
| `src/kext/RTW88PCIDevice.cpp:365` | `rtw88_force_wifi_only();` commented out |
| Version strings | `1.2.1-coex` in `DriverBuild`, IOLog banner, device label, `kmod_info` |

Everything else is upstream **1.2.1** (2026-11-11), including the new **PCI TX serialization**
fix: normal traffic and background scans used to contend for the same slot in the NIC transmit
queue, which stalled large downloads while the system still reported Wi-Fi as connected.

---

## Status

**In active use** on the machine above together with:

| Component | Version |
|---|---|
| `AirPortRTW.kext` | 1.2.1-coex (this build) |
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

## Build

Requires macOS with Command Line Tools (full Xcode is not needed).

```sh
./scripts/bootstrap-deps.sh   # MacKernelSDK + rtw88-stable core + firmware blobs
make airport                  # -> build/out/AirPortRTW.kext
```

`bootstrap-deps.sh` pins the rtw88 core to `d029a677c49266fad86750714eb5612becd134d3` and embeds
`rtw8821c_fw.bin` / `rtw8822b_fw.bin` / `rtw8822c_fw.bin` into the bundle.

---

## Credits

- **[JoMei9019-real/AirPort-RTW](https://github.com/JoMei9019-real/AirPort-RTW)** — upstream AirPortRTW driver (v1.2.1), the base of this build.
- **[torvalds/linux](https://github.com/torvalds/linux)** — `drivers/net/wireless/realtek/rtw88`, reference for the coexistence / RF arbitration logic.
- **[OpenIntelWireless](https://github.com/OpenIntelWireless)** — community reference for native-style macOS Wi-Fi driver structure.
- **[acidanthera/Lilu](https://github.com/acidanthera/Lilu)** — kernel patching framework used by the ecosystem this driver lives in.
- **[acidanthera/WhateverGreen](https://github.com/acidanthera/WhateverGreen)** and **[acidanthera/BrcmPatchRAM](https://github.com/acidanthera/BrcmPatchRAM)** — for the general kext packaging conventions.

If your project should be credited and is missing, please open an issue.

---

## Disclaimer

Provided as-is, no warranty. Keep a bootable backup EFI before swapping Wi-Fi drivers.
