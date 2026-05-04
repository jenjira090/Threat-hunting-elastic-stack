# VMware Lab Setup: Kali Linux, Windows Server 2019, and Ubuntu Elastic Server

## Project Purpose

This document explains how to set up three virtual machines in VMware Workstation Pro for a threat hunting lab using Elastic Stack.

The lab contains:

| Machine | Role | Operating System |
|---|---|---|
| Kali Linux | Attacker machine | Kali Linux |
| Windows Server 2019 | Target machine | Windows Server 2019 |
| Ubuntu Server | Security analytics platform | Ubuntu Server |

---

## Lab Architecture

```text
Kali Linux
   |
   | Attack simulation
   v
Windows Server 2019
   |
   | Telemetry collection
   v
Ubuntu Elastic Server
   |
   | Elasticsearch + Kibana + Elastic Security
   v
Threat Hunting / Detection
```

---

## 1. VMware Workstation Pro Setup

### 1.1 Install VMware Workstation Pro

1. Download VMware Workstation Pro.
2. Run the installer.
3. Keep the default installation options.
4. If Hyper-V is enabled, VMware may show a warning that nested virtualization is not available.
5. Continue with the installation if you need WSL2 or Docker on the Windows host.

---

## 2. VMware Network Configuration

The lab uses two VMware networks:

| Network | Purpose |
|---|---|
| NAT / VMnet8 | Internet access for updates and downloads |
| Host-only / VMnet1 | Private lab communication between VMs |

### 2.1 Open Virtual Network Editor

1. Open VMware Workstation Pro.
2. Go to `Edit`.
3. Click `Virtual Network Editor`.

### 2.2 Verify Networks

Make sure these networks exist:

- `VMnet8` configured as NAT
- `VMnet1` configured as Host-only

You do not need `VMnet0` for this lab.

---

## 3. Kali Linux VM Setup

### 3.1 Import or Create Kali VM

If using the prebuilt Kali VMware image:

1. Download the Kali VMware image.
2. Extract the ZIP file to a permanent location, for example:

```text
C:\VMs\Kali-Linux\
```

3. Open VMware Workstation Pro.
4. Click `Open a Virtual Machine`.
5. Select the Kali `.vmx` file.
6. Add the VM to VMware.

### 3.2 Recommended Kali Hardware Settings

| Setting | Value |
|---|---|
| RAM | 2 GB minimum, 4 GB recommended |
| CPU | 2 cores |
| Disk | Default from Kali image |
| Network Adapter 1 | NAT |
| Network Adapter 2 | Host-only |
| Display | Enable 3D acceleration if available |

### 3.3 Add Second Network Adapter

1. Power off the Kali VM.
2. Open `Edit virtual machine settings`.
3. Select `Network Adapter`.
4. Set it to `NAT`.
5. Click `Add`.
6. Select `Network Adapter`.
7. Set the second adapter to `Host-only`.
8. Click `OK`.

### 3.4 Start Kali and Verify Network

Start Kali and open a terminal.

Run:

```bash
ip a
```

Expected interfaces:

```text
lo
eth0
eth1
```

- `eth0` is usually NAT.
- `eth1` is usually Host-only.

Test internet access:

```bash
ping google.com
```

### 3.5 Update Kali

Run:

```bash
sudo apt update && sudo apt upgrade -y
```

If asked whether services should restart automatically during package upgrades, choose `Yes`.

### 3.6 Install VMware Integration Tools

Run:

```bash
sudo apt install open-vm-tools open-vm-tools-desktop -y
sudo reboot
```

### 3.7 Take Snapshot

After Kali is working:

1. Go to `VM`.
2. Select `Snapshot`.
3. Click `Take Snapshot`.
4. Name it:

```text
kali-network-ready
```

---

## 4. Windows Server 2019 VM Setup

### 4.1 Create Windows Server VM

1. Download the Windows Server 2019 ISO.
2. Open VMware Workstation Pro.
3. Click `Create a New Virtual Machine`.
4. Select `Typical`.
5. Choose the Windows Server ISO.
6. Select Windows Server as the guest OS.
7. Name the VM:

```text
Windows-Target
```

8. Choose a VM location, for example:

```text
C:\VMs\Windows-Target\
```

### 4.2 Recommended Windows Server Hardware Settings

