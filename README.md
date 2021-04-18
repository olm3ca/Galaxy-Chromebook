# Samsung Galaxy Chromebook: PRODUCT(RED) Hackbook
All the red without the product! Install guide for multibooting Mac OS, Linux, ChromeOS and more.

|      |       |
|------------|-------------|
|<img src="replaceme.jpg" width="300">|<img src="replaceme.png" width="600">|

### Follow For Updates
The goal of this repo will be to detail all steps necessary for multi-boot of various operating systems including Mac OS, Linux and Brunch. Not all of the functionality is working. This will be updated frequently as fixes are identified. 

### Mandatory Disclaimer

The process described in this document could cause irreversible damage to your expensive laptop, and
you should prepare yourself mentally and emotionally for that outcome before you begin. I accept absolutely no responsibility for the consequences of anyone choosing to follow or ignore any of the instructions in this document, and make no guarantees about the quality or effectiveness of the
software in this repo.

### Galaxy Chromebook Hardware
Our specs include:
-  CPU: Intel i5-10210U (Cometlake)
-  GPU: Intel UHD 620
-  RAM: 8GB Soldered to motherboard
-  Audio Codec: ALC256
-  Wifi/BT Card: Intel AX201
-  Touchpad: ELAN
-  SSD: 256GB NVME M.2 - Easily upgradable following [MrHG78's guide](https://www.youtube.com/watch?v=QAyFRj-gORI).

### Linux and Windows 10 Current Status
In booting Linux Mint 20.1, all hardware worked out of the box. Other distros may have different results, YMMV. It is expected that Windows 10 would also fully support this hardware, since it is quite similar to the Galaxy Book Flex.

### MacOS Big Sur Current Status
In configuring MacOS Big Sur with Opencore, the following is the latest hardware status: 

| Feature            | Status               | Notes                                                             |
|--------------------|----------------------|-------------------------------------------------------------------|
| WiFi               | Working              | Working                                                           |
| Bluetooth          | Working              | Working                                                           |
| Suspend / Sleep    | Working              | Working                                                           |
| Touchpad           | Working              | Voodoo ELAN                                                       |
| Graphics Accel.    | Working              | Platform ID 00009B3E / Device ID 9B3E0000)                        |
| Sound              | Working              | ALC256                                                            |
| Keyboard backlight | Working              |                                                                   |
| Touchscreen        | Working              | With VoodooI2C.kext and VoodooI2CHID.kext                         |


## Step 1: Firmware Write Protect

