# hackintosh-deskmini-310-i7-8700-OpenCore

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
Version: 0.6.4

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
AppleALC.kext | 1.5.5 | Driver for Realtek ALC233
IntelMausi.kext | 1.0.4 | Driver for Intel® Gigabit I219V
Lilu.kext | 1.5.0 | Fundation kext for many other kexts
WhateverGreen.kext | 1.4.5 | A composite kext that addresses graphics related issues, requires `Lilu.kext`
VirtualSMC.kext | 1.1.9 | SMC emulator, vital to booting hackintosh
SMCProcessor.kext | 1.1.9 | Companion kext for `VirtualSMC.kext`
SMCSuperIO.kext | 1.1.9 | Companion kext for `VirtualSMC.kext`
USBPorts.kext | - | Customized kext to work around USB port limit
NVMeFix.kext | 1.0.4 | Improve compatibility and power management of 3rd party M2 SSDs 
XHCI-unsupported.kext | 0.9.2 | Needed for non-native USB controllers 

## Drivers

Driver | Purpose
------ | -------
HfsPlus.efi | Needed for seeing HFS volumes (ie. macOS Installers and Recovery partitions/images) 
OpenRuntime.efi | Replacement for [AptioMemoryFix.efi](https://github.com/acidanthera/AptioFixPkg), used as an extension for OpenCore to help with patching boot.efi for NVRAM fixes and better memory management 
OpenCanopy.efi | For GUI and boot-chime 

## Useful Apps

| App   | Home Page                        | Usage                              |
| ----- | -------------------------------- | ---------------------------------- |
| Stats | https://github.com/exelban/stats | Open source version of iStat Menus |


