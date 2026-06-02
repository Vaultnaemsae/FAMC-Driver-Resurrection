# FAMC Driver Resurrection

FAMC made Liquid Foot MIDI controllers that were truly ahead of their time, but they are now unsupported, particularly on modern macOS. The following should help users keep their devices manageable via a driver-signing workaround for Parallels Desktop Windows 11 ARM VMs.

Please note: this is not an official FAMC or FTDI procedure. It is a community workaround for keeping unsupported hardware usable.

**This process assumes you are working inside a Windows 11 ARM VM in Parallels Desktop.**

## Prerequisites

Download `FAMC Driver Resurrection.zip`. It contains two PowerShell scripts:

- `patch-famc-lf-plus-inf-files.ps1`
- `sign-famc-lf-plus-arm64-driver.ps1`

The original FAMC LF+ Editor installer, `SetupLF+Editor_64bit.exe`, is required, but is not included in this repository because it is third-party copyrighted software. Users must provide their own copy.

You must also download the official FTDI Windows ARM64 driver yourself from FTDI:

```text
https://ftdichip.com/wp-content/uploads/2025/03/CDM-v2.12.36.20-Universal-Driver-for-ARM64-WHQL-Certified.zip
```

It is strongly recommended that you download and extract the FTDI driver **inside the Windows 11 ARM VM**, not on macOS. This avoids file permission, path, quarantine, and copy/extraction issues between macOS and Windows.

To create the `.cat` files and sign the driver, you must have **Visual Studio 2022 Build Tools** installed.

Choose:

```text
Desktop development with C++
```

Make sure the following are selected:

- MSVC v143 C++ build tools
- C++ CMake tools for Windows
- Windows 11 SDK
- Windows Driver Kit / WDK integration component, if shown

I found it useful to separately install the latest **Windows 11 SDK** and **Windows 11 WDK**.

## Important Parallels VM setting

Before Windows test-signing mode is enabled, shut down the Windows VM and check the Parallels configuration.

1. Shut down the Windows VM completely.
2. Open the VM configuration.
3. Go to **Hardware > Boot Order > Advanced Settings**.
4. In the **Boot flags** field, enter:

   ```text
   vm.efi.secureboot=0
   ```

5. Start the Windows VM again.

This disables Secure Boot for the VM. It is required because Windows may refuse to enable test-signing mode while Secure Boot is active.

If Secure Boot is still enabled, the signing script may fail when it tries to run:

```powershell
bcdedit /set testsigning on
```

## Procedure

### 1. Install the LF+ Editor

Install the LF+ Editor from the original installer:

```text
SetupLF+Editor_64bit.exe
```

In addition to installing the LF+ Editor software, it will create a folder full of outdated drivers at the following location:

```text
C:\Program Files\FAMC\Drivers
```

You may delete the old driver files if you wish, but keep the folder.

### 2. Download the official FTDI Windows ARM64 driver

Inside the Windows VM, download the official FTDI Windows ARM64 driver:

```text
https://ftdichip.com/wp-content/uploads/2025/03/CDM-v2.12.36.20-Universal-Driver-for-ARM64-WHQL-Certified.zip
```

The recommended download location is the Windows VM’s `Downloads` folder.

The exact download/extraction location does not matter permanently, because the extracted folder will be moved in the next steps.

### 3. Extract and rename the FTDI driver folder

Extract the FTDI driver ZIP.

Rename the extracted folder to:

```text
ARM64
```

After renaming, the folder should contain subfolders such as:

```text
Image
x86
```

The important original FTDI INF files should be located here:

```text
ARM64\Image\ftdibus.inf
ARM64\Image\ftdiport.inf
```

### 4. Move the ARM64 folder into the FAMC driver folder

Move the renamed `ARM64` folder into:

```text
C:\Program Files\FAMC\Drivers
```

The final layout should be:

```text
C:\Program Files\FAMC\Drivers\ARM64
C:\Program Files\FAMC\Drivers\ARM64\Image
C:\Program Files\FAMC\Drivers\ARM64\x86
```

The original FTDI INF files should now be here:

```text
C:\Program Files\FAMC\Drivers\ARM64\Image\ftdibus.inf
C:\Program Files\FAMC\Drivers\ARM64\Image\ftdiport.inf
```

### 5. Copy the repository scripts into the ARM64 folder

Copy the two included PowerShell scripts into:

```text
C:\Program Files\FAMC\Drivers\ARM64
```

