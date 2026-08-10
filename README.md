# Acer E5-571 OpenCore Journey

This repository is my complete OpenCore / macOS experiment on an Acer Aspire E5-571.

I'm not creating this repository just to upload a finished EFI folder.

The real purpose is to document what happens while trying to make macOS work on this machine.

Every setting I change, every boot result, black screen, strange behavior, useful Windows command, OpenCore log and hardware detail I find will eventually be written here.

A lot of information about older laptops like this is scattered between old forum posts, GitHub repositories, OpenCore documentation and discussions that sometimes disappear years later.

Another problem is that two laptops can both be called "Acer Aspire E5-571" while having slightly different hardware inside.

Different Wi-Fi cards, different displays, different storage controllers, different firmware revisions or other small hardware differences can completely change the result.

Because of that, I'm trying not to blindly copy another person's EFI.

I want to understand this particular machine first.

And I'm keeping the failed attempts too.

Sometimes knowing what **didn't work** is more useful than downloading somebody else's finished EFI.

---

# The machine

The laptop I'm working on is:

**Acer Aspire E5-571**

Current confirmed information:

| Part | Information |
|---|---|
| Manufacturer | Acer |
| Model | Aspire E5-571 |
| CPU | Intel Core i5-4210U |
| CPU Base Frequency | 1.70 GHz |
| CPU Architecture | Intel Haswell |
| CPU Cores | 2 |
| CPU Threads | 4 |
| RAM | 8 GB |
| Graphics reported by Windows | Intel(R) HD Graphics |
| Storage | SSD + HDD |
| Windows | Windows 10 Home Single Language |
| Bootloader being tested | OpenCore |
| Target | macOS |

Some hardware information is intentionally not filled in yet.

I would rather write:

`Unknown - still checking`

than guess a device model and leave incorrect information in this repository.

Things that still need exact identification include:

- exact Intel graphics device ID
- exact framebuffer requirements
- internal display information
- audio codec
- Ethernet controller
- Wi-Fi card
- Bluetooth controller
- touchpad controller
- keyboard controller
- SATA controller
- USB controller layout
- BIOS version
- SSD model
- HDD model
- PCI device IDs
- ACPI device names
- internal display connector path

These will be added after I verify them directly from the machine.

---

# Why I'm doing this

The basic goal sounds simple:

**Get macOS running properly on this Acer Aspire E5-571 using OpenCore.**

But I don't want the result to be just:

`EFI.zip`

and nothing else.

I want to know why the configuration works.

If something doesn't work, I want to know where it starts going wrong.

My approach is basically:

1. Identify the actual hardware.
2. Check the current OpenCore configuration.
3. Understand what a setting is supposed to do.
4. Change as little as possible during one experiment.
5. Boot the laptop.
6. Observe exactly what happens.
7. Save useful logs or screenshots.
8. Write down the result.
9. Compare it with the previous test.
10. Only then make another change.

I don't want to change ten unrelated settings at the same time and later say:

> somehow it works now

because then I haven't actually learned what fixed the problem.

That also makes troubleshooting much harder if the same problem returns later.

---

# Current state

The current main problem is a **black screen during the OpenCore boot process**.

The laptop has been able to execute OpenCore far enough to produce OpenCore debug information.

Parts of the boot log have shown successful DataHub operations such as:

```text
OCDH: Setting DataHub ... name - Success
OCDH: Setting DataHub ... Model - Success
OCDH: Setting DataHub ... SystemSerialNumber - Success
OCDH: Setting DataHub ... system-id - Success
```

This is useful because it tells me that OpenCore is not simply failing immediately after the machine powers on.

OpenCore is executing code and progressing through at least part of its initialization.

The screen disappears somewhere later in the process.

Exactly where and why is still under investigation.

I don't want to assume this is purely a GPU problem yet.

Possible areas that eventually need to be checked include:

- OpenCore graphical output
- UEFI GOP behavior
- integrated graphics initialization
- DeviceProperties
- WhateverGreen configuration
- framebuffer configuration
- boot arguments
- internal display handling
- firmware behavior
- OpenCore UEFI output settings

