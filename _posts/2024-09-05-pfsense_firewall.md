---

layout: post
title:  Building a Cybersecurity Lab Part 3 - pfSense Configurations
description: Learn how to configure your pfSense firewall, assign static IPs, and set up rules for different subnets in your home lab. Follow along as we walk you through the pfSense interface, DNS resolver settings, and advanced configurations, ensuring a secure and efficient network environment.
date: 05-09-2024
categories: [Documentation,security,Home-lab]
tags: [cybersecurity,security,home lab,mac m1,blue team,networking,pfSense]

image: 
    path: /assets/images/pfsense2/pf0.png
    alt: Cyber Security Home Lab Setup

---

In this part of the setup, we will finish the pfSense configuration for our home lab, followed by setting up firewall rules for different subnets. If you’ve been following along, we’re getting closer to fully securing and customizing our network environment. Let’s get started!

### Completing the pfSense General Setup

#### Accessing the Web Portal

To start, hop onto your **Kali Linux VM**, fire up the browser, and navigate to `https://192.168.10.1`. Now, don’t be alarmed when you see the **“Warning: Potential Security Risk Ahead”** message. This happens because we’re using an unsecured HTTP (no “S” for secure), but it’s all good here. Just click **Advanced**, then **Accept the Risk and Continue**, and you’ll land on the **pfSense Web UI login page**.
Here, log in using the default credentials:

- **Username**: `admin`
- **Password**: `pfsense`

![pfsenseLogin](/assets/images/pfsense2/pf1.png){: w="400" h="200" }
_pfSense Web Login Page_

Now that we’re in, let’s step through the initial configuration process.

#### General Information

First, provide a **Hostname** and **Domain Name**. You can name these however you like. The hostname will help identify your pfSense VM within the network. 

Next, uncheck the **Override DNS** option, then hit **Next**. From there, select your **Timezone** and keep moving forward by clicking **Next**.

#### WAN Setup and Private Networks

When you get to the section about **RFC1918 Private Networks**, make sure to **uncheck the Block RFC1918 Private Networks** option. Here’s why: since we’re using a private IP address on our WAN interface instead of a public one, it’s not a real WAN connection in the traditional sense. This setup allows communication between our internal network and the outside world through the host.

No need to tweak any other settings here, just click **Next**.

#### Securing pfSense

At this point, you’ll be asked to set a new password for the admin user. Don’t skip this step—create a strong password and store it securely.

![pfsenseLogin](/assets/images/pfsense2/pf2.png)
_pfsense Admin account password_


Once that’s done, click **Reload** to apply the changes, followed by **Finish**. With this, the onboarding process is complete, and you’ll be able to access the pfSense dashboard.

### Customizing Interfaces: Renaming for Clarity

Next up, let’s make sure our interfaces are properly labeled.

1. From the navigation bar, go to **Interfaces -> OPT1**.

![pfsenseLogin](/assets/images/pfsense2/pf3.png)
_Displaying list of interfaces_


2. In the **Description** field, type in `TARGET_MACHINE`. This helps us identify this subnet later. Scroll down and hit **Save**.
3. A popup will appear—click **Apply Changes**.

![pfsenseLogin](/assets/images/pfsense2/pf4.png)
_Renaming interface OPT1_

The same step was taken to rename OPT2 and OPT3 to `Security_Onion` and `Active_Directory` respectively. 

![pfsenseLogin](/assets/images/pfsense2/pf5.png)
_Showing all renamed interfaces_

### Configuring the DNS Resolver

Now, let’s ensure proper DNS resolution within our network.

1. Go to **Services -> DNS Resolver**.
2. Scroll down to find DHCP Registration and Static DHCP and enable them. 

![pfsenseLogin](/assets/images/pfsense2/pf6.png)
_Enabling DHCP registration and Static DHCP on pfSense_

3. Scroll back to the top and click **Advanced Settings**.
4. Once in **Advanced Resolver Options**, enable prefetch support and Prefetch DNS key support, then scroll to the bottom and click **Save**.

![pfsenseLogin](/assets/images/pfsense2/pf7.png)
_Advanced Resolver Settings on pfSense_

Remember to hit **Apply Changes** when prompted.

### Advanced Configuration for Better Performance

