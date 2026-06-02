# FAMC Driver Resurrection

FAMC made Liquid Foot MIDI controllers that were truly ahead of their time, but they are now unsupported, particularly on modern macOS. The following should help users keep their devices manageable via a driver-signing workaround for Parallels Desktop Windows 11 ARM VMs.

Please note: this is not an official FAMC or FTDI procedure. It is a community workaround for keeping unsupported hardware usable.

## Prerequisites

Download `FAMC Driver Resurrection.zip`. It contains three key items:

- `FTDIBUS.INF` — modified
- `FTDIPORT.INF` — modified
- PowerShell script: `sign-famc-lf-plus-arm64-driver.ps1`

You will also need the original 64-bit LF+ Editor software installer:

`SetupLF+Editor_64bit.exe`

This installer is too large to include here.

To create the `.cat` files and sign the driver, you must install **Visual Studio 2022 Build Tools**.

Choose:

`Desktop development with C++`

Make sure the following are selected:

- MSVC v143 C++ build tools
- C++ CMake tools for Windows
- Windows 11 SDK
- Windows Driver Kit / WDK integration component, if shown

I found it useful to separately install the latest **Windows 11 SDK** and **Windows 11 WDK**.

## Important Parallels VM setting

If all your other files are now in place, before Windows test-signing mode is enabled, shut down the Windows VM and check the Parallels configuration.

1. Shut down the Windows VM completely.
2. Open the VM configuration.
3. Go to **Hardware > Boot Order > Advanced Settings**.
4. In the **Boot flags** field, enter:

   ```text
   vm.efi.secureboot=0
   ```
   
5. Start the Windows VM again.

This disables Secure Boot for the VM. It is required because Windows may refuse to enable test-signing mode while Secure Boot is active.

If Secure Boot is still enabled, the script may fail when it tries to run:

```powershell
bcdedit /set testsigning on
```

## Procedure

1. Install the LF+ Editor from the original installer:

   `SetupLF+Editor_64bit.exe`

   In addition to installing the LF+ Editor software, it will also create a folder full of outdated drivers at the following location:

   `C:\Program Files\FAMC\Drivers`

   You may delete the old driver files if you wish, but keep the folder.

2. Download the official FTDI Windows ARM64 driver here:

   `https://ftdichip.com/wp-content/uploads/2025/03/CDM-v2.12.36.20-Universal-Driver-for-ARM64-WHQL-Certified.zip`

3. Extract the FTDI driver folder and shorten the extracted folder name to:

   `ARM64`

4. Place the newly named `ARM64` folder inside:

   `C:\Program Files\FAMC\Drivers`

   The resulting path should be:

   `C:\Program Files\FAMC\Drivers\ARM64`

5. Place the included PowerShell script inside:

   `C:\Program Files\FAMC\Drivers\ARM64`

   The script should be located here:

   `C:\Program Files\FAMC\Drivers\ARM64\sign-famc-lf-plus-arm64-driver.ps1`

6. Replace the two `.inf` files inside:

   `C:\Program Files\FAMC\Drivers\ARM64\Image`

   with the two included modified `.inf` files:

   - `FTDIBUS.INF`
   - `FTDIPORT.INF`

7. Open **PowerShell as Administrator** and execute:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
cd "C:\Program Files\FAMC\Drivers\ARM64"
.\sign-famc-lf-plus-arm64-driver.ps1
```

8. The script signs the ARM64 driver package and enables Windows test-signing mode.

   If this is the first time enabling test-signing, reboot Windows before installing the driver.

9. After reboot, if you skipped the install step in the script, open **PowerShell as Administrator** again and execute:

```powershell
cd "C:\Program Files\FAMC\Drivers\ARM64\Image"
pnputil /add-driver ".\ftdibus.inf" /install
pnputil /add-driver ".\ftdiport.inf" /install
```

10. Disconnect and reconnect the Liquid Foot USB cable.

    Make sure the FAMC device is connected to the **Windows VM**, not macOS, under the Parallels USB device menu/settings.

11. Open **Windows Device Manager** and check that the device has registered.

    You should see entries similar to:

    - `FAMC LF Controller`
    - `FAMC LF+ Series (COMx)`

    The COM number may vary.

12. Open the LF+ Editor and click **Connect**. Confirm operation.

13. You may exit test mode and reboot the device after you have confirmed correct operation.
```powershell
bcdedit /set testsigning off
shutdown /r /t 0
```
14. Additionally, you may remove the secure boot flag from your VM's advanced settings after the installation is complete and you have confirmed device connection. 

## Notes

If the script fails while enabling test-signing mode, shut down the Windows VM, add `vm.efi.secureboot=0` to the Parallels Boot flags field, boot Windows again, and rerun the script.

If the device does not appear in the LF+ Editor, disconnect and reconnect the USB cable, then confirm in Parallels that the USB device is assigned to Windows rather than macOS.

This process uses a self-signed test certificate created locally inside the Windows VM. It is not an official Microsoft, FTDI, or FAMC driver signature.