For now the symptom is simply recorded as:

**Black screen.**

---

# Black screen investigation

This is the main thing I'm working on right now.

One configuration option I found during testing was:

`BuiltinGraphics`

Its original value was:

`False`

I changed it to:

`True`

Then I saved the configuration and restarted the laptop.

Result:

**Black screen again.**

So at this point I know:

`BuiltinGraphics = True`

does **not** solve the problem by itself.

That doesn't automatically mean the option is wrong.

It only means changing this single value did not fix the current symptom.

Another display-related value currently seen in the configuration is:

`Resolution = Max`

This is also being kept in mind while investigating the display path.

I'm not claiming either of these values is responsible for the black screen yet.

They are simply known variables in the current configuration.

---

# First recorded tests

I want every meaningful experiment to eventually have its own test number.

For now the first known tests can be recorded like this.

## OC-0001

### Configuration

`BuiltinGraphics = False`

### Observed result

Black screen problem present.

### Conclusion

No solution.

The configuration was able to reach part of the OpenCore initialization process, but the display problem remained.

---

## OC-0002

### Changed

`BuiltinGraphics`

From:

`False`

To:

`True`

### Known related setting

`Resolution = Max`

### Reason

Testing whether changing OpenCore's built-in graphics handling changes the display behavior.

### Result

Black screen again after reboot.

### Conclusion

Changing `BuiltinGraphics` from `False` to `True` did not solve the problem by itself.

### Next step

Do not start changing unrelated settings randomly.

First investigate the graphics and display initialization path and compare the OpenCore configuration with the actual hardware installed in this laptop.

---

# How I want to record future tests

Every important change should eventually have something similar to this:

```text
Test ID:
OC-0003

Date:
YYYY-MM-DD

Changed:
Setting name

Previous value:
Old value

New value:
New value

Reason:
Why I changed it.

Expected result:
What I thought might happen.

Actual result:
What actually happened.

Boot status:
Booted / Black screen / Freeze / Reboot / Error

Log:
Available / Not available

Screenshot:
Available / Not available

Decision:
Keep / Revert / Investigate further

Notes:
Anything strange or important.
```

This might look excessive when testing one setting.

But after 30, 40 or 50 experiments it becomes extremely useful.

Without this, it becomes very easy to forget:

- which value was originally used
- which test caused a new problem
- whether an option actually changed anything
- whether a setting was later reverted
- which OpenCore configuration produced a particular log

I want to avoid that.

---

# What I actually want to understand

I'm not only trying to make the screen turn on.

I want to know **why** it turns on.

For the graphics side, the rough chain I'm interested in understanding is:

```text
Intel iGPU
    ↓
PCI device
    ↓
ACPI device
    ↓
OpenCore
    ↓
DeviceProperties
    ↓
Lilu / WhateverGreen
    ↓
framebuffer configuration
    ↓
display connector
    ↓
internal laptop panel
    ↓
macOS graphics initialization
```

If something breaks somewhere in this chain, randomly trying boot arguments might occasionally make the system boot, but it doesn't tell me what was actually wrong.

So I want to investigate the individual pieces.

---

# Windows is part of the investigation

I'm keeping Windows available on the laptop during this project.

Windows is extremely useful for identifying the machine's actual hardware.

Instead of trusting random Acer specification pages, I can query the hardware directly.

Information collected from Windows will eventually include:

- Device Manager information
- Hardware IDs
- PCI IDs
- USB IDs
- PnP information
- graphics adapter information
- storage devices
- network controllers
- audio devices
- BIOS information
- ACPI information
- monitor/display information
- disk information
- USB devices

Useful CMD and PowerShell commands will also be saved.

This gives me a hardware reference even if Windows is removed or reinstalled later.

---

# Windows commands

Commands that help identify hardware or diagnose the system will be stored under:

`Windows/commands-used.md`

For example, Windows PnP information can be useful for finding exact USB hardware.

Commands and their results should eventually be documented together instead of only keeping the command itself.

The idea is to record:

