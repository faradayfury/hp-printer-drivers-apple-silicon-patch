# HP Printer Drivers — Apple Silicon & macOS Compatibility Patch

## Download & Install

**Pre-patched package** — download and install in one step:

1. Download `HewlettPackardPrinterDrivers-patched.pkg` from the [latest release](https://github.com/faradayfury/hp-printer-drivers-apple-silicon-patch/releases/latest)
2. Install it:
   ```bash
   sudo installer -pkg HewlettPackardPrinterDrivers-patched.pkg -target /
   ```
3. If you're on Apple Silicon, make sure Rosetta 2 is installed:
   ```bash
   softwareupdate --install-rosetta
   ```

## The Problem

Apple's official **HewlettPackardPrinterDrivers** package (v10.6, dated Oct 2021) refuses to install on modern Macs due to two artificial restrictions in the installer's `Distribution` file:

1. **Architecture lock** — The installer only declares `x86_64` support, so macOS blocks it on Apple Silicon (M1/M2/M3/M4) Macs entirely.
2. **macOS version cap** — A JavaScript `InstallationCheck()` function rejects any macOS version above 15.0 (Sequoia), throwing a fatal error.

The actual driver binaries inside the package work fine under Rosetta 2. Only the **installer metadata** blocks installation — the drivers themselves are not broken.

## What Was Changed

**Only the `Distribution` file was modified.** No driver binaries, scripts, or payloads were touched.

### Change 1: Allow Apple Silicon

```xml
<!-- ORIGINAL -->
<options hostArchitectures="x86_64"/>

<!-- PATCHED -->
<options hostArchitectures="x86_64,arm64"/>
```

Added `arm64` to the `hostArchitectures` attribute so the installer runs on Apple Silicon Macs.

### Change 2: Remove macOS version cap

```js
// ORIGINAL
function InstallationCheck(prefix) {
    if (system.compareVersions(system.version.ProductVersion, '15.0') > 0) {
        my.result.message = system.localizedStringWithFormat('ERROR_25CBFE41C7', '15.0');
        my.result.type = 'Fatal';
        return false;
    }
    return true;
}

// PATCHED
function InstallationCheck(prefix) {
    return true;
}
```

Removed the version check that blocked installation on macOS versions newer than 15.0.

### Verification: Nothing else changed

| Component | Modified? |
|-----------|-----------|
| `Distribution` | Yes (2 changes above) |
| `HewlettPackardPrinterDrivers.pkg/Payload` | No (identical MD5: `e0576d1db286a4878d4e89b7d0f0dbd9`) |
| `HewlettPackardPrinterDrivers.pkg/Bom` | No (identical MD5: `9acc9cd5d19e6c29bfff6dbd4e8f9270`) |
| `HewlettPackardPrinterDrivers.pkg/PackageInfo` | No |
| `HewlettPackardPrinterDrivers.pkg/Scripts` | No (identical contents) |
| `Resources/` (localizations, license) | No |

## How to Reproduce the Patch

You can use the included script to patch the original DMG yourself:

```bash
./patch.sh HewlettPackardPrinterDrivers.dmg
```

Or do it manually:

```bash
# 1. Mount the original DMG
hdiutil attach HewlettPackardPrinterDrivers.dmg -nobrowse

# 2. Extract the pkg
mkdir hp_pkg && cd hp_pkg
xar -xf /Volumes/HP_PrinterSupportManual/HewlettPackardPrinterDrivers.pkg

# 3. Edit Distribution — make the two changes described above
#    a) Add arm64:  hostArchitectures="x86_64,arm64"
#    b) Remove the version check in InstallationCheck()

# 4. Re-pack into a new pkg
xar -cf ../HewlettPackardPrinterDrivers-patched.pkg *

# 5. Unmount
hdiutil detach /Volumes/HP_PrinterSupportManual

# 6. Install the patched pkg
sudo installer -pkg HewlettPackardPrinterDrivers-patched.pkg -target /
```

## Supported Printers

This driver package includes PPDs for **280+ HP printer models** across the following families:

### LaserJet
HP LaserJet 1010, 1012, 1015, 1150, 1160 series, Pro MFP M125-M126, Pro MFP M127-M128, Color LaserJet 3500, 3550, 3600, Color LaserJet Pro MFP M176, M177, 100 color MFP series, 200 color MFP M276, 300 color MFP M375, 400 color MFP M475, 400 MFP M425, 500 color MFP M570, CM1312 MFP, CM1410, M1522 MFP, M1530 MFP, M2727 MFP, Pro MFP M225-M226, Pro MFP M521, X476-X576 MFP

### OfficeJet
HP Officejet 100 Mobile L411, 150 Mobile L511, 2620, 4000 K210, 4100-4600 series, 4630, 5500-5740 series, 6000-6800 series, 7000-7610 series, 8040 series, H470, J3600-J6400 series, K7100, Pro 3610, 3620, 6230, 6830, 8000-8660 series, Pro K550-K8600, Pro L7300-L7700

### DeskJet
HP Deskjet 460, 1000-1510 series, 2000-2640 series, 3000-3900 series, 4510-4640 series, 5400-5900 series, 6500-6980 series, 9800, D730-D5500 series, F300-F4500 series, Ink Advantage 2010/2060/2640/4640, K109/K209

### ENVY
HP ENVY 100, 110, 120, 4500, 5530, 5640, 5660, 7640 series

### Photosmart
HP Photosmart 140-470 series, 2570-3300 series, 5510-7520 series, 7400-8700 series, A310-A820 series, B010-B8500 series, C309-C8100 series, D110-D7500 series, Ink Adv K510, Plus B209/B210, Prem C310/C410, Premium C309, Pro B8300/B8800/B9100, Wireless B109, eStn C510

### Designjet
HP Designjet 111, 130, 500/500ps plus, 510/510ps, T120-T920, Z2100-Z5200

### PSC
HP PSC 1000-2500 series

## Notes

- The drivers run under **Rosetta 2** on Apple Silicon. Make sure Rosetta is installed (`softwareupdate --install-rosetta`).
- This package includes drivers for many HP printers (LaserJet, OfficeJet, DeskJet, etc.), not just a single model.
- The original DMG is Apple's own distribution from `swscan.apple.com`, package identifier `com.apple.pkg.HewlettPackardPrinterDrivers`.
- **No proprietary binaries are modified or redistributed** — only the installer metadata (Distribution XML) is patched.
- Rosetta 2 will be removed from macOS in macOS 28, diminished only to gaming support and Linux translation.

## Disclaimer

This patch is **unofficial** and is not endorsed by, affiliated with, or supported by Apple Inc. or HP Inc. It is provided as-is, without warranty of any kind. Use at your own risk.

- No proprietary software is modified or redistributed — only the open XML installer metadata is patched.
- This may stop working with future macOS updates if Apple changes the installer framework.
- Always verify your printer works after installation.
