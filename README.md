# hackintosh-deskmini-310-i7-8700-OpenCore

- [Hardware](#hardware)
- [BIOS](#bios)
- [Display Connectors](#display-connectors)
- [OpenCore](#opencore)
- [config.plist](#configplist)
- [ACPI](#acpi)
- [Kexts](#kexts)
- [Drivers](#drivers)
- [Quick Tips](#quick-tips)
- [Useful Apps](#useful-apps)
- [Installing/Dualbooting Windows](#installingdualbooting-windows)


## Hardware
- Case & Mobo: ASRock DeskMini H310
- CPU: Intel i7-8700 w/ Noctua NH-L9i
- RAM: Corsair Memory Kit 16GB DDR4 2400MHz SODIMM Memory (CMSX16GX4M2A2400C16)
- SSD: Samsung 860 EVO 500 GB
- WiFi & Bluetooth: BCM94360CS2 w/ M.2 adapter
- Display: BenQ GW2480 (DisplayPort, VESA)

## BIOS
- v3.10: works out-of-box
- Beyond v3.10: DSDT patch required

    ```
    comment: Fix RTC _STA bug (fix asrock new bios failed to boot)
    Find: A00A9353 54415301
    Replace: A00A910A FF0BFFFF
    ```
- Settings
    1. Load UEFI Defaults
    2. Advanced
        - Onboard HD Audio: Enabled
        - USB Configuration - XHCI Hand-off: Enabled
        - Super IO Configuraton - Serial Port: Disabled

## Display Connectors
- *DisplayPort*: best option, has signal since start of boot and should work out-of-box; if your display only has HDMI connectors, a DisplayPort to HDMI cable is recommended, note that the only confirmed-working model so far is DP111 by Ugreen (PS176 chipset).
- *HDMI*: can get working with extra hack for multi-screen setup; only receives signal after entring MacOS.

## OpenCore
Version: 0.8.9

## config.plist

Mostly following the OpenCore Install Guide [here](https://dortania.github.io/OpenCore-Install-Guide/config.plist/coffee-lake.html).

- Booter

    - Quirks

        > Fixing KASLR slide values ([Reference](https://dortania.github.io/OpenCore-Install-Guide/extras/kaslr-fix.html#fixing-kaslr-slide-values))

        - AvoidRuntimeDefrag: True
        - DevirtualiseMmio: True
        - EnableSafeModeSlide: True
        - ProtectUefiServices: False
        - ProvideCustomSlide: True
        - RebuildAppleMemoryMap: True

- DeviceProperties

    - Add

        - PciRoot(0x0)/Pci(0x1f,0x3)

            - device-id: cKEAAA==
            - layout-id: HAAAAA==

        - PciRoot(0x0)/Pci(0x2,0x0)

            > Enable HDMI output and resolve other iGPU issues.

            - AAPL,ig-platform-id: BwCbPg==
            - device-id: mz4AAA==
            - enable-hdmi20: AQAAAA==
            - framebuffer-con0-enable: AQAAAA==
            - framebuffer-con0-pipe: EgAAAA==
            - framebuffer-con1-busid: AgAAAA==
            - framebuffer-con1-enable: AQAAAA==
            - framebuffer-con1-pipe: EgAAAA==
            - framebuffer-con1-type: AAgAAA==
            - framebuffer-con2-enable: AQAAAA==
            - framebuffer-con2-index: /////w==
            - framebuffer-fbmem: AACQAA==
            - framebuffer-patch-enable: AQAAAA==
            - framebuffer-portcount: AgAAAA==
            - framebuffer-unifiedmem: AAAAgA==

- Kernel

    - Quirks

      > Already supports CFG Lock unlocking

      - AppleCpuPmCfgLock: False
      - AppleXcpmCfgLock: False

## ACPI

| SSDT               | Purpose                                                      |
| ------------------ | ------------------------------------------------------------ |
| SSDT-PLUG.aml      | To allow the kernel's XCPM (XNU's CPU Power Management) to take over our CPU's power management |
| SSDT-EC-USBX.aml   | 1) On desktops, the EC (or better known as the embedded controller) isn't compatible with `AppleACPIEC` driver, to get around this we disable this device when running macOS <br /><br />2) `AppleBusPowerController` will look for a device named `EC`, so we will want to create a fake device for this kext to load onto. `AppleBusPowerController` also requires a `USBX` device to supply USB power properties for Skylake and newer, so we will bundle this device in with the `EC` fix |
| SSDT-PMC.aml       | NVRAM support                                                |
| SSDT-SBUS-MCHC.aml | Fix `AppleSMBus` support in macOS ([Reference](https://dortania.github.io/Getting-Started-With-ACPI/Universal/smbus.html#what-this-ssdt-does)) |

## Kexts

> All of the following kexts are available on this [repo](https://1drv.ms/f/s!AiP7m5LaOED-m-J8-MLJGnOgAqnjGw) courtesy of Goldfish64. Each kext is auto-built whenever a new commit is made. ([Reference](https://hackintosh.gitbook.io/-r-hackintosh-vanilla-desktop-guide/gathering-kexts))

Kext | Version | Purpose
---- | ------- | -------
AppleALC.kext | 1.7.9 | Driver for Realtek ALC233 (Mic & HDMI audio breaks in 1.5.9 and 1.6.0)
IntelMausi.kext | 1.0.7 | Driver for Intel® Gigabit I219V
Lilu.kext | 1.6.3 | Fundation kext for many other kexts
WhateverGreen.kext | 1.6.4 | A composite kext that addresses graphics related issues, requires `Lilu.kext`
VirtualSMC.kext | 1.3.0 | SMC emulator, vital to booting hackintosh
SMCProcessor.kext | 1.3.0 | Companion kext for `VirtualSMC.kext`
SMCSuperIO.kext | 1.3.0 | Companion kext for `VirtualSMC.kext`
USBPorts.kext | - | Customized kext to work around USB port limit
NVMeFix.kext | 1.1.0 | Improve compatibility and power management of 3rd party M2 SSDs 
XHCI-unsupported.kext | 0.9.2 | Needed for non-native USB controllers 

## Drivers

Driver | Purpose
------ | -------
HfsPlus.efi | Needed for seeing HFS volumes (ie. macOS Installers and Recovery partitions/images) 
OpenRuntime.efi | Replacement for [AptioMemoryFix.efi](https://github.com/acidanthera/AptioFixPkg), used as an extension for OpenCore to help with patching boot.efi for NVRAM fixes and better memory management 
OpenCanopy.efi | For GUI and boot-chime 

## Quick Tips

- To make OpenCore remember last boot entry and set it as default, use **CTRL + ENTER** to select the boot entry.

- There are two ways to check the running OpenCore version:

  1. Use the plain text theme and find the version at the bottom of the boot screen

  2. Execute the following in a terminal (note that there was a bug in OpenCore prior to 0.6.7 that leads to stale version name written to the nvram, reset nvram to fix):

     ```shell
     % nvram 4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102:opencore-version
     4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102:opencore-version	REL-069-2021-05-03
     ```

## Useful Apps

| App   | Home Page                        | Usage                              |
| ----- | -------------------------------- | ---------------------------------- |
| Stats | https://github.com/exelban/stats | Open source version of iStat Menus |

## Installing/Dualbooting Windows

- Recommended to create the Windows bootable USB installer on a Windows machine (either with the official Media Creation Tool or [Rufus](https://rufus.ie/) if you already have the ISO image). Making it on macOS with the rsync + wimlib approach wasn't successful in my several attempts.

- AFAIK It's not possible to install Windows on a machine that has multiple EFI partitions, which happens if you are installing Windows on a separate disk. To work around this, I used DISM to apply the Windows image manually with the steps elaborated below:

  1. Boot into the Windows installer and next all the way until you reach the "Where do you want to install Windows" page. Create the partition to host the Windows OS with your preferred size. In my experience, the installer will create 3 partitions automatically: EFI partition (100MB), MSR partition (16MB), and the Primary partition. Remember to format the Primary partition. Now, hit the cross button top right which will navigate you back to the installer home screen.

  2. Press SHIFT + F10 to launch the Command Prompt.

  3. Enter `diskpart` followed by `list vol`. From the list, locate the installer USB volume as well as the Windows volume that has just been created, note down their corresponding letter in the `Ltr` column. For illustration purpose, we'll use `C` as the Windows volume, and `D` as the installer USB volume. Enter `exit` to get back to where we started.

  4. Enter the following to locate the index of the Windows edition you'd like to install (change `install.wim` to `install.esd` if the installer USB was created with the official Media Creation Tool):

     ```shell
     dism /Get-WimInfo /WimFile:D:\Sources\install.wim
     ```

     We'll assume the edition is at `Index: 1` here as an example.

  5. Use the following command to apply the image and install Windows onto the target partition (again, change `install.wim` to `install.esd` if the installer USB was created with the official Media Creation Tool):

     ```shell
     dism /Apply-Image /ImageFile:D:\Sources\install.wim /index:1 /ApplyDir:C:\
     ```

  6. Now we need to install the Windows bootloader. Go back to `diskpart` then `list vol`, locate the EFI partition that the Windows installer created for us, then enter `select volume X` where `X` is the volume number. Once the correct volume is selected, enter `assign letter=Z` to make sure we can reference the volume by letter `Z` later. Enter `exit` to get back out. Lastly, enter the following to create the bootloader in the EFI partition so OpenCore can detect it later:

     ```shell
     bcdboot C:\Windows /s Z: /f UEFI
     ```

  7. You can now exit the installer, reboot, and launch Windows via OpenCore. If the Windows boot option is not picked up, ensure you have set `Misc - Security - ScanPolicy` to `0` in `config.plist`.

  8. Necessary drivers i.e. WiFi, Bluetooth, Audio, Chipset, etc are included in the `WIN10_Drivers` folder.
