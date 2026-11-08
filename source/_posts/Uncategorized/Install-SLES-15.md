---
title: Install SUSE Linux Enterprise Server 15 SP7
description: Install SUSE Linux Enterprise Server 15 SP7 in VMware Workstation and complete the initial network, storage, and user configuration.
date: 2026-11-05 19:10:26
tags:
  - Install
  - SLES
categories:
  - Uncategorized
---

SUSE Linux Enterprise Server (SLES) 15 SP7 can be installed as a virtual machine with VMware Workstation. This guide uses the full installation ISO, configures a static network and a custom disk layout, and finishes with a text-mode login. The hostname, IP addresses, DNS servers, and passwords shown below are examples; replace them with values that match your environment.

## 1. Download the ISO

Open the [SUSE Linux Enterprise Server download page](https://www.suse.com/download/sles/), select **SLES 15 SP7** and the **x86_64** architecture.

![SUSE Linux Enterprise Server download page](./Install-SLES-15/01-suse-download-page.png)

Choose the installation media that fits the deployment. The **Full DVD** ISO supports an offline installation, while the **Online Media** requires access to SUSE repositories during installation. This guide uses the Full DVD image.

![SLES 15 SP7 ISO download options](./Install-SLES-15/02-sles-15-sp7-download-options.png)

## 2. Create the virtual machine

In VMware Workstation, select **File > New Virtual Machine**.

![Create a new VMware virtual machine](./Install-SLES-15/03-new-virtual-machine.png)

Select **Typical (recommended)** and continue.

![Select the typical virtual machine configuration](./Install-SLES-15/04-typical-configuration.png)

Choose **Installer disc image file (iso)** and browse to the SLES 15 SP7 ISO.

![Select the SLES installation ISO](./Install-SLES-15/05-installation-iso.png)

Enter a virtual machine name and location. The example uses `Aurobay-2512`.

![Set the virtual machine name and location](./Install-SLES-15/06-vm-name-and-location.png)

Set the virtual disk capacity to 200 GB. Keeping the disk split into multiple files is suitable for a typical VMware Workstation deployment.

![Set the virtual disk capacity](./Install-SLES-15/07-disk-capacity.png)

The default summary allocates 2 GB of memory and two processor cores. These values are sufficient only for a basic test, so select **Customize Hardware** before powering on the VM.

![Review the default virtual machine hardware](./Install-SLES-15/08-vm-summary-default-hardware.png)

Increase the memory to 16 GB for this example.

![Set the virtual machine memory to 16 GB](./Install-SLES-15/09-custom-memory.png)

Configure one processor with eight cores, then review the updated hardware summary.

![Configure eight processor cores](./Install-SLES-15/10-custom-processors.png)

![Review the customized virtual machine hardware](./Install-SLES-15/11-vm-summary-custom-hardware.png)

Power on the VM and select **Installation** from the SLES boot menu.

![Select installation from the SLES boot menu](./Install-SLES-15/12-boot-menu.png)

## 3. Configure installation options

Select the installation language, keyboard layout, and **SUSE Linux Enterprise Server 15 SP7** product, then select **Next**.

![Select the language, keyboard, and product](./Install-SLES-15/13-language-and-product.png)

Read and accept the SUSE license agreement.

![Accept the SUSE license agreement](./Install-SLES-15/14-license-agreement.png)

### Configure the network

The network page initially shows `eth0` without an assigned address. Select the adapter and choose **Edit**.

![Open the network adapter configuration](./Install-SLES-15/15-network-overview.png)

Select a static address and enter the example address `192.168.80.110/24` with hostname `Aurobay-2512`.

![Configure the static IP address and hostname](./Install-SLES-15/16-static-ip-and-hostname.png)

On the **Hostname/DNS** tab, disable DHCP-based hostname assignment if required, then enter the DNS servers for the environment. The example uses `192.168.80.2`, `114.114.114.114`, and `8.8.8.8`.

![Configure hostname and DNS settings](./Install-SLES-15/17-hostname-and-dns.png)

Open the **Routing** tab and add a default route.

![Open the routing table editor](./Install-SLES-15/18-routing-table-empty.png)

Set the example gateway to `192.168.80.2` and select `eth0` as the device.

![Add the default gateway](./Install-SLES-15/19-default-gateway.png)

Confirm that the routing table contains the expected default route before continuing.

![Review the configured routing table](./Install-SLES-15/20-routing-table-configured.png)

### Select modules and the system role

For an offline lab system, select **Skip Registration**. For a production system with repository access, register the system so it can receive security updates.

![Skip system registration for the offline example](./Install-SLES-15/21-skip-registration.png)

Keep the **Basesystem Module**. Add any modules required by the workload; this example also selects the **Server Applications Module**.

![Select the Basesystem Module](./Install-SLES-15/22-basesystem-module.png)

![Select the Server Applications Module](./Install-SLES-15/23-server-applications-module.png)

Use **Add-On Products** only when an additional ISO or local repository is required.

![Add optional installation products](./Install-SLES-15/24-add-on-products.png)

Select **Text Mode** for a server installation. Choose **X Window System** instead if a graphical desktop is required.

![Select the text-mode system role](./Install-SLES-15/25-system-role-text-mode.png)

## 4. Configure disk partitions

The installation proposes a guided partition layout. Select **Expert Partitioner** to create the custom layout used in this guide.

![Open the expert partitioner](./Install-SLES-15/26-suggested-partitioning.png)

Before deleting or changing partitions, verify that the correct virtual disk is selected. This example starts with an automatically proposed layout.

![Review the proposed partition layout](./Install-SLES-15/27-default-partition-layout.png)

Delete the proposed partitions while keeping the small **BIOS Boot Partition**. This partition is required by the example's BIOS/GPT boot layout. If the VM uses UEFI, keep the EFI System Partition instead.

![Keep only the BIOS Boot partition](./Install-SLES-15/28-bios-boot-partition.png)

Create an 8 GB swap partition.

![Set the swap partition size](./Install-SLES-15/29-swap-partition-size.png)

Set the partition role to **Swap**.

![Set the swap partition role](./Install-SLES-15/30-swap-partition-role.png)

Use the **Swap** filesystem and the **Linux Swap** partition ID.

![Configure the swap filesystem](./Install-SLES-15/31-swap-filesystem.png)

The partition list should now contain the BIOS Boot and swap partitions.

![Review the partition layout after adding swap](./Install-SLES-15/32-partition-layout-after-swap.png)

Create the root partition with the remaining `191.99 GB`.

![Set the root partition size](./Install-SLES-15/33-root-partition-size.png)

In this example, select **Data and ISV Applications** as the partition role.

![Select the root partition role](./Install-SLES-15/34-root-partition-role.png)

Format the partition with **XFS**, select the **Linux** partition ID, and mount it at `/`.

![Configure the root XFS filesystem](./Install-SLES-15/35-root-filesystem.png)

Confirm that the final disk layout contains the BIOS Boot, swap, and root partitions.

![Review the final partition layout](./Install-SLES-15/36-final-partition-layout.png)

Review the manual partitioning summary and continue.

![Review the manual partitioning summary](./Install-SLES-15/37-partition-summary.png)

## 5. Complete and run the installation

Select the required region and time zone. The example uses **Asia > Shanghai**.

![Select the system time zone](./Install-SLES-15/38-time-zone.png)

Enable NTP synchronization with a reachable NTP server. The example uses the SUSE pool.

![Configure NTP synchronization](./Install-SLES-15/39-ntp-configuration.png)

Local user creation is skipped in this example. For a production server, create a non-root administrative account and use it for normal access.

![Configure the local user](./Install-SLES-15/40-local-user.png)

Set the root password. Use a strong password that meets the required complexity policy.

![Set the root password](./Install-SLES-15/41-root-password.png)

The installer warns when the password is weak. Choose a stronger password instead of confirming a weak one, even though the example dialog allows it.

![Weak root password warning](./Install-SLES-15/42-weak-password-warning.png)

Review the installation settings. Confirm the firewall and SSH options match the intended access model, then select **Install**.

![Review the installation settings](./Install-SLES-15/43-installation-settings.png)

Confirm the installation to begin modifying the virtual disk.

![Confirm the installation](./Install-SLES-15/44-confirm-installation.png)

Wait while the packages are installed. Do not power off the virtual machine during this phase.

![SLES package installation progress](./Install-SLES-15/45-installation-progress.png)

After installation and reboot, the system reaches a text-mode login prompt. The screenshot confirms the example hostname `Aurobay-2512` and address `192.168.80.110`.

![SLES first login console](./Install-SLES-15/46-first-login-console.png)

After the first login, remove the installation ISO, verify the hostname and network configuration, and register the system if repository access is available. Then install updates and VMware guest integration packages such as `open-vm-tools` when the configured repositories provide them.