pfSense can be tuned to improve network performance. Here’s how:

1. Navigate to **System -> Advanced** and open the **Networking tab**.
2. Scroll down to the **Network Interfaces** section and enable `Hardware Checksum Offloading` options to boost performance.

![pfsenseLogin](/assets/images/pfsense2/pf8.png)
_pfSense configuration for better performance_

3. Save the changes, and pfSense will prompt you to reboot. Click **OK** to reboot and apply the configurations.

![pfsenseLogin](/assets/images/pfsense2/pf9.png)
_pfSense Reboot for changes_

After the reboot, log back into the dashboard using the new password.

### Assigning a Static IP to Kali Linux

We’ll assign a static IP to our Kali Linux VM to make applying firewall rules easier.

1. Go to **Status -> DHCP Leases**.
2. In the Leases section, you should see your Kali VM listed with its current IP. Click the `+` icon next to it to assign a static IP.
![pfsenseLogin](/assets/images/pfsense2/pf10.png)
_DHCP Leases on LAN_

3. Enter `192.168.10.2` as the IP address and click **Save**.

![pfsenseLogin](/assets/images/pfsense2/pf11.png)
_Editing DHCP Leases on LAN_

4. Apply the changes via the popup.

Now, let’s ensure the VM is using the static IP by refreshing its network settings:

- Run the command `sudo ip l set eth0 down && sudo ip l set eth0 up` in the terminal.
- Verify the IP using `ip a l eth0`.

![pfsenseLogin](/assets/images/pfsense2/pf12.png)
_Verifying changes on Kali_

### Setting Up Firewall Rules

Finally, we’ll define firewall rules to control traffic between subnets. First, head to **Firewall -> Rules**.

#### LAN Rules

1. In the **LAN** tab, click **Add rule to top**.

![pfsenseLogin](/assets/images/pfsense2/pf14.png)
_Adding Firewall rules on a LAN on pfSense_

2. Configure the rule as follows:
   - **Action**: Block
   - **Address Family**: IPv4+IPv6
   - **Protocol**: Any
   - **Source**: LAN subnets
   - **Destination**: WAN subnets
   - **Description**: Block access to services on WAN interface
3. Save the rule and apply the changes.

#### Target Machine Interface Rules

Next, switch to the **TARGET_MACHINE** tab. Here, we’ll create rules to control traffic to and from our target subnet.

1. Before setting up rules, go to **Firewall -> Aliases** and create an alias for private IP ranges:

![pfsenseLogin](/assets/images/pfsense2/pf15.png)
_Firewall aliase on pfSense_

A firewall alias in pfSense (or other firewall systems) is essentially a shortcut or a named group used to simplify firewall rule management. Instead of manually specifying individual IP addresses, networks, or ports in your firewall rules, you can create an alias, group them together, and reference that alias in the rules. This makes managing rules more efficient, especially in large or complex networks. Input the following:

   - **Name**: `RFC1918`
   - **Description**: `Private IPv4 Addresses`
   - **Type**: `Networks`
   - **Network 1**: `10.0.0.0/8`
   - **Network 2**: `172.16.0.0/12`
   - **Network 3**: `192.168.0.0/16`
   - **Network 4**: `169.254.0.0/16`
   - **Network 5**: `127.0.0.0/8`

![pfsenseLogin](/assets/images/pfsense2/pf16.png)
_Firewall aliase on pfSense_ 

### What is RFC1918?

**RFC1918** defines private IP address ranges that are used for internal networks, not accessible from the internet. These private IPs allow devices within a network to communicate with each other securely while staying hidden from the outside world.

#### Private IP Ranges:
- **10.0.0.0 – 10.255.255.255** (Large networks)
- **172.16.0.0 – 172.31.255.255** (Medium networks)
- **192.168.0.0 – 192.168.255.255** (Small networks, common in homes)

Using RFC1918 IPs keeps internal traffic isolated and secure, perfect for private setups like home labs.
   