```text
Command:
<command>

Reason:
Why I ran it.

Important result:
What useful information came back.

Used for:
Which OpenCore/macOS problem this helped investigate.
```

---

# Hardware identification

The hardware section will eventually contain separate documentation for important devices.

Planned structure:

```text
Hardware/
├── specifications.md
├── CPU.md
├── GPU.md
├── Display.md
├── Audio.md
├── Ethernet.md
├── WiFi-Bluetooth.md
├── Storage.md
├── USB.md
├── PCI-devices.md
└── screenshots/
```

For every important device I want to record information like:

```text
Device name
Manufacturer
Windows device name
PCI ID
USB ID
Subsystem ID
Driver
ACPI device
macOS support status
Required kext
Required ACPI modification
OpenCore configuration
Working / Not working
Notes
```

If I don't know something yet, it stays unknown.

No guessing.

---

# Graphics documentation

Graphics deserves its own section because it is currently the biggest problem.

The current known graphics information is:

```text
Device:
Intel integrated graphics

CPU generation:
Haswell

CPU:
Intel Core i5-4210U

Windows device name:
Intel(R) HD Graphics

Exact PCI ID:
Not verified yet

OpenCore status:
Under investigation

macOS status:
Not working yet

Current symptom:
Black screen
```

Before making complicated framebuffer changes, I want to verify the actual graphics device information first.

Things to investigate later:

- exact PCI ID
- ACPI path
- device name
- framebuffer
- platform-id
- device-id requirements
- connector configuration
- internal panel
- HDMI output
- WhateverGreen behavior
- graphics acceleration
- brightness control
- sleep/wake graphics behavior

---

# OpenCore logs

Boot logs are not going to be deleted just because a boot failed.

Failed logs can be extremely useful.

They will eventually be stored under:

```text
OpenCore/logs/
```

When possible, each important log should be associated with the test that produced it.

For example:

```text
OpenCore/logs/

OC-0001/
├── opencore-log.txt
├── config-notes.txt
└── result.txt

OC-0002/
├── opencore-log.txt
├── config-notes.txt
└── result.txt

OC-0003/
├── opencore-log.txt
├── config-notes.txt
└── result.txt
```

The idea is simple.

Six months later I should still be able to answer:

**Which configuration produced this log?**

without guessing.

---

# Configuration history

Every meaningful configuration change should have a record.

Example:

```text
Test ID:
OC-0007

Changed:
BuiltinGraphics

Previous:
False

New:
True

Reason:
Testing whether OpenCore graphics output behavior changes.

Result:
Black screen remained.

Decision:
Further investigation required.
```

Configuration history will eventually make it possible to trace the entire troubleshooting process from the first experiment to the final working configuration.

---

# EFI

Eventually there may be a complete EFI directory in this repository.

But I don't want the EFI folder to become the only useful part of the project.

A final EFI without explanation is useful for copying.

A documented EFI is useful for understanding.

The OpenCore section will probably look something like:

```text
OpenCore/
├── EFI/
├── configuration-notes.md
├── boot-args.md
├── ACPI.md
├── Kexts.md
├── Drivers.md
├── DeviceProperties.md
├── UEFI.md
└── logs/
```

Important configuration decisions should be explained.

If something was added, I want to know why it was added.

If something was removed, I want to know why it was removed.

---

# ACPI

ACPI will be handled carefully.

I don't want to collect random SSDTs just because another Haswell laptop used them.

For every SSDT or ACPI change, I want to be able to answer:

- What problem is this supposed to solve?
- Does this Acer actually need it?
- What ACPI device is being modified?
- What does the original ACPI table contain?
- Can the change be verified?
- What happens if the SSDT is removed?
- Is this a generic laptop fix or specific to this machine?

If something is required because of the Acer firmware, that should be documented.

---

# Kexts

Kexts will also be documented individually.

Instead of only keeping a folder containing:

```text
Lilu.kext
WhateverGreen.kext
VirtualSMC.kext
...
```

I want notes explaining why each one exists.

Example:

