# IR Runbook — Web Application Attack
**Framework:** SANS PICERL  
**MITRE ATT&CK:** T1190 — Exploit Public-Facing Application  
**Severity:** Critical  

---

## Preparation

- Web application logs ingested into Wazuh
- DVWA used as test target for attack simulation
- Docker container logs monitored for anomalous activity

---

## Identification

**Indicators:**
- SQL injection patterns in web logs (`' OR`, `UNION SELECT`, `--`)
- XSS payloads in request parameters (`<script>`, `onerror=`)
- Unusual file uploads (`.php`, `.phtml` extensions)
- Command injection attempts (`|`, `&&`, `;` in input fields)
- Repeated 500 errors or unusual response sizes

**Confirmed in lab (DVWA testing):**
- SQL injection: extracted all user password hashes via UNION SELECT
- XSS: executed JavaScript and extracted session cookies
- Command injection: executed OS commands via unsanitized ping field
- File upload: uploaded PHP web shell, achieved remote code execution

---

## Containment

1. Take application offline or put in maintenance mode immediately
2. Block attacker IP at WAF/firewall
3. Preserve logs before any remediation
4. Identify what data may have been accessed or exfiltrated

```bash
# Take Docker container offline
docker stop <container_name>

# Preserve logs
docker logs <container_name> > incident_$(date +%Y%m%d).log
```

---

## Eradication

1. Review all uploaded files — remove any web shells
2. Patch the vulnerable application code
3. Rotate all database credentials
4. Invalidate all active sessions
5. Review database for unauthorized queries or data changes

---

## Recovery

1. Deploy patched version of application
2. Implement WAF rules for common injection patterns
3. Enable input validation and output encoding
4. Test fixes against known payloads before bringing back online

---

## Lessons Learned

**From DVWA testing:**
- Low difficulty: no defenses — trivially exploitable
- Medium difficulty: client-side controls only — bypassed via HTML inspection
- High difficulty: server-side blacklisting — partially effective but still bypassable
- **Key finding:** Blacklisting is always weaker than whitelisting. Proper input validation (only accept known-good input) is the correct defense, not trying to block known-bad input
