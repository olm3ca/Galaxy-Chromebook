# Samsung Galaxy Chromebook: PRODUCT(RED) Multiboot Hackbook
All the red without the product! Install guide for multibooting Mac OS, Linux, ChromeOS, & Windows 10.

|      |       |
|------------|-------------|
|<img src="UEFI.jpeg" width="300">|<img src="replaceme.png" width="600">|

### Follow For Updates
The goal of this repo will be to detail all steps necessary for multi-boot of various operating systems. Not all of the functionality is working. This will be updated frequently as fixes are identified. 

### Mandatory Disclaimer

You know this already: The process described in this document could cause irreversible damage to your expensive laptop, and
you should prepare yourself mentally and emotionally for that outcome before you begin. I accept absolutely no responsibility for the consequences of anyone choosing to follow or ignore any of the instructions in this document, and make no guarantees about the quality or effectiveness of the
software in this repo.

### Galaxy Chromebook Hardware
Specs:
-  CPU: Intel i5-10210U (Cometlake)
-  GPU: Intel UHD 620
-  RAM: 8GB Soldered to motherboard
-  Audio: Intel Multimedia Controller (only works with Sound Open Firmware on Linux & Chrome OS). On CrOS, uses driver sof-cmlda7219max 
-  Wifi/BT Card: Intel AX201
-  Touchpad: Synaptics TM3579-001 (only works on Linux and Chrome OS). 
-  SSD: 256GB NVME M.2 - Easily upgradable following [MrHG78's guide](https://www.youtube.com/watch?v=QAyFRj-gORI).

### OS Compatibility Current Status
In booting Manjaro Linux with kernel 5.10 and Sound Open Firmware baked in, all hardware worked out of the box. This distro is the easiest to install, whereas others such as Ubuntu will need to be upgraded to a newer kernel and SOF will need to be configured.

### OS Current Status
The following table shows hardware working based on the OS installed: 

| Hardware           | Manjaro Linux        | Mac OS Catalina     | Windows 10      | Brunch		|
|--------------------|----------------------|---------------------|-----------------|-------------------|
| WiFi               | Working              | Working             | Working         | Working		|
| Bluetooth          | Working              | Working             | Working	    | Working		|
| Suspend / Sleep    | Not Working (fixable)| Not Working         | Not Working.    | Not Working	|
| Touchpad           | Working	            | Not Working         | Not Working     | Working           |
| Graphics Accel.    | Working              | Not Working	  | Working    	    | Working 		|
| Sound              | Working (SOF, 5.10)  | Not Working         | Not Working	    | Working (headphones problem noted below)
| Keyboard backlight | Not Working          |                                                                   |
| Touchscreen        | Not Working          | ELAN Touchscreen                         |


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

 - For Linux, install the distro of your choice to a partition at the end of the SSD. Currently, Manjaro Linux works best as it already incorporates Sound Open Firmware natively, which enables sound on this device. Other distros will work, but you will need to configure sound on your own.
 - For Brunch, this guide will have additional steps for following the [GetDroidTips tutorial](https://www.getdroidtips.com/install-chrome-os/) on booting Chrome OS from an image on a partition. You will want to start with Recovery v88 on this device, as wifi works. [Go to CrOS Updates](https://cros-updates-serving.appspot.com/) and search for "hatch", then download v88. To upgrade to v89 and above, you will need to replace the wifi driver in /lib/firmware with the file in this repo (iwlwifi-QuZ...) 
 - An example grub config file may look like this, if you have EFI on partition 1, Mac OS on partition 2, Chrome OS on partition 3 and Linux on partition 4. This assumes you installed grub and refind. Currently, using grup to chainload opencore is problematic (thus why refind is used). 

```
menuentry "ChromeOS" {
	img_part=/dev/nvme0n1p3
	img_path=/chromos.img
	search --no-floppy --set=root --file $img_path
	loopback loop $img_path
	linux (loop,7)/kernel-4.19 boot=local noresume noswap loglevel=7 disablevmx=off \
		cros_secure cros_debug options=native_chromebook_image,iwlwifi_backport,enable_updates loop.max_part=16 img_part=$img_part img_path=$img_path \
		console= vt.global_cursor_default=0 brunch_bootsplash=default
	initrd (loop,7)/lib/firmware/amd-ucode.img (loop,7)/lib/firmware/intel-ucode.img (loop,7)/initramfs.img
}
menuentry "Mac OS"{
   insmod part_gpt
   search --no-floppy --set=root --fs-uuid 67E3-17ED
   chainloader /EFI/refind/refind_x64.efi
}
```
