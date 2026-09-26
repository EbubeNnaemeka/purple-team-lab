# Purple Team Lab — Attack Simulation & Detection Validation

Combines the [Active Directory lab](https://github.com/EbubeNnaemeka/active-directory-lab) and [SIEM detection lab](https://github.com/EbubeNnaemeka/siem-detection-lab): simulates real attacker techniques against the AD environment using Atomic Red Team, then validates whether the Splunk detections actually catch them — closing the loop between building infrastructure and defending it.

## Why this project

Most entry-level candidates can describe attacker techniques from studying for a certification. This project proves it hands-on: attack executed → telemetry generated → detection fired (or didn't, and got tuned until it did) → documented end to end.

## Prerequisites
- The AD lab (`active-directory-lab`) built and running
- The SIEM lab (`siem-detection-lab`) ingesting logs from the same environment
- [Atomic Red Team](https://github.com/redcanaryco/atomic-red-team) installed on a **disposable/snapshotted** test client — never run against a production system
- PowerShell 5.1+, execution policy relaxed on the test VM only

## Install Atomic Red Team

```powershell
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing)
Install-AtomicRedTeam -getAtomics
```

## Techniques exercised

| Technique | ATT&CK ID | Atomic Test # | Detection validated |
|---|---|---|---|
| Kerberoasting | T1558.003 | Atomic Test #1 | New detection added (see below) |
| Credential dumping (Mimikatz LSASS) | T1003.001 | Atomic Test #1 | Sysmon Event ID 10 alert |
| PowerShell encoded command | T1059.001 | Atomic Test #2 | [`powershell-encoded-command.spl`](https://github.com/EbubeNnaemeka/siem-detection-lab/blob/main/detections/powershell-encoded-command.spl) |

Full run log with commands, expected telemetry, and actual Splunk results: [`attack-log.md`](attack-log.md).

## Run a technique + validate detection

```powershell
Invoke-AtomicTest T1558.003 -TestNumbers 1
```

Then in Splunk, search for the resulting telemetry within the run window and confirm the detection fires. Document the result (fired / didn't fire / partial) in `attack-log.md`, and if it didn't fire, tune the rule and re-run — that tuning iteration is the actual value of this project.

## New detection added from this exercise

The SIEM lab has no Kerberoasting rule, so this test is expected to expose a gap. Draft rule to validate against your own telemetry: [`detections/kerberoasting.spl`](detections/kerberoasting.spl), written against the expected telemetry (Event ID 4769 with RC4 encryption type, which modern Kerberos shouldn't be using).

## Resume bullet (use once you have completed and verified the lab)

> Simulated Kerberoasting and credential-dumping attacks (Atomic Red Team, MITRE ATT&CK T1558.003/T1003.001) against a self-built AD lab; validated SIEM detection coverage and authored a new detection rule after identifying a gap in the original ruleset.

## Repo contents

```
├── README.md
├── attack-log.md
└── detections/
    └── kerberoasting.spl
```

## Safety note

All testing was performed in an isolated, host-only-networked lab with no connection to production systems or the internet-facing network, using disposable VM snapshots reverted after each test run.
