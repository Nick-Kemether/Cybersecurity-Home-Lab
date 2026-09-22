# Atomic Red Team — Adversary Emulation

## Overview

Installed Atomic Red Team on Kali Linux to emulate MITRE ATT&CK techniques in a controlled environment and validate Wazuh detection coverage.

## Setup

Installed PowerShell on Kali Linux and deployed Atomic Red Team:

```bash
sudo apt install powershell -y
```

```powershell
IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing);
Install-AtomicRedTeam -getAtomics -Force
```

## Techniques Staged for Emulation

| Technique | ID | Description |
|---|---|---|
| Brute Force | T1110 | Credential brute forcing via SSH |
| Command & Scripting | T1059 | Execution via command interpreter |
| OS Credential Dumping | T1003 | Extracting credentials from OS |
| Remote System Discovery | T1018 | Enumerating systems on the network |

## Current Status

Environment configured and Atomic Red Team installed. Full emulation runs and Wazuh detection validation in progress. Detection rules and hunt findings will be documented as exercises are completed.

## Already Validated Detections

Prior to Atomic Red Team, the following techniques were manually simulated and confirmed detected by Wazuh:

| Technique | ID | Tool Used | Wazuh Rule | Result |
|---|---|---|---|---|
| Brute Force: Password Guessing | T1110.001 | Hydra | 5760 | ✅ Detected |
| Network Scanning | T1046 | Nmap | Suricata integration | ✅ Detected after gap remediation |
