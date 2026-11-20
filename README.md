# Home SOC Lab: Detection, Triage and Reporting

A small, isolated home lab for practising security operations (SOC) work: collecting and monitoring logs in a SIEM, detecting simulated attacks, triaging alerts, and writing incident reports.

**Status: in progress.** The SIEM is built and running. Network detection (Suricata) and the attack scenarios are next. This README is updated as each stage is completed.

## Goals
- Run a SIEM and an IDS on a Linux server I built and configured myself
- Generate realistic alerts by attacking the lab from Kali Linux
- Triage each alert and write an incident report (timeline, evidence, indicators, MITRE ATT&CK mapping, escalation decision)
- Write short SOPs/playbooks and a shift handover note

## Architecture

```
Windows host (VMware Workstation)
|
+-- VMnet1  host-only  192.168.153.0/24   <- all lab traffic, isolated from the internet
|     +-- Kali Linux          (attacker)
|     +-- Metasploitable2     (vulnerable target, host-only ONLY)
|     +-- wazuh-lab           (Ubuntu Server: Wazuh SIEM, Suricata planned)
|
+-- VMnet8  NAT        192.168.109.0/24   <- package downloads only (Kali, wazuh-lab)
```

## Lab environment

| VM | Role | Resources | Networks |
| --- | --- | --- | --- |
| wazuh-lab (Ubuntu Server 26.04.1 LTS) | SIEM / EDR server, IDS sensor | 8 GB RAM, 4 vCPU, 60 GB disk | Host-only + NAT |
| Kali Linux 2026.2 | Attacker | 4 GB RAM, 4 vCPU | Host-only + NAT |
| Metasploitable2 | Intentionally vulnerable target | default | Host-only only |

**Isolation:** the vulnerable machine is never connected to NAT, bridged, or any real network. Attacks happen only on the host-only network.

## Progress

| Stage | Status |
| --- | --- |
| Virtual network plan (host-only + NAT) | Done |
| Ubuntu Server VM built from ISO | Done |
| Wazuh 4.14 all-in-one install (indexer, manager, dashboard) | Done |
| Suricata IDS installed and sent to Wazuh | Planned (next) |
| Scenario 1: Nmap scan detection | Planned |
| Scenario 2: SSH brute force detection | Planned |
| Scenario 3: Suspicious Windows activity (Sysmon) | Optional |
| Scenario 4: File integrity monitoring | Planned |
| Scenario 5: Phishing email analysis | Planned |
| Monitoring health check (stop a sensor, detect the gap) | Planned |
| Playbooks and shift handover note | Planned |

## Repository layout

```
docs/          build log and notes
scenarios/     one folder per scenario, each with an incident report
playbooks/     SOP-style triage playbooks
config/        custom rules and configuration files
screenshots/   evidence images
```

Start with [docs/build-log.md](docs/build-log.md) to see how the lab was built, including the problems I hit and how I fixed them.

## Safety notes
- Everything runs in an isolated lab for learning purposes.
- No credentials, certificates, or generated key archives are committed to this repo.