```text
Kext:
WhateverGreen.kext

Purpose:
Graphics-related patches and support.

Why it is here:
Required for graphics configuration being tested.

Status:
To be verified.

Current notes:
Black screen problem still under investigation.
```

Version numbers should also be recorded.

That matters because OpenCore and kext behavior can change between versions.

A configuration that worked years ago may not behave exactly the same with newer components.

---

# OpenCore drivers

Drivers should receive similar documentation.

For each driver:

```text
Driver name
Version
Purpose
Why it is required
Whether it is currently necessary
Related UEFI setting
Test result
```

I don't want unused drivers sitting inside the EFI simply because they came with somebody else's configuration.

---

# Boot arguments

Boot arguments are especially easy to forget.

Every important boot argument should eventually have:

```text
Argument
Reason
Date
Test ID
Expected behavior
Actual behavior
Still required?
```

Temporary debug arguments should not quietly become permanent parts of the EFI.

If an argument was only used during troubleshooting, it should be removed once it is no longer needed.

---

# BIOS

BIOS settings will have their own section.

For every relevant setting I want to record:

```text
Setting name
Original value
Changed value
Reason
Boot result
Required?
```

I don't want to write generic instructions such as:

`Disable X`

unless I can confirm that the option actually exists on this Acer firmware or that another method is required.

If the Acer BIOS hides an important setting, that itself is useful information and should be documented.

---

# Screenshots

Screenshots will be used when they actually help explain something.

They will probably be organized like:

```text
screenshots/
├── BIOS/
├── Windows/
├── OpenCore/
├── Errors/
└── macOS/
```

I also want screenshots to have meaningful file names.

Instead of:

```text
IMG_3938.png
IMG_3939.png
Screenshot1.png
Screenshot2.png
```

I would rather use:

```text
opencore-builtingraphics-false.png
opencore-builtingraphics-true.png
windows-display-adapter.png
black-screen-OC-0002.png
bios-main-page.png
```

It is a small detail, but it makes the repository much easier to investigate later.

---

# SMBIOS privacy

Real SMBIOS information from this machine will **not** be published.

Before uploading any public EFI, private machine-specific values must be removed or replaced.

This includes:

- SystemSerialNumber
- MLB
- SystemUUID
- ROM

Possibly sensitive screenshots and logs will also be checked before publication.

Anyone using this repository should generate their own SMBIOS information.

Do not copy another person's serial information into your own installation.

This repository is not intended to distribute valid Apple identity information.

---

# Successful and failed configurations

Both successful and failed configurations will stay documented.

A failed configuration is not useless.

For example:

```text
BuiltinGraphics = True
Result = Black screen
```

already means I don't need to unknowingly repeat exactly the same experiment later.

Over time the repository should become a map of the laptop:

```text
this works
this doesn't
this changes something
this changes nothing
this breaks boot
this improves boot
this is still unknown
```

That is more useful than pretending every experiment went perfectly.

---

# Planned repository structure

The current planned structure is:

```text
Acer-E5-571-OpenCore-Journey/
│
├── README.md
│
├── CHANGELOG.md
│
├── Hardware/
│   ├── specifications.md
│   ├── CPU.md
│   ├── GPU.md
│   ├── Display.md
│   ├── Audio.md
│   ├── Ethernet.md
│   ├── WiFi-Bluetooth.md
│   ├── Storage.md
│   ├── USB.md
│   ├── PCI-devices.md
│   └── screenshots/
│
├── BIOS/
│   ├── settings.md
│   └── screenshots/
│
├── Windows/
│   ├── system-info.md
│   ├── hardware-ids.md
│   ├── commands-used.md
│   └── screenshots/
│
├── OpenCore/
│   ├── configuration-notes.md
│   ├── boot-args.md
│   ├── ACPI.md
│   ├── Kexts.md
│   ├── Drivers.md
│   ├── DeviceProperties.md
│   ├── UEFI.md
│   ├── EFI/
│   └── logs/
│
├── Troubleshooting/
│   ├── black-screen.md
│   ├── graphics.md
│   ├── boot-errors.md
│   └── test-history.md
│
└── macOS/
    ├── installation.md
    ├── working.md
    └── not-working.md
```

