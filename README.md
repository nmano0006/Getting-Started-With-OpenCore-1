![logo](https://i.imgur.com/dtoe9Oj.png)

# Getting Started With OpenCore 0.5.1

A brief guide to using the OpenCore boot-loader for hackintoshes


### OpenCore is an alternative bootloader to CloverEFI or Chameleon. It is not only for Hackintosh and can also be used on real macs for purposes that require an emulated EFI. It also aims to have the ability to boot Windows and Linux without the need for using different acpi tables. It has a clean codebase and aims to stay closer to how a real mac bootloader functions. Kext injection has been greatly improved. While already functioning well,
OpenCore should be considered in Public Beta stage at this time and is intended to be used by experienced hackintosh users, developers, or users who are happy to recover a system which fails to boot or becomes broken in some way.

**This guide may not always be able to keep up with every change to OpenCore,** 
**(currently OpenCore is in active development,and therefore a moving target)** 
**please keep that in mind when compiling the latest version of OpenCore.**
**To be safe, use release versions of OpenCore rather than the latest commits.** (0.0.4 Current Release)

## This guide is intended to complement the excellent opencore "configuration.pdf" rather than be used instead of it. If you did not already do so, please read it now: 
## [Full Documentation From Acidanthera GitHub](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/Configuration.pdf)


# Current known issues
### Refer to opencore bugtracker for current known bugs [here](https://github.com/acidanthera/bugtracker/issues) 

### Opencore Pro's
* One average, OpenCore systems boot faster than those using Clover.
* OpenCore offers better overall security and better support for FileVault.
* OpenCore supports boot hotkey support via boot.efi - hold Option or ESC at startup to choose a boot device, Command+R to enter Recovery or Command+Option+P+R to reset NVRAM.
* OpenCore is designed with the future in mind, and uses modern methods to inject kernel extensions without breaking System Integrity Protection.
* BootCamp switching and boot device selection are supported by reading NVRAM variables set by Startup Disk just like a real mac.
* Furture development for AptioMemoryFix is directly tied to OpenCore, specifically being absorbed into OpenCore itself with the FwRuntimeVariable.efi being used as an extension.
* UEFI and Legacy boot modes are supported.

### OpenCore Must's
* Kernel extensions are loaded in the order specified in your config file, so you must load an extension's dependencies before you load the extension itself. For example, Lilu must be loaded before WhateverGreen or VirtualSMC.
* ACPI patches and SSDTs are applied to all operating systems. 
* Some systems require pure UEFI mode to boot. (This setting is commonly called "Windows 8/10 Mode" by motherboard manufacturers.)
* Issues can occur if NVMe devices are set up as SATA devices in the BIOS.

# Getting Started

## Requirements:

* OpenCorePkg (Recommend to build from scratch instead of using the prebuilt package as OpenCore is constantly being updated. As of writing we're on Version 0.5.1 even though the current official release is 0.5.0). Easiest way to build is via the macbuild.tool or Use Pavo's Opencore Builder App to Create the Entire EFI Folder (Requires Xcode Installed)

## Recommended BIOS settings:

### Disable:
* Fast Boot
* CFG Lock(MSR 0xE2 write protection)
* VT-d(can be enabled if you set DisableIoMapper to YES)
* CSM
* Thunderbolt
* Intel SGX
* Intel Platform Trust
### Enable:
* VT-x
* Above 4G decoding
* Hyper Threading
* Execute Disable Bit
* EHCI/XHCI Hand-off
* OS type: Windows 8.1/10

# EFI Drivers

For the majority of systems, you only need 3 .efi drivers to get up and running:

* [ApfsDriverLoader.efi](https://github.com/acidanthera/AppleSupportPkg/releases)
   * Needed for seeing APFS volumes.
* [VboxHfs.efi](https://github.com/acidanthera/AppleSupportPkg/releases) or HfsPlus.efi
   * Needed for seeing HFS volumes.
* [FwRuntimeServices.efi](https://github.com/acidanthera/AppleSupportPkg/releases)
   * Replacement for [AptioMemoryFix.efi](https://github.com/acidanthera/AptioFixPkg), used for patching boot.efi for NVRAM fixes and better memory management.
   
For extra functionality with OpenCore:

* [AppleGenericInput.efi](https://github.com/acidanthera/AppleSupportPkg)
   * Used for Apple Hot keys and FileVault support, some firmwares may not register all the keys while booting so this can be substituted with [UsbKbDxe.efi](https://github.com/acidanthera/AppleSupportPkg). 

* [VirtualSmc.efi](https://github.com/acidanthera/VirtualSMC/releases)
   * Used for proper FileVault support, cannot be used with FakeSMC
   
For a full list of compatible drivers, see 11.2 Properties in the [OpenCorePkg Docs](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/Configuration.pdf)
   
# Kexts

Please refer to [Kexts.md](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/Kexts.md) for a full list of supported kexts

# SSDTs

So you see all those SSDTs in the AcpiSamples folder and wonder whether you need any of them. Well we'll be going over a couple to see whether you need them. Do note you'll need to compile these SSDTs with [MaciASL](https://github.com/acidanthera/MaciASL/releases) and please read them before compiling. Some require you to adjust them for your specific system(ie: EC0 to H_EC for SSDT-EC-USBX)


* [SSDT-AWAC](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-AWAC.dsl)
   *  This is the [300 series RTC patch](https://www.hackintosh-forum.de/forum/thread/39846-asrock-z390-taichi-ultimate/?pageNo=2), needed for certain Z390 systems.
* [SSDT-RTC0](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-RTC0.dsl)
   * Alternative to [SSDT-AWAC](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-AWAC.dsl) when not compatible with your system.
* [SSDT-EC-USBX](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-EC-USBX.dsl)
   * Needed to setup USB power and such correctly, prefered option over renaming XHCI. This SSDT is meant for Skylake+ systems, please use
   [SSDT-EC](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-EC.dsl) and [SSDT-EHCx_OFF](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-EHCx_OFF.dsl) for older systems.

* [SSDT-EC](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-EC.dsl)
   * Needed to setup USB power correctly on pre-skylake systems.

* [SSDT-EHCx_OFF](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-EHCx_OFF.dsl)
   * Prefered alternative over renaming EHCI for setting up USB correctly on pre-skylake systems.

* [SSDT-PLUG](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-PLUG.dsl)
   * Sets `PluginType`, Clover altrenative would be under `Acpi -> GenerateOptions -> PluginType`. Do note that this SSDT is made for systems where `AppleACPICPU` attaches `CPU0`, though some systems have theirs starting at `PR00` so adjust accordingly.

* [SSDT-SBUS-MCHC](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-SBUS-MCHC.dsl)
   * Adds a SMbus device and fixes DeviceProperties injection via `_DSM` which is found in your DSDT table. Most can ignore.


`USB drive formatted as MacOS Journaled with GUID partition map. This is to test opencore without overwriting your working Clover.
Knowledge of how a hackintosh works and what files yours requires.
A previously setup and functioning hackintosh is assumed: which you are happy to potentially break.
Time and patience. Without these, you are wasting your effort.
*Sign out of all apple services until you are sure you have MLB and ROM sections of smbios set to match your previous Clover set up.  Not doing so could cause said services to cease to function, or worst case block your machine.`

* ### [InsanelyMac-Discord](https://discord.gg/EdmdKRW) If you need any extra help join our discord.
* ### [Pavo's OCBuilder ](https://github.com/Pavo-IM/ocbuilder/releases) Creates a Basic EFI Structure with Basic Kexts which are needed to Boot **XCODE App is required**



# Creating the USB

Creating the USB is simple, format a USB stick (any size will suffice) as MacOS Journaled with GUID partition map. 

![Formatting the USB](https://i.imgur.com/7SNlbto.png)

Next, mount the EFI partition on the USB with either diskutil terminal command or Clover Configurator.

![mountEFI](https://i.imgur.com/yCWBGoJ.png)

By default, the EFI partition will be empty.


# EFI folder structure

To setup OpenCore’s folder structure, copy the files from OpenCorePkg so your EFI looks like the one below:

```

   |--EFI
   |   |--BOOT
   |   |   |--BOOTx64.efi
   |   |--OC
   |   |   |--ACPI
   |   |   |--Drivers
   |   |   |   |--ApfsDriverLoader.efi
   |   |   |   |--AppleGenericInput.efi
   |   |   |   |--AppleUiSupport.efi
   |   |   |   |--FWRuntimeServices.efi
   |   |   |   |--UsbKbDxe.efi
   |   |   |   |--VBoxHfs.efi
   |   |   |   |--VirtualSmc.efi (Filevault Only))
   |   |   |--Kexts
   |   |   |   |--AppleALC.kext
   |   |   |   |--CPUFriend.kext
   |   |   |   |--Lilu.kext
   |   |   |   |--SMCBatteryManager.kext (Laptop)
   |   |   |   |--SMCLightSensor.kext (Laptop)
   |   |   |   |--SMCProcessor.kext
   |   |   |   |--SMCSuperIO.kext
   |   |   |   |--VirtualSMC.kext
   |   |   |   |--WhateverGreen.kext
   |   |   |--OpenCore.efi
   |   |   |--Tools
   |   |   |   |--CleanNvram.efi
   |   |   |   |--Shell.efi
   |   |   |   |--VerifyMsrE2.efi

```

Place necessary .efi drivers from AppleSupportPkg and AptioFixPkg into the *drivers* folder, then kexts/ACPI into their respective folders.

# Setting up the config.plist

While sharing the name, the config.plist in OpenCore, is very different from Clover config.plist, they **cannot** be mixed and matched. It is also not recommended to duplicate every patch and option from your clover config. 

First, duplicate the `sample.plist`, rename it to `config.plist` and open in your .plist editor of choice.


The config contains a number of sections:

* **ACPI:** This is for loading, blocking and patching the ACPI.
* **Booter** UEFI modifications on Apple bootloader (boot.efi).
* **DeviceProperties:** This is where you'd inject PCI device properties or set Intel Framebuffer.
* **Kernel:** Where we tell OpenCore what kexts to load, what order to load and which to block.
* **Misc:** Settings for OpenCore's boot loader itself.
* **NVRAM:** This is where we set certain NVRAM properties like boot flags and SIP.
* **Platforminfo:** This is where we setup your SMBIOS.
* **UEFI:** UEFI drivers and related options. 

We can delete *#WARNING -1* and  *#WARNING -2* You did heed the warning didn't you?

# 1. ACPI 

**Add:** Here you add your SSDTs or custom DSDT. (SSDT-EC.aml for example)

**Block**: Certain systems benefit from dropping some acpi tables, most modern desktops however require nothing in this section.

**Patch**: In OpenCore we should be keeping ACPI device renames to a minimum as they are often harmful and unnecessary. If your system absolutely needs something, you should add it in this section. Refer to configuration.pdf.

For those who need DSDT patches for things like XHC controllers can utilize the [SSDT-EC-USBX.dsl](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-EC-USBX.dsl) or use similar Device Property patching like what's seen with Framebuffer patching And to grab the location of such devices can use [gfxutil](https://github.com/acidanthera/gfxutil/releases).

* Example, common device renames are handled now by WhateverGreen on-the-fly and in a safer way:
- GFX0 to IGPU
- HECI to IMEI
* Do NOT do these in the config.plist nor in DSDT/SSDT.

* Do NOT rename EC0 to EC as this can cause an incompatible kext (AppleACPIEC) to load and cause strange issues at any time or a non bootable system.


For those who utilize [300 series RTC patch](https://www.hackintosh-forum.de/forum/thread/39846-asrock-z390-taichi-ultimate/?pageNo=2) can use the [SSDT-AWAC.dsl](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-AWAC.dsl).


 ## Quirks: 
 
* **FadtEnableReset**: NO
   * Enable reboot and shutdown on legacy hardware, not recommended unless needed
* **NormalizeHeaders**: NO
   * Cleanup ACPI header fields, only relevant for macOS High Sierra 10.13
* **RebaseRegions**: NO
   * Attempt to heuristically relocate ACPI memory regions, not needed unless custom DSDT is used.
* **ResetHwSig**: NO
   * Needed for hardware that fail to maintain hardware signature across the reboots and cause issues with waking from hibernation
* **ResetLogoStatus**: NO
   * Workaround for OEM Windows logo not drawing on systems with BGRT tables.



&#x200B;

# 2. Booter


* This section allows to apply different kinds of UEFI modifications on Apple bootloader (boot.efi). The modifications
currently provide various patches and environment alterations for different firmwares. Some of these features were
originally implemented as a part of AptioMemoryFix.efi, which is no longer maintained. 


**NOTES:**

- Most up-to-date UEFI firmware (check your motherboard vendor website).
- Fast Boot and Hardware Fast Boot disabled in firmware settings if present.
- Above 4G Decoding or similar enabled in firmware settings if present. Note, that on some motherboards (notably ASUS WS-X299-PRO) this option causes adverse effects, and must be disabled. While no other motherboards with the same issue are known, consider this option to be first to check if you have erratic boot failures.
- DisableIoMapper quirk enabled, or VT-d disabled in firmware settings if present, or ACPI DMAR table dropped.
- No ‘slide‘ boot argument present in NVRAM or anywhere else. It is not necessary unless you cannot boot at all or see No slide values are usable! Use custom slide! message in the log.
- CFG Lock (MSR 0xE2 write protection) disabled in firmware settings if present. Cconsider patching it if you have enough skills and no option is available. See VerifyMsrE2 nots for more details.
- CSM (Compatibility Support Module) disabled in firmware settings if present. You may need to flash GOP ROM on NVIDIA 6xx/AMD 2xx or older. Use GopUpdate or AMD UEFI GOP MAKER in case you are not sure how.
- EHCI/XHCI Hand-off enabled in firmware settings only if boot stalls unless USB devices are disconnected.
- VT-x, Hyper Threading, Execute Disable Bit enabled in firmware settings if present.
- While it may not be required, sometimes you have to disable Thunderbolt support, Intel SGX, and Intel Platform Trust in firmware settings present.


## Booter Quirks

* **AvoidRuntimeDefrag**: YES
   * Fixes UEFI runtime services like date, time, NVRAM, power control, etc
* **DevirtualiseMmio**: NO
   * Reduces Stolen Memory Footprint, expands options for `Slide=N` values but may not be ompatible with all boards. Generally useful for APTIO V firmwares(Broadwell+)
* **DisableSingleUser**: NO
   * Disables use of `Cmd+S` and `-s`, this is closer to the behaviour of T2 based machines
* **DisableVariableWrite**: NO
   * Needed for systems with non-functioning NVRAM like Z390 and such
* **DiscardHibernateMap**: NO
   * Reuse original hibernate memory map, only needed for certain legacy hardware 
* **EnableSafeModeSlide**: YES
   * Allows for slide values to be used in Safemode
* **EnableWriteUnprotector**: YES
   * Removes write protection from CR0 register during their execution
* **ForceExitBootServices**: NO
   * Ensures ExitBootServices calls succeeds even when MemoryMap has changed, don't use unless necessary\) 
* **ProtectCsmRegion**: NO
   * Needed for fixing artifacts and sleep-wake issues, AvoidRuntimeDefrag resolves this already so avoid this quirk unless necessary
* **ProvideCustomSlide**: YES
   * If there's a conflicting slide value, this option forces macOS to

      use a pseudo-random value. Needed for those receiving `Only N/256 slide values are usable!` debug message
* **SetupVirtualMap**: YES
   * Fixes SetVirtualAddresses calls to virtual addresses
* **ShrinkMemoryMap**: NO
   * Needed for systems with large memory maps that don't fit, don't use unless necessary


# Fixing Certain NVRAM Issues



**AvoidRuntimeDefrag** : Set to YES for Enabled NVRAM Reading.

**EnableWriteUnprotector** : Set to YES for Enabled NVRAM Writing.

* NVRAM read tests should display the NVRAM information in the Hackin tool/NVRAM correctly.
* NVRAM write testing shall ensure that the starting disk  was correctly. (Default name was must be Macintosh HD)
* Tested on Asus X299, Z370M-Plus II, and Gigabyte Z370 AORUS Gaming 5 and 7.
* This feature is based on an OpenCore 0.0.4 08082018 distribution and works with the FwRuntimeService.efi driver.


# 4. DeviceProperties

**Add**: Sets device properties from a map.

This section is set up via Headkaze's [_Intel Framebuffer Patching Guide_](https://www.insanelymac.com/forum/topic/334899-intel-framebuffer-patching-using-whatevergreen/?tab=comments#comment-2626271) and applies only one actual property to begin, which is the _ig-platform-id_. The way we get the proper value for this is to look at the ig-platform-id we intend to use, then swap the pairs of hex bytes.

If we think of our ig-plat as `0xAABBCCDD`, our swapped version would look like `DDCCBBAA`

The two ig-platform-id's we use are as follows:

* `0x3E9B0007` - this is used when the iGPU is used to drive a display
   * `07009B3E` when hex-swapped
* `0x3E920003` - this is used when the iGPU is only used for compute tasks and doesn't drive a display
   * `0300923E` when hex-swapped

Worth noting that for 10.12 -&gt; 10.13.5, you would need to fake the iGPU to the same values in the Kaby Lake guide, as this was before native Coffee Lake iGPU showed up.

We also add 2 more properties, framebuffer-patch-enable and framebuffer-stolenmem. The first enables patching via WhateverGreen.kext, and the second sets the min stolen memory to 19MB. This is usually unnecessary, as this can be configured in BIOS.

I added another section as well that shows a fake `device-id` for the i3-8100's UHD 630. This has a different device id than the UHD 630 found on the 8700k, for instance, \(`3e918086` vs `3e928086` \).

For this - we follow a similar procedure as our above ig-platform-id hex swapping - but this time, we only work with the first two pairs of hex bytes. If we think of our device id as 0xAABB0000, our swapped version would look like 0xBBAA0000. We don't do anything with the last 2 pairs of hex bytes.

The device-id fake is set up like so:

* `0x3e920000` - this is the device id for the UHD 630 found on an 8700k
   * `923e0000` when hex swapped

Note: FakeID is only required for High Sierra, Mojave doesn't require this

`PciRoot(0x0)/Pci(0x1f,0x3)` -&gt; `Layout-id`

* Applies AppleALC audio injection, you'll need to do your own research on which codec your motherboard has and match it with AppleALC's layout. [AppleALC Supported Codecs](https://github.com/acidanthera/AppleALC/wiki/Supported-codecs).

Keep in mind that some motherboards have different device locations, you can find yours by either examining the device tree in IOReg or using [gfxutil](https://github.com/acidanthera/gfxutil/releases). Please note that ADR for HDAS/HDEF is 0x001F0003 and Path = PciRoot(0x0)/Pci(0x1f,0x3), PciRoot(0x0)/Pci(0x1b,0x0) is for previous series

Do note that `layout-id` is a `Data` value meaning you will need to convert from `Number` to `HEX` so `Layout=5` would be interpreted as `<05000000>` and `Layout=11` would be `<0B000000>`

Fun Fact: The reason the byte order is swapped is due to [Endianness](https://en.wikipedia.org/wiki/Endianness), specifcally Little Endians that modern CPUs use for ordering bytes. The more you know!

**Block**: Removes device properties from map, for us we can ignore this

# 5. Kernel

**Add**: Here we can specify kexts to inject from our EFI into the kernel kextcache. 
Order of kexts is important, they are loaded in this order. Plugins for other kexts should always come after the main kext. Lilu should be first, then Lilu plugins like WhateverGreen and VirtualSMC. 

* **BundlePath** 
   * Name of the kext
   * ex: `Lilu.kext`
* **Enabled** 
   * Self explaitroy, either enables or diables the kext
* **Executableath** 
   * Path to the actual executable hidden within the kext, you can see what path you kext has by right clicking and selecting `Show Package Contents`. Generally they'll be `Contents/MacOS/Kext` but some have kexts hiddin within under `Plugin` folder. Do note that Plist only kexts do not need this filled in.
   * ex: `Contents/MacOS/Lilu`
* **PlistPath** 
   * Path to the `info.plist` hidden within the kext
   * ex: `Contents/Info.plist`

**Emulate**: Needed for spoofing unsupported CPUs like Pentiums and Celerons

* **CpuidMask**: When set to Zero, original CPU bit will be used
   * `<Clover_FCPUID_Extended_to_4_bytes_Swapped_Bytes> | 00 00 00 00 | 00 00 00 00 | 00 00 00 00`
   * ex: CPUID `0x0306A9` would be `A9 06 03 00 | 00 00 00 00 | 00 00 00 00 | 00 00 00 00`
* **CpuidData**: The value for the CPU spoofing
   * `FF FF FF FF | 00 00 00 00 | 00 00 00 00 | 00 00 00 00`
   * Swap `00` for `FF` if needing to swap with a longer value

**Block**: Blocks kexts from loading. Sometimes needed for disabling Apple's trackpad driver for some laptops.

**Patch**: Patches kexts \(this is where you would add newer USB port limit patches and AMD CPU patches. Do note that the XhciPortLimit quirk is preferred for USB port limit patches\).

**Quirks**:

* **AppleCpuPmCfgLock**: NO 
  * Only needed when CFG-Lock can't be disabled in BIOS, Clover counterpart would be AppleICPUPM
* **AppleXcpmCfgLock**: NO 
  * Only needed when CFG-Lock can't be disabled in BIOS, Clover counterpart would be KernelPM
* **AppleXcpmExtraMsrs**: NO 
  * Disables multiple MSR access needed for unsupported CPUs like Pentiums and certain Xeons
* **CustomSMBIOSGuid**: NO 
  * Performs GUID patching for UpdateSMBIOSMode Custom mode. Usually relevant for Dell laptops
* **DisableIOMapper**: YES 
  * Needed to get around VT-D if  either unable to disable in BIOS or needed for other operating systems
* **ExternalDiskIcons**: YES 
  * External Icons Patch, for when internal drives are treated as external drives but can also make USB drives internal. For NVMe on Z87 and below you just add built-in property via DeviceProperties.
* **LapicKernelPanic**: NO 
  * Disables kernel panic on AP core lapic interrupt, generally needed for HP systems. Clover equivalent is `Kernel LAPIC`
* **PanicNoKextDump**: YES 
  * Allows for reading kernel panics logs when kernel panics occurs
* **ThirdPartyTrim**: NO 
  * Enables TRIM, not needed for NVMe but AHCI based drives may require this. Please check under system report to see if your drive supports TRIM
* **XhciPortLimit**: YES 
  * This is actually the 15 port limit patch, don't rely on it as it's not a guaranteed solution for fixing USB. Please create a [USB map](https://usb-map.gitbook.io/project/) when possible as.

The reason being is that UsbInjectAll reimplements builtin macOS functionality without proper current tuning. It is much cleaner to just describe your ports in a single plist-only kext, which will not waste runtime memory and such



# 6. Misc

**Boot**: Settings for boot screen \(leave as-is unless you know what you're doing\)

* **HibernateMode**: None
   * Best to avoid hibernation with hackintoshes all together
* **HideSelf**: YES
   * Hides the EFI partition as a boot option in OC's boot picker
* **PollAppleHotKeys**: YES
   * Allows you to use Apple's hot keys during boot, needs to be used in conjunction with either AppleGenericInput.efi or UsbKbDxe.efi depending on the firmware. Popular commands:
      * `Cmd+V`: Enables verbose
      *  `Cmd+Opt+P+R`: Cleans NVRAM 
      * `Cmd+R`: Boots Recovery partition
      * `Cmd+S`: Boot in Singleuser mode
      * `Option/Alt`: Shows boot picker when `ShowPicker` set to `NO`, alternative is `ESC` key
* **Timeout**: `5`
  * This sets how long OpenCore will wait until it automatically boots from the default selection
* **ShowPicker**: YES
  * Shows OpenCore's UI, needed for seeing your available drives or set to NO to follow default option
* **UsePicker**: YES
  * Uses OpenCore's default GUI, set to NO if you wish to use a different GUI

**Debug**: Debug has special use cases, leave as-is unless you know what you're doing.

* **DisableWatchDog**: NO \(May need to be set for YES if macOS is stalling on something while booting, generally avoid unless troubleshooting\)

**Security**: Security is pretty self-explanatory.

* **AllowNvramReset**: YES
   * Allows for NVRAM reset both in the boot picker and when pressing `Cmd+Opt+P+R`
* **RequireSignature**: NO
  * We won't be dealing vault.plist so we can ignore
* **RequireVault**: NO
  * We won't be dealing vault.plist so we can ignore as well
* **ScanPolicy**: `0` 
* `0` allows you to see all drives available, please refer to [Security](extras/secuirty.md) section for furthur details

**Tools** Used for running OC debugging tools like clearing NVRAM
* **Name** 
   * Name shown in OpenCore
* **Enabled** 
   * Self explanitory, enables or disables
* **Path** 
   * Path to file after the `Tools` folder
   * ex: [Shell.efi](https://github.com/acidanthera/OpenCoreShell/releases)

**Entires**: Used for specifying iregular boot paths that can't be found naturally with OpenCore
* **Name**
   * Name shown in boot picker
* **Enabled**
   * Self explanitory, enables or disables
* **Path**
   * PCI route of boot drive, can be found with the [OpenCoreShell](https://github.com/acidanthera/OpenCoreShell) and the `map` command
   * ex: `PciRoot(0x0)/Pci(0x1D,0x4)/Pci(0x0,0x0)/NVMe(0x1,09-63-E3-44-8B-44-1B-00)/HD(1,GPT,11F42760-7AB1-4DB5-924B-D12C52895FA9,0x28,0x64000)/\EFI\Microsoft\Boot\bootmgfw.efi`


# Emulated NVRAM and NVRAM

**Add**: 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14 \(Booter Path, majority can ignore but \)

* UIScale:
    * 01: Standard resolution(Clover equivalent is `0x28`)
    * 02: HiDPI (generally required for FileVault to function correctly on smaller displays, Clover equivalent is `0x2A`\)

7C436110-AB2A-4BBB-A880-FE41995C9F82 \(System Integrity Protection bitmask\)

* **boot-args**:
   * `-v` - this enables verbose mode, which shows all the behind-the-scenes text that scrolls by as you're booting instead of the Apple logo and progress bar.  It's invaluable to any Hackintosher, as it gives you an inside look at the boot process, and can help you identify issues, problem kexts, etc.
   * `dart=0` - this is just an extra layer of protection against Vt-d issues, keep in mind this requires SIP to be disabled
   * `debug=0x100` - this prevents a reboot on a kernel panic. That way you can \(hopefully\) glean some useful info and follow the breadcrumbs to get past the issues.
   * `keepsyms=1` - this is a companion setting to debug=0x100 that tells the OS to also print the symbols on a kernel panic. That can give some more helpful insight as to what's causing the panic itself.
   * `shikigva=40` - this flag is specific for Nvidia users.  It enables a few Shiki settings that do the following \(found [here](https://github.com/acidanthera/WhateverGreen/blob/master/WhateverGreen/kern_shiki.hpp#L35-L74)\):
      * 8 - AddExecutableWhitelist - ensures that processes in the whitelist are patched.
      * 32 - ReplaceBoardID - replaces board-id used by AppleGVA by a different board-id. Do note that this generally needed for systems running Nvidia GPUs
* **csr-active-config**: Settings for SIP, generally recommended to manually change this within Recovery partition with `csrutil` via the recovery partition

csr-active-config is set to `E7030000` which effectively disables SIP. You can choose a number of other options to enable/disable sections of SIP. Some common ones are as follows:

* `00000000` - SIP completely enabled
* `30000000` - Allow unsigned kexts and writing to protected fs locations
* `E7030000` - SIP completely disabled
* **nvda\_drv**: &lt;&gt; 
   * For enabling Nvidia WebDrivers, set to 31 if running a [Maxwell or Pascal GPU](https://github.com/khronokernel/Catalina-GPU-Buyers-Guide/blob/master/README.md#Unsupported-nVidia-GPUs). This is the same as setting nvda\_drv=1 but instead we translate it from [text to hex](https://www.browserling.com/tools/hex-to-text)
* **prev-lang:kbd**: &lt;&gt; 
   * Needed for non-latin keyboards

**Block**: Forcibly rewrites NVRAM variables, not needed for us as `sudo nvram` is prefered but useful for those edge cases. Note that `Add` will not overwrite values already present in NVRAM

**LegacyEnable**: NO

* Allows for NVRAM to be stored on nvram.plist, needed for systems without native NVRAM

**LegacySchema**

* Used for assigning NVRAM variables, used with LegacyEnable set to YES

## Emulated NVRAM

So this section is for those who don't have native NVRAM, Hardware to have incompatible native NVRAM with macOS are the Z390-300 series chipsets:

* B360
* B365
* H310
* H370
* Q370
* Z390

## Making the nvram.plist

Outlay's to making a NVRAM.plist file, Requires the following:

Change these settings within the config.plist:

**Booter Section**
* `DisableVariableWrite`: set to `YES`
**NVRAM Section**
* `LegacyEnable`: set to `YES`
* `LegacySchema`: NVRAM variables set and injected into OpenCore and compares these variables present in nvram.plist
**Security Section**
* `ExposeSensitiveData`: set to `0x3` (Which allows all data exposure)

And within your EFI:

* `FwRuntimeServices.efi` (Needed for sleep, wake and shutdown and other services to work correctly (Goes in the EFI/OC/Drivers Folder)

Now grab the 'LogoutHook.command' and place it somewhere safe like within your user directory:

``/Users/(your username)/LogoutHook/LogoutHook.command``

Open up terminal and run the following:

``sudo defaults write com.apple.loginwindow LogoutHook /Users/(your username)/LogoutHook/LogoutHook.command``

Now you have emulated NVRAM, Just to note that for macOS to support the `-x` flag and work correctly which is unavailable on 10.12 and below. ``nvram.mojave`` fixes this by injecting it instead of the system based one.



# 8. Platforminfo

* This section used be filled in correctly to avoid errors, if using non automatic setup make sure (DATA, BOOLEAN, STRING) types are set as shown in the ```Sampleconfig.plist```

or setting up the SMBIOS info, we'll use acidanthera's [_macserial_](https://github.com/acidanthera/MacInfoPkg/releases) application.

To get the SMBIOS info generated with macserial, you can run it with the `-a` argument \(which generates serials and board serials for all supported platforms\). You can also parse it with grep to limit your search to one SMBIOS type.

For this Coffee Lake example, I chose the iMac18,1 SMBIOS - this is done intentionally for compatibility's sake. There are two main SMBIOS used for Kaby Lake:

* `iMac18,1` - this is used for computers utilizing the iGPU for displaying.
* `iMac18,3` - this is used for computers using a dGPU for displaying, and an iGPU for compute tasks only.

To get the SMBIOS info generated with macserial, you can run it with the -a argument \(which generates serials and board serials for all supported platforms\). You can also parse it with grep to limit your search to one SMBIOS type.

With our iMac18,1 example, we would run macserial like so via the terminal:

`macserial -a | grep -i iMac18,1`

Which would give us output similar to the following:

```text
   iMac18,1 | C02T8SZNH7JY | C02707101J9H69F1F
   iMac18,1 | C02VXBYDH7JY | C02753100GUH69FCB
   iMac18,1 | C02T7RY6H7JY | C02706310GUH69FA8
   iMac18,1 | C02VD07ZH7JY | C02737301J9H69FCB
   iMac18,1 | C02TQPYPH7JY | C02720802CDH69FAD
   iMac18,1 | C02VXYYVH7JY | C02753207CDH69FJC
   iMac18,1 | C02VDBZ0H7JY | C02737700QXH69FA8
   iMac18,1 | C02VP0H6H7JY | C02746300CDH69FJA
   iMac18,1 | C02VL0W9H7JY | C02743303CDH69F8C
   iMac18,1 | C02V2NYMH7JY | C02728600J9H69FAD
```

The order is Product \| Serial \| Board Serial \(MLB\)

The `iMac18,1` part gets copied to Generic -&gt; SystemProductName.

The `Serial` part gets copied to Generic -&gt; SystemSerialNumber.

The `Board Serial` part gets copied to Generic -&gt; MLB.

We can create an SmUUID by running `uuidgen` in the terminal \(or it's auto-generated via my GenSMBIOS script\) - and that gets copied to Generic -&gt; SystemUUID.

We set Generic -&gt; ROM to either an Apple ROM \(dumped from a real Mac\), your NIC MAC address, or any random MAC address \(could be just 6 random bytes, for this guide we'll use `11223300 0000`\)

**Automatic**: YES

* Generates PlatformInfo based on Generic section instead of DataHub, NVRAM, and SMBIOS sections
* **SpoofVendor**: YES
    * Swaps vendor field for Acidanthera, generally not safe to use Apple as a vendor in most case
**UpdateDataHub**: YES

* Update Data Hub fields

**UpdateNVRAM**: YES

* Update NVRAM fields

**UpdateSMBIOS**: YES

* Updates SMBIOS fields

**UpdateSMBIOSMode**: Create

* Replace the tables with newly allocated EfiReservedMemoryType, use Custom on Dell laptops requiring CustomSMBIOSGuid quirk

# 9. UEFI

**ConnectDrivers**: YES

* Forces .efi drivers, change to NO will automatically connect added UEFI drivers. This can make booting slightly faster, but not all drivers connect themselves. E.g. certain file system drivers may not load.

**Drivers**: Add your .efi drivers here

* **Input** 
1. KeyForgetThreshold : Remove key unless it was submitted during this timeout in milliseconds.
* AppleKeyMapAggregator protocol is supposed to contain a fixed length buffer of currently pressed keys. However, the majority of the drivers only report key presses as interrupts and pressing and holding the key on the keyboard results in subsequent submissions of this key with some defined time interval. As a result we use a timeout to remove once pressed keys from the buffer once the timeout expires and no new submission of this key happened.
This option allows to set this timeout based on your platform. The recommended value that works on the majority of the platforms is 5 milliseconds. For reference, holding one key on VMware will repeat it roughly every 2 milliseconds and the same value for APTIO V is 3-4 milliseconds. Thus it is possible to set a slightly lower value on faster platforms and slightly higher value on slower platforms for more responsive input.

2. KeyMergeThreshold : Assume simultaneous combination for keys submitted within this timeout in milliseconds.
* Similarly to KeyForgetThreshold, this option works around the sequential nature of key submission. To be able to recognise simultaneously pressed keys in the situation when all keys arrive sequentially, we are required to set a timeout within which we assume the keys were pressed together.
Holding multiple keys results in reports every 2 and 1 milliseconds for VMware and APTIO V respectively. Pressing keys one after the other results in delays of at least 6 and 10 milliseconds for the same platforms. The recommended value for this option is 2 milliseconds, but it may be decreased for faster platforms and increased for slower.

3. KeySupport : Enable internal keyboard input translation to AppleKeyMapAggregator protocol.
* This option activates the internal keyboard interceptor driver, based on AppleGenericInput aka AptioIntputFix), to fill AppleKeyMapAggregator database for input functioning. In case a separate driver is used, such as UsbKbDxe, this option should never be enabled.

4. KeySupportMode : Set internal keyboard input translation to AppleKeyMapAggregator protocol mode.
* **Auto** : Performs automatic choice as available with the following preference: AMI, V2, V1. 
* **V1** : Uses UEFI standard legacy input protocol EFI_SIMPLE_TEXT_INPUT_PROTOCOL.
* **V2** : Uses UEFI standard modern input protocol EFI_SIMPLE_TEXT_INPUT_EX_PROTOCOL. 
* **AMI** : Uses APTIO input protocol AMI_EFIKEYCODE_PROTOCOL.

5. KeySwap : Swap Command and Option keys during submission.
* This option may be useful for keyboard layouts with Option key situated to the right of Command key.

6. PointerSupport : Enable internal pointer driver.
* This option implements standard UEFI pointer protocol (EFI_SIMPLE_POINTER_PROTOCOL) through select OEM protocols. The option may be useful on Z87 ASUS boards, where EFI_SIMPLE_POINTER_PROTOCOL is broken.

7. PointerSupportMode : Set OEM protocol used for internal pointer driver.
* Currently the only supported variant is ASUS, using specialised protocol available on select Z87 and Z97 ASUS boards. More details can be found in [LongSoft/UefiTool#116](https://github.com/LongSoft/UEFITool/pull/116).

8. TimerResolution : Set architecture timer resolution.
* This option allows to update firmware architecture timer period with the specified value in 100 nanosecond units. Setting a lower value generally improves performance and responsiveness of the interface and input handling.
The recommended value is 50000 (5 milliseconds) or slightly higher. ASUS boards use 60000 for the interface. Apple boards use 100000.

**Protocols**: (Most values can be ignored here as they're meant for real Macs/VMs)

* **ConsoleControl**: YES
    * Replaces Console Control protocol with a builtin version,  set to YES otherwise you may see text output during booting instead of nice Apple logo. Required for most APTIO firmware
* **FirmwareVolume**: NO
    * Fixes UI regarding Filevault, set to YES for better FileVault compatibilty
* **HashServices**: NO
    * Fixes incorrect cusor size when running FileVault, set to YES for better FileVault compatibilty
* **UnicodeCollation**: NO
    * Some older firmware have broken unicode collation, fixes UEFI shell compatibility on these systems(generally IvyBridge and older)


**Quirks**:

* **AvoidHighAlloc**: NO
   * Workaround for when te motherboard can't properly access higher memory in UEFI Boot Services. Avoid unless necessary\(affected models: GA-Z77P-D3 \(rev. 1.1\)\)
* **ExitBootServicesDelay**: `0`
   * Only required for very specific use cases like setting to `5` for ASUS Z87-Pro running FileVault2
* **IgnoreInvalidFlexRatio**: NO
   * Fix for when MSR\_FLEX\_RATIO \(0x194\) can't be disabled in the BIOS, required for all pre-skylake based systems
* **IgnoreTextInGraphics**: NO
   * Fix for UI corruption when both text and graphics outputs happen, set to YES with SanitiseClearScreen also set to YES for pure Apple Logo\(no verbose screen\)
* **ProvideConsoleGop**: YES
   * Enables GOP\(Graphics output Protcol\) which the macOS bootloader requires for console handle
* **ReleaseUsbOwnership**: NO
* Releases USB controller from firmware driver, avoid unless you know what you're doing. Clover equivalent is `FixOwnership`
* **RequestBootVarRouting**: NO
   * Redirects AptioMemeoryFix from `EFI_GLOBAL_VARIABLE_GUID` to `OC\_VENDOR\_VARIABLE\_GUID`. Needed for when firmware tries to delete boot entries and is recommended to be enabled on all systems for correct update installation, Startup Disk control panel functioning, etc.
* **ReplaceTabWithSpace**: NO
   * Depending on firmware, some system may need this to properly edit files in the UEFI shell when unable to handle Tabs. This swaps it for spaces instead but majority can ignore it but do note that ConsoleControl set to True may be needed
* **SanitiseClearScreen**: NO
   * Fixes High resolutions displays that display OpenCore in 1024x768, recommened for user with 1080P+ displays
* **ClearScreenOnModeSwitch**: NO
   * Needed for when half of the previously drawn image remains, will force black screen before switching to TextMode. Do note that ConsoleControl set to True may be needed

# AMD Zen Based CPU's and Threadripper

## Starting Point

You'll want to start with the sample.plist that OpenCorePkg provides you and rename it to config.plist. Next, open up your favourite XML editor like Xcode and we can get to work.

Kernel patches:
* [Ryzen/Threadripper(17h)](https://cdn.discordapp.com/attachments/611462337446281236/630939338133209139/OC-patches-17h.plist) (10.13, 10.14, and 10.15)
* [Bulldozer/Jaguar(15h/16h)](https://cdn.discordapp.com/attachments/611462337446281236/630939338133209138/OC-patches-15_16h.plist) (10.13, 10.14, and 10.15)

Do note that these patches are pulled from the [AMD OS X discord](https://discord.gg/EfCYAJW) and that they're made for OpenCore 0.5.0. Users of 0.5.1 will need to wait for updated patches.

**Extra Importamt Note**: Current kernel patches will only work with [041045f](https://github.com/acidanthera/OpenCorePkg/tree/cd481d011bab6dc887629fe15962cd8f3cbed775) and older. To ease users experiences, please use [OpenCorePkg 0.5.0](https://github.com/acidanthera/OpenCorePkg/releases)



# Time to boot with OpenCore!


![AboutThisMac](https://i.imgur.com/BjQwhPC.png)

# Making Opencore your default Bootloader

When you are satisfied opencore boots your system correctly, simply mount your Clover efi partition, (back it up somewhere safe) and overwrite it with your OpenCore one. Certain system BIOS may require you to manually remove Clover as an EFI boot option (rarely some system might need a factory reset to permanently remove it).


# Legacy Install

OpenCore supports DuetPkg which emulates a UEFI environment for legacy systems.

To start, you need the following:

* BootInstall.command
* Install source


Within your OpenCore build folder, navigate to `Utilities/BootInstall`. Here you'll find a file called `BootInstall.command`. What this does is install DuetPkg to your desired drive.


Now you'll want to run `BootInstall.command`, do note that you may need `sudo` for this to work correctly on newer versions of macOS

```text
sudo Utilities/BootInstall/BootInstall.command
```


This will give you a list of available disks, choose yours and you will be prompted to write a new MBR. Choose yes`[y]` and you'll be finished.
This will provide you with an EFI partition with a `boot` file, this is where we'll add our OpenCore EFI.




# TroubleShooting

# Stuck on EndRandomSeed
Couple problems:
* `ProvideConsoleGop` is likely missing as this is needed for transitioning to the next screen, this was originally part of AptioMemoryFix but is now within OpenCore as this quirk

Other possible problem is that some users either forget or cannot disable CFG-Lock in the BIOS(specifically relating to a locked 0xE2 MSR bit for power managment, obviously much safer to turn off CFG-Lock). When this happens, there's a couple fixes:

* Enable `AppleXcpmCfgLock`, this disables `PKG_CST_CNFIG_CONTROL` within the XNU itself and likely the cause of the stall. Clover equivalent is `KernelPm`
* Enable `AppleXcpmCfgLock`, this disables `PKG_CST_CNFIG_CONTROL` within AppleIntelCPUPowerManagment. Clover equivalent is `AppleIntelCPUPM`


# Still waiting on root device

* Gernally seen as a USB error, couple ways to fix:
   * if you're hitting the 15 port limit, you can temporarily get around this with `XhciPortLimit` but for long term use we recommend making a [USBmap](https://github.com/InsanelyHack/USBMap). CorpNewt also has a guide for this: [USBmap Guide](https://usb-map.gitbook.io/project/)
   * Other issue can be that certain firmwares won't pass USB ownership to macOS, to fix this we can enable `ReleaseUsbOwnership`. Clover equivalent is `FixOwnership`

# iMessage and Siri Broken

* En0 device not setup as `Built-in`, couple ways to fix:
   * Find PCI path for your NIC with [gfxutil](https://github.com/acidanthera/gfxutil/releases)(ex: ethernet@0). Then via DeviceProperties in your config.plist, apply the property of `built-in` with the value of `01` and type `Data`
   * [NullEthernet.kext](https://bitbucket.org/RehabMan/os-x-null-ethernet/downloads/) + [SSDT-RMNE](https://github.com/RehabMan/OS-X-Null-Ethernet/blob/master/ssdt-rmne.aml)

# Windows Startup Disk can't see APFS drives

* Outdated Bootcamp drivers(generally ver 6.0 will come with brigadier, BootCamp Utility in macOS provides newer version like ver 6.1). InsanelyHack has also forked brigadier fixing these issues as well: [brigadier](https://github.com/InsanelyHack/brigadier)

# Incorrect resolution with OpenCore

* Follow [Hiding Verbose](verbose.md) for correct setup, set `UIScale` to `02` for HiDPI
* Users also have noticed that setting `ConsoleMode` to Max will sometimes fail, leaving it empty can help

# Receiving "Failed to parse real field of type 1"
* A value is set as `real` when it's not supposed to be, generally being that Xcode converted `HaltLevel` by accident:
```
<key>HaltLevel</key>
<real>2147483648</real>
```
To fix, swap `real` for `integer`:
```
<key>HaltLevel</key>
<integer>2147483648</integer>
```

# Booting OpenCore reboots to BIOS

* Incorrect EFI folder structure, make sure all of your OC files are within an EFI folder located on your ESP(EFI system partition)

![Directory Structure from OpenCore's DOC](https://i.imgur.com/9RyBQ0L.png)


# How to Enable a bootchime?

* Copy BootChimeDxe.efi & AudioDxe.efi to OC/Drivers folder.
* Copy BootChimeCgf.efi & HdaCodecDump.efi to OC/Tools folder.
* Bootchime can be played through the WAV file in the EFI root folder.
* Add BootChimeDxe.efi & AudioDxe.efi drivers to UEFI/Drivers in Config.plist.
* Add BootChimeCfg.efi & HdaCodecDump.efi drivers to Misc/Tools in Config.plist.
* Reboot & wait a few seconds.
* The default sound reproduction is line 1 of the built-in sound, and the speaker volume depends on the NVRAM setting.


# Credits
* [khtonokernel] (https://github.com/khronokernel/Opencore-Vanilla-Desktop-Guide) For the Orginal Guide
* [Apple](https://www.apple.com) for MacOS.
* [Acidanthera](https://github.com/acidanthera) for everything they contribute to hackintosh. :)
* [Pavo-IM](https://github.com/Pavo-IM) for Opencore Builder and edits
* [ZISQO](https://github.com/zisqo) to translate this guide for korean language and update gigabyte and asus data.
* [MacProDude](https://github.com/MacProDude) for images and guide rewrite
* [AlGrey](https://amd-osx.com/forum/memberlist.php?mode=viewprofile&u=10918&sid=e0feb8a14a97be482d2fd68dbc268f97) AMD Kernel patches
* [InsanelyMac](https://www.insanelymac.com) For finding issues and solutions to different hardware issues


# Developers of OpenCore:

* [Download-Fritz](https://github.com/Download-Fritz)
* [Goldfish64](https://github.com/Goldfish64)
* [savvamitrofanov](https://github.com/savvamitrofanov)
* [vit9696](https://github.com/vit9696)


