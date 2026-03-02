---
# the default layout is 'page'
icon: fas fa-info-circle
order: 5
---

Hello, I’m Tylor.

I build and break things on purpose to have a better understanding of how and why things work the way they do.

This site documents my cybersecurity journey through hands-on lab work. Everything here is built inside my own homelab and configured, misconfigured, fixed, and fully understood before it’s written about.

---

## What I Focus On

- Blue Team fundamentals
- Detection engineering concepts
- SIEM and telemetry analysis
- Windows logging & Sysmon
- DNS control & network visibility
- Digital forensics labs
- Infrastructure troubleshooting

I’m especially interested in how systems generate logs, how telemetry tells a story, and how small configuration mistakes create real world failure points.

---

## Why This Blog Exists

I got tired of just watching tutorials and thinking I was learning.

So I started building. I was already documenting my process in building the labs for myself but I realized I should be publishing them online as well.

If I am taking the time to build these labs, break them, and doing troubleshooting, this knowledge should ***NOT*** be gatekept as someone else may experience a similar issue. Maybe my documentation can help a fellow cyber professional debug faster. Hopefully it will save someone hours of frustration!

Either way, this information is better shared than buried deep in my private notes folder.

This lab exists to:

- Understand infrastructure from the inside
- See what “normal” network behavior actually looks like
- Practice troubleshooting real failures
- Document everything so I don’t forget it

If something breaks, I write about it.
If something works, I explain why.

---

## Philosophy

Security isn’t paranoia.

It’s visibility.

It’s knowing what traffic leaves your network.
It’s understanding what your systems are actually doing.
It’s reducing noise so real signals stand out.

Most of what you’ll find here isn’t polished enterprise architecture, it’s practical, iterative learning.

**Build > Break > Fix > Document**

---

## Current Lab Stack

- Unraid server (24/7 infrastructure host)
- Pi-hole (LAN DNS control)
- Wazuh SIEM
- Windows Server DC + Sysmon w/ 2 agents
- Raspberry Pi services
- Segmented internal lab network
- Plex Media Server
- T-Pot honeypot VM

Everything is built to simulate real environments and generate meaningful telemetry.

---

*If it's not broken, fix it til it is*
