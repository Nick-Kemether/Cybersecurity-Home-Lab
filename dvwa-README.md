# DVWA Web Application Security Testing

Hands-on web application vulnerability testing using DVWA (Damn Vulnerable Web Application) running in Docker. Tested each vulnerability category across low, medium, and high difficulty levels to understand both attack techniques and the defenses that stop them.

---

## Environment

- DVWA running in Docker container on Ubuntu 22.04
- Accessed via Firefox on the same VM
- All testing contained within isolated lab environment

```bash
docker run -d -p 80:80 --name dvwa vulnerables/web-dvwa
```

---

## SQL Injection

The application passes user input directly into SQL queries without sanitization.

**Low — Basic bypass:**
```
' OR 1=1 --
```
Returned all users in the database. Followed up with UNION SELECT to extract the database version and dump all password hashes.

```
' UNION SELECT user, password FROM users --
```

Cracked the extracted MD5 hashes using CrackStation — recovered plaintext passwords for multiple accounts.

**Medium — Client-side bypass:**
Input field replaced with a dropdown to prevent direct injection. Bypassed by using Firefox's inspector to edit the HTML option value directly, injecting the payload at the DOM level before submission. Client-side controls provide no real security.

**High — Server-side filter bypass:**
Filter blocked `--` comment syntax. Switched to `#` which MySQL also accepts as a comment character and isn't in the blacklist.

```
1' OR 1=1#
```

**Takeaway:** SQL injection defenses fail when built on blacklists. Parameterized queries are the only real fix.

---

## Cross-Site Scripting (XSS)

Unsanitized user input rendered directly as HTML in the browser.

**Low — Basic script injection:**
```html
<script>alert('hacked')</script>
```
Alert fired. Followed up with `document.cookie` to extract the active session cookie — demonstrating session hijacking potential.

**Medium — Nested tag bypass:**
Filter stripped `<script>` tags on one pass. Bypassed by nesting the tag inside itself so removing the inner one reconstructs the outer:
```html
<scr<script>ipt>alert('hacked')</scr<script>ipt>
```

**High — Event handler bypass:**
Filter blocked `<script>` and `<img>` tags but not `<body>`. Used an onload event handler instead:
```html
<body onload=alert('hacked')>
```

**Takeaway:** Blacklisting HTML tags is unwinnable — there are too many tags and event handlers that execute JavaScript. Output encoding is the correct defense.

---

## Command Injection

Application passes user input to OS commands without sanitization.

**Low:**
```
127.0.0.1 && cat /etc/passwd
```
Chained a second command using `&&`. Dumped the full `/etc/passwd` file.

**Medium:**
Filter blocked `&&`. Bypassed using pipe character:
```
127.0.0.1 | cat /etc/passwd
```

**High:**
Comprehensive blacklist blocked `&&`, `|`, `;`, `` ` ``, `$`. Analyzed the source code and found the filter blocked `'| '` (pipe with trailing space) but not a bare pipe with no space:
```
127.0.0.1|cat /etc/passwd
```
No space around the pipe — filter didn't match, command executed.

**Takeaway:** Blacklists fail on variations the author didn't anticipate. A whitelist that only accepts valid IP format (`\d+\.\d+\.\d+\.\d+`) would have stopped all three levels cold.

---

## File Upload

Application allows file uploads without validating content type.

**Low:**
Uploaded a PHP web shell directly:
```php
<?php system($_GET["cmd"]); ?>
```
Accessed the file via URL and executed arbitrary OS commands including `whoami` and `cat /etc/passwd`. Full remote code execution achieved.

**Medium:**
Filter checked file extension — rejected `.php`. Renamed to `.php.jpg`, uploaded successfully. Server wouldn't execute it as PHP due to extension. Attempted `.phtml` bypass — blocked. Full bypass requires MIME type manipulation via Burp Suite.

**Takeaway:** Extension checking alone is insufficient. Servers should validate actual file content, restrict upload directories from execution, and use randomized filenames.

---

## Key Findings

| Vulnerability | Low | Medium | High |
|---|---|---|---|
| SQL Injection | ✅ Bypassed | ✅ Bypassed (DOM edit) | ✅ Bypassed (`#` comment) |
| XSS | ✅ Bypassed | ✅ Bypassed (nested tags) | ✅ Bypassed (event handler) |
| Command Injection | ✅ Bypassed | ✅ Bypassed (`\|`) | ✅ Bypassed (no-space pipe) |
| File Upload | ✅ RCE achieved | ⚠️ Partial (needs Burp) | — |

---

## Broader Lessons

**Blacklists always lose.** Every medium and high level defense was a blacklist. Every one was bypassed. The pattern was consistent — the developer blocked the obvious cases and missed a variation.

**Source code review accelerates testing.** DVWA lets you view the source. In the command injection high level, reading the filter directly revealed the gap. In real assessments, this is why attackers look for exposed source, config files, and error messages.

**Defense in depth matters.** No single control stopped all attacks. Real applications need input validation, output encoding, WAF rules, and monitoring working together.

---

*Nick Kemether — Cybersecurity Analyst*
