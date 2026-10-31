# Malware Analysis Sandbox Lab — Capstone Project (Unit 1)

### Student & Project Information
- **Student Name**: Sarthak Sunil Thorat
- **PRN**: 2125UCSM1105
- **Subject**: NPC (Network Protection & Cyber Security)
- **Unit**: Unit 1 — Capstone Project
- **Topic**: *Malware Analysis Sandbox Using VirtualBox or VMware*
- **GitHub Repository**: [https://github.com/sarthak833/NPC-Capstone-Project](https://github.com/sarthak833/NPC-Capstone-Project)

---

## Overview

This repository contains the complete, reproducible laboratory submission for **Unit 1 Capstone Project** in **Network Protection & Cyber Security (NPC)**.

The lab is built on **VMware Workstation 26** with **Debian 13.7 (trixie)** operating inside a strictly isolated **host-only (VMnet1)** virtual network with no default gateway or internet connectivity by design.

It features complete educational payloads (reverse shell and bind shell), automated host listeners, unattended Debian preseed configuration, full evidence transcripts, system console screenshots, and formal submission reports in both DOCX and PDF formats.

---

## Submission Deliverables & Repository Structure

```
├── screenshots/                                  # All VM console screenshots (baseline, isolation, shells, restore)
│   ├── 02-clean-boot-console.png
│   ├── 03-isolation-verified.png
│   ├── 04-reverse-shell-active.png
│   ├── 05-bind-shell-active.png
│   ├── 06-post-compromise-state.png
│   ├── 07-after-restore-verified.png
│   └── 08-live-demo-reverse-shell.png
├── Report.docx                                   # Complete Capstone Submission Report (Word Document)
├── Report.pdf                                    # Complete Capstone Submission Report (PDF Document, < 10 MB)
├── malware_sandbox_traffic.pcapng                # Captured network traffic of reverse/bind shell interaction
├── README.md                                     # Project documentation, student metadata & setup guide
├── payloads/                                     # Educational shell script payloads
│   ├── lab_reverse.sh                            # Reverse shell (/dev/tcp -> host:4444 with beacon & reconnect)
│   └── lab_bind.sh                               # Python3-based multi-client bind shell (TCP:4445)
├── listeners/                                    # Analyst host listener and client tooling
│   ├── c2_listener.py                            # Multi-stage C2 listener & session logger
│   ├── bind_connect.py                           # Automated bind shell client & transcript recorder
│   ├── fb2png.py                                 # Guest framebuffer decoder (1280x800 BGRX -> PNG)
│   ├── guest_console.sh                          # Guest console renderer and framebuffer capture tool
│   └── vncshot.py                                # VNC automated capture utility
├── build/                                        # Provisioning files
│   ├── preseed.cfg                               # Debian unattended installation answer file
│   └── lab_key.pub                               # Public SSH key for key-based authentication
├── evidence/                                     # Verifiable evidence logs and raw transcripts
│   ├── hashes.txt                                # Cryptographic hashes (MD5 & SHA-256) of payloads
│   ├── isolation_check.log                       # Proof of host-only network isolation (no internet)
│   ├── reverse_session.log                       # C2 reverse shell session transcript
│   ├── reverse-shell-during.log                  # In-guest process and socket inspection during C2
│   ├── bind_session.log                          # Bind shell session transcript
│   ├── bind-shell-launch.log                     # In-guest bind shell service verification
│   ├── post-compromise-changes.log               # Persistence simulation logs (crontab, backdoor user)
│   ├── restore_verification.log                  # 7-point post-revert verification transcript
│   ├── snapshot_ops.log                          # Snapshot operations log (vmrun)
│   └── screenshots/                              # Raw screenshot evidence directory
└── report/                                       # Report source and renders
    ├── Malware-Analysis-Sandbox-Lab-Report-Sarthak-Thorat-2125UCSM1105.docx
    ├── build/                                    # DOCX generation tooling
    │   ├── generate.js
    │   └── package.json
    └── render/
        └── Malware-Analysis-Sandbox-Lab-Report-Sarthak-Thorat-2125UCSM1105.pdf
```

---

## Lab Network Architecture

```
Windows Host (Analyst Workstation)                Isolated Debian 13 VM (Sandbox Guest)
+-----------------------------------+             +-----------------------------------+
| VMnet1 Adapter                    |  Host-Only  | enp0s17                           |
| IP: 192.168.116.1/24              |<----------->| IP: 192.168.116.200/24 (Static)   |
|                                   |             |                                   |
| - c2_listener.py   (TCP/4444) <---+-- Reverse --+-- Outbound callback (/dev/tcp)   |
| - bind_connect.py ------------->--+--- Bind ----+-> LISTEN 0.0.0.0:4445 (python3)   |
| - ssh / scp (key-only auth) --->--+--- Mgmt ----+-> sshd :22                        |
+-----------------------------------+             +-----------------------------------+
             NO default gateway on guest -> Statically impossible to access internet
```

---

## End-to-End Methodology

1. **Unattended Provisioning**: Remastered Debian 13.7 installer using `build/preseed.cfg` with key-based authentication (`build/lab_key.pub`), static IP `192.168.116.200/24`, and omitted default gateway.
2. **Clean Baseline Snapshot**: VM powered off, then captured snapshot: `01-CLEAN-BASELINE`.
3. **Isolation Verification**: Verified zero routes to internet, empty `/etc/resolv.conf`, unreachable external IPs (8.8.8.8) and DNS, with functional host-only ping (`evidence/isolation_check.log`).
4. **Reverse Shell Execution**: Initiated `c2_listener.py` on host TCP/4444; launched `payloads/lab_reverse.sh` via guest. Evaluated process tree (`bash` + `/bin/sh -i`), socket state, and auto-reconnection behavior.
5. **Bind Shell Execution**: Executed `payloads/lab_bind.sh` on guest listening on `0.0.0.0:4445`; connected using `listeners/bind_connect.py`.
6. **Persistence Simulation**: Planted persistence artifacts: `@reboot` crontab, backdoor user `svcupdate` (UID 1001), hidden binary `/usr/local/bin/.sysupdate`, and loot marker.
7. **Snapshot Reversion & Verification**: Reverted VM using `vmrun revertToSnapshot ... 01-CLEAN-BASELINE`. Executed 7-point audit verifying all persistent files, backdoor users, and crontab entries were purged (`evidence/restore_verification.log`).
8. **VirusTotal Hash Query**: Queried SHA-256 hashes of both payloads (`evidence/hashes.txt`). Confirmed no prior reports exist; per lab rules, zero samples were uploaded to external platforms.

---

## Key Results Summary

- **Reverse Shell**: Spawns two distinct processes (`bash` loop + interactive `/bin/sh`). Demonstrates resilient reconnections every 5 seconds upon disconnect.
- **Bind Shell**: Single resident `python3` process binding to `0.0.0.0:4445`, dynamically spawning subshells per incoming socket.
- **Persistence Assessment**: Crontab modifications, backdoor user accounts, and filesystem artifacts survived guest reboots, underscoring the inadequacy of live memory-only forensics.
- **Snapshot Integrity**: Reverting to `01-CLEAN-BASELINE` successfully restored 100% of the baseline environment with zero residual artifacts.

---

## Academic Integrity & Safety

All credentials, passwords, and host keys used in this laboratory are strictly fictitious and isolated to this exercise. The guest virtual machine had zero internet connectivity at all times. In compliance with safe analysis policies, no malware sample or laboratory artifact was uploaded to VirusTotal or any third-party threat intelligence repository.
