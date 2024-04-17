# Guest Machine Guide
Guide for setup of a guest Libvirt/QEMU Virtual Machine (VM). Includes general overview and references, and optimizations for Windows guests and hardware-passthrough (VFIO).

**[View develop branch...](https://github.com/portellam/guest-machine-guide/tree/develop)**

#### Related Projects:
**[Auto Xorg](https://github.com/portellam/auto-Xorg) | [Deploy VFIO](https://github.com/portellam/deploy-vfio) | [Generate Evdev](https://github.com/portellam/generate-evdev)  | [Libvirt Hooks](https://github.com/portellam/deploy-vfio) | [Power State Virtual Machine Manager](https://github.com/portellam/deploy-vfio)**

## Table of Contents
- [1. About](#1-about)

## 1. About
The purpose of this document is to inform a new or returning user how to optimize a Guest machine right n
this moment, without demanding greater research and time.

This document does not serve to replace existing knowledge-bases such as the [ArchLinux Wiki](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF) the [VFIO forum (Reddit)](https://old.reddit.com/r/vfio). If you have any unexpected questions, wish to fact-check, or want to expand your knowledge, then please visit these places!

Copy and paste what you need from here and/or any example XML files, to your Guest XML file.

## TODO:
- [x] add About.
- [ ] add Documentation for
  - [ ] guest XML layout
    - [ ] name syntax
    - [ ] introduction
    - [ ] memory
    - [ ] cpu topology (1/2)
    - [ ] system information spoofing
    - [ ] features
    - [ ] cpu topology (2/2)
    - [ ] power management
    - [ ] qemu command line
    - [ ] qemu overrides
  - [ ] host optimizations
  - [ ] guest optimizations
  - [ ] benchmarking performance
- [ ] add References
