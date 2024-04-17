# Guest Machine Guide
Guide for setup of a Libvirt/QEMU Virtual Machine (VM). Includes general overview and references, and optimizations for Windows VMs and hardware-passthrough (VFIO).

[deploy-VFIO](https://github.com/portellam/deploy-vfio) **|** [auto-Xorg](https://github.com/portellam/auto-Xorg) **|**[generate-evdev](https://github.com/portellam/generate-evdev) **|** [libvirt-hooks](https://github.com/portellam/deploy-vfio) **|** [pwrstat-virtman](https://github.com/portellam/deploy-vfio)

**[View develop branch...](https://github.com/portellam/guest-machine-guide/tree/develop)**

## Table of Contents
- [1. About](#1-about)
- [2. Related Projects](#2-related-projects)

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
