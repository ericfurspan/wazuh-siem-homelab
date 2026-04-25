# Wazuh SIEM Home Lab

A personal home lab deploying Wazuh as a SIEM/XDR platform for hands-on practice with threat detection, log analysis, file integrity monitoring, and incident response. Built with a MacBook Pro (Apple Silicon M2) using Docker.

## Architecture

```
┌─────────────────────────────────────────────┐
│               macOS Host (M2)               │
│                                             │
│  ┌──────────────────────────────────────┐   │
│  │         Docker Network               │   │
│  │                                      │   │
│  │  ┌─────────────────────┐             │   │
│  │  │   Wazuh Manager +   │             │   │
│  │  │   Indexer +         │◄────────┐   │   │
│  │  │   Dashboard         │         │   │   │
│  │  │   (single-node)     │         │   │   │
│  │  └─────────────────────┘         │   │   │
│  │                                  │   │   │
│  │  ┌─────────────────────┐         │   │   │
│  │  │  ubuntu-agent-01    │─────────┘   │   │
│  │  │  Ubuntu 22.04       │  logs/      │   │
│  │  │  Wazuh Agent        │  alerts     │   │
│  │  └─────────────────────┘             │   │
│  └──────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

**Components:**

- **Wazuh Manager/Indexer/Dashboard** -- single-node Docker stack
- **ubuntu-agent-01** -- Ubuntu 22.04 Docker container enrolled as a Wazuh agent
- **macOS agent** -- host machine also enrolled as a Wazuh agent

## What Was Detected

The following detection scenarios were simulated and validated:

| Scenario                              | Rule IDs      | Severity          | MITRE               |
| ------------------------------------- | ------------- | ----------------- | ------------------- |
| SSH brute force (non-existent user)   | 5710, 5712    | Level 5, Level 10 | T1110 - Brute Force |
| File integrity: add / modify / delete | 554, 550, 553 | Level 5-7         | T1565.001           |
| CIS Ubuntu 22.04 Benchmark (SCA)      | 19004, 19014  | Level 7-9         | N/A                 |
| CVE detection (31 vulnerabilities)    | 23504, 23505  | Level 7-10        | N/A                 |

## Repository Structure

```
wazuh-siem-homelab/
├── README.md
├── lab.md                        # Setup guide and known issues
├── incident-report.md            # IR-2026-001: SSH brute force simulation
└── screenshots/
    ├── dashboard/                # Wazuh overview dashboard
    ├── ssh-brute-force/          # SSH auth alerts and rule details
    ├── fim/                      # File integrity monitoring events
    ├── mitre/                    # MITRE ATT&CK dashboard
    └── compliance/               # CVE detection and CIS benchmark results
```

## Documentation

- [Lab Setup Guide](lab.md) -- deployment steps, agent configuration, known issues
- [Incident Report IR-2026-001](incident-report.md) -- SSH brute force detection walkthrough

## Tools & Technologies

- [Wazuh](https://wazuh.com) -- SIEM/XDR platform
- Docker -- containerized deployment on macOS
- Ubuntu 22.04 LTS -- monitored endpoint
- MITRE ATT&CK -- threat classification framework