| Setting | Value |
|---|---|
| RAM | 4 GB minimum, 6-8 GB recommended |
| CPU | 2 cores |
| Disk | 60 GB or more |
| Network Adapter 1 | NAT |
| Network Adapter 2 | Host-only |

### 4.3 Install Windows Server

1. Start the VM.
2. Choose Windows Server 2019 with Desktop Experience.
3. Complete the installation.
4. Set the Administrator password.
5. Log in after installation.

### 4.4 Configure Network

Make sure Windows Server has:

- One NAT adapter
- One Host-only adapter

Open Command Prompt and run:

```cmd
ipconfig
```

Confirm that Windows has IP addresses for the VMware networks.

### 4.5 Enable Remote Desktop

1. Open Server Manager.
2. Go to `Local Server`.
3. Find `Remote Desktop`.
4. Enable Remote Desktop.

### 4.6 Create Test Users

Open PowerShell or Command Prompt as Administrator:

```powershell
net user user1 Password123! /add
net user user2 Password123! /add
net user user3 Password123! /add
```

These users will be used later for password spraying simulation.

### 4.7 Take Snapshot

Take a snapshot and name it:

```text
windows-target-clean
```

---

## 5. Ubuntu Server VM Setup for Elastic Stack

### 5.1 Create Ubuntu Server VM

1. Download Ubuntu Server ISO.
2. Open VMware Workstation Pro.
3. Click `Create a New Virtual Machine`.
4. Select `Typical`.
5. Choose the Ubuntu Server ISO.
6. Select Linux / Ubuntu 64-bit.
7. Name the VM:

```text
Elastic-Server
```

8. Choose a VM location, for example:

```text
C:\VMs\Elastic-Server\
```

### 5.2 Recommended Ubuntu Hardware Settings

| Setting | Value |
|---|---|
| RAM | 6 GB minimum, 8 GB recommended |
| CPU | 2-4 cores |
| Disk | 80 GB or more |
| Network Adapter 1 | NAT |
| Network Adapter 2 | Host-only |

Elastic Stack needs more memory than Kali and Windows.

### 5.3 Install Ubuntu Server

1. Start the VM.
2. Follow the Ubuntu Server installer.
3. Create a username and password.
4. Enable OpenSSH if you want remote access.
5. Complete installation.
6. Reboot.

### 5.4 Update Ubuntu

Run:

```bash
sudo apt update && sudo apt upgrade -y
```

### 5.5 Verify Network

Run:

```bash
ip a
```

Confirm that the VM has:

- NAT IP address
- Host-only IP address

Test internet access:

```bash
ping google.com
```

### 5.6 Take Snapshot

Take a snapshot and name it:

```text
ubuntu-elastic-clean
```

---

## 6. Verify VM-to-VM Communication

After all three machines are running, verify that they can communicate through the Host-only network.

### 6.1 Find IP Addresses

On Kali and Ubuntu:

```bash
ip a
```

On Windows Server:

```cmd
ipconfig
```

Write down the Host-only IP address for each VM.

### 6.2 Test Ping

From Kali, ping Windows Server:

```bash
ping <windows-host-only-ip>
```

From Kali, ping Ubuntu Elastic Server:

```bash
ping <ubuntu-host-only-ip>
```

From Ubuntu, ping Windows Server:

```bash
ping <windows-host-only-ip>
```

If ping does not work, check Windows Firewall or network adapter settings.

---

## 7. Final VM Summary

| VM | Adapter 1 | Adapter 2 | Purpose |
|---|---|---|---|
| Kali Linux | NAT | Host-only | Attacker |
| Windows Server 2019 | NAT | Host-only | Target |
| Ubuntu Server | NAT | Host-only | Elastic Stack |

---

## 8. Project Status

Completed:

- VMware Workstation Pro installed
- Kali Linux VM configured
- Windows Server 2019 VM configured
- Ubuntu Server VM configured
- NAT and Host-only networking configured
- Initial snapshots created

Planned next steps:

- Install Elasticsearch
- Install Kibana
- Install Elastic Agent
- Install Sysmon on Windows Server
- Ingest Windows logs into Elastic
- Simulate password spraying from Kali
- Create detection rules in Kibana
- Build Python AI threat hunting agent

---

## 9. Notes

This lab is for educational and defensive security research only.

Do not run attack tools against systems you do not own or have explicit permission to test.

