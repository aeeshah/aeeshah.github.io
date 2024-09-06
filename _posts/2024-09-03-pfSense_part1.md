---

layout: post
title:  Building a Cybersecurity Lab Part 1- pfSense Basic Setup & Configuration.
date: 03-09-2024
categories: [Documentation,security,Home-lab]
tags: [cybersecurity,security,home lab,mac m1,blue team,networking,pfSense]

image: 
    path: /assets/images/pfSense/pfSense0.png
    alt: Cyber Security Home Lab Setup

---

This part of the lab focuses on a basic configuration for pfSense required to onboard the subnets that make up this lab. pfSense is a powerful, open-source firewall and router software that provides robust security features. It will act as the central point of control for traffic entering and exiting the network segments.

## Why pfSense?

pfSense is highly regarded in the cybersecurity community for its flexibility, performance, and comprehensive feature set. It allows us to create sophisticated network configurations, enforce security policies, and monitor traffic. By setting up pfSense, the lab’s network will be secure and segmented, which is crucial for effective practice and testing.

## Step-by-Step Setup Guide

### 1. **Download and Install pfSense**

1. **Download pfSense**: Go to the [pfSense official website](https://www.pfsense.org/download/) and download the latest version of pfSense suitable for your architecture (x86 for most setups). Choose the ISO image for installation. 

> Due to changes in the download process, I downloaded from a mirror at “[Netgate Downloads](https://atxfiles.netgate.com/mirror/downloads/)” version 2.7.2 
{: .prompt-tip} 

2. **Create a New VM in UTM**:
   - Open UTM and click on create a new virtual machine.
   ![welcome screen UTM](/assets/images/pfSense/pfSense1.png){: w="400" h="200" }
   _UTM Welcome Screen_

   - For the operating system, choose `Other`.

   ![Operating system selection](/assets/images/pfSense/pfSense2.png){: w="400" h="200" }
   _Operating System Selection_

   - Select **CD/DVD image** as the boot device and select the downloaded pfSense ISO as the boot image.

    ![Boot Image](/assets/images/pfSense/pfSense3.png){: w="400" h="200" }
    _Boot image and ISO file_

   - Configure the VM with the following settings:
     - **Architecture**: `x86_64`
     - **System**: `Standard PC (Q35 + ICH9, 2009)`
     - **Memory**: `512MB RAM`
     - **CPU**: `1 core`
     - **Storage**: `2GB ` 
   This should be sufficient for pfSense.

    ![VM Configuration](/assets/images/pfSense/pfSense4.png){: w="400" h="200" }
    _pfSense Configuration_

3. **Configuring Network Adapters in UTM**:

For WAN select the following;
Network mode: `Bridged` (if you want pfSense to connect to the external network).
Bridged Interface: `en0` or any other available
Emulated Network card: `virtio-net-pci` (Paravirtualized Network)  or `e1000` based on your performance and compatibility needs.


Ensure correct network adapters are set up in UTM for pfSense:

- **WAN Interface**: This connects to the external network or the internet.
  - **Network mode**: `Bridged`
  - **Bridged Interface**: `en0` or any other available
  - **Emulated Network card**: `virtio-net-pci` or `e1000`.

![WAN network](/assets/images/pfSense/pfSense5.png){: w="400" h="200" }
_WAN Interface Configuration_

- **LAN Interface**: This connects to your internal lab network.
  - **Network mode**: `Host only`
  - **Emulated Network card**: `virtio-net-pci`.  
  Repeat this process for each additional LAN. I created four (4) in total, one for each lab.  
  Click `Save` to apply the changes and start pfSense.

![LAN network](/assets/images/pfSense/pfSense6.png){: w="400" h="200" }
_LAN Interface Configuration_

3. **Install pfSense**:
   - On boot, a banner will appear followed by text. Press Enter to accept the agreement.

   ![Terms](/assets/images/pfSense/pfSense7.png){: w="400" h="200" }
    _pfSense Terms_

   - Press Enter to install pfSense.

    ![Install pfSense](/assets/images/pfSense/pfSense8.png){: w="400" h="200" }
    _pfSense Install_

   - Select `Auto UFS` for partitioning, 

   ![Partition](/assets/images/pfSense/pfSense9.png){: w="400" h="200" }
    _pfSense Partitioning_

   then choose the entire disk, 

   ![Partitioning](/assets/images/pfSense/pfSense10.png){: w="400" h="200" }
    _pfSense Partition_
   
   and `GPT GUID Partition table` for the partition scheme.

   ![Partition scheme](/assets/images/pfSense/pfSense11.png){: w="400" h="200" }
    _pfSense Partition Scheme_

   - Commit the changes and wait for the installation to complete. 
   
   ![Confirmation](/assets/images/pfSense/pfSense12.png){: w="400" h="200" }
    _Confirmation_

   Once done, press `Enter` to reboot.

   ![Reboot](/assets/images/pfSense/pfSense13.png){: w="400" h="200" }
   _Reboot_

>After reboot, remove the ISO from the boot sequence else when you power pfSense it will display the pfSense installer again 
{: .prompt-warning} 

  - Launch UTM on your Mac.
  - Choose the recently created pfSense virtual machine from the list.
  - from the preview, click on CD/DVD and select clear to remove the ISO file
 
 ![Remove Boot Sequence](/assets/images/pfSense/pfSense14.png){: w="400" h="200" }
   _Remove Boot Sequence_
     

### 2. **Initial Configuration**

Once pfSense reboots, you will configure the adapters from the VM settings.

1. Should VLANs be set up now? `n` 

 ![Vlan setup](/assets/images/pfSense/pfSense15.png){: w="400" h="200" }
   _VLAN Setup_

2. Enter interface names:
   - **WAN**: vtnet0
   - **LAN**: vtnet1
   - **OPT1**: vtnet2
   - **OPT2**: vtnet3
   - **OPT3**: vtnet4

   Proceed with `Y` 

 ![Vlan Interface](/assets/images/pfSense/pfSense16.png){: w="400" h="200" }
   _VLAN Interfaces_

   The WAN interface has received an IPv4 address from UTM’s DHCP. However, the OPT1 and OPT2 interfaces currently lack IP addresses. To ensure the IP addresses for the LAN, OPT1, OPT2 & OPT3 interfaces remain consistent after each reboot, we will assign static IPv4 addresses to the interfaces.  

![pfSense Interfaces](/assets/images/pfSense/pfSense17.png){: w="400" h="200" }
   _pfSense Interfaces_

   - Select `Option 2` to set an IP address.
   - For `LAN` following this following:
     - Configure IPv4 address LAN interface viaa DHCP? `n`
     - Enter the new LAN IPv4 address. > `192.168.10.1`
     - Enter the new LAN IPv4 subnet bit count (1 to 32): `24`
     - For a LAN, press <ENTER> for none: press `enter`
     - Configure IPv6 address LAN interface via DHCP6? `n`
     - Enter the new LAN IPv6 address. Press <ENTER> for none: Press `enter`
     - Do you want to enable the DHCP server on LAN?: `y`
     - Enter the start address of the IPv4 client address range: `192.168.10.11`
     - Enter the end address of the IPv4 client address range: `192.168.10.243`
     - Do you want to revert to HTTP as the webConfigurator protocol?: `n`

![LAN Interface](/assets/images/pfSense/pfSense18.png){: w="400" h="200" }
   _LAN Interface Setup_


   Repeat the process for the other interfaces using different ip addresses.

>For the Active Directory lab, the Domain Controller (DC) will handle DHCP. Thus, DHCP is disabled for OPT3.
{: .prompt-info} 

For Active Directory use the following:
- Configure IPv4 address LAN interface viaa DHCP? `n`
- Enter the new LAN IPv4 address. > `192.168.20.1`
- Enter the new LAN IPv4 subnet bit count (1 to 32): `24`
- For a LAN, press <ENTER> for none: **press enter**
- Configure IPv6 address LAN interface via DHCP6? `n`
- Enter the new LAN IPv6 address. Press <ENTER> for none: **Press enter**
- Do you want to enable the DHCP server on OPT3?: `n`
- Do you want to revert to HTTP as the webConfigurator protocol?: `n`

![AD Interface](/assets/images/pfSense/pfSense19.png){: w="400" h="200" }
   _Active Directory Interface Setup_
   
   
The IP addresses for each interface are now configured.

![pfSense](/assets/images/pfSense/pfSense20.png){: w="400" h="200" }
   _pfSense Interfaces_

3. **Shut Down pfSense**:

With this, we've successfully completed the interface setup in pfSense we can then shut down and halt the system. To achieve this 
   - Enter **Option 6** to halt the system.
   - Confirm with `Y`.

![pfSense Halt](/assets/images/pfSense/pfSense21.png){: w="400" h="200" }
   _pfSense Halt

Additional configurations will be handled after setting up Kali Linux in the next module. From Kali, we’ll access the pfSense web interface for further setup. pfSense will act as the default gateway and firewall for the home lab.

>pfSense will serve as the default gateway and firewall for the home lab. It’s important to boot the pfSense VM first, and once it’s running, the other VMs in the lab can be started.
{: .prompt-info} 

## What’s Next?

In the next part of the lab, we’ll set up Kali Linux on the LAN interface to continue configuring pfSense through its web interface. This Kali Linux VM will be used both to configure and manage pfSense, as well as serve as the attack machine to target vulnerable systems on the OPT2 interface.



