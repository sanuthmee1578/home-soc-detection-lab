# Build Log

A running record of how the lab was built, in order, including what went wrong. Newest work goes at the bottom.

## Stage 1: Network and VM planning (19 Nov 2026)

**Goal:** an isolated lab network, plus a separate network for internet access during installs.

- VMware Workstation has two networks I use:
  - **VMnet1, host-only, 192.168.153.0/24**: all lab traffic.
  - **VMnet8, NAT, 192.168.109.0/24**: downloads and updates only.
- Existing VMs from earlier labs: Kali Linux and Metasploitable2. Metasploitable2 is host-only only, because it is deliberately vulnerable.
- Decision: build **one new Ubuntu Server VM** to hold the SIEM and, later, the IDS. Kali attacks it.

Screenshot: `screenshots/01-vmware-virtual-network-editor.png`

## Stage 2: Ubuntu Server VM

**Specs:** Ubuntu Server 26.04.1 LTS (installed from the official ISO), 8 GB RAM, 4 vCPUs, 60 GB disk, two network adapters (host-only and NAT). Hostname `wazuh-lab`. OpenSSH server enabled so I can manage it from Windows PowerShell.

**Steps**
1. Created the VM in VMware from the ISO and stored it on the D: drive.
2. Ran the text installer: standard server install, default networking (DHCP on both adapters), custom storage size, OpenSSH server, no extra snaps.
3. Updated the system (`apt update && apt upgrade`).
4. Took a clean snapshot.

**Problems and fixes**
- **VM stored in the wrong place.** The wizard defaulted to a OneDrive folder, which would try to sync a huge, constantly changing disk file. I moved the VM files to `D:\cyber\siem lab`, removed the old library entry and reopened the `.vmx`, choosing "I moved it".
- **Installer mirror step.** I cleared the mirror address while editing it, then pressed Ctrl+Z, which opened a side shell inside the installer. I typed `exit` to return and re-entered the standard Ubuntu mirror URL. Lesson: avoid Ctrl+Z in the installer.
- **Only half the disk was used.** The default storage layout gave the root volume about 29 GB of a 60 GB disk. I extended `ubuntu-lv` to the full free space, because Wazuh's indexer stores a lot of data. Root is now about 56.7 GB.
- **DHCP addresses change.** The host-only address moved from `192.168.153.130` to `192.168.153.131` after a power cycle. I re-check with `ip -4 a` each time I start the VM.

Screenshots: `screenshots/02-siem-lab-vm-settings-and-snapshot.png`, `screenshots/03-ssh-login-and-memory.png`

## Stage 3: Wazuh (SIEM/EDR) install

**Version:** Wazuh 4.14.10, all-in-one deployment (indexer, manager, dashboard and Filebeat on one host), using the official installation assistant from the Wazuh Quickstart:

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

**Notes**
- The installer warned that Ubuntu 26.04 is not on Wazuh's list of recommended systems, and that installation "may not work properly". It passed the hardware check, continued, and finished without errors. Install time was about 15 minutes.
- Verified all services are running:
  ```bash
  sudo systemctl is-active wazuh-indexer wazuh-manager wazuh-dashboard filebeat
  ```
  Result: four lines of `active`.
- Logged in to the dashboard from the Windows host at `https://<server host-only IP>` (browser certificate warning is expected for a lab with a self-signed certificate).
- The dashboard shows "no agents registered". That is expected, because the server monitors itself and Suricata will run on the same machine. In the last 24 hours it already showed 46 medium and 26 low severity alerts from the server's own events.
- Admin credentials were saved to a password manager. They are not in this repo.
- Took a VMware snapshot named **wazuh-installed** as a rollback point before adding Suricata.

Screenshots: `screenshots/04-wazuh-install-complete.png` (credentials redacted), `screenshots/05-wazuh-dashboard.png`

## Next: Stage 4, Suricata and the first scenario

Planned steps:
1. Install Suricata, set `HOME_NET` to the host-only network and the capture interface to `ens33`, update rules, and enable the service.
2. Configure the Wazuh manager to read `/var/log/suricata/eve.json`.
3. From Kali, run an Nmap scan against the server and confirm the alert appears in Wazuh under `rule.groups:suricata`.
4. Write the first incident report in `scenarios/01-nmap-scan/`.