Before you start, you'll need to open the write protect for this machine's CR50 security chip. Start by [reading this wiki by MrChromebox](https://wiki.mrchromebox.tech/Firmware_Write_Protect) to understand what you'll be doing. For this laptop, the requirements include:

- A [SuzyQable CCD Debugging cable](https://www.sparkfun.com/products/14746), ~$15 USD + shipping
- A USB-A to USB-C adapter
- You must be in developer mode. Turn off the device and hold Power + ESC + REFRESH until the recovery screen appears, then press CTRL+D. It takes about 30 minutes to switch over. Plug in the laptop and make some coffee.
- Read the [Firmware Write Protect](https://wiki.mrchromebox.tech/Firmware_Write_Protect) wiki article again. Start at the section entitled "Disabling WP on CR50 Devices via CCD."
- The device will ask you to press the power button several times during 3 minutes. Once finished, it will power off.
- Upon booting again, it will switch back to regular mode. You must shutdown and switch back over to Developer Mode. Same procedure as before and another 30 minutes of waiting.
- Verify at the end that WP has been disabled with `sudo gsctool -a -I`  


## Step 2: UEFI Firmware Utility Script: MrChromebox's Coreboot installer
Note, once you take this step you cannot boot to regular Chrome OS. Luckily, there is a work-in-progress Brunch install currently being developed to allow booting to ChromeOS alongside other operating systems. More soon.

- Read MrChromebox's install [instructions carefully.](https://mrchromebox.tech/#fwscript)
- The latest 4.13-based firmware supports all Cometlake ChromeOS devices, including this laptop.
- Make sure you are still in Developer Mode.
- `cd; curl -LO mrchromebox.tech/firmware-util.sh`
- `sudo install -Dt /usr/local/bin -m 755 firmware-util.sh`
- `sudo firmware-util.sh`
- Follow the on-screen prompts and make sure you save a backup of the stock firmware!

## Step 3: Install MacOS Big Sur
This guide uses the lastest version of Opencore. You are encouraged to build your own EFI to boot with, but I have provided one to help you. Please note that you WILL need to customize it prior to using this full time.
 
1. Download and set up your Mac OS X Big Sur USB install media. [gibMacOS](https://github.com/corpnewt/gibMacOS) is a great tool for downloading it. 
    - Before you make the install USB, make sure it is formatted as Mac OS Extended (Journaled) with GUID Partition Map.
    - To create the installer on a Mac in Terminal: `sudo /Applications/Install\ macOS\ Big\ Sur.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume` and replace MyVolume with the name of your target drive.

2. Download the **EFI folder** zip from the Releases section of this repo. Unzip it. You will need the entire contents, starting from the EFI folder itself.
    
3. When the Big Sur install media is ready, mount the EFI partition with the [MountEFI](https://github.com/corpnewt/MountEFI) utility and copy the contents of the latest EFI linked above into this partition.
    - Make sure to copy the entire contents of the EFI above, starting from the EFI folder itself. So inside the EFI partition it should start with EFI, followed by BOOT and OC folders, etc. For more information visit the OpenCore [guide](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/opencore-efi.html)

4. Now, boot from the Big Sur installer. In Disk Utility, go to Show All Devices in the top left, and then select the entire drive to format it as APFS.
    - After about 10 minutes or so, it will reboot. Go back into the boot menu and select your Big Sur install media. In the opencore boot menu you should now see "Mac OS Install" as a menu item. Select that to continue the installation. 
    - The second phase of the installation will continue for about 15-20 minutes. 
  
5. Before you can boot from the new Big Sur installation, you will need to copy the EFI to your insternal SSD drive using the same procedure from step 3.  

6. On first boot, if you have some hardware issues, read the Opencore guide for [Cometlake devices.](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/coffee-lake-plus.html#starting-point). You may need to make adjustments to your config.plist file. Instructions on how to edit this file are below.

7. **Required final step 1**: This is important, as the EFI you downloaded does not include a serial number, which will prevent iMessage, Facetime and other Mac services from working.
    - Download [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) 
    - Mount your EFI on the Mac OS build you have installed. 
    - Run GenSMBIOS, follow the prompts in order. 
    - Step 2 - The config.plist is located in EFI / OC / drag and drop the config.plist file into Terminal.
    - Step 3 - build type, ours is **MacBookPro16,1** 
    - It will generate a serial number to be used on your machine. Yours must be different than the one used in the included EFI in order to set up iCloud / iMessage / Facetime / etc. 
    - Final tip: our serial numbers on hackintosh builds should **not** pass validation on [Apple's Check Coverage site](https://checkcoverage.apple.com/). You will want to verify that before using your serial number.

8. **Required final step 2** Download [ProperTree](https://github.com/corpnewt/ProperTree) and mount your EFI partition. Open the config.plist file.
    - In ProperTree, change the "ROM" section in the PlatformInfo/Generic to the actual MAC address of your wifi card. 

9. In the EFI on this repo, USBInjectAll.kext is included. The purpose of this kext was to create mappings for any USB devices, however it is no longer maintained and should be replaced. For best results, you can:
    - A) Use the USBMap.kext included in this repo OR, even better, 
    - B) Make your own especially if you have USB devices connected that are different than any standard USB C hub. Making your own USBMap.kext is really easy and is described in the OpenCore [guide here](https://dortania.github.io/OpenCore-Post-Install/usb/intel-mapping/intel.html)

10. Read the [OpenCore guide](https://dortania.github.io/OpenCore-Install-Guide/) on how to improve this hackintosh build and contribute here.

11. Still being worked on: 
    - Battery percentage

## Step 4: Multiboot on the Internal SSD.
Once Mac OS and Opencore are working, you can install other Operating Systems to the internal SSD. 

 - For Linux, install the distro of your choice to a partition at the end of the SSD and configure Grub to boot with an option to load Linux and Opencore.
 - For Brunch, this guide will have additional steps for following the [GetDroidTips tutorial](https://www.getdroidtips.com/install-chrome-os/) on booting Chrome OS from an image on a partition. Check back soon for more details. 
 - An example grub config file may look like this to include Opencore as a boot option:

```
menuentry "OSX" --class osx --class os {
savedefault
insmod fat
insmod chain
search --no-floppy --set=root --label SYSTEM
chainloader /EFI/BOOT/BOOTx64.efi
}
```
