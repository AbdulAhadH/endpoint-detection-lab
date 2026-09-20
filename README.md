# Enterprise Endpoint Detection Lab (Wazuh EDR/XDR)

A hands-on lab replicating day-one EDR analyst/engineer work: deploying an open-source
EDR/XDR platform across a small fleet, authoring detection rules mapped to MITRE ATT&CK,
validating them with real attack simulations, and tuning out false positives.

> **Status:** 🚧 In progress — infrastructure deployed, detection engineering underway.

## Resume summary (target)

> Deployed Wazuh EDR and Sysmon across 3 Windows/Linux endpoints; authored 18 detection
> rules mapped to MITRE ATT&CK, validated with Atomic Red Team.

## Architecture

```
                    ┌─────────────────────────────┐
                    │   Wazuh Server (Ubuntu VM)   │
                    │  - Wazuh Manager             │
                    │  - Wazuh Indexer             │
                    │  - Wazuh Dashboard           │
                    └──────────────┬────────────────┘
                                   │  TLS (1514/1515)
              ┌────────────────────┼────────────────────┐
              │                    │                    │
    ┌─────────▼─────────┐ ┌───────▼────────┐  ┌─────────▼─────────┐
    │  Windows Endpoint  │ │ Windows/Linux  │  │  Windows/Linux     │
    │  Wazuh Agent       │ │  Endpoint #2   │  │  Endpoint #3       │
    │  + Sysmon          │ │  Wazuh Agent   │  │  Wazuh Agent       │
    └────────────────────┘ └────────────────┘  └─────────────────────┘
```

*(Diagram will be replaced with an actual image at `docs/architecture.png` once the
full fleet is deployed.)*

## What this lab covers

- [x] Deploy Wazuh manager, indexer, and dashboard
- [x] Resolve TLS certificate chain issues across indexer/dashboard/filebeat
- [x] Deploy first Wazuh agent (Windows) and confirm active status
- [ ] Deploy 2nd–3rd endpoint agents
- [ ] Install Sysmon (SwiftOnSecurity baseline) on Windows endpoints
- [ ] Point Wazuh at the Sysmon log channel
- [ ] Run Atomic Red Team simulations against mapped ATT&CK techniques
- [ ] Author custom Wazuh detection rules (informed by SigmaHQ) mapped to ATT&CK IDs
- [ ] Validate each rule by re-running its corresponding Atomic test
- [ ] Tune rules to reduce false positives
- [ ] Document agent-health monitoring process
- [ ] (Stretch) Use Velociraptor for a DFIR collection task

## Tech stack

| Component | Purpose |
|---|---|
| [Wazuh](https://wazuh.com) | Open-source EDR/XDR — manager, indexer, dashboard, agents |
| [Sysmon](https://github.com/SwiftOnSecurity/sysmon-config) | Deep Windows telemetry (process, network, registry) |
| [Atomic Red Team](https://github.com/redcanaryco/atomic-red-team) | Attack simulation mapped to MITRE ATT&CK |
| [MITRE ATT&CK](https://attack.mitre.org) | Technique framework used to map detections |
| [Sigma](https://github.com/SigmaHQ/sigma) | Reference detection logic, adapted into Wazuh rules |
| [Velociraptor](https://docs.velociraptor.app) *(stretch)* | Endpoint DFIR / forensic collection |

## Repo structure

```
wazuh-edr-lab/
├── README.md                    # You are here
├── docs/
│   ├── deployment.md            # Full deployment log, incl. cert-chain troubleshooting
│   ├── agent-health-checks.md   # How to verify agent health day-to-day
│   └── architecture.png         # Environment diagram
├── detection-rules/
│   ├── local_rules.xml          # Custom Wazuh detection rules
│   └── rule-notes.md            # ATT&CK technique mapped to each rule, and why
├── atomic-tests/
│   └── test-log.md              # Which Atomic tests were run, when, and the result
└── screenshots/
    └── ...                      # Dashboard alerts, agent status, rule hits, etc.
```

## Environment

- **Wazuh server:** Ubuntu VM — Wazuh 4.14.x (manager + indexer + dashboard, single-node)
- **Endpoint(s):** Windows VM(s) running Wazuh agent + Sysmon
- All agent↔manager traffic is TLS-encrypted; certs generated via the Wazuh certificate
  generation tool (see `docs/deployment.md` for the full cert-chain debugging process).

## Notes

This lab intentionally documents the *troubleshooting process*, not just the end state —
real EDR deployment work involves exactly this kind of certificate, service, and config
debugging, and that process is captured in `docs/deployment.md`.