2. Now, go back to **Firewall -> Rules** under the **TARGET_MACHINE** tab, and add rules:
   - **Rule 1**: Allow traffic within the **TARGET_MACHINE** subnet.

    ![pfsenseLogin](/assets/images/pfsense2/pf17.png)
    _Allow traffic on Target Machine_ 

     - **Rule 2**: Allow traffic from the Security Onion subnets.

    ![pfsenseLogin](/assets/images/pfsense2/pf18.png)
    _Allow traffic from Security Onion to Target machine_ 

   - **Rule 3**: Allow traffic from the Kali Linux VM (`192.168.10.2`).

    ![pfsenseLogin](/assets/images/pfsense2/pf19.png)
    _Allow traffic from Kali linux to Target machine_ 

   - **Rule 4**: Allow traffic to any non-private IPv4 address (use the alias and select **Invert match**).

    ![pfsenseLogin](/assets/images/pfsense2/pf20.png)
    _Allow traffic to any non-private address_ 

   - **Rule 4**: Block all remaining traffic.

    Save the rules and apply changes.

    ![pfsenseLogin](/assets/images/pfsense2/pf21.png)
    _Summary of firewall rules on Target Machine_ 


#### Security Onion Interface Rules
For configuring firewall rules on Security Onion, similar principles apply as with Metasploitable, but with specific considerations based on the role Security Onion plays in your network. Here's how to set up firewall rules for Security Onion:

### Firewall Rule Configuration for Security Onion

1. **Allow Traffic Within the Subnet**
   - **Action**: `Pass`
   - **Interface**: `Secuity_Onion`
   - **Address Family**: `IPv4`
   - **Protocol**: `Any`
   - **Source**: `Security_Onion subnets`
   - **Destination**:`Security_Onion subnets`
   - **Description**: `Allow internal network traffic`

2. **Allow Traffic to and From Target_Network**
   - **Action**: `Pass`
   - **Interface**: `Secuity_Onion`
   - **Address Family**: `IPv4`
   - **Protocol**: `Any`
   - **Source**:  `Security_Onion subnets`
   - **Destination**: `Target_Machine_IP` 
   - **Description**: Allow traffic between Security Onion and Metasploitable.

3. **Allow Traffic to Kali Linux**
   - **Action**: Pass
   - **Interface**: `Secuity_Onion`
   - **Address Family**: `IPv4`
   - **Protocol**: `Any`
   - **Source**:  `Security_Onion subnets`
   - **Destination**: **Kali Linux IP (192.168.10.2)**
   - **Description**: Allow traffic between Security Onion and Kali Linux 

4. **Allow Traffic to Non-Private IPv4 Addresses**
   - **Action**: Pass
   - **Interface**: `Secuity_Onion`
   - **Address Family**: `IPv4`
   - **Protocol**: `Any`
   - **Source**:  `Security_Onion subnets`
   - **Destination**: **Any non-private IP** (use an alias with Invert match to cover non-private IPs)
   - **Description**: Allow traffic to external (public) IP addresses

5. **Block All Remaining Traffic**
   - **Action**: Block
   - **Interface**: `Secuity_Onion`
   - **Address Family**: `IPv4/IPv6`
   - **Protocol**: `Any`
   - **Source**:  `Security_Onion subnets`
   - **Destination**: `Any`
   - **Description**: Block all other traffic 

6. **Apply Changes:**
   - After adding the rules, click on `Apply Changes` to ensure they take effect.

 ![pfsenseLogin](/assets/images/pfsense2/pf22.png)
_Summary of firewall rules on Security Onion Machine_ 

#### Active Directory Interface Rules

Similarly, define rules for the **Active Directory** subnet:

1. Block traffic to WAN.
2. Block traffic to the **TARGET_MACHINE** subnet.
3. Allow traffic to other subnets and the internet.

Save and apply changes.

 ![pfsenseLogin](/assets/images/pfsense2/pf23.png)
    _Summary of firewall rules on Active Directory Machine_ 
### Wrapping Up: Reboot pfSense

With all the rules in place, we’ll reboot pfSense to make sure everything sticks.

1. Navigate to **Diagnostics -> Reboot**.
2. Click **Submit** and once pfSense comes back online, log in to check everything is running smoothly.

### What’s Next?

In the next module, we'll be adding vulnerable machines to the **TARGET_MACHINE** subnet and testing connectivity from Kali. Follow along [here]({{ site.baseurl }}{% post_url 2024-09-06-metasploitable %}).

Let’s dive in and add some vulnerable VMs!

