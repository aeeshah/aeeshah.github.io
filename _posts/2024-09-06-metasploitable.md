---

layout: post  
title: Building a Cybersecurity Lab Part 4 - Setting up Metasploitable 2 as a Target Machine on Mac M1 Using UTM 
description: Learn how to install Metasploitable 2 on a Mac M1 using UTM with this step-by-step guide. Ideal for penetration testing and cybersecurity training.
date: 05-09-2024  
categories: [Documentation, security, Home-lab]  
tags: [cybersecurity, security, home lab, mac m1, blue team, networking, pfSense]  

image:  
    path: /assets/images/meta/msf0.png  
    alt: Cyber Security Home Lab Setup  

---

After setting up pfSense, I decided to add **Metasploitable 2** as my target machine for penetration testing. **Metasploitable 2** is intentionally vulnerable, making it the perfect target for anyone wanting to practice ethical hacking in a controlled environment. In this post, I'll share how I set it up on my **Mac M1** with **UTM**.


### **Step-by-Step Guide to Installing Metasploitable 2 on Mac M1**

#### **1. Download Metasploitable 2**
First, I grabbed **Metasploitable 2** from the [SourceForge website](https://sourceforge.net/projects/metasploitable/). It comes as a **VMDK** file, which I needed to convert to **QCOW2** to be compatible with UTM.

#### **2. Convert VMDK to QCOW2 Format**
This was the tricky part. Since **UTM** doesn't work with **VMDK** files, I had to convert it using **QEMU**. Luckily, it wasn’t too hard once I got **QEMU** installed through **Homebrew**.

- Install Homebrew if you haven’t already:

   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

- Then, install **QEMU**:

   ```bash
   brew install qemu
   ```

- Next, convert the **VMDK** file to **QCOW2**:

   ```bash
   qemu-img convert -f vmdk -O qcow2 /path/to/metasploitable.vmdk /path/to/metasploitable.qcow2
   ```

Replacing the file paths with the location of my **VMDK** and the destination for the new **QCOW2** file was straightforward.

#### **3. Create a New VM in UTM**
With the **QCOW2** file ready, I opened **UTM** and created a new virtual machine:

1. Selected **Emulate** and chose **Linux** as the operating system.
2. Set the architecture to `x86_64`.

#### **4. Configure the VM**
Now came the fun part—configuring the VM! Here's what I set:

- **Memory**: 512MB (plenty for Metasploitable 2).
- **CPU**: 1 core.
- **Storage**: Imported the **QCOW2** file with at least 10GB of space.

#### **5. Network Settings**
I went with **Host-Only** mode for the network, which isolates the VM from my main network and provides a safe testing environment. I also selected **virtio-net-pci** as the network card.

#### **6. Start the VM**
Once everything was set, I clicked **Start**, and just like that, the virtual machine booted into **Metasploitable 2**!

#### **7. Log in to Metasploitable 2**
The default login credentials were all I needed to access the system:

- **Username**: `msfadmin`
- **Password**: `msfadmin`

#### **8. Verify Network Setup**
To make sure my network was configured correctly, I ran the following command:

```bash
ifconfig
```

