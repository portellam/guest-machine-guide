# Guest Machine Guide
### In Development
Guide for setup of a guest Libvirt/QEMU Virtual Machine (VM). Includes general
overview and references, and optimizations for Windows guests and
hardware-passthrough (VFIO).

#### View this repository on [Codeberg][01] or [GitHub][02].
[01]: https://codeberg.org/portellam/guest-machine-guide
[02]: https://github.com/portellam/guest-machine-guide
##

## Table of Contents
- [1. Why?](#1-why)
- [2. Related Projects](#2-related-projects)
- [3. Documentation](#3-documentation)
- [4. Host Optimizations](#4-host-optimizations)
- [5. Guest Optimizations](#5-guest-optimizations)
- [6. Guest XML Layout](#6-guest-xml-layout)
    - [6.1. Syntax](#61-syntax)
    - [6.2. First Lines in XML](#62-first-lines-in-xml)
    - [6.3. Memory](#63-memory)
    - [6.4. CPU Topology (1 / 2)](#64-cpu-topology-1--2)
    - [6.5. System Information Spoofing](#65-system-information-spoofing)
    - [6.6. Features](#66-features)
    - [6.7. CPU Topology (2 / 2)](#67-cpu-topology-2--2)
    - [6.8. Power Management](#68-power-management)
    - [6.9. Devices](#69-devices)
    - [6.10. QEMU Command Line](#610-qemu-command-line)
    - [6.11. QEMU Overrides](#611-qemu-overrides)
- [7. Benchmarking Guest Performance](#7-benchmarking-guest-performance)
- [8. Contact](#8-contact)
- [9. References](#9-references)

## Contents
### 1. Why?
The purpose of this document is to inform a new or returning user how to
optimize a Guest machine, without demanding greater research and time.

This document does not serve to replace existing knowledge-bases. If you have
any unexpected questions, wish to fact-check, or want to expand your knowledge,
then please visit these places!

Copy and paste what you need from here and/or any example XML files, to your
Guest XML file.

### 2. Related Projects
To view other relevant projects, visit [Codeberg][21]
or [GitHub][22].

[21]: https://codeberg.org/portellam/guest-machine-guide
[22]: https://github.com/portellam/guest-machine-guide

### 3. Documentation
- What is VFIO?[<sup>[12]</sup>](#12)
- VFIO Discussion and Support[<sup>[13]</sup>](#13)
- Hardware-Passthrough Guide[<sup>[9]</sup>](#9)
- Virtual Machine XML Format Guide[<sup>[16]</sup>](#16)

### 4. Host Optimizations
TODO: add here.

### 5. Guest Optimizations
TODO: add here.

### 6. Guest XML Layout
Below is an *incomplete* layout for building a Guest machine. The lines include
additional features, of which are absent when creating a Guest XML (with the
`virsh` CLI command or `virt-manager` GUI application).

#### 6.1. Syntax
```xml
<parent_tag_name attribute_name="attribute_value">
  <child_tag_name>child_tag_value</child_tag_name>
</parent_tag_name>
```

#### 6.2. First Lines in XML

| `<domain>` Tag     | Attribute    | Value                                          | Description                                        |
| ------------------ | ------------ | ---------------------------------------------- | -------------------------------------------------- |
|                    | `xmlns:qemu` | `"http://libvirt.org/schemas/domain/qemu/1.0"` | Enable QEMU [command lines](#qemu-command-line) and [overrides](#qemu-overrides).           |
|                    | `type`       | `"kvm"`                                        | Enable QEMU [command lines](#qemu-command-line) and [overrides](#qemu-overrides).           |
| `<name/>`[<sup>1</sup>](#21a-name-best-practice)          | none         | text                                           | Name of the Guest.                                 |

##### 2.a. `<name/>` Best practice:
**Note:** The following formatting examples are a personal preference of the
Author.

**Format:** `purpose`**_**`vendor`\***_**`operating system`**_**`architecture`**_**`chipset`**_**`firmware`**_**`topology`

**\*** Optional, if Host machine contains two (2) or more video devices
(GPU/VGA).
  - Example systems and names:
    - Modern gaming machine:&ensp;&ensp;&ensp;&ensp;&ensp;&nbsp;`game_nvidia_win10_x64_q35_uefi_6c12t`
    - Older 2000s gaming machine:&ensp;`retro_amd_winxp_x86_i440fx_bios_2c4t`
    - Retro 1990s gaming machine:&ensp;`retro_3dfx_win98_x86_i440fx_bios_1c1t`
    - Intel MacOS workstation:&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;`work_macos_amd_x64_q35_uefi_6c12t`

  - Purpose of the Guest (and suggested names):
    - Gaming PC:&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;`game`
    - Legacy/Retro PC:&ensp;&nbsp;`retro`
    - Server:&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;`server`
    - Workstation PC:&ensp;&ensp;`work`
    - etc.

  - Vendor name of the Video device:
    - AMD:&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;`amd`
    - Intel:&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;`intel`
    - NVIDIA:&ensp;&ensp;&ensp;&nbsp;`nvidia`
    - emulated:&ensp;`virtgpu`

    - non-mainstream or legacy:
      - 3DFX:&ensp;`3dfx`.

  - Short name of the Operating System (OS):
    - Apple Macintosh:&ensp;&ensp;&ensp;`macos`
    - Linux:&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;`arch`, `debian`, `redhat`, `ubuntu`
    - Microsoft Windows:&ensp;`win98`, `winxp`, `win10`
    - etc.

  - Short name of the CPU architecture:
    - AMD/Intel 32-bit:&ensp;[`x86`](#15)
    - AMD/Intel 64-bit:&ensp;[`x64`](#14)
    - ARM 32-bit:&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&nbsp;[`aarch32`](#2)
    - ARM 64-bit:&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&nbsp;[`aarch64`](#3)
    - etc.

  - Virtualized chipset:
    - I440FX:&ensp;[`i440fx`](#6)
      - Emulated, older chipset by Intel.
      - Does support legacy Guests (example: Windows NT 5 and before, XP/2000, 9x).
      - PCI bus only; expansion devices will exist on a PCI topology.
      - Will accept PCIe devices.

    - Q35:&ensp;&ensp;&ensp;&nbsp;[`q35`](#10)
      - Virtual, newer platform.
      - Native PCIe bus; expansion devices will exist on a PCIe topology.
      - Does not support legacy Guests.

  - Firmware:
    - BIOS:&ensp;[`bios`](#4)
    - UEFI:&ensp;&nbsp;[`uefi`](#11)

  - Short-hand of Core topology:&ensp;`4c8t`
    - Given the amount of physical cores (example: 4).
    - Given the amount of logical threads per core (2 * 4 = 8).

#### 6.3. Memory
To gather information of system memory, execute: `free --kibi --total --wide`
For more information on this topic, click [here](#1).

| `<memory>` Tag | Attribute | Value    | Description                                                       |
| -------------- | --------- | -------- | ----------------------------------------------------------------- |
|                | parent    | a number | The total amount of memory.                                       |
|                | `unit`    | `"KiB"`  | Amount of memory in Kibibytes (1024 bytes, Kilo is 1000 bytes).   |

| `<currentMemory>` Tag | Attribute | Value    | Description                                                       |
| --------------------- | --------- | -------- | ----------------------------------------------------------------- |
|                       | parent    | a number | The current amount of memory.                                     |
|                       | `unit`    | `"KiB"`  | Amount of memory in Kibibytes (1024 bytes, Kilo is 1000 bytes).   |

| `<memoryBacking>` Tag | Attribute | Value       | Description                                                       |
| --------------------- | --------- | ----------- | ----------------------------------------------------------------- |
| `<allocation/>`       | `mode`    | `immediate` | Specifies how memory allocation is performed.                     |
| `<discard/>`          | none      | none        | TODO: define, what is this?                                                   |
| [`<hugepages/>`](#5)  | none      | none        | Enable Huge memory pages.                                         |
| `<nosharepages/>`     | none      | none        | Prevents the Host from merging the same memory used among Guests. |

##### 3.a. `<hugepages/>`
- Static allocation of *Host* memory pages into *Guest* memory pages.
- **Huge:** Memory page size greater than 4K bytes (2M or 1G bytes). The greater
the size, the lower the Host overhead.
- Dynamic *Host* memory page allocation is more flexible, but will require
defragmentation before use as *Guest* memory pages (before a Guest machine may start).
- **Warning:** If the specified *Guest* memory pages exceeds the allocated
*Host* memory pages, then the Guest machine will fail to start.

For more information on this topic, click [here](#5).

#### 6.4. CPU Topology (1 / 2)
To gather information about your CPU, execute: `lscpu | grep --extended-regexp --ignore-case "per core|per socket|socket"`

| `<vcpu>` Tag | Attribute   | Value      | Description                                                     |
| ------------ | ----------- | ---------- | --------------------------------------------------------------- |
|              | parent      | a number   | Specify number of Host threads for Guest (same as `<cpu>` tag). |
|              | `placement` | `"static"` | Static allocation of Guest CPU threads.                         |

| `<iothreads>` Tag | Attribute | Value       | Description                                                                                  |
| ----------------- | --------- | ----------- | -------------------------------------------------------------------------------------------- |
|                   | none      | a number    | Specify number of Host threads to manage storage block devices (see as `<iothreadpin>` tag). |

| `<cputune>` Tag | Attribute  | Value       | Description                                  |
| --------------- | ---------- | ----------- | -------------------------------------------- |
| `<vcpupin>`     | `vcpu`     | a number    | Guest CPU: the Guest thread ID number.[<sup>1</sup>](#1-vcpupin-vcpu)      |
| `<vcpupin>`     | `cpuset`   | a number    | Guest CPU: the Host thread ID number.[<sup>2</sup>](#2-vcpupin-cpuset)      |
| `<emulatorpin>` | `cpuset`   | a number    | Guest IRQ: the Guest IRQ thread ID number.[<sup>3</sup>](#3-emulatorpin-cpuset) |
| `<iothreadpin>` | `iothread` | a number    | Guest IO: the Guest IO thread ID number.    |
| `<iothreadpin>` | `cpuset`   | a number    | Guest IO: the Host thread ID numbers.[<sup>4</sup>](#4-iothreadpin-cpuset)      |

##### 4.a. `<vcpupin vcpu>`
- Count does not exceed value as defined in `<vcpu placement>`.

##### 4.b. `<vcpupin cpuset>`
- Threads should not overlap Host process threads.

##### 4.c. `<emulatorpin cpuset>`
- Emulator threads handle Interrupt Requests for Guest hardware emulation.
- Threads should not overlap Guest CPU threads as defined in `vcpupin cpuset`.

##### 4.d. `<iothreadpin cpuset>`
- IO threads handle IO processes for Guest virtual drives/disks.
- Threads should not overlap Guest CPU threads as defined in `vcpupin cpuset`.

##### 4.e. Example XML:
```xml
  <!-- Given a 4-core, 8-thread CPU... -->
  <vcpu placement="static">4</vcpu>           <!-- Statically allocate four (4) cores to Guest. -->
  <iothreads>1</iothreads>                    <!-- Define one (1) thread to IO. -->
  <cputune>
    <vcpupin vcpu="0" cpuset="2"/>            <!-- Guest CPU: use the third core, first thread. -->
    <vcpupin vcpu="1" cpuset="6"/>            <!-- Guest CPU: use the third core, second thread. -->
    <vcpupin vcpu="2" cpuset="3"/>            <!-- Guest CPU: use the fourth core, first thread. -->
    <vcpupin vcpu="3" cpuset="7"/>            <!-- Guest CPU: use the fourth core, second thread. -->
    <emulatorpin cpuset="1,4"/>               <!-- Guest IRQ: use the second core, two threads. -->
    <iothreadpin iothread="1" cpuset="1,4"/>  <!-- Guest IO: use the second core, two threads. -->
  </cputune>
```

#### 6.5. System Information Spoofing
To gather information about your BIOS, execute:

&ensp;`sudo dmidecode --type bios | grep --extended-regexp --ignore-case "vendor|version|release date"`

To gather information about your system, execute:

&ensp;`sudo dmidecode --type system | grep --extended-regexp --ignore-case "manufacturer|product name|version|serial number|sku number|family"`

```xml
  <!-- BIOS and System spoofing (you may copy your actual info). -->
  <sysinfo type="smbios">                                       <!-- This line is necessary! -->
    <bios>
      <entry name="vendor">American Megatrends Inc.</entry>     <!-- AMI is the industry standard BIOS vendor. -->
      <entry name="version">version_of_bios_firmware</entry>
      <entry name="date">MM/DD/YYYY</entry>
    </bios>
    <system>
      <entry name="manufacturer">vendor_of_motherboard</entry>
      <entry name="product">product_name</entry>
      <entry name="version">Default string</entry>
      <entry name="serial">Default string</entry>
      <entry name="sku">Default string</entry>
      <entry name="family">Default string</entry>
    </system>
  </sysinfo>
```

#### 6.6. Features

TODO: make the following inline XML into chart, describe each feature.

```xml
    <hyperv mode="custom">
      <relaxed state="on"/>
      <vapic state="on"/>
      <spinlocks state="on" retries="8191"/>
      <vpindex state="on"/>
      <runtime state="on"/>
      <synic state="on"/>

      <stimer state="on">
        <direct state="on"/>
      </stimer>

      <reset state="on"/>
      <vendor_id state="on" value="1234567890ab"/>
      <frequencies state="on"/>
      <reenlightenment state="on"/>
      <tlbflush state="on"/>
      <ipi state="on"/>
      <evmcs state="on"/>
    </hyperv>
    <kvm>
      <hidden state="on"/>
    </kvm>
    <vmport state="off"/>
    <ioapic driver="kvm"/>
  </features>
```

#### 6.7. CPU Topology (2 / 2)
To gather information about your CPU, execute: `lscpu | grep --extended-regexp --ignore-case "per core|per socket|socket"`

##### 7.a. Example output:
```
Thread(s) per core:                 2
Core(s) per socket:                 8
Socket(s):                          1
```

TODO: make the following inline XML into chart, describe each feature.

```xml
  <cpu mode="host-passthrough" check="none" migratable="on">  <!-- Spoof the CPU info, with the actual CPU info. -->
    <topology sockets="1" dies="1" cores="6" threads="2"/>
    <cache mode="passthrough"/>
    <feature policy="disable" name="hypervisor"/>
    <feature policy="disable" present="yes"/>
    <timer name="tsc" present="yes" mode="native"/>
  </clock>
```

| `<cpu>` Tag  | Attribute    | Value                | Description                                                     |
| ------------ | ------------ | -------------------- | --------------------------------------------------------------  |
| parent       | `mode`       | `"host-passthrough"` | Spoof the Guest CPU info, with the actual Host CPU info.        |
|              | `check`      | `"none"`             | TODO: add here.                                                 |
|              | `migratable` | `"on"`               | TODO: add here.                                                 |
| `<topology>` | `sockets`    | a number             | The number of CPU sockets, or maxmium number of dies.           |
|              | `dies`       | a number             | The number of CPU dies (typically one).                         |
|              | `cores`      | a number             | The number of CPU cores.                                        |
|              | `threads`    | a number             | The number of CPU threads per core (typically a factor of two). |
| `<cache>`    | `mode`       | `"passthrough"`      | Passthrough the Host CPU cache.                                 |
| `<feature>`  | `policy`     | `"disable"`          | Disables policy.                                                |
|              | `name`       | `"hypervisor"`       | TODO: add here.                                                 |
| `<feature>`  | `policy`     | `"disable"`          | Disables policy.                                                |
|              | `present`    | `"yes"`              | TODO: add here.                                                 |
| `<timer>`    | `name`       | `"tsc"`              | TODO: add here.                                                 |
|              | `present`    | `"yes"`              | TODO: add here.                                                 |
|              | `mode`       | `"native"`           | TODO: add here.                                                 |

#### 6.8. Power Management
```xml
  <!-- Power Management -->
  <pm>
    <suspend-to-mem enabled="yes"/>   <!-- Enable S3 Suspend (Sleep) -->
    <suspend-to-disk enabled="yes"/>  <!-- Enable S4 Suspend (Hibernation) -->
  </pm>
```

#### 6.9. Devices
For more information on this topic, click [here](#8).

TODO: make the following inline XML into chart, describe each feature???
```xml
  <!-- Emulated, Paravirtual, Passed-through Real PCI/e, and Shared Memory devices -->
  <devices>
  ...
  </devices>
```

##### 6.9.a. Emulated Devices

TODO: make this inline XML?
- QEMU
- RedHat
- Other legacy devices
  - SoftGPU https://github.com/JHRobotics/softgpu

lorem ipsum

##### 6.9.b. Real/Passthrough Hardware Devices
TODO: make this inline XML?

  - AMD
    - Integrated
    - Dedicated
    - Reset bug
  - Intel
    - Integrated
    - Dedicated
      - **Note:** Not much research done regarding this topic. Please refer to any mentioned guides, the Reddit forum, or use an Internet search engine with the keywords `intel gpu vfio`.
  - NVIDIA
    - Problems:
      - Boot bug (Solution: Use known-good copy of given GPU's video BIOS or VBIOS).
      - TODO: add references, programs used, instructions, and XML here.

##### 6.9.c. Memory Devices

TODO: make this inline XML?
  - Looking Glass
  - Scream
  - ???

#### 6.10. QEMU Command Line

TODO: make the following inline XML into chart, describe each feature.
  - Evdev

```xml
  <qemu:commandline>...</qemu:commandline>  <!-- Add Evdev here -->
```

#### 6.11. QEMU Overrides

TODO: make the following inline XML into chart, describe each feature.

```xml
  <qemu:override>...</qemu:override>
```

### 7. Benchmarking Guest Performance
TODO: add here.

### 8. Contact
Do you need help? Please visit the [Issues][81] page.

[81]: https://github.com/portellam/guest-machine-guide/issues

### 9. References
#### 1.
&nbsp;&nbsp;**8.2. Memory Tuning on Virtual Machines Red Hat Enterprise Linux 7**.
Red Hat Customer Portal. Accessed June 15, 2024.

&nbsp;&nbsp;&nbsp;&nbsp;<sup>https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/virtualization_tuning_and_optimization_guide/sect-virtualization_tuning_optimization_guide-memory-tuning.</sup>

#### 2.
&nbsp;&nbsp;**AARCH64**. Wikipedia, May 30, 2024.

&nbsp;&nbsp;&nbsp;&nbsp;<sup>https://en.wikipedia.org/wiki/AArch64.</sup>

#### 3.
&nbsp;&nbsp;**ARM Architecture Family**. Wikipedia, May 31, 2024.

&nbsp;&nbsp;&nbsp;&nbsp;<sup>https://en.wikipedia.org/wiki/ARM_architecture_family#AArch32.</sup>

#### 4.
&nbsp;&nbsp;**BIOS**. Wikipedia, June 7, 2024.

&nbsp;&nbsp;&nbsp;&nbsp;<sup>https://en.wikipedia.org/wiki/BIOS.</sup>

#### 5.
&nbsp;&nbsp;**Hugepages**. Hugepages - Debian Wiki. Accessed June 15, 2024.

&nbsp;&nbsp;&nbsp;&nbsp;<sup>https://wiki.debian.org/Hugepages.</sup>

#### 6.
&nbsp;&nbsp;**I440fx PC (PC-I440fx, PC)**. i440fx PC (pc-i440fx, pc) - QEMU documentation.
Accessed June 15, 2024.

&nbsp;&nbsp;&nbsp;&nbsp;<sup>https://www.qemu.org/docs/master/system/i386/pc.html.</sup>

#### 7.
&nbsp;&nbsp;**Intel 440FX**. Wikipedia, August 20, 2023.

&nbsp;&nbsp;&nbsp;&nbsp;<sup>https://en.wikipedia.org/wiki/Intel_440FX.</sup>

#### 8.
&nbsp;&nbsp;**PCI Overview**. QEMU Wiki. Accessed June 15, 2024.

&nbsp;&nbsp;&nbsp;&nbsp;<sup>https://wiki.qemu.org/images/f/f6/PCIvsPCIe.pdf.</sup>

#### 9.
&nbsp;&nbsp;**PCI passthrough via OVMF**. ArchWiki. Accessed June 14, 2024.

&nbsp;&nbsp;&nbsp;&nbsp;<sup>https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF.</sup>

#### 10.
&nbsp;&nbsp;**Q35**. QEMU Wiki. Accessed June 15, 2024.

&nbsp;&nbsp;&nbsp;&nbsp;<sup>https://wiki.qemu.org/images/4/4e/Q35.pdf.</sup>

#### 11.
&nbsp;&nbsp;**UEFI**. Wikipedia, June 7, 2024.

&nbsp;&nbsp;&nbsp;&nbsp;<sup>https://en.wikipedia.org/wiki/UEFI.</sup>

#### 12.
&nbsp;&nbsp;**VFIO - ‘Virtual Function I/O’ - The Linux Kernel Documentation**.
The linux kernel. Accessed June 14, 2024.

&nbsp;&nbsp;&nbsp;&nbsp;<sup>https://www.kernel.org/doc/html/latest/driver-api/vfio.html.</sup>

#### 13.
&nbsp;&nbsp;**VFIO Discussion and Support**. Reddit. Accessed June 14, 2024.

&nbsp;&nbsp;&nbsp;&nbsp;<sup>https://www.reddit.com/r/VFIO/.</sup>

#### 14.
&nbsp;&nbsp;**X64**. Wikipedia, July 27, 2023.

&nbsp;&nbsp;&nbsp;&nbsp;<sup>https://en.wikipedia.org/wiki/X64.</sup>

#### 15.
&nbsp;&nbsp;**X86**. Wikipedia, May 18, 2024.

&nbsp;&nbsp;&nbsp;&nbsp;<sup>https://en.wikipedia.org/wiki/X86.</sup>

#### 16.
&nbsp;&nbsp;**XML Design Format**. GitHub - libvirt/libvirt.
Accessed June 18, 2024.

&nbsp;&nbsp;&nbsp;&nbsp;<sup>https://github.com/libvirt/libvirt/blob/master/docs/formatdomain.rst.</sup>

## TODO:
- [x] add About.
- [ ] add Documentation for:
  - [x] guest XML layout
    - [x] name syntax
    - [x] introduction
    - [ ] memory
    - [ ] cpu topology (1/2)
    - [ ] system information spoofing
    - [ ] features
    - [ ] cpu topology (2/2)
    - [ ] power management
    - [ ] qemu command line
      - [ ] evdev macro
      - [ ] gpu stuff?
    - [ ] qemu overrides
  - [ ] host optimizations
  - [ ] guest optimizations
  - [ ] benchmarking performance
- [x] add References.
- [ ] use collapsable dropdowns (https://gist.github.com/pierrejoubert73/902cc94d79424356a8d20be2b382e1ab).
- [ ] add Guest XML file with all lines referenced in "Guest XML Layout".
- [ ] add pictures of virt-manager.
- [ ] flesh out "Guest XML Layout".
- [ ] how to setup virtio disks (virtio, virtio-scsi).