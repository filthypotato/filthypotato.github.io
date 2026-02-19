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
By us connecting a Windows 11 client machine, and then using a PowerShell script to generate over 1000 user account, allows us for
practical simulation and hand on learning with Active Directory, Group Policies,

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
  - Memory: 4096MiB
  - CPUs: 2
- Virtual Hard Disk:
  - 60.00 GiB
    - My boot drive did not have enough space, so `Select or create custom storage`

- Network selection
  - Virtual Network 'default': NAT
    - This prevents exposing your IP and Host system to the VM.

- Now `Finish!`

**Great! You now have the Virtual Machine setup, the next steps will involve the Windows Setup Wizard inside of the VM**

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

Check Default NAT Network
  - In Arch termnial:

  ```bash
  virsh net-list --all
  ```
You should see:

```cpp
  default active
```
If this unfortunately does not work, try these commands:

```bash
  sudo virsh net-start default
  sudo virsh net-autostart default
```
***This is the INTERNET network!***


# Create Internal Network (INTNET)

Open virt-manager:

  - Edit -> Connection Details

  - Virtual Networks

  - Click + (Add Network)

Configure:

  - Name: `intnet`

  - Mode: `Isolated`

  - IPv4:

    - Address: 172.16.0.1

    - Netmask: 255.255.255.0

  - Disable DHCP

*Finish*

**Now you should have:**

  - `default` -> internet

  - `intnet` -> internal lab


# Inside Your Windows Server VM

Now we will rename the Network Adapters

1: Open:

    - Control Panel

    - Network and Sharing Center

    - Change Adapter Settings

You should see and rename:

  - Ethernet -> _INTERNET_

  - Ethernet 2 -> _INTNET_

Right click the `_INTNET_` -> Properties
Internet Protocol Version 4 -> Properties

Set:

  ```
  IP: 172.16.0.1
  Mask: 255.255.255.0
  Gateway: (blank)
  DNS: 127.0.0.1
  ```
Now we can rename the Server to DC

- Start

  - System

  - Rename this PC

Rename to:

  `DC`

***Now Restart***

Now lets attach the Networks to the Windows Server VM

Go inside `virt-manager`

  - Right click Windows Server VM

  - Open

  - Show Hardware Details

Add:

  NIC 1:

    - Source: `default`

    - Device mode: `e1000` (you can use virtio if you have the drivers installed, but for this I do not)

  NIC 2:

    - Source: `intnet`

    - Device model: `e1000`

  **Apply**

  *Now run VM!*

Open Command Prompt inside of your Windows VM

```ngnix
ipconfig
```

You will see:

```nginx
_INTERNET_ -> DHCP address 192.168.x.x (or close too)
_INTNET_ -> 172.16.0.1
```
This means it is working!







More screenshots and attack scenarios coming soon.