This structure is not final.

If the project grows and another layout makes more sense, I'll reorganize it.

The point is to make information easy to find, not to keep a folder structure just because I created it on the first day.

---

# Working / not working

Once macOS starts properly, I'll maintain a hardware status list.

For now most things remain unknown because I haven't reached the point where they can be properly tested.

| Feature | Status | Notes |
|---|---|---|
| OpenCore boot | Testing | Current display issue |
| OpenCore graphical output | Problem | Black screen investigation |
| Internal display | Not properly tested | Current black screen |
| Graphics acceleration | Unknown | |
| Keyboard | Unknown | |
| Touchpad | Unknown | |
| USB | Unknown | |
| Audio | Unknown | |
| Ethernet | Unknown | |
| Wi-Fi | Unknown | |
| Bluetooth | Unknown | |
| SSD | Unknown | |
| HDD | Unknown | |
| Battery status | Unknown | |
| Sleep | Unknown | |
| Wake | Unknown | |
| Brightness control | Unknown | |
| HDMI | Unknown | |

I'm deliberately leaving things as `Unknown` until I actually test them.

---

# Things I don't want to do

A few rules for myself during this project.

**Don't claim something works before testing it.**

**Don't copy an EFI and call the project finished.**

**Don't change many unrelated settings during one experiment.**

**Don't delete failed experiments.**

**Don't publish private SMBIOS information.**

**Don't assume another E5-571 has exactly the same hardware.**

**Don't blindly trust an old guide without checking what the setting actually does.**

**Don't fix something without trying to understand what fixed it.**

**Don't fill unknown hardware information with guesses just to make the documentation look complete.**

---

# What counts as progress

Progress doesn't only mean reaching the macOS desktop.

Finding out that a particular setting does nothing is progress.

Finding the exact PCI ID is progress.

Finding where the screen goes black is progress.

Finding a configuration that makes things worse is also useful because it removes one possibility.

The project is basically narrowing down the problem one test at a time.

---

# End goal

Best case:

I end up with a stable macOS installation on the Acer Aspire E5-571 and a clean OpenCore EFI that I actually understand.

The final configuration should not contain random patches, drivers, SSDTs or boot arguments that I can't explain.

But even if some hardware never works perfectly, I still want this repository to contain enough information that another E5-571 owner can follow the investigation from the beginning instead of starting from zero.

The final result isn't supposed to be only:

`Here is my EFI.`

I want this repository to answer questions like:

- What hardware is actually inside this laptop?
- What version of the E5-571 am I working with?
- What did I try?
- Why did I try it?
- What happened after the change?
- What failed?
- What improved something?
- What fixed something?
- What is still broken?
- What is still unknown?
- Which changes were reverted?
- Which changes are specific to this laptop?
- Which parts might apply to other E5-571 machines?
- What should another owner check before copying anything?
- Which OpenCore configuration produced a certain boot log?

If I can answer those questions by looking through this repository, then the documentation has done its job.

---

# Current investigation summary

```text
Machine:
Acer Aspire E5-571

CPU:
Intel Core i5-4210U

Architecture:
Intel Haswell

Cores / Threads:
2 / 4

RAM:
8 GB

Graphics:
Intel integrated graphics

Exact graphics PCI ID:
Pending verification

Storage:
SSD + HDD

Windows:
Windows 10 Home Single Language

Bootloader:
OpenCore

Current main issue:
Black screen during OpenCore startup/display process

Known configuration:
Resolution = Max

Recorded test:
BuiltinGraphics False -> True

Result:
Black screen remained

Current conclusion:
BuiltinGraphics = True does not solve the problem by itself.

Next direction:
Identify the graphics and display path properly and continue changing one relevant variable at a time.
```

---

# Project status

**Work in progress.**

There are still a lot of unknowns.

That's fine.

I'd rather slowly turn the unknowns into verified information than fill this repository with assumptions.

The machine decides what is true.

The logs, hardware IDs and test results decide what stays in the documentation.

Work continues.
