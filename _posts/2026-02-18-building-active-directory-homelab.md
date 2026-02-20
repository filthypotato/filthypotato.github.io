---
title: "Building an Active Directory Homelab"
date: 2026-02-18 14:00:00 -0800

categories: [Homelab, Active Directory]
tags: [Windows Server, AD, Cybersecurity]
image:
  path: "/assets/img/ad-banner.webp"
  alt: Active Directory Setup
---

## Introduction

In this lab, I set up an Active Directory Domain Controller using Windows Server 2025 and connected a Windows 10 client to simulate a real-world enterprise environment.
By us connecting a Windows 10 client machine, and then using a PowerShell script to generate over 1000 user account, allows us for
practical simulation and hand on learning with Active Directory, Group Policies, user management, and domain based configuration.

## Lab Topology

  > **Lab-only note:** I’m using simple passwords like `Password1` to keep the lab fast to rebuild.
  > Do **not** do this in production.

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


## Virtualization Platform
  - [virt-manager (KVM/QEMU)](https://virt-manager.org/)
  - Virtualization must be enabled. (Intel VT-x or AMD-V)

Terminal command to install rerequisites.

```bash
sudo pacman -S --needed qemu-full virt-manager virt-viewer dnsmasq vde2 bridge-utils libvirt edk2-ovmf
```
Enables and start libvirtd

```bash
sudo systemctl enable --now libvirtd
sudo usermod -aG libvirt $USER
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


## Installation Files

 - [Windows Server 2025 ISO](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2025)
 - [Windows 10 ISO](https://www.microsoft.com/en-us/software-download/windows10ISO)
 - [PowerShell Create Account Script w/ Names (GitHub)](https://github.com/filthypotato/ActiveDirectoryLab)

## Verify Virtualization Support

Run:

```bash
lscpu | grep Virtualization
```

You should see **`VT-x`** or **`AMD-V`**


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

### DC: Windows Server Setup Wizard

Select Options as they appear:
  - Select the operating system you would like installed

    - Windows Server 2025 Standard Evaluation (Desktop Experience)

  - Which type of installation do you want?

    - Custom: Install Windows only (advanced)

    - Click next to the following screem to select your default partition.

  - Customize Settings:

    - Password: Password1

        - (Easy password makes it easy for lab setup)

### Configuring NIC's (Internal/External Networks)

Check Default NAT Network
  - In Arch termnial:

  ```bash
  virsh net-list --all
  ```
You should see:

  - default active

If this unfortunately does not work, try these commands:

```bash
  sudo virsh net-start default
  sudo virsh net-autostart default
```
***This is the INTERNET network!***


### Create Internal Network (INTNET)

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


### Inside Your Windows Server VM

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

### Configuring Active Directory Domain Services

Now that we have the configured NIC's, we can setup our Domain (Active Directory Domain Server).

Open `Server Manager` -> select `Add role and features`

Click next until you get to `Select destination server/Server Selection` tab.

Wait until `Server Roles` window pops up.

  - Select Active Directory Domain Services, click Add Features, click Next.

  - Click Next through the rest of the setup, click Install.

  - Click Close after install completes.

Now on Server Manager, there will be a Notification icon in top right (will be a yellow caution symbol).

Select it and select "Promote this server to a domain controller".

This will open the Active Directory Services Configuration Window, you can choose these options as follows.

  - Deployment Configuration:

    - Add new forest.

    - Root domain name: mydomain.com

  - Domain Controller Options:

    - Wait for window to load..

      - Password & Confirm Password: Password1

  - Click Next through the rest of the settings, click Install

### Creating dedicated domain admin user account

Open Active Directory Users and Computers.

Right-click on `mydomain.com` > Select New > Organizational Unit

  - Name: "_ADMINS"

    - Right-click "_ADMINS" group > New > User

  - For this part setup the Admin account with your information using this format:

    - First name: Peter

    - Last name: Smith

    - User logon name: p-smith@mydomain.com

  - Next

    - Password & Confirm Password: Password1

    - Uncheck "User must change password at next logon"

    - Check "Password never expires"

  - Next > Finish

In order to set your New Account as a Domain Admin, select the user account, right-click > Properties.

  - Member Of > Add

  - In "Enter the object names to select (examples):"

    - Type "domain admins".

    - Check Names

    - Click OK, Apply, then OK.

Now we are able to sign out and select the new user account we created.

Go ahead and sign out, then on the Sign-In screen, select Other Users, and enter in the credentials for your user you created.

  - Example:

    - Username: p-smith

    - Password: Password1

### Network Configuration

### Configure Remote Access Server (RAS)/NAT

Now it is time to setup a RAS/NAT to allow our client virtual machines to be on a private virtual network,
but we still want to be able to access the Internet through the DC.

Open Server Manager > select "Add roles and features".

Click Next through the Wizard, and select these options as they appear:

  - [Server Selection] Select the DC

  - [Sever Roles] Check Remote Access

  - [Role Services] Check DirectAccess and VPN (RAS)

    - Click Add Features

    - Check Routing

  - Click Next through the rest of the Wizard, click Install

  - When done, click Close

Navigate to Server Manager > Tools > Routing and Remote Access

  - Right-click on 'DC (local)' and select Configure and Enable Routing and Remote Access

  - Click nect through the Wizard, and select these options as they appear:

    - [Configuration] Select Network address tranlation (NAT)

    - [NAT Internet Connection] Select "Use this public interface to connect to the Internet"

      - Select _INTERNET_

      > **NOTE:**
      > If these options are unavaliable, try closing and reopening Routing and Remote Access

  - Continue clicking Next through the Wizard then click Finish.

Now there will be a green symbol next to "DC (local)" in Routing and Remote Access,
the setup has been configured ***correctly!*** Now our client VMs should be able to connect
once we setep DCHP for them!


### Configuring DCHP Server

Open Server Manager > select "Add roles and features".

  - Click next through the Wizard, choosing these options as they appear:

    - [Server Selection] Select DC.mydomain.com

    - [Server Roles] Select DHCP Server > Add Features

    - Click Next through the rest of the settings, click Install.

  - Navigate to Server Manager > Tools > DHCP

    - Expand the dc.mydomain.com drop-down menu.

    - Right-click on IPv4 and select New Scope…

  - Click next through the Wizard, choosing these options on the following tabs:

    - [Scope Name] Name: 172.16.0.100-200

    - [IP Address Range]

      - Start IP Address: 172.16.0.100

      - End IP Address: 172.16.0.200

      - Length: 24

      - Subnet mask: 255.255.255.0

    - [Router (Default Gateway)] IP address: 172.16.0.1 (and click Add!)

  - Click Next through the rest of the settings, and click Finish.

    - Right-click on dc.mydomain.com and select Authorize.

    - Right-click on dc.mydomain.com and select Refresh.

    - A green symbol should appear next to IPv4 and IPv6 to show that DHCP is now properly configured!

### How to enable internet access from this virtual machine

 > **Note:**
 > In a real production environment, we do not want to allow the domain server to access the Internet.

  - Inside of Server Manager:

    - Select the Local Server tab on the left-hand side.

    - Find the properties set for “IE Enhanced Security Configuration: On”.

    - Change the setting to Off for both Administrators and Users.

### Automating User Account Creation with PowerShell

Download the PowerShell script from the repository above and pasting the link into Internet Explorer.

Download and open the .zip file, choose the Compressed Folder Tools options tab and press Extract All.

Browse to your Desktop folder and select Extract.

The `names.txt` file contains a list of names that our PowerShell script will use to populate our Active Directory with user accounts.

Open `names.txt`, add your name to the file, and save it.

Run Windows PowerShell ISE as Administrator.

In PowerShell click File > Open and navigate to the AD_PS Folder/CREATE_USERS.ps1.

  - Type the command “Set-ExecutionPolicy Unrestricted” into the PowerShell Terminal, and press enter.

   - Select “Yes to all” on the window that appears.

  - Type the command “cd C:\Users\p-smith\Desktop\ActiveDirectoryHomeLab”.

    - (Replace “p-smith” with the username for the admin account that you created.)
  - Press the Run Script button (or the F5 key) to start executing the script.

***Now if you open Active Directory Users and Computers, expand mydomain.com, and click the _USERS folder, you can see all of the users inside!***

  > **Note:**
    that the way we formatted these user accounts is different than how we set up the admin account:
    p-smith (Administrator)
    p-smith (User)
  >



## Setting up Windows 10 Client

Open Virt Manager in Terminal:
```bash
virt-manager
```

Click `Create a new virtual machine`

- `Local install media (ISO Image, or CDROM)` > Forward

- `Browse` for ISO Image

- Select ISO Image for Windows 10

- Make sure on Chose the operating system you are installing -> `Microsoft Windows 10` > Foward

- Click `yes` on the next window

***I did the same setup options as the Win Server:***

- Hardware

  - Memory: 4096MiB

  - CPUs: 2

- Virtual Hard Disk:

  - 60.00 GiB

    - My boot drive did not have enough space, so `Select or create custom storage`

      - + to create new volume

      - Set storage size

      - Finish

    - Choose the new win10.qcow2 image

    - Forward

  - Name: CLIENT1

  - Network selection:

    - Virtual network 'intnet': Isolated network (***!IMPORTANT***)

  - Check

  - Finish

***Now boot the VM**
You will need to press enter when it says boot from CD or DVD....
  - Use arrow keys.
    - Choose QEMU DVD-ROM
      - Press Enter again

Now continue through the Windows Setup Wizard with these options:

Continuing through the Windows Setup wizard, these are the options you should select when prompted to:

  - Activate Windows

    - “I don’t have a product key”.

  - Select the operating system you want to install.

    - Windows 10 Pro (IMPORTANT!)

  - Which type of installation do you want?

    - Custom: Install Windows only (advanced)

    - Click next to the following screen to select the default partition.

  - Windows will reboot - Continue through until you get to the steps below

  - Let’s connect you to a network

    - “I don’t have internet”

    - Continue with limited setup

  - Who’s going to use this PC?

    - Name: user

    - Password: Password1

  - Services

    - For privacy settings, I like to disable everything, and skip customization.

And there we go! Now if you open Command Prompt and type `ipconfig` then all of your internet settings should be assigned from your Domain Controller. (As long as you have both VMs running at the same time!)

You should have access to the internet via the Domain Controller, and you can verify this by running the command `ping google`, or `ping mydomain.com` to ping the DC and get a response.

There is only one final step in joining the Domain Controller, and that is:

  - Open Settings > System > About > Related Settings > Rename this PC (advanced).

  - [Computer Name] Press the “Change…” button.

    - Computer name: CLIENT1

    - Member Of: Domain: mydomain.com

    - A Windows Security tab will open, and you can enter the user account you created for yourself to give the PC access to the domain. (jsmith/Password1)

    - Allow the PC to restart to join it to the domain.

When the VM reaches the sign in screen, you can now choose the “Other user” option and sign-in with any of the user accounts that we created earlier. A new profile will be built whenever a new user signs in.

And then you're all done!! :D YUURRRR

## Issues Conclusion

After publishing this lab, I noticed that the page was loading slower than expected. I ran a Cloudflare syntheic monitor and it showed me Largest Contentful Paint (LCP) was nearly 4 seconds.

The issue turned out to be the hero image for this post. The original banner image was 3.8MB, which significantly delayed page rendering.

In order to fix this, I:

- Resized the image to 1200px width
- Converted it from PNG to WebP
- Stripped metadata
- Reduced quality to 65 for optimal compression

```bash
magick assets/img/ad-banner.png -resize 1200x -strip -quality 65 assets/img/ad-banner.webp
```

After deploying the optimized image and purging Cloudflare cache, LCP improved from ~3.9s to 734ms and the performance score increased to 99.

This was a good reminder that large media assets can drastically impact web performance, even on static sites.


## Lab Conclusion

Building this Active Directory homelab gave me hands-on experience with core enterprise infrastructure concepts that are difficult to fully understand through theory alone.

By configuring a Windows Server 2025 Domain Controller, joining a Windows 10 client to the domain, and automating user creation with PowerShell, I was able to simulate a small-scale enterprise environment. This lab reinforced how critical proper network configuration, DNS setup, and role management are in domain environments.

Generating over 1,000 users with a script also highlighted how automation plays a major role in systems administration. In real-world environments, repetitive tasks are rarely performed manually. Understanding how to script and manage bulk changes is essential for scalability and security.

This lab also helped me better understand:

  - How domain authentication flows between client and controller

  - The importance of internal DNS in Active Directory

  - Organizational Units and user management at scale

  - How Group Policy can be leveraged in structured environments

Going forward, I plan to expand this lab by:

  - Implementing Group Policy Objects (GPOs)

  - Adding a file server role

  - Simulating attack scenarios for detection practice

  - Monitoring authentication logs and failed login attempts

  > **NOTE:**
  > This will be done with Blake Faust, a learning Cyber Professional like myself. You can find his blog for his attack scenario while I will be doing detection practice.
  > [Blake Faust](https://blake-faust.com/)
  > ***This is currently a WIP...***

Overall, this project strengthened both my technical understanding of Active Directory and my ability to design, deploy, and troubleshoot enterprise-style infrastructure in a controlled environment.
