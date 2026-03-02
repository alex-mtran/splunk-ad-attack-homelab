# Splunk Active Directory Attack Homelab (IN PROGRESS)
Home lab for practicing Active Directory security monitoring and attack detection using Splunk SIEM. Built with Oracle VirtualBox VMs for log ingestion and analysis. The server versions are the latest as of the creation of this project.

This project demonstrates:
- Active Directory security monitoring and log analysis
- Attack simulation using common techniques and tools
- SIEM (Splunk) configuration for Windows event log ingestion
- Threat detection and incident response workflows

---

## Table of Contents 
1. [Initial Setup](#initial-setup-anchor-point)
   * [Windows 10](#windows-10-anchor-point)
   * [Kali Linux](#kali-linux-anchor-point)
   * [Active Directory Domain Controller](#active-directory-domain-controller-anchor-point)
   * [Splunk Server](#splunk-server-anchor-point)

---

<a name="initial-setup-anchor-point"></a>
## Initial Setup

### Lab Architecture

Host Machine: Windows 11 Home <br> 
Virtual Machine Manager: Oracle VirtualBox <br> 
Network: Internal network (192.168.50.0/24)

| Machine | OS | IP Address | Role |
|---------|-----|------------|------|
| Splunk Server | Ubuntu Server | 192.168.100.10 | SIEM / SOAR |
| Active Directory | Windows Server 2022 | 192.168.100.7 | Domain Controller |
| Target-PC | Windows 10 | 192.168.100.3 | Target machine |
| Attacker | Kali Linux | 192.168.100.200 | Attack machine |

<img width="681" height="656" alt="Project architecture" src="https://github.com/user-attachments/assets/65f4f6e6-de98-41de-9f42-ade3008f3797" />

### Oracle VirtualBox

Oracle VirtualBox is a free, open-source hypervisor that allows for running multiple operating systems simultaneously on a single host machine. It will serve as the foundation for this entire lab environment.

1. <a href="https://www.virtualbox.org/wiki/Downloads" target="_blank">Download Oracle VirtualBox</a>.
2. Run the installer and follow the installation wizard with default settings.
3. Verify installation by opening VirtualBox Manager.

#

<a name="windows-10-anchor-point"></a>
### Windows 10

The Windows 10 machine will serve as our target endpoint representing a typical user workstation in an enterprise environment. This machine will be domain-joined, monitored by Sysmon, and forward logs to Splunk.

#### Download Windows 10

1. <a href="https://www.microsoft.com/en-ca/software-download/windows10" target="_blank">Download Windows 10 ISO</a>.
2. Click **Download Now** to download the Media Creation Tool.
3. Run the tool and select **Create installation media (USB flash drive, DVD, or ISO file)**.
4. Choose **ISO file** and save it to a known location.

#### Create Virtual Machine

1. Open **Oracle VirtualBox Manager**.
2. Click **New** to create a new virtual machine.
3. Fill in:

   **Virtual Name and OS**
   
   <img width="785" alt="VM summary before creation" src="https://github.com/user-attachments/assets/2d3701a7-7148-4256-b490-6252465b382d" />

   **Virtual Hardware**

   <img width="783" alt="VM hardware settings" src="https://github.com/user-attachments/assets/fde5a99c-0519-4b33-89f6-a60b7575e063" />

   **Virtual Hard Disk**

   <img width="781" alt="VM virtual hard disk settings" src="https://github.com/user-attachments/assets/5ecfd760-858b-4d9c-a273-1f3dcb4ed6af" />

4. Click **Finish**

#### Install Windows 10

1. In Oracle VirtualBox Manager, select and start the Windows 10 VM.
2. The Windows installation screen will begin automatically. Configure Windows setup:

  <img width="957" height="857" alt="Windows setup" src="https://github.com/user-attachments/assets/eebf25d0-4bcd-4baa-a7c9-c167947ae030" />

  * Click **Next** on the language/region selection screen
  * Click **Install now**
  * Select **I don't have a product key**
  * Select **Windows 10 Pro** (required for domain joining)
  * Check **I accept the license terms** and click **Next**
  * Select **Custom: Install Windows only (advanced)**
  * Select the unallocated disk space and click **Next**

Windows will now install. This process takes 10-15 minutes and will reboot several times automatically. Once installation completes, Windows will boot to the desktop with your configured username. Your Windows 10 virtual machine is now up and running.

#### Configure Windows 10

1. Select **Set up for personal use** and Click **Next**.
2. Click **Offline Account**.
3. Click **Limited Experience**.
4. Enter username and password.
5. Click **Not now** or **Skip** for optional features until reaching the Windows homescreen.

#### NAT Network

The NAT Network adapter enables internet connectivity and inter-VM communication. This network adapter must be enabled on all of the machines in the network.

1. In **Oracle VirtualBox Manager**, navigate to **File > Tools > Network**.

<img width="415" height="741" alt="Path to network tab" src="https://github.com/user-attachments/assets/bdc5f6d8-c06e-4691-93b6-c03a6ea86226" />

2. Select **NAT Networks > Create**.
   * Name the network to **SplunkNetwork**.
   * Set IPv4 range to **192.168.100.0/24**.
   * Click **Apply**.

<img width="924" height="746" alt="Create NAT Network" src="https://github.com/user-attachments/assets/f3ad2f4f-4cbf-4d7c-be0c-44e785826810" />

3. Open **Settings** for the Windows 10 VM.

<img width="927" height="740" alt="Open Windows 10 settings" src="https://github.com/user-attachments/assets/692c4133-7eb4-41d9-9a52-c26d4e1f8e14" />

4. Navigate to **Network** and set **Attached to** to **NAT Network**.
5. Select **SplunkNetwork** from the dropdown.

<img width="773" height="510" alt="Enable NAT Network for Windows 10 machine" src="https://github.com/user-attachments/assets/72eeac86-8061-4e4d-a396-4c11d0b87b14" />

6. Click **OK**

> **Note:** NAT Network allows multiple VMs to share the host's IP address for internet access while enabling VM-to-VM communication. This differs from a standard NAT adapter, which isolates VMs from each other. <a href="https://www.youtube.com/watch?v=Fhdxk4bmJCs" target="_blank">Learn more about VirtualBox network adapters</a>.

#### Sysmon

Sysmon (System Monitor) provides detailed, persistent telemetry (or logging) of system activity to the Windows event log via detecting malicious activity, monitoring unauthorized changes to critical files, and analyzing network connections or process creation. 

#### Atomic Red Team

Atomic red team is a PowerShell-based execution framework built around the MITRE ATT&CK framework. It provides a library of simple tests that generate real, detectable malicious telemetry on the target machine.

#

<a name="kali-linux-anchor-point"></a>
### Kali Linux

The Kali Linux machine will serve as our attacker, representing a threat actor within the security environment. This machine will generate malicious telemetry to the Active Directory domain and target Windows machine that we can detect and analyze in Splunk.

#### Download Kali Linux

1. Download the Kali Linux virtual machine image from the <a href="https://www.kali.org/get-kali/#kali-virtual-machines" target="_blank">official Kali Linux website</a>.

<img width="966" height="678" alt="Kali Linux Download Page" src="https://github.com/user-attachments/assets/d1d13ae5-8b60-4b9a-a53b-a37b2de5dc8b" />

> **Note:** The default credentials for this machine are `kali`/`kali`.

2. Extract the downloaded archive using <a href="https://www.7-zip.org/" target="_blank">7-zip</a>.
<img width="587" height="387" alt="Extract Kali Linux folder with 7-zip" src="https://github.com/user-attachments/assets/a56655e5-a136-48fa-9f8c-38944a5fa281" />

3. Open Kali Linux `.vbox` file using **Oracle VirtualBox** to import the machine.

<img width="840" height="621" alt="Open Kali Linux .vbox file" src="https://github.com/user-attachments/assets/e11fa948-8552-46d7-aa44-44b1c0f5354d" />

<img width="919" height="740" alt="Oracle VirtualBox" src="https://github.com/user-attachments/assets/70c5ae93-babe-4e0c-9538-c388c3ab055b" />

4. Once imported, open **Settings** for the Kali Linux VM and navigate to the **Network** tab.
5. Set **Attached to** to **NAT Network**, then select the **SplunkNetwork** from the dropdown.
6. Click **OK**.

<img width="773" height="514" alt="Enable NAT Network for Kali Linux machine" src="https://github.com/user-attachments/assets/33801a75-1b8a-44b1-a593-fe44a8bc9eda" />

#### Updating Login Information

Kali Linux does not allow changing hte username of the default user through standard methods. To rename the account, we must first switch to the root user.

1. Start the Kali Linux VM and sign in with the default credentials `kali`/`kali`.
2. Open a terminal using **Ctrl + Alt + T** or click the Terminal icon in the top bar.

<img width="1278" height="883" alt="Open terminal" src="https://github.com/user-attachments/assets/ecec3baa-08a6-46ea-9ca1-f5c08b1087e8" />

> **Note:** Kali Linux won't let you change the username of the default user account. We will switch to this **root** user to adjust the username of the root user.

3. Switch to the root user and update the root password.

<img width="1056" height="883" alt="Terminal sign into root user" src="https://github.com/user-attachments/assets/9aaaadd8-ca80-4671-aeaf-8b84e4a81db3" />

4. Power off the machine and sign back in as **root**.
5. Follow <a href="https://www.hexzilla.com/p/change-username-hostname-on-kali-linux-2025-update" target="_blank">this guide</a> to rename the default username.

<img width="1078" height="956" alt="Change username" src="https://github.com/user-attachments/assets/0b74029b-49d5-47cf-9ea4-5d8e08c90190" />

6. Power off the machine and sign back in as **attacker**.
7. Set a new password.

<img width="1077" height="945" alt="Change password" src="https://github.com/user-attachments/assets/dc7f90b9-9b40-4c13-95a9-e3212ddc0b78" />

#### Configuring a Static IP address

1. Open a terminal and run the following commands to note your current gateway, IP address, and subnet mask:

```bash
ip r
ifconfig
```

<img width="1078" height="949" alt="IP configuration" src="https://github.com/user-attachments/assets/a214ae06-953d-4125-8600-634c20965cad" />

2. Right-click the network icon in the taskbar and select **Edit Connections...**
3. Select **Wired connection 1** and click the settings icon to open its configuration.

<img width="1079" height="949" alt="Path visual to wired connection settings" src="https://github.com/user-attachments/assets/50e1f032-c788-4204-aee2-d242471def12" />

4. Navigate to the **IPv4 Settings** tab and change the **Method** from **Automatic (DHCP)** to **Manual**.
5. Click **Add** and enter the following values:

|  Field  |  Value  |
|---------|-----|
|  Address  |  `192.168.100.200`  |
|  Netmask  |  `255.255.255.0`  |
|  Gateway  |  `192.168.100.1`  |

6. Click **Save**.

<img width="1079" height="948" alt="Add static IP address" src="https://github.com/user-attachments/assets/f1d09dda-6a0e-4986-9a9f-7134083bef92" />

7. Reboot machine and verify the new IP address.

<img width="1078" height="947" alt="Verify changed IP address" src="https://github.com/user-attachments/assets/6a1d3609-618c-476f-87d1-9abaa2952215" />

#

<a href="active-directory-domain-controller-anchor-point"></a>
### Active Directory Domain Controller

The Active Directory Domain Controller (ADDC) is both the infrastructure backbone and the system we're monitoring for signs of compromise. The DC handles standard administrative functions such as authentication requests, group policies, and replication while also generating the logs that Splunk collects to enable attack detection and analysis. We will deploy the Active Directory Domain Services on a Windows Server 2022.

#### Download Windows Server 2022

1. Download the Windows Server image from the <a href="https://info.microsoft.com/ww-landing-windows-server-2022.html" target="_blank">official Windows Server website</a>.
2. Open **Oracle VirtualBox** and click **New**
3. Fill in:

   **Virtual Name and OS**
   
   <img width="781" height="557" alt="Windows Server 2022 Virtual Name and OS" src="https://github.com/user-attachments/assets/b215a0a5-66a5-4f8b-973a-79e473da4bbb" />

   **Virtual Hardware**

   <img width="779" height="556" alt="Windows Server 2022 Virtual Hardware" src="https://github.com/user-attachments/assets/e80c3ef1-11c6-4cf8-b648-fbdc5fda36b1" />

4. Leave the rest as default and click **Finish**

#### Configuring Windows Server 2022
1. Start the **ADDC01** server.
2. Click **Next** and **Install Now**.
3. Select **Windows Server 2022 Standard Evaluation (Desktop Experience)** for the operating system.

<img width="1027" height="849" alt="Windows Server 2022 Operating System" src="https://github.com/user-attachments/assets/e87baee5-563a-4c9d-b78d-2d4460e22d7b" />

4. Accept the terms and agreements and click **Next**.
5. Select **Custom: Install Microsoft Server Operating System only (advanced)** and click **Next**.

<img width="1030" height="849" alt="Windows Server 2022 Installation Type" src="https://github.com/user-attachments/assets/689497c2-0421-44f6-ad67-f54c4cb1764c" />

6. Create a password and click **Finish**.
> **Note:** To unlock via **Ctrl + Alt + Delete** on a virtual machine, click the **Input** button at the top bar and under the **Keyboard** dropbar there is a button for that key command.
> 
> <img width="1021" height="850" alt="Ctrl + Alt + Delete on virtual machine" src="https://github.com/user-attachments/assets/a56dab30-e737-4551-a589-9a6ffac40953" />

#### Installing Active Directory Domain Services



#

<a href="splunk-server-anchor-point"></a>
### Splunk Server

The Splunk Server acts as the central log aggregation and analysis platform for this lab environment. It ingests and indexes Windows Event Logs forwarded from the Domain Controller and other endpoints via the Splunk Universal Forwarder for us to search, correlate, and visualize security events. We will deploy the Splunk Server on an Ubuntu Server 24.04. 

#### Download Ubuntu Server 24.04.4 LTS

1. Download the Ubuntu Server 24.04.4 LTS image from the <a href="https://ubuntu.com/download/server" target="_blank">official Ubuntu server website</a>.
2. Open **Oracle VirtualBox** and click **New**
3. Fill in:

   **Virtual Name and OS**
   
   <img width="782" height="555" alt="Ubuntu Server 24.04.4 LTS virtual name and os" src="https://github.com/user-attachments/assets/41f9269a-f475-42c0-a6c4-2c99c11f69e2" />

   **Virtual Hardware**

   <img width="783" height="559" alt="Ubuntu Server 24.04.4 LTS virtual hardware" src="https://github.com/user-attachments/assets/4f277242-0e8a-4ee3-aa78-df206dc59d69" />

   **Virtual Hard Disk**

   <img width="783" height="559" alt="Ubuntu Server 24.04.4 LTS virtual hard disk" src="https://github.com/user-attachments/assets/4c687748-cc28-4efb-bc95-ec73bdf87603" />

4. Leave the rest as default and click **Finish**

#### Configuring Ubuntu Server 24.04.4 LTS
1. Start the **Splunk** server.
2. Select **Try or Install Ubuntu Server** and hit **Enter** on the keyboard.
3. Keep the defaults and continue until reaching the profile setup page.

<img width="1155" height="918" alt="Splunk server profile page" src="https://github.com/user-attachments/assets/ed5dee9b-2188-47b4-ae8d-a01b0ab26457" />

4. Fill out the profile page.
5. Keep defaults and reboot the machine by clicking **Reboot Now**.
6. Press **Enter** on the keyboard when prompted with the errors to continue with reboot.
7. Sign in after reboot finishes.
> **Note:** Inputting keys into the password field will not output any visible characters to the screen. This is intentional. Fill out the password regardless.

<img width="1276" height="883" alt="Sign into splunk server" src="https://github.com/user-attachments/assets/cad71b39-db0b-4966-9732-72d72fb652df" />

8. Update and upgrade repository.

<img width="992" height="879" alt="Splunk update and upgrade repository commands" src="https://github.com/user-attachments/assets/d8b0127c-36dc-47fe-976d-1765d8465116" />

9. Press **Enter** on the keyboard when the terminal finishes scanning for updates.

> **Note:** Don't forget to add/change network adapter to NAT Network on all machines (Splunk, Windows 10 Splunk Forwarder, ADDC01)!

