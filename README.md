# Splunk Active Directory Attack Homelab (IN PROGRESS)
Home lab for practicing Active Directory security monitoring and attack detection using Splunk SIEM. Built with Oracle VirtualBox VMs for log ingestion and analysis.

This project demonstrates:
- Active Directory security monitoring and log analysis
- Attack simulation using common techniques and tools
- SIEM (Splunk) configuration for Windows event log ingestion
- Threat detection and incident response workflows

---

## Table of Contents 
1. [Initial Setup](#initial-setup-anchor-point)

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

<img width="681" height="656" alt="image" src="https://github.com/user-attachments/assets/65f4f6e6-de98-41de-9f42-ade3008f3797" />

### Oracle VirtualBox

Oracle VirtualBox is a free, open-source hypervisor that allows for running multiple operating systems simultaneously on a single host machine. It will serve as the foundation for this entire lab environment.

1. <a href="https://www.virtualbox.org/wiki/Downloads" target="_blank">Download Oracle VirtualBox</a>
2. Run the installer and follow the installation wizard with default settings
3. Verify installation by opening VirtualBox Manager

### Windows 10 Target Machine

#### Download Windows 10 ISO

The Windows 10 machine will serve as our target endpoint representing a typical user workstation in an enterprise environment. This machine will be domain-joined, monitored by Sysmon, and forward logs to Splunk.

1. <a href="https://www.microsoft.com/en-ca/software-download/windows10" target="_blank">Download Windows 10 ISO</a>
2. Click **Download Now** to download the Media Creation Tool
3. Run the tool and select **Create installation media (USB flash drive, DVD, or ISO file)**
4. Choose **ISO file** and save it to a known location

#### Create Virtual Machine

1. Open **Oracle VirtualBox Manager**
2. Click **New** to create a new virtual machine
3. Fill in:

   **Virtual Name and OS**
   
   <img width="785" alt="VM summary before creation" src="https://github.com/user-attachments/assets/2d3701a7-7148-4256-b490-6252465b382d" />

   **Virtual Hardware**

   <img width="783" alt="VM hardware settings" src="https://github.com/user-attachments/assets/fde5a99c-0519-4b33-89f6-a60b7575e063" />

   **Virtual Hard Disk**

   <img width="781" alt="VM virtual hard disk settings" src="https://github.com/user-attachments/assets/5ecfd760-858b-4d9c-a273-1f3dcb4ed6af" />

4. Click **Finish**

#### Install Windows 10

1. In Oracle VirtualBox Manager, select and start **Windows 10 Splunk Forwarder** virtual machine
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

1. Select **Set up for personal use** and Click **Next**
2. Click **Offline Account**
3. Click **Limited Experience**
4. Enter username and password
5. Click **Not now** or **Skip** for optional features until reaching the Windows homescreen

#### NAT Network

The NAT Network adapter enables internet connectivity and inter-VM communication.

1. In **Oracle VirtualBox Manager**, navigate to **File > Tools > Network**

<img width="415" height="741" alt="image" src="https://github.com/user-attachments/assets/bdc5f6d8-c06e-4691-93b6-c03a6ea86226" />

2. Select **NAT Networks > Create**
   * Name the network to **SplunkNetwork**
   * Set IPv4 range to **192.168.100.0/24**
   * Click **Apply**

<img width="924" height="746" alt="image" src="https://github.com/user-attachments/assets/f3ad2f4f-4cbf-4d7c-be0c-44e785826810" />

3. Open settings for the **Windows 10 Splunk Forwarder** VM

<img width="927" height="740" alt="image" src="https://github.com/user-attachments/assets/692c4133-7eb4-41d9-9a52-c26d4e1f8e14" />

4. Navigate to **Network** and set **Attached to: NAT Network**
5. Select **SplunkNetwork**

<img width="773" height="510" alt="image" src="https://github.com/user-attachments/assets/72eeac86-8061-4e4d-a396-4c11d0b87b14" />

6. Click **OK**

> **Note:** NAT Network allows multiple VMs to share the host's IP address for internet access while enabling VM-to-VM communication. This differs from a standard NAT adapter, which isolates VMs from each other. <a href="https://www.youtube.com/watch?v=Fhdxk4bmJCs" target="_blank">Learn more about VirtualBox network adapters</a> 

#### Sysmon

Sysmon (System Monitor) provides detailed, persistent telemetry (or logging) of system activity to the Windows event log via detecting malicious activity, monitoring unauthorized changes to critical files, and analyzing network connections or process creation. 

#### Atomic Red Team

Atomic red team is a PowerShell-based execution framework built around the MITRE ATT&CK framework. It provides a library of simple tests that generate real, detectable malicious telemetry on the target machine.
