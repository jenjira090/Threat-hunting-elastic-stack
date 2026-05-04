# Threat Hunting Lab with Elastic Stack

## Project Overview - status - developing

This project simulates a real-world **Security Operations Center (SOC)** environment using virtual machines. It demonstrates how to detect and investigate attacks such as **password spraying** using the Elastic Stack and an AI-driven threat hunting layer.

---

## Architecture

```
Attack Simulation Layer
│
├── Kali Linux (Attacker)
└── Windows Server 2019 (Target)

Telemetry Collection Layer
│
└── Elastic Agent + Sysmon

Security Analytics Platform
│
└── Elastic Stack
   ├── Elasticsearch
   ├── Kibana
   └── Elastic Security

AI Threat Hunting Layer (Planned)
│
└── Python AI Agent
   ├── Elasticsearch API queries
   ├── Detection logic
   ├── LLM-based reasoning
   └── Report generation
```

---

## Lab Environment

All components are deployed using:

* VMware Workstation Pro

### Virtual Machines

| VM Name        | OS                  | Role                |
| -------------- | ------------------- | ------------------- |
| Kali-Attacker  | Kali Linux          | Attack simulation   |
| Windows-Target | Windows Server 2019 | Target system       |
| Elastic-Server | Ubuntu Server       | Logging & analytics |

---

## Network Configuration

Each VM is configured with two network adapters:

* **NAT** → Internet access
* **Host-only (VMnet1)** → Internal lab communication

### Purpose:

* Enables controlled attack simulation
* Isolates lab traffic from external networks

---

## Initial Setup

### 1. Kali Linux (Attacker)

* Prebuilt VM imported
* Updated system packages:

```bash
sudo apt update && sudo apt upgrade -y
```

* Network interfaces verified:

```bash
ip a
```

---

### 2. Windows Server 2019 (Target)

* Installed with Desktop Experience
* Remote Desktop enabled
* Test users created for attack simulation:

```powershell
net user user1 Password123! /add
net user user2 Password123! /add
net user user3 Password123! /add
```

---

### 3. Ubuntu Server (Elastic Stack)

* Installed as monitoring server
* Planned components:

  * Elasticsearch
  * Kibana
  * Elastic Agent

---

## Telemetry Collection (Planned)

* Install Sysmon on Windows
* Install Elastic Agent
* Collect:

  * Logon events (Event ID 4624, 4625)
  * Process creation
  * Network activity

---

## Attack Simulation (Planned)

From Kali Linux:

* Password spraying using tools like:

  * hydra
  * crackmapexec

Goal:

* Generate authentication failure logs
* Simulate real attack patterns

---

## Detection Strategy (Planned)

Using Kibana:

* Detect multiple failed logins
* Identify single source IP targeting multiple accounts
* Correlate authentication events

---

## AI Threat Hunting Layer (Planned)

A Python-based agent will:

* Query Elasticsearch
* Identify suspicious patterns
* Use LLM reasoning to classify attacks
* Generate investigation reports

---

## Snapshots

Snapshots are used to maintain clean states:

* `kali-network-ready`
* `kali-updated-clean`

---

## Next Steps

* [ ] Install Elastic Stack
* [ ] Configure Sysmon on Windows
* [ ] Ingest logs into Elasticsearch
* [ ] Create detection rules in Kibana
* [ ] Simulate password spraying attacks
* [ ] Build Python AI threat hunting agent

---

## ⚠️ Notes

* This lab is isolated and intended for educational purposes only
* Do not run attack tools outside a controlled environment

---

## Learning Objectives

* Understand SOC architecture
* Practice threat detection using Elastic Stack
* Simulate real-world attacks safely
* Build AI-assisted threat hunting workflows

---

## Author

Jenjira Punkalong

---

## License

This project is for educational use.