The folder should now contain:

```text
patch-famc-lf-plus-inf-files.ps1
sign-famc-lf-plus-arm64-driver.ps1
Image
x86
```

### 6. Patch the official FTDI INF files

Open **PowerShell as Administrator** and run:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
cd "C:\Program Files\FAMC\Drivers\ARM64"
.\patch-famc-lf-plus-inf-files.ps1
```

The patch script adds support for the FAMC Liquid Foot / Liquid Router / X-Series device IDs present in the original FAMC-modified driver files:

```text
VID_0403&PID_87C0
VID_0403&PID_87C1
VID_0403&PID_87C2
VID_0403&PID_87C3
VID_0403&PID_87C4
VID_0403&PID_87C5
VID_0403&PID_87C6
```

This process has been tested with a Liquid Foot+ Series device using `VID_0403&PID_87C0`. The other included FAMC IDs are taken from the original FAMC-modified driver files but have not all been individually tested.

The script also adds the required ARM64 catalog entries needed for signing.

After patching, the device names should be set to:

```text
FAMC LF Controller
FAMC LF+ Series
```

### 7. Sign the patched ARM64 driver package

Open **PowerShell as Administrator** and run:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
cd "C:\Program Files\FAMC\Drivers\ARM64"
.\sign-famc-lf-plus-arm64-driver.ps1
```

The signing script will:

- generate new ARM64 catalog files
- create or reuse a local test-signing certificate
- trust that certificate locally
- sign `ftdibus.cat`
- sign `ftdiport.cat`
- enable Windows test-signing mode

If this is the first time enabling test-signing mode, reboot Windows before installing the driver.

### 8. Reboot Windows

After the signing script has completed, reboot the Windows VM:

```powershell
shutdown /r /t 0
```

After reboot, confirm test-signing mode is enabled:

```powershell
bcdedit /enum
```

Look for:

```text
testsigning Yes
```

### 9. Install the signed driver

Open **PowerShell as Administrator** and run:

```powershell
cd "C:\Program Files\FAMC\Drivers\ARM64\Image"
pnputil /add-driver ".\ftdibus.inf" /install
pnputil /add-driver ".\ftdiport.inf" /install
```

### 10. Connect the Liquid Foot device to the Windows VM

Disconnect and reconnect the Liquid Foot USB cable.

In Parallels Desktop, make sure the device is assigned to the **Windows VM**, not macOS.

### 11. Confirm Device Manager entries

Open **Windows Device Manager** and check that the device has registered.

You should see entries similar to:

```text
FAMC LF Controller
FAMC LF+ Series (COMx)
```

The COM number may vary.

### 12. Open the LF+ Editor

Open the LF+ Editor and click:

```text
Connect
```

Confirm that the editor can communicate with the device.

### 13. Optional: exit test-signing mode after confirming operation

After you have confirmed that the LF+ Editor connects correctly, you may exit Windows test-signing mode and reboot:

```powershell
bcdedit /set testsigning off
shutdown /r /t 0
```

If the driver stops working after exiting test-signing mode, re-enable test-signing mode:

```powershell
bcdedit /set testsigning on
shutdown /r /t 0
```

### 14. Optional: restore Parallels Secure Boot setting

After installation is complete and you have confirmed device connection, you may remove the Parallels boot flag from the VM’s advanced settings.

Remove:

```text
vm.efi.secureboot=0
```

If restoring Secure Boot causes problems with the driver or with test-signing mode, add the flag again.

## Notes

If the signing script fails while enabling test-signing mode, shut down the Windows VM, add `vm.efi.secureboot=0` to the Parallels Boot flags field, boot Windows again, and rerun the script.

If the device does not appear in the LF+ Editor, disconnect and reconnect the USB cable, then confirm in Parallels that the USB device is assigned to Windows rather than macOS.

This process uses a self-signed test certificate created locally inside the Windows VM. It is not an official Microsoft, FTDI, or FAMC driver signature.

This repository does not include the original FAMC LF+ Editor installer or the original FTDI driver package. Users must provide the FAMC installer themselves and download the official FTDI ARM64 driver from FTDI.

The included patch script targets the known FAMC/Liquid Foot FTDI hardware ID range `VID_0403&PID_87C0` through `VID_0403&PID_87C6`, based on the original FAMC-modified driver files. It should not be assumed to support unknown FAMC hardware using a different VID/PID.
