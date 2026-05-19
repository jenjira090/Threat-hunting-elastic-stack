# Threat Hunting Lab - Network Setup Guide

This document explains the complete network setup for a 3-VM threat hunting lab using VMware Workstation.

The lab environment consists of:

```
10.0.5.4 → Ubuntu Server (Elastic Stack)
10.0.5.6 → Kali Linux (Attacker)
10.0.5.7 → Windows Server (Target)
```

The setup uses a VMware Host-only network (VMnet1) so all virtual machines can communicate internally.

## 1. VMware Network Setup ##

### 1.1 Open Virtual Network Editor ###

Open VMware Workstation.

Go to:

```
Edit → Virtual Network Editor
```
Click:

```
Change Settings
```

to run with Administrator privileges.

### 1.2 Configure VMnet1 ###

Select:

```
VMnet1
``` 

Set:

```
Network Type: Host-only
Subnet IP:    10.0.5.0
Subnet Mask:  255.255.255.0
```

Enable:

```
✔ Connect a host virtual adapter to this network
```

Disable:

```
✘ Use local DHCP service to distribute IP addresses
```

The final configuration should look similar to:

```
VMnet1
Type: Host-only
Subnet IP: 10.0.5.0
Subnet Mask: 255.255.255.0
DHCP: Disabled
Host adapter: Connected
```
## 2. VMware VM Network Adapter Configuration ##

Each VM should use:
```
Network Adapter 1 → VMnet1 (Host-only)
Network Adapter 2 → NAT (Optional Internet Access)
```
### 2.1 Configure Adapter 1 ###

For EACH VM:

```
VM → Settings → Network Adapter
```

Set:

```
✔ Connected
✔ Connect at power on
Network connection: Custom
Specific virtual network: VMnet1
```

### 2.2 Configure Adapter 2 (Optional) ###

Adapter 2 provides internet access for package installation and updates.

Set:

```
Network connection: NAT
```

This adapter is optional but recommended.

## 3. Ubuntu Server Network Configuration ##

Ubuntu Server is used for Elastic Stack.

Target IP:

```
10.0.5.4
```

### 3.1 Verify Interfaces ###

Run:

``` Bash
ip a
```

Example result:

```
ens33 → VMnet1 interface
ens37 → unused or NAT interface
```
In this lab:

```
ens33 = VMnet1
```

### 3.2 Configure Static IP ###

Edit Netplan configuration:

```
sudo nano /etc/netplan/01-netcfg.yaml
```

If the file does not exist:

``` Bash
ls /etc/netplan/
```

Some Ubuntu versions may use:

```
00-installer-config.yaml
```

### 3.3 Netplan Configuration ###

Example configuration:

```
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: no
      addresses:
        - 10.0.5.4/24
      routes:
        - to: default
          via: 10.0.5.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
```

### 3.4 Save File ###

Inside nano:

```
Ctrl + O
Enter
Ctrl + X
```

### 3.5 Fix Permissions ###

Run:

``` Bash
sudo chmod 600 /etc/netplan/01-netcfg.yaml
``` 

This removes the warning:

```
Permissions for /etc/netplan/01-netcfg.yaml are too open
```

### 3.6 Apply Netplan ###

Run:

``` Bash
sudo netplan try
``` 

Press:

```
Enter
```

Then:

``` Bash
sudo netplan apply
```

Possible warning:

```
Cannot call Open vSwitch: ovsdb-server.service is not running
```

This warning is harmless and can be ignored.


### 3.7 Verify Ubuntu IP ###

Run:

``` Bash
ip a
```

Expected:
```
ens33 → 10.0.5.4
```

## 4. Kali Linux Network Configuration ##

Kali Linux is used as the attacker machine.

Target IP:
```
10.0.5.6
```

### 4.1 Verify Interfaces

Run:

``` Bash
ip a
```

Example:

```
eth0 → VMnet1 interface
eth1 → NAT interface
```
In this lab:

```
eth0 = VMnet1
eth1 = NAT
```

### 4.2 Open Network Manager ###

Run:

``` Bash
sudo nmtui
```

Select:
```
Edit a connection
```

### 4.3 Configure eth0 ###

Select the interface:
```
eth0
```

Set:
```
IPv4 Configuration: Manual
```

Then configure:
```
Address: 10.0.5.6/24
Gateway: 10.0.5.1
DNS:     8.8.8.8
```

Save:
```
OK → Back → Quit
```

### 4.4 Restart Networking ###

Run:

``` Bash
sudo systemctl restart NetworkManager
```

### 4.5 Verify Kali IP ###

Run:
``` Bash
ip a
``` 

Expected:
```
eth0 → 10.0.5.6
eth1 → 192.168.x.x (NAT)
```

## 5. Windows Server Network Configuration ##

Windows Server is used as the target machine.

Target IP:
```
10.0.5.7
```

### 5.1 Open Network Connections ###

Press:

```
Win + R
```

Run:

```
ncpa.cpl
```

### 5.2 Configure IPv4 ###

Find the adapter connected to VMnet1.

Open:
```
Properties → Internet Protocol Version 4 (TCP/IPv4)
```
 
Configure:
```
IP address:      10.0.5.7
Subnet mask:     255.255.255.0
Default gateway: 10.0.5.1
Preferred DNS:   8.8.8.8
```

Click:
```
OK → OK
```

### 5.3 Verify Windows IP ###

Open PowerShell:
```
ipconfig
```

Expected:
```
IPv4 Address: 10.0.5.7
```

## 6. Enable Ping on Windows Server ##

Windows blocks ICMP (ping) by default.

Open PowerShell as Administrator:

```
New-NetFirewallRule -DisplayName "Allow ICMPv4 Ping" -Protocol ICMPv4 -IcmpType 8 -Action Allow
```

## 7. Connectivity Testing ##

After configuring all three VMs:

### 7.1 Ubuntu Tests ###

Run:

``` Bash
ping 10.0.5.6
ping 10.0.5.7
```

### 7.2 Kali Tests ###

Run:

``` Bash
ping 10.0.5.4
ping 10.0.5.7
```

### 7.3 Windows Tests ###

Run:

``` PowerShell
ping 10.0.5.4
ping 10.0.5.6

```
### 7.4 Successful Final State

All machines should successfully communicate.

Final topology:

VMnet1 (Host-only)
      |
------------------------------------------------
|                     |                        |
10.0.5.4              10.0.5.6                10.0.5.7
Ubuntu Server         Kali Linux              Windows Server
Elastic Stack         Attacker                Target

