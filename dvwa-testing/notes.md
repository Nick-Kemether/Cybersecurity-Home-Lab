# DVWA Testing Notes

Ran DVWA in Docker on the Ubuntu VM and worked through the main vulnerability categories at low, medium, and high difficulty. The point wasn't just to exploit things — I wanted to understand what the defenses actually looked like in code and why they failed.

```bash
docker run -d -p 80:80 --name dvwa vulnerables/web-dvwa
```

---

## SQL Injection

**Low**
Basic bypass worked immediately:
```
' OR 1=1 --
```
Got all users back. Then used UNION SELECT to pull password hashes, cracked them with CrackStation. Recovered plaintext passwords for every account in the database.

**Medium**
They replaced the text field with a dropdown. Opened the browser inspector, edited the option value directly in the HTML, and submitted the injection that way. Client-side controls don't stop anything.

**High**
The `--` comment syntax was filtered. Switched to `#` which MySQL also treats as a comment and wasn't in the blacklist. Worked first try.

The consistent lesson across all three: blacklists lose. You block the obvious thing and miss a variation. Parameterized queries are the only real fix.

---

## XSS (Reflected)

**Low**
```html
<script>alert(document.cookie)</script>
```
Alert fired and showed the active session cookie. In a real scenario that cookie goes to the attacker and they're logged in as you without ever touching your password.

**Medium**
Filter stripped `<script>` tags. Nested the tag inside itself so when the filter removed the inner one it rebuilt the outer:
```html
<scr<script>ipt>alert('hacked')</scr<script>ipt>
```

**High**
Script and img tags blocked. Used a body tag with an onload handler instead:
```html
<body onload=alert('hacked')>
```
The filter was blocking specific tags by name, not all HTML. There are enough tags and event handlers that you can always find one that isn't on the list.

---

## Command Injection

**Low**
```
127.0.0.1 && cat /etc/passwd
```
Dumped the passwd file. The app was passing input directly to a shell command with no sanitization.

**Medium**
`&&` was blocked. Used a pipe instead:
```
127.0.0.1 | cat /etc/passwd
```

**High**
Most separators blocked. Looked at the source code — the filter blocked `'| '` (pipe with a space after it) but not a bare pipe. No space around the pipe:
```
127.0.0.1|cat /etc/passwd
```
Got through. This one was satisfying to figure out because the answer was sitting in the source the whole time.

---

## File Upload

**Low**
Uploaded a PHP web shell:
```php
<?php system($_GET["cmd"]); ?>
```
Navigated to the file URL, added `?cmd=whoami` to the query string. Got `www-data` back — full remote code execution through a file the server let me upload.

**Medium**
Rejected `.php` extension. Renamed to `.php.jpg`, uploaded fine. Server wouldn't execute it as PHP though so no RCE. Getting full execution here would need Burp Suite to manipulate the MIME type on the way in.

---

## Overall Takeaways

Blacklisting is the wrong approach for almost everything here. Every single medium and high level defense was a blacklist and every one had a bypass. The developers blocked the thing they thought of and missed the variation.

The right defenses — parameterized queries, output encoding, whitelist input validation — show up in the "view source" for impossible difficulty. Comparing the vulnerable code to the fixed code side by side was probably the most useful part of this whole exercise.
