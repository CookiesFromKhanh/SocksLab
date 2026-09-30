# Home SOC & Detection Engineering Lab

A self-built home lab for learning SOC analyst and detection engineering skills:
Windows + Linux endpoints instrumented with Sysmon, a Wazuh SIEM, attack
simulation with Atomic Red Team, and hand-written Sigma detections mapped to
MITRE ATT&CK.

## Goal

Understand the full detection lifecycle — generate attacker telemetry, spot it
in logs, write a detection rule for it, and document it like an analyst would —
rather than just reading theory.

## Lab architecture

```
                    ┌─────────────────────────────┐
                    │   Desktop 1 (i5-12400, 16GB) │
                    │        Proxmox VE host        │
                    │      hostname: pve             │
                    │   192.168.100.2/24 (mgmt)      │
                    └───────────────┬────────────────┘
                                    │
        ┌───────────────┬──────────┴──────────┬─────────────────┐
        │               │                     │
┌───────▼──────┐ ┌───────▼───────┐   ┌─────────▼────────┐
│ win10-endpoint│ │ ubuntu-endpoint│   │  Wazuh-SIEM (VM 102)│
│   Windows 10   │ │ Ubuntu Server  │   │   Ubuntu Server      │
│  + Sysmon      │ │  + Wazuh agent │   │  192.168.0.117        │
│  + Wazuh agent │ │                │   │  (all-in-one install) │
└────────────────┘ └────────────────┘   └───────────────────────┘

Laptop (remote access only, not part of the lab itself) →
    connects to Proxmox web UI / Wazuh dashboard over 192.168.0.x
```

| Component | Role | Notes |
|---|---|---|
| Desktop 1 (i5-12400, 16GB, GTX/RTX 1050 Ti) | Proxmox VE host | Always-on, runs everything |
| win10-endpoint | Attack target | Sysmon (SwiftOnSecurity config) + Wazuh agent |
| ubuntu-endpoint | Secondary target | Wazuh agent |
| Wazuh-SIEM (VM ID 102) | SIEM | All-in-one Wazuh install (manager, indexer, dashboard) |
| Laptop | Remote access | Not hosting any lab component — just views the dashboard |

> Desktop 2 (G6405, 8GB) is currently unused — not enough RAM to add reliably.

## Tech stack

- **Hypervisor:** Proxmox VE
- **SIEM:** Wazuh (all-in-one)
- **Endpoint telemetry:** Sysmon (SwiftOnSecurity config) on Windows
- **Attack simulation:** Atomic Red Team
- **Detection format:** Sigma → translated to Wazuh custom rules
- **Detection mapping:** MITRE ATT&CK

## Repo structure

```
.
├── README.md
├── docs/
│   └── setup/
│       ├── proxmox-setup.md
│       ├── wazuh-install.md
│       └── sysmon-config.md
├── detections/
│   ├── T1053.005-scheduled-task/
│   │   ├── detection.md
│   │   ├── sigma-rule.yml
│   │   └── wazuh-rule.xml
│   └── T1059.001-powershell/
│       └── ...
├── journal/
│   ├── 2026-09-21.md
│   └── 2026-09-25.md
└── assets/
    └── screenshots/
```

## Current status

- [x] Proxmox VE installed on Desktop 1
- [x] win10-endpoint, ubuntu-endpoint, Wazuh-SIEM VMs created
- [ ] Wazuh all-in-one install completed
- [ ] Sysmon deployed on win10-endpoint
- [ ] First Atomic Red Team test run
- [ ] First Sigma detection written

## How to reproduce

1. Install Proxmox VE on the host machine (see `docs/setup/proxmox-setup.md`)
2. Create the three VMs listed above
3. Install Wazuh all-in-one on the SIEM VM (see `docs/setup/wazuh-install.md`)
4. Install Sysmon + Wazuh agent on the Windows endpoint
5. Install the Wazuh agent on the Linux endpoint
6. Confirm both agents report into the Wazuh dashboard
7. Pick a technique from `detections/`, snapshot the target VM, and run the
   Atomic test

## Detection template

Every file under `detections/<technique-id>/detection.md` follows:

- **Technique** — name + ATT&CK ID
- **What the attacker is doing** — 2-3 plain sentences
- **Telemetry generated** — log source, event ID, screenshot
- **Detection logic** — the Sigma rule + how it maps to fields
- **False positives** — legitimate activity that looks the same
- **Analyst response** — what to check, how to confirm, what to do next
- **Test result** — did it fire, was it tuned

## Why this lab exists

Built as part of my path toward a SOC Analyst / Incident Responder role,
alongside my cybersecurity coursework at the University of Turku.
