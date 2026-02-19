---
title: "Building an Active Directory Homelab"
date: 2026-02-18 14:00:00 -0800

categories: [Homelab, Active Directory]
tags: [Windows Server, AD, Cybersecurity]
image:
  path: "/assets/img/ad-banner.png"
  alt: Active Directory Setup
author: Tylor Romine
---

## Introduction

In this lab, I set up an Active Directory Domain Controller using Windows Server 2025 and connected a Windows 10 client to simulate a real-world enterprise environment.

This homelab allows me to practice:

- Domain configuration
- Group Policy
- User management
- Attack simulations
- Blue team logging & monitoring

## Network Overview

- Windows Server (Domain Controller)
- Windows 10 Client
- Internal NAT network
- Static IP for DC
- DNS pointed to DC

## Prerequisites

Files needed to follow along:

  > **Note:** This lab was built using virt-manager on a Linux host (KVM/QEMU)
  > If you are using VMWare or VirtualBox, steps may differ.

### Virtualization Platform
  - [virt-manager (KVM/QEMU)](https://virt-manager.org/)
  - Virtualization must be enabled. (Intel VT-x or AMD-V)

Terminal command to install rerequisites.

```bash
sudo pacman -S qemu virt-manager virt-viewer dnsmasq vde2 bridge-utils openbsd-netcat qemu-full virt-manager libvirt edk2-ovmf dnsmasq bridge-utils
```
Enables and start libvirtd

```bash
sudo systemctl enable libvirtd
sudo systemctl start libvirtd
```
Checks if running

```bash
systemctl status libvirtd
```
Adds yourself to the libvirt group

```bash
sudo usermod -aG libvirt $USER
```
Then **log out** and log back
 (or reboot)


### Installations Files

 - [Windows Server 2025 ISO](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2025)
 - [Windows 11 ISO](https://www.microsoft.com/en-us/software-download/windows11)

### Verify Virtualization Support

Run:

```bash
lscpu | grep Virtualization
```

You should see **`VT-x`** or **`AMD-V`**


# Procedures

## Setting up Windows Server 2025

Download Virt Manager by using the steps above if you have not already.

Open Virt Manager in Terminal:
```bash
virt-manager
```

Click `Create a new virtual machine`

- `Local install media (ISO Image, or CDROM)` > Forward

- `Browse` for ISO Image

- Select ISO Image for Windows Server 2025

- Make sure on Chose the operating system you are installing -> `Microsoft Windows Server 2025` > Foward

- Click `yes` on the next window

These are the setup options that I used:
- Hardware
  - Base Memory: 4096MiB
  - Processors: 2
- Virtual Hard Disk:
  - 60.00 GiB

- CPUs: 1 or 2 is fine, depending on performance > Forward

- Adjust storage size to what you want. 60G is ideal.

- My boot drive did not have enough space, so `Select or create custom storage`

- `Check` Customize configuration before install

- On Network selection > Leave as Virtual Network 'default': NAT

  - This prevents exposing your IP and Host system to the VM.

- Make sure you have enough memory allocated, I used `4096`

- Double check NIC tab is still set to `Virtual network 'default': NAT`

Then top left of window pane > `Begin Installation`

# DC: Windows Server Setup Wizard

Select Options as they appear:
  - Select the operating system you would like installed

    - Windows Server 2025 Standard Evaluation (Desktop Experience)

  - Which type of installation do you want?

    - Custom: Install Windows only (advanced)

    - Click next to the following screem to select your default partition.

  - Customize Settings:

    - Password: Password1

        - (Easy password makes it easy for lab setup)

# Configuring NIC's (Internal/External Networks)



More screenshots and attack scenarios coming soon.

