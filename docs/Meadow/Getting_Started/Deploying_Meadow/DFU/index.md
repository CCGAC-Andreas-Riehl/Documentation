---
layout: Meadow
title: Meadow OS Deployment
subtitle: Flashing the Meadow with the latest OS via Device Firmware Upgrade (DFU).
---

**NOTE** Steps to update Meadow OS have changed! Please review the steps carefully before updating to beta 4.x.

When you receive your Meadow board, it will need to have the latest Meadow.OS uploaded, or _flashed_, to it. To do this, you'll need to:

 1. [Download the latest binaries](http://developer.wildernesslabs.co/Meadow/Getting_Started/Downloads/) including the OS, CLI and Network binaries.
 2. Put the device into Device Firmware Upgrade (DFU) mode.
 3. Upload the files to the device. 

You can follow this detailed step by step guide for both macOS and Windows: 

## Step 1: Install dfu-util

We'll use the _dfu-util_ app to flash the firmware files to Meadow. 

### Windows

*Note - Windows users can flash the latest Meadow OS from within Visual Studio 2019.*

You can download dfu-util from [Sourceforge](http://dfu-util.sourceforge.net/releases/dfu-util-0.9-win64.zip).

Extract the zip to a convenient location that you can access using the Terminal/Command Prompt.

### macOS

For macOS, you'll first need to install [Brew](https://brew.sh/), if you don't already have it. Once brew is installed, you can use it install dfu-util:

 1. Open the terminal.
 2. Execute the following command:

   ```bash
   brew install dfu-util
   ```

### Linux (Debian/Ubuntu)

You can install dfu-util using the **apt** package manager.

 1. Open the terminal.
 2. Execute:

   ```bash
   sudo apt-get install dfu-util
   ```

## Step 2: Put the device into DFU Bootloader mode.

To update the OS, Meadow must be in _DFU bootloader_ mode. To enter this mode, the `BOOT` button needs to be held down while the board boots up. This can be accomplished one of two ways.

**If the board is disconnected:** hold the `BOOT` button down and connect the board to your computer via a Micro USB Cable.

![Primary USB port](./primary_usb.png){:standalone}

**If the board is connected:** hold the `BOOT` button down, and then press and release the `RST` (Reset) button. Then release the `BOOT` button. 

## Step 3: Download Meadow OS, CLI and network binaries 

Go to the [Downloads page](http://developer.wildernesslabs.co/Meadow/Getting_Started/Downloads/) and download the Beta 4.0 Meadow.OS binaries, the Beta 4.0 CLI, and the Meadow network binaries.

Unzip everything to a common folder - the instructions below assume the OS and network binaries are in the same folder and the CLI is in a `Meadow.CLI` sub-folder relative to the binaries.

## Step 4: Upload Meadow.OS and network binaries

**NOTE** Steps to update Meadow OS have changed! Please review the steps carefully before updating to beta 4.x.

The instructions are essentially the same on all supported platforms (Windows, macOS, Linux).

On **Windows**, you'll need to make the `dfu-util.exe` executable accessible. You can either:

 1. Add it's location to the PATH.

   **OR**
 * Copy `dfu-util.exe` and `libusb.dll` to your working folder.

   **OR**
 * Use a full qualified path when launching dfu-util. (e.g. `c:\Meadow\dfu-util-0.9-win64\dfu-util.exe`)

To flash Meadow to the board:

 1. Unzip the Meadow.OS, network and CLI zips as stated in Step 3 above.
 2. Open the Command Prompt (Windows) or Terminal (macOS/Linux).
 3. Navigate to the folder that contains the Meadow OS bin file.
 4. Enter `dfu-util --list` to see a list of dfu enabled devices:

  ![dfu-util --list (Windows)](./dfu_serial.png){:standalone}

  **Note:** Meadow will show four (4) DFU devices when in bootloader mode. All four devices will have the same serial number.
  
  **Help for Windows error: "Cannot open DFU device 0483:df11"** This error can occur when Windows uses
  a default driver for the USB device that doesn't support the commands needed for DFU. Use
  [Zadig](https://zadig.akeo.ie/) to replace the default driver with a WinUSB driver. Refer to
  [Scott Hanselman's blog]( https://www.hanselman.com/blog/HowToFixDfuutilSTMWinUSBZadigBootloadersAndOtherFirmwareFlashingIssuesOnWindows.aspx)
  for more details.

 5. Select and copy the serial number of your Meadow board.
 6. Execute the following command, replacing `[DEVICE_SERIAL]` with the serial number you found in
 the previous step. Each command should complete with `File downloaded successfully`

   ```bash
   dfu-util -a 0 -S [DEVICE_SERIAL] -D Meadow.OS.bin -s 0x08000000
   ```
   
 7. When the flash is complete, press the reset (**RST**) button to reset Meadow and exit boot mode.
 8. Disable mono (may need to run twice if you get an exception the first time):  
    `mono ./Meadow.CLI/Meadow.CLI.exe -s /dev/tty.usbmodem01 --MonoDisable`
 9. Reset F7 - press the reset (**RST**) button.
 10. Upload new Mono Runtime:  
    `mono ./Meadow.CLI/Meadow.CLI.exe --WriteFile -f Meadow.OS.Runtime.bin`  
 11. Move the runtime into it's special home on the 2MB partition:  
    `mono ./Meadow.CLI/Meadow.CLI.exe --MonoFlash`  
 12. Reset F7 - press the reset (**RST**) button.
 13. Upload the ESP32 bootloader:  
  `mono ./Meadow.CLI/Meadow.CLI.exe --Esp32WriteFile -f bootloader.bin --McuDestAddr 0x1000` (note progress percentage may be incorrect)
 14. Upload the ESP32 partition table:  
  `mono ./Meadow.CLI/Meadow.CLI.exe --Esp32WriteFile -f partition-table.bin --McuDestAddr 0x8000`
 15. Upload the ESP32 Meadow Comms application:  
  `mono ./Meadow.CLI/Meadow.CLI.exe --Esp32WriteFile -f MeadowComms.bin --McuDestAddr 0x10000`

Your board is now ready to have a Meadow application deployed to it!

#### Notes:

 * If you only have one dfu enabled device connected to your PC, you can omit `-S [DEVICE_SERIAL]`.
 * Linux may require `sudo` to access USB devices.

## [Next - Hello, Meadow](/Meadow/Getting_Started/Hello_World/)
