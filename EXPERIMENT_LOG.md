# Acer Aspire E5-571 OpenCore — Detailed Experiment Log

> Last updated: 10 August 2026
> Status: Unfinished research project — macOS boot is not yet reliable.

## 1. Purpose of This Project

This repository documents my attempt to install and boot macOS Monterey on an Acer Aspire E5-571 using OpenCore.

This is not presented as a finished or universally working EFI release. It is a record of the real process: hardware investigation, disk conversion, firmware changes, EFI creation, ACPI work, configuration experiments, boot logs, black-screen failures and the solutions I tested.

I am publishing the unsuccessful attempts as well as the successful ones because the failed stages often contain the most useful information for someone working on similar Haswell-era Acer hardware.

## 2. Confirmed Hardware and System Information

The machine used for this project is:

| Component                | Information                                               |
| ------------------------ | --------------------------------------------------------- |
| Laptop                   | Acer Aspire E5-571                                        |
| Mainboard identification | Acer EA50_HB                                              |
| Processor                | Intel Core i5-4210U                                       |
| CPU generation           | Intel Haswell                                             |
| Base frequency           | 1.70 GHz                                                  |
| CPU layout               | 2 cores / 4 logical processors                            |
| Integrated graphics      | Intel HD Graphics 4400                                    |
| Memory                   | 8 GB RAM                                                  |
| Internal display         | 1366 × 768                                                |
| Windows installation     | Windows 10 Home Single Language                           |
| Storage layout           | Windows SSD and an additional HDD of approximately 465 GB |
| Wireless adapter         | Qualcomm Atheros AR956x                                   |
| Touchpad identification  | ELAN0501                                                  |
| Installation USB         | Approximately 30 GB SanDisk USB drive                     |

Some hardware details may differ between Acer E5-571 variants. Therefore, this work should not be treated as a ready-made configuration for every laptop carrying the same model name.

## 3. Initial Condition

The laptop originally ran Windows 10 and used a legacy/MBR-oriented boot configuration.

Before starting the OpenCore work, I checked:

* The CPU and platform generation.
* Available memory.
* Storage devices and partitions.
* Integrated graphics.
* BIOS boot options.
* Whether Windows could be converted without reinstalling it.
* Whether the machine could boot in UEFI mode.

Windows was installed on the SSD. The second internal drive was intended to provide space for macOS-related testing.

## 4. Converting the Windows Disk from MBR to GPT

The Windows disk was first checked with Microsoft’s `mbr2gpt` validation process.

The validation completed successfully. The conversion command was then executed and the disk was converted from MBR to GPT without reinstalling Windows.

After conversion:

1. The laptop was restarted into BIOS.
2. The boot mode was changed from Legacy/CSM operation to UEFI.
3. Secure Boot was checked.
4. Secure Boot was initially seen enabled during the work and was later disabled for OpenCore testing.
5. The firmware settings were saved with `F10`.
6. Windows remained discoverable through its UEFI bootloader.

Later OpenCore logs also detected the Microsoft boot file:

`EFI\Microsoft\Boot\bootmgfw.efi`

This confirmed that Windows Boot Manager was still visible to the UEFI/OpenCore environment.

## 5. OpenCore and Supporting Tools

The main OpenCore version used during this project was:

* OpenCore 1.0.7

Both RELEASE and DEBUG packages were downloaded during the experiments. The DEBUG build was useful because it produced boot logs that could be inspected after unsuccessful starts.

The following supporting tools were also used:

* ProperTree
* SSDTTime
* `ocvalidate`
* `macrecovery.py`
* OpenCore DEBUG and RELEASE packages

ProperTree was used to edit `config.plist` and perform OC Snapshot operations after changing ACPI files, drivers and kexts.

SSDTTime was used to generate or assist with the ACPI files and patches required for this laptop.

The macOS recovery files were prepared for a Monterey installation. The tested Monterey version/build was:

* macOS Monterey 12.7.6
* Build 21H1320

## 6. USB and EFI Structure

A SanDisk USB drive of approximately 30 GB was used.

The Windows drive letter assigned to the OpenCore partition changed during the work. Paths seen at different stages included:

* `J:\EFI\OC\`
* `F:\EFI\OC\`

When inspected from macOS-related environments, the volume was also identified as:

* `/Volumes/OPENCORE`

The basic EFI structure was organised as follows:

```text
EFI
├── BOOT
│   └── BOOTX64.EFI
└── OC
    ├── ACPI
    ├── Drivers
    ├── Kexts
    ├── Resources
    ├── Tools
    ├── OpenCore.efi
    └── config.plist
```

OpenCore was successfully located from:

`EFI\BOOT\BOOTX64.EFI`

One boot log showed OpenCore being found on the USB’s EFI area even while the machine also contained GPT partitions and the Windows UEFI installation.

## 7. ACPI Work

The following ACPI files were added and enabled during the project:

* `SSDT-EC.aml`
* `SSDT-HPET.aml`
* `SSDT-PLUG.aml`
* `SSDT-PNLF.aml`
* `SSDT-XOSI.aml`

Their intended roles were:

| ACPI file       | Intended purpose                                                |
| --------------- | --------------------------------------------------------------- |
| `SSDT-EC.aml`   | Provide a macOS-compatible embedded controller setup            |
| `SSDT-HPET.aml` | Help resolve IRQ and HPET-related conflicts                     |
| `SSDT-PLUG.aml` | Attach processor power-management properties                    |
| `SSDT-PNLF.aml` | Provide internal display backlight support                      |
| `SSDT-XOSI.aml` | Assist with OSI handling, mainly for input-device compatibility |

A rename related to the touchpad/GPIO path was also added:

* `GPIO_STA` → `XSTA`

SSDTTime produced a `patches_OC.plist` file during the ACPI work. The relevant patches were transferred into the main OpenCore configuration instead of using the generated file as a complete replacement.

All ACPI entries were checked in ProperTree and enabled in the `ACPI -> Add` section.

## 8. UEFI Drivers

The main drivers used were:

* `OpenRuntime.efi`
* `OpenHfsPlus.efi`

At different experimental stages, the Drivers folder also contained:

* `Ps2KeyboardDxe.efi`
* `OpenCanopy.efi`
* `ResetNvramEntry.efi`

Not all of these remained active in every configuration.

The logs confirmed that `OpenRuntime.efi` and `OpenHfsPlus.efi` could be loaded successfully.

One log reported:

```text
Got 3 drivers
Watchdog status is 0
```

Another warning seen in the log was related to missing FSB frequency data:

```text
Failed to get FSBFrequency data ... - Not Found
```

This line was recorded for investigation, but it was not treated by itself as proof of the final boot failure.

## 9. Kernel Extensions

The core kext versions prepared during the project included:

| Kext          | Version |
| ------------- | ------- |
| Lilu          | 1.7.2   |
| VirtualSMC    | 1.3.7   |
| WhateverGreen | 1.7.0   |
| AppleALC      | 1.9.7   |

Additional kexts used or prepared during later stages included:

* `VoodooPS2Controller.kext`
* `USBToolBox.kext`
* `UTBMap.kext`

The input-device investigation also involved the VoodooI2C method because the laptop reported an ELAN0501 I2C touchpad. The final touchpad configuration was not completed successfully.

The main functions of the kexts were:

* Lilu: patching framework required by several other kexts.
* VirtualSMC: Apple SMC emulation.
* WhateverGreen: Intel integrated graphics support and graphics-related patching.
* AppleALC: onboard audio support.
* VoodooPS2Controller: PS/2 keyboard and input-device support.
* USBToolBox and UTBMap: USB port mapping.

## 10. Important `config.plist` Settings

The configuration was edited repeatedly while boot results were compared.

Confirmed settings used during the work included:

```text
ShowPicker = True
Timeout = 30
DisableWatchDog = True
ProvideConsoleGop = True
Resolution = Max
```

The OpenCore picker was intentionally kept visible so Windows, macOS/recovery and OpenCore auxiliary entries could be inspected.

The configuration contained the enabled ACPI files, drivers and kexts listed above.

The `BuiltinGraphics` setting was initially set to `False`. It was later changed to `True` during graphics/console troubleshooting.

After this change, while `Resolution` remained set to `Max`, the laptop restarted into a black screen before the OpenCore picker appeared. The result was reproduced on another restart.

This does not prove that `BuiltinGraphics=True` was the only cause. It only records the exact sequence observed during the test.

## 11. SMBIOS Experiments

The original hardware was identified in logs as:

* Acer Aspire E5-571
* Mainboard/platform: EA50_HB

The macOS SMBIOS values tested during different stages included:

* `MacBookPro11,4`
* `MacBookPro11,1`

The purpose was to find a Haswell-era Mac profile compatible with the CPU and Intel integrated graphics.

The project does not publish the generated private SMBIOS identity values. The following fields must always be regenerated by each user:

* `SystemSerialNumber`
* `MLB`
* `ROM`
* `SystemUUID`

Any future EFI uploaded to this repository must contain placeholders or newly generated test values instead of my personal platform identity.

## 12. Intel HD Graphics 4400 Investigation

The Acer uses Intel HD Graphics 4400, integrated into the Haswell i5-4210U processor.

WhateverGreen was installed and the following areas were investigated:

* Haswell iGPU handling.
* DeviceProperties injection.
* Internal display output.
* OpenCore console/GOP behaviour.
* Framebuffer-related configuration.
* `ProvideConsoleGop`.
* `BuiltinGraphics`.
* Resolution handling.

The firmware reported four GOP display modes:

* 1366 × 768
* 1024 × 768
* 800 × 600
* 640 × 480

The current firmware mode matched the laptop panel’s native 1366 × 768 resolution.

Because the exact final framebuffer/device-property combination was not proven stable, this repository does not claim a finished Intel HD 4400 graphics configuration.

## 13. OpenCore Picker and OpenCanopy Results

OpenCore successfully reached stages where it:

* Located `BOOTX64.EFI`.
* Loaded `config.plist`.
* Loaded the main UEFI drivers.
* Detected several possible filesystems.
* Detected Windows Boot Manager.
* Detected APFS-related structures in later tests.
* Offered or attempted to offer boot entries through the picker.

OpenCanopy was present during part of the testing, but one log showed it being skipped.

Other UI-related errors included:

```text
OCUI: Failed to load images
OC: External interface failure, fallback to builtin - Unsupported
```

OpenCore then attempted to fall back to its built-in interface.

This suggested a problem with the graphical picker resources or configuration rather than an inability to start OpenCore itself.

## 14. APFS and Boot Scanning

The boot logs showed that OpenCore scanned multiple filesystems. One log recorded three potential filesystems.

The USB EFI was found and the Windows UEFI loader was detected.

At some stages, APFS recovery or bless-related targets were not found. At a later stage, APFS was detected, but one boot attempt stopped after the last visible line:

```text
apfs.efi: LocateProtocol(AppleLogging) succeeded
```

This line may only have been the final visible message rather than the actual cause of the freeze. It is documented here exactly because diagnosing Hackintosh boot failures requires separating the last printed line from the real failure.

## 15. Black-Screen Behaviour

Several black-screen situations occurred, and they did not all behave identically.

### Black screen before the OpenCore picker

During the console/graphics experiments:

1. `BuiltinGraphics` was changed from `False` to `True`.
2. `Resolution` was left at `Max`.
3. The configuration was saved.
4. The laptop was restarted.
5. The screen remained black before the OpenCore picker appeared.
6. A second restart produced the same result.

This was one of the most important failures in the project because it happened before macOS itself was selected.

### Later recovery from the black screen

During later input-device testing, the screen opened again after another restart. This showed that the black-screen condition was not necessarily a permanent hardware failure.

The OpenCore menu could again be awaited and the `Macintosh HD` entry was used as part of the later boot attempts.

## 16. Touchpad and Input-Device Investigation

The laptop’s touchpad was identified as:

* ELAN0501
* I2C-connected device

The touchpad appeared to involve IRQ/GPIO and ACPI compatibility issues.

The following components or changes were involved in the investigation:

* `SSDT-HPET.aml`
* `SSDT-XOSI.aml`
* GPIO-related ACPI rename
* VoodooPS2Controller
* VoodooI2C-oriented troubleshooting
* `-vi2c-force-polling`

The boot argument tested was:

```text
-vi2c-force-polling
```

After adding it, OpenCore was still able to reach the screen during a later attempt, meaning the argument did not permanently break OpenCore startup.

However, after an NVRAM reset, another boot attempt appeared to stop around:

```text
apfs.efi: LocateProtocol(AppleLogging) succeeded
```

The touchpad was not confirmed fully operational. It remains one of the unresolved parts of the project.

## 17. Wi-Fi Status

The internal wireless adapter was identified as Qualcomm Atheros AR956x.

It did not provide working native Wi-Fi in the tested macOS Monterey configuration.

No claim is made that the internal AR956x adapter is supported by the current experimental EFI. Wireless networking therefore remains unresolved.

## 18. USB Mapping

The later kext set included:

* `USBToolBox.kext`
* `UTBMap.kext`

These were prepared to provide a mapped USB configuration rather than depending permanently on generic USB injection.

USB mapping still needs to be verified under a stable macOS boot before it can be considered finished.

## 19. Main Results

| Test                                 | Result                      |
| ------------------------------------ | --------------------------- |
| Windows MBR validation               | Successful                  |
| MBR-to-GPT conversion                | Successful                  |
| UEFI boot conversion                 | Successful                  |
| Windows Boot Manager detection       | Successful                  |
| OpenCore `BOOTX64.EFI` detection     | Successful                  |
| `config.plist` loading               | Successful                  |
| OpenRuntime loading                  | Successful                  |
| OpenHfsPlus loading                  | Successful                  |
| Firmware GOP mode detection          | Successful                  |
| Native 1366 × 768 mode detection     | Successful                  |
| OpenCanopy graphical interface       | Unreliable / resource error |
| Built-in OpenCore interface fallback | Attempted                   |
| APFS detection                       | Reached during later tests  |
| Monterey boot reliability            | Not solved                  |
| Intel HD 4400 acceleration           | Not confirmed               |
| Internal AR956x Wi-Fi                | Not working                 |
| ELAN0501 touchpad                    | Not solved                  |
| Stable everyday macOS system         | Not achieved                |

## 20. Current Status

The project is currently unfinished.

The machine can reach important OpenCore stages, and several parts of the configuration have been verified:

* UEFI boot works.
* The OpenCore loader is found.
* The main configuration is loaded.
* Core UEFI drivers load.
* Windows Boot Manager is detected.
* The firmware exposes the correct 1366 × 768 display mode.
* APFS-related stages can be reached.

However, the system is not yet a stable Hackintosh.

The remaining problems include:

* Intermittent black screen before the picker.
* OpenCanopy image/resource errors.
* Unreliable macOS/Monterey startup.
* Intel HD 4400 graphics configuration.
* ELAN0501 I2C touchpad support.
* Internal Atheros AR956x Wi-Fi.
* Final USB mapping verification.
* Final audio verification.
* Selecting and validating the best SMBIOS.

## 21. Important Warning

This is an experimental research log, not a ready-to-use EFI.

Do not copy the configuration without checking:

* Your exact CPU.
* Your graphics hardware.
* Your display resolution.
* Your storage controller.
* Your Wi-Fi card.
* Your touchpad connection.
* Your BIOS version.
* Your own ACPI tables.

Never use another person’s SMBIOS identity values. Generate your own serial number, MLB, ROM and UUID.

Always keep a Windows recovery option and a backup of important files before changing partitions, firmware settings or bootloaders.

## 22. Next Investigations

The next technical steps are:

1. Return to a minimal text-based OpenCore picker.
2. Remove unnecessary graphical OpenCanopy variables during diagnosis.
3. Compare the last configuration that displayed the picker with the configuration that produced the pre-picker black screen.
4. Recheck `BuiltinGraphics`, `ProvideConsoleGop` and resolution behaviour individually.
5. Validate the entire configuration with the OpenCore 1.0.7 version of `ocvalidate`.
6. Review the Haswell Intel HD 4400 DeviceProperties.
7. Test macOS boot before adding touchpad-specific arguments.
8. Add input-device changes one at a time.
9. Verify USB mapping only after obtaining a repeatable macOS boot.
10. Keep private SMBIOS information out of all public commits.

## 23. Final Note

This repository is a record of learning through real testing.

I did not begin with a prebuilt EFI. The configuration was assembled, edited and tested step by step. Some changes improved the process, some caused new failures, and some produced unclear results that required another test.

The goal is to keep the history honest. If the installation becomes fully stable, this log will make it possible to see not only the final solution, but also the exact path that led to it.
