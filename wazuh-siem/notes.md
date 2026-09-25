Threat Hunting Notes

These are notes from hunt exercises I ran in the lab after simulating attacks. The goal was to practice the process of forming a hypothesis, querying the SIEM, and figuring out what I could and couldn't find.

Hunt 1 — Looking for brute force activity

Hypothesis: An attacker is trying to brute force credentials somewhere on the network.

Started with a broad query in Wazuh for authentication failures:

rule.id: 5760

Found a spike from a single IP. Narrowed it down:

rule.id: 5760 AND data.srcip: 192.168.1.125

300+ failures in under 5 minutes — clearly automated. Hydra running from Kali. Detection worked, alert was accurate, and the MITRE mapping (T1110.001) was correct out of the box.

What I'd look for next in a real environment: did any of those authentication attempts succeed? That's the question that changes the severity of the response.

Hunt 2 — Looking for reconnaissance

Hypothesis: Something is scanning the network to map out what's running.

First query came back empty:

rule.groups: network_scan

Nothing. That's when I realized the gap — Wazuh wasn't watching the network layer at all. Host-based logging doesn't capture someone scanning your ports from outside.

After integrating Suricata, re-ran the hunt:

rule.groups: network_scan

Got hits. Same Nmap scan that generated nothing before now showed up clearly with source IP, timing, and the ports that were probed.

The hunt found a detection gap before an attacker could exploit it. That felt like the point of the exercise.

Hunt 3 — Web application attacks

Hypothesis: Someone is probing or exploiting the web server.

Pulled nginx access logs from the Docker container and looked for patterns:

Requests with SQL characters (', UNION, --)
Script tags in request parameters
Requests to /hackable/uploads/ for unusual file types
High volume of requests from a single IP in a short window

All of those showed up in the DVWA testing logs. The patterns are distinct enough that you could write Wazuh rules to alert on them in a real web server deployment.

Next step for this one: get nginx logs flowing into Wazuh automatically so I don't have to pull them manually.

What I'm taking away from this

The gap finding in Hunt 2 was the most useful thing. I wasn't looking for the gap — I was looking for the attack. Finding out the tool couldn't see what I was hunting for is exactly the kind of thing that matters in a real environment.

The process matters more than any individual query. Form a hypothesis, go look, document what you found and what you didn't, fix the gaps, retest. Doing it in a lab where you already know the answer helps you build the muscle for when you don't.

Content

The Role The Schonfeld Cybersecurity Operations team is seeking an analyst—a hands-on individual who sets the standard for incident response, threat management and risk mitigation while driving continuous improvement of our security controls. The Cybersecurity Analyst must proactively adapt to the

PASTED
