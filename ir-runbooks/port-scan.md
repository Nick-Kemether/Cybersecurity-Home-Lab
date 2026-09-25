# Port Scan / Reconnaissance — Incident Response Notes
MITRE: T1046 | Severity: Medium

---

## What triggered this

Suricata ET SCAN alerts showing up in Wazuh. Before I integrated Suricata, port scans weren't being detected at all — this runbook also documents that gap and how I closed it.

---

## The detection gap

Running Nmap from Kali against the Wazuh server generated zero alerts. Wazuh's agent-based monitoring watches what happens on the host but doesn't see network traffic. A full port scan can happen and you'd never know.

Fix: installed Suricata on the SIEM VM, pointed it at the right interface, added the eve.json log file to Wazuh's monitored sources.

```bash
sudo apt install suricata -y
# Edit /etc/suricata/suricata.yaml — set interface to match your NIC
sudo systemctl restart suricata
```

In ossec.conf:
```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

After that, the same Nmap scan generated alerts immediately.

---

## Responding to a scan alert

First question: who is this?

- Internal IP — could be authorized scanning, IT doing asset discovery, or a compromised internal host
- External IP — more concerning, treat as hostile until proven otherwise

Check whether the scan was followed by anything else. A scan by itself is reconnaissance. A scan followed by connection attempts to specific ports means they found something interesting.

---

## Containment

If external and clearly malicious, block at the firewall. In OPNsense:
Firewall → Rules → WAN → add block rule for the source IP

If internal, isolate the host and investigate — something on your network is scanning.

---

## After the fact

A scan tells you what an attacker can see. Use it:
- Pull the Suricata log and see which ports responded
- Those are your exposed services — do they all need to be exposed?
- Anything that doesn't need to be public-facing should be firewalled off

The scan is free recon for your own environment if you use it right.
