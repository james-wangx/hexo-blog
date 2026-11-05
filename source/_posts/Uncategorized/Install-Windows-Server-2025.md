---
title: Install Windows Server 2025
description: Install Windows Server 2025
date: 2026-11-05 19:13:27
tags:
  - Install
  - Windows Server
categories:
  - Uncategorized
---

## 1. Download the ISO

Open the [Windows Server 2025 Evaluation Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2025) and select **Download the ISO**.

![Windows Server 2025 Evaluation Center download page](./Install-Windows-Server-2025/01-download-page.png)

Complete the trial registration form. The information is used to provide access to the evaluation download.

![Windows Server 2025 evaluation registration form](./Install-Windows-Server-2025/02-evaluation-registration.png)

Select the preferred language and the 64-bit ISO download. Save the ISO to a location that VMware can access.

![Windows Server 2025 language and ISO download options](./Install-Windows-Server-2025/03-download-language-options.png)

## 2. Create the virtual machine

In VMware Workstation, select **File > New Virtual Machine**.

![Create a new VMware virtual machine](./Install-Windows-Server-2025/04-new-virtual-machine.png)

Choose **Custom (advanced)** to configure the virtual hardware manually.

![Select the custom virtual machine configuration](./Install-Windows-Server-2025/05-custom-installation.png)

Keep the hardware compatibility level at **Workstation 25H2 or later** unless the VM must run on an older VMware host.

![Select the VMware hardware compatibility level](./Install-Windows-Server-2025/06-hardware-compatibility.png)

Choose **Installer disc image file (iso)** and browse to the Windows Server 2025 ISO.

![Select the Windows Server 2025 ISO](./Install-Windows-Server-2025/07-installation-source.png)

Select **Microsoft Windows** and version **Windows Server 2025**.

![Select the Windows Server 2025 guest operating system](./Install-Windows-Server-2025/08-guest-operating-system.png)

Enter a VM name and location. The example uses `TcLicServer`, but any descriptive name can be used.

![Enter the virtual machine name and location](./Install-Windows-Server-2025/09-vm-name-and-location.png)

Select **UEFI** as the firmware type. Enable Secure Boot if required by your environment.

![Select the UEFI firmware type](./Install-Windows-Server-2025/10-firmware-type.png)

The example assigns one processor with two cores and 2 GB of memory. Increase these values if the host has enough resources and the VM will run additional workloads.

![Configure the processor count and cores](./Install-Windows-Server-2025/11-processor-configuration.png)

![Configure the virtual machine memory](./Install-Windows-Server-2025/12-memory-size.png)

Select **NAT** so the VM can access the network through the host.

![Select the NAT network type](./Install-Windows-Server-2025/13-network-type.png)

Use the recommended **LSI Logic SAS** controller for Windows Server 2025.

![Select the LSI Logic SAS controller](./Install-Windows-Server-2025/14-io-controller-type.png)

Select **NVMe** as the virtual disk type.

![Select the NVMe virtual disk type](./Install-Windows-Server-2025/15-disk-type.png)

Create a new virtual disk and set its capacity to 100 GB. Do not allocate all disk space immediately, and keep the split-virtual-disk option selected.

![Select the virtual disk options](./Install-Windows-Server-2025/16-disk-selection.png)

![Set the virtual disk capacity](./Install-Windows-Server-2025/17-disk-capacity.png)

![Set the virtual disk file name](./Install-Windows-Server-2025/18-disk-file.png)

Review the summary and select **Finish** to create the VM.

![Review the virtual machine summary](./Install-Windows-Server-2025/19-ready-to-create-vm.png)

Before starting the VM, open its settings and confirm that the ISO is selected under **CD/DVD**, with **Connect at power on** enabled.

![Confirm the Windows Server installation ISO is attached](./Install-Windows-Server-2025/20-attach-installation-iso.png)

## 3. Install Windows Server 2025

Power on the VM and press any key when **Press any key to boot from CD or DVD** appears.

![Boot the virtual machine from the installation media](./Install-Windows-Server-2025/21-boot-from-cd.png)

Wait for the Windows boot logo and the setup program to start.

![Windows Server boot logo](./Install-Windows-Server-2025/22-windows-boot-logo.png)

Select the installation language, time and currency format, and keyboard or input method, then select **Next**.

![Select the Windows Setup language](./Install-Windows-Server-2025/23-language-settings.png)

![Select the keyboard and input method](./Install-Windows-Server-2025/24-keyboard-settings.png)

Select **Install Windows Server**.

![Select the Windows Server installation option](./Install-Windows-Server-2025/25-setup-option.png)

Choose the required edition. This example uses **Windows Server 2025 Standard Evaluation (Desktop Experience)**, which includes the graphical desktop. The Datacenter edition provides additional virtualization and scale features, while Server Core does not include the desktop experience.

![Select the Windows Server 2025 edition](./Install-Windows-Server-2025/26-image-edition.png)

Review and accept the license terms.

![Accept the Windows Server license terms](./Install-Windows-Server-2025/27-license-terms.png)

Select the unallocated virtual disk and continue. Setup can create the required partitions automatically.

![Select the Windows Server installation location](./Install-Windows-Server-2025/28-install-location.png)

Review the selected edition and installation target, then select **Install**.

![Review the Windows Server installation settings](./Install-Windows-Server-2025/29-ready-to-install.png)

Windows Setup copies the installation files and may restart the VM several times. Leave the virtual machine running until setup is complete.

![Windows Server 2025 installation progress](./Install-Windows-Server-2025/30-installation-progress.png)

Setup may restart and continue through additional installation phases. The percentage and number of restarts can vary depending on hardware and updates.

![Windows Server 2025 setup continues at 42 percent](./Install-Windows-Server-2025/31-installation-progress-42-percent.png)

## 4. Complete the first sign-in

When Windows asks for the built-in Administrator password, enter and confirm a strong password that meets the local security requirements.

![Set the built-in Administrator password](./Install-Windows-Server-2025/32-administrator-password.png)

After the restart, the lock screen may prompt for `Ctrl+Alt+Delete`. Use **VM > Send Ctrl+Alt+Del**; `Ctrl+Alt+Insert` is the equivalent VMware keyboard shortcut.

![Windows Server lock screen](./Install-Windows-Server-2025/33-unlock-screen.png)

![Send Ctrl+Alt+Delete from the VMware VM menu](./Install-Windows-Server-2025/34-send-ctrl-alt-delete.png)

Select **Administrator**, enter the password, and continue.

![Sign in with the Administrator account](./Install-Windows-Server-2025/35-administrator-sign-in.png)

The first sign-in may briefly show a black screen while Windows initializes the user profile and desktop services. Wait a few minutes before restarting the VM. If the black screen persists, send `Ctrl+Alt+Delete` and try opening Task Manager; a memory increase from 2 GB to 4 GB or more can also help on slower hosts.

When the diagnostic data page appears, **Required only** is sufficient for a basic evaluation environment. Select **Accept** to continue.

![Select the required diagnostic data option](./Install-Windows-Server-2025/36-diagnostic-data-settings.png)

Windows then opens the desktop.

![Windows Server 2025 desktop](./Install-Windows-Server-2025/37-windows-server-desktop.png)

## 5. Complete the initial configuration

After the desktop loads, install **VMware Tools** and restart the VM. Then disconnect the installation ISO, run Windows Update, and configure the computer name, time zone, and network settings required by the environment.
