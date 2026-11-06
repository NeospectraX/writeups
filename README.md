<div align="center">

# 🚩 Writeups

**CTF & vulnerable-machine writeups** — recon, exploitation, and privilege-escalation methodology (TryHackMe, HackTheBox, PortSwigger, RootMe, OverTheWire, and other platforms).

![TryHackMe](https://img.shields.io/badge/TryHackMe-212C42?style=for-the-badge&logo=tryhackme&logoColor=red)
![HackTheBox](https://img.shields.io/badge/HackTheBox-9FEF00?style=for-the-badge&logo=hackthebox&logoColor=black)
![PortSwigger](https://img.shields.io/badge/PortSwigger-FF6633?style=for-the-badge&logo=portswigger&logoColor=white)
![RootMe](https://img.shields.io/badge/RootMe-000000?style=for-the-badge&logo=rootme&logoColor=white)
![OverTheWire](https://img.shields.io/badge/OverTheWire-1D1D1D?style=for-the-badge&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=for-the-badge)

</div>

---

## 📂 Index

| # | Machine / Room | Platform | Difficulty | Attack Path |
|:-:|---|:-:|:-:|---|
| 1 | [🏴 Bounty Hacker](./bounty-hacker-writeup-github.md) | ![THM](https://img.shields.io/badge/THM-red?style=flat-square) | 🟢 Easy | Anonymous FTP → Credential Harvesting → SSH (Hydra Bruteforce) → Privesc via `sudo tar` ([GTFOBins](https://gtfobins.github.io/gtfobins/tar/)) |
| 2 | [🏴 RootMe](./rootme-writeup.md) | ![THM](https://img.shields.io/badge/THM-red?style=flat-square) | 🟢 Easy | Directory Enum → File Upload Filter Bypass (`.phtml`) → PHP Reverse Shell → Shell Stabilization → Root |
| 3 | [🏴 Fool's Mate](<./# Fool's Mate TryHackMe Writeup>) | ![THM](https://img.shields.io/badge/THM-red?style=flat-square) | 🟡 Easy–Medium | Client-Side Logic Analysis → Broken Access Control (CWE-602) → Direct API Bypass |
| 4 | [🏴 U.A. High School](./ua-high-school-writeup.md) | ![THM](https://img.shields.io/badge/THM-red?style=flat-square) | 🟢 Easy | Unauthenticated Command Injection (`index.php?cmd=`) → Steganography (Corrupted JPEG Header + `steghide`) → Lateral Move → Sudo Filter Bypass (Blacklist→Exact-Match, `eval`) → Root |
| 5 | [🏴 Crawl4AI — Hooks RCE](./portswigger-vapt-reports/VAPT_Report_Crawl4AI_RCE.pdf) | ![PortSwigger](https://img.shields.io/badge/PortSwigger-FF6633?style=flat-square) | 🔴 Critical | API Recon (`/openapi.json`) → Python `exec()` Hook Injection → RCE as root → Flag Exfiltration |
| 6 | [🏴 SecureCorp Data Vault — SSI Injection](./portswigger-vapt-reports/VAPT_Report_SecureCorpDataVault_SSI.pdf) | ![PortSwigger](https://img.shields.io/badge/PortSwigger-FF6633?style=flat-square) | 🔴 Critical | Stored Note → `.shtml` SSI Processing → `#exec` Directive Injection → RCE |
| 7 | [🏴 SecurePortal — Werkzeug Debug RCE](./portswigger-vapt-reports/VAPT_Report_SecurePortal_WerkzeugRCE.pdf) | ![PortSwigger](https://img.shields.io/badge/PortSwigger-FF6633?style=flat-square) | 🔴 Critical | Base64 PIN Disclosure → Forced Debug-Console Crash → Werkzeug PIN Auth → `os.popen()` RCE |
| 8 | [🏴 VM2 Sandbox Escape (CVE-2023-32314)](./portswigger-vapt-reports/VAPT_Report_VM2_Sandbox_Escape.pdf) | ![PortSwigger](https://img.shields.io/badge/PortSwigger-FF6633?style=flat-square) | 🔴 Critical | `Error.stack` Proxy Trap (Host-Realm) → `vm2` Sandbox Escape → Host RCE |
| 9 | [🏴 File Path Traversal — 5 Bypass Variants](./portswigger-vapt-reports/VAPT_Report_PathTraversal_MultiVariant.pdf) | ![PortSwigger](https://img.shields.io/badge/PortSwigger-FF6633?style=flat-square) | 🟠 High | Non-Recursive Filter Bypass → Zero-Sanitization → Double URL-Encoding → Start-of-Path Validation Bypass → Null-Byte Extension Bypass |

*New writeups are appended to this table as they're completed.*

---

## 🗂️ Methodology

Every writeup follows a consistent structure:

```
1️⃣  Reconnaissance      → service / attack-surface enumeration (Nmap, Gobuster, DevTools)
2️⃣  Initial Access      → exploitation path used to gain a foothold
3️⃣  Privilege Escalation → technique used to escalate to root/admin
4️⃣  Summary             → condensed technique table + key lessons
```

---

## 📁 PortSwigger VAPT Engagement Reports

The [`portswigger-vapt-reports/`](./portswigger-vapt-reports/) folder holds full, professional-format VAPT engagement reports (PDF) for individual PortSwigger Web Security Academy labs — each role-played as a real client engagement with Document Control, Scope, Methodology (PTES), CVSS 3.1 scoring, reproducible PoC steps with real Burp Suite evidence, a Remediation Roadmap, and appendices. These sit alongside the shorter CTF-style writeups above as a separate, deeper deliverable format.

---

## 📊 Stats

![Total Writeups](https://img.shields.io/badge/Writeups-9-blue?style=flat-square)
![Platforms](https://img.shields.io/badge/Platforms-2-orange?style=flat-square)
![Easy](https://img.shields.io/badge/Easy-3-brightgreen?style=flat-square)
![Easy--Medium](https://img.shields.io/badge/Easy--Medium-1-yellow?style=flat-square)
![High](https://img.shields.io/badge/High-1-orange?style=flat-square)
![Critical](https://img.shields.io/badge/Critical-4-red?style=flat-square)

---

## ⚠️ Scope & Disclaimer

> All writeups document activity performed against intentionally vulnerable machines hosted on legitimate training platforms (TryHackMe, HackTheBox, etc.), within their sanctioned lab environments. Flags, IPs, and credentials referenced are specific to ephemeral instances tied to those platforms and hold no value outside that context. This repository is intended purely for educational and skill-documentation purposes.

---

<div align="center">

## 🔗 Connect

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/ashok-siravi)
[![Medium](https://img.shields.io/badge/Medium-12100E?style=for-the-badge&logo=medium&logoColor=white)](https://siraviashok4721.medium.com/)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/NeospectraX)

</div>
