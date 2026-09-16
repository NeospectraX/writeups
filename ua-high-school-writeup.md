# TryHackMe: U.A. High School, Command Injection to Root via Sudo Filter Bypass

**Room:** U.A. High School (TryHackMe), Easy
**Category:** Web Exploitation, Linux Privilege Escalation
**Tools used:** `nmap`, `dirsearch`/`gobuster`, `curl`/Burp, netcat, `hexeditor`, `steghide`, `ssh-keygen`
**Author:** Ashok Siravi

---

## TL;DR

This box goes from unauthenticated remote code execution to root with no exploit development required, only enumeration and one filter logic flaw. A command injection in `index.php?cmd=` yields a shell as the web user. Enumeration of the filesystem surfaces a base64-encoded passphrase and an oversized JPEG whose header has been deliberately corrupted; repairing it and running `steghide` recovers credentials for a second account, `deku`. `sudo -l` under that account permits root execution of a "feedback" script whose input filter was rewritten from a substring blacklist to an exact-match check, a one-character-class regression that lets `$(id)` sail past validation and into `eval`. The same primitive, combined with an `${IFS}` whitespace bypass, is enough to write an SSH key into `/root/.ssh/authorized_keys` and obtain a persistent root shell.

---

## 1. Reconnaissance

_Fill in your actual scan output here, this is the one step missing from your recall, add it before publishing:_

```bash
nmap -n -sSCV <target-ip>
```

Ports open: `22` (SSH), `80` (HTTP). *(confirm against your own output)*

The web root gave no obvious clues, so the next step was content discovery:

```bash
dirsearch -u http://<target-ip>/
```

This surfaced `/assets/`, a blank page with no error. A blank 200 response is worth noting: a genuinely missing path usually 404s, so a blank 200 suggests a script that's running but failing silently on missing parameters. Probing common extensions confirmed a hit:

```bash
dirsearch -u http://<target-ip>/assets/index.php
```

`index.php` responded with no error, pointing at a script expecting query parameters it wasn't getting.

---

## 2. Initial Access: Unauthenticated Command Injection

Guessing a common parameter name confirmed command execution:

```
GET /assets/index.php?cmd=whoami
```

The response came back base64-encoded. That's output obfuscation, not protection: it hides the result from a casual glance in the browser, but does nothing to stop the injection itself. Decoding it confirmed shell access as the web service user.

**Exploitation, reverse shell:**
Swapped the `cmd` parameter for a reverse shell payload (via [revshells.com](https://www.revshells.com/) or a hand-built one-liner) and caught the connection with a `netcat` listener, giving an interactive shell.

**Root cause:** user-controlled input reaches a shell execution sink (`exec`/`shell_exec`/`system` in PHP) with no sanitization. Whoever wrote this endpoint solved for hiding the output, not for blocking the input.

---

## 3. Post-Exploitation Enumeration

With shell access, enumeration of `/var/www/` turned up a hidden directory containing a file with a base64-encoded passphrase, noted for later use.

Enumeration of `/var/www/html/assets/images/` revealed a JPEG of unusually large size relative to what it should be. Size mismatch on an image file is one of the more reliable steganography signals: legitimate site assets don't carry a few extra megabytes of payload.

---

## 4. Steganography: Recovering Hidden Credentials

Pulled the oversized image to the attack host:

```bash
wget http://<target-ip>/assets/images/<file>.jpg
```

Opening it in `hexeditor` showed the file's magic bytes were corrupted. The image data itself was intact, just mislabeled at the header level, which is why standard tools refused to open it.

**Steps:**
1. Compared the file's header bytes against the [JPEG file signature reference](https://en.wikipedia.org/wiki/List_of_file_signatures) and corrected them in `hexeditor`.
2. Extracted the embedded payload:
   ```bash
   steghide extract -sf <fixed-file>.jpg
   ```
3. Supplied the base64-decoded passphrase recovered during enumeration (Section 3) when `steghide` prompted for one.
4. Extraction produced `creds.txt`, a password for local user **deku**.

**Root cause:** credentials protected by security-through-obscurity (a deliberately corrupted signature plus steganography) instead of proper secrets management. Once the file was flagged as anomalous by its size, obscurity bought a few minutes and nothing more.

---

## 5. Lateral Movement

Logged in as **deku** with the recovered credentials and grabbed `user.txt`.

---

## 6. Privilege Escalation: Sudo Filter Bypass (Blacklist to Exact-Match Regression)

`sudo -l` as `deku`:

```
User deku may run the following commands on this host:
    (root) /opt/NewComponent/feedback.sh
```

The script takes free-form "feedback" input, checks it against a list of "dangerous characters," and, if judged safe, executes:

```bash
eval "echo $feedback"
```

The intended design was a wildcard blacklist: reject the input if it *contains* any of a set of dangerous substrings, such as `$(`, backticks, or `;`. What's actually implemented is an exact-match comparison: reject the input only if it *equals* one of those strings exactly.

```bash
# Intended (substring/wildcard check), correctly blocks this:
[[ "$input" == *'$('* ]]

# Actual (exact-match check), does NOT block $(id):
[[ "$input" == '$(' ]]
```

`$(id)` is not equal to `$(`, so the filter passes it straight through.

**PoC:**
```bash
./feedback.sh
# feedback: $(id)
```
The filter check effectively evaluates `"$(id)" != "$("`, which is true (no match), so the input is treated as safe and `eval "echo $(id)"` runs as **root**.

**Root cause:** the filtering logic was quietly downgraded from substring/pattern matching to exact-string matching, without anyone re-checking what security guarantee the original blacklist was providing. It's a one-character-class change with a critical impact. The deeper issue sits underneath the filter, though: `eval` on any user-influenced string is dangerous no matter how the check around it is written, because the check will always be reasoning about strings the attacker controls.

---

## 7. Root Access and Persistence

Reusing the same `eval` injection, whitespace filtering (where present) was bypassed with `${IFS}`, bash's Internal Field Separator, standing in for literal spaces, to append an SSH public key to root's `authorized_keys`:

```bash
$(echo${IFS}ssh-ed25519${IFS}AAAA...<attacker_pubkey>${IFS}>>${IFS}/root/.ssh/authorized_keys)
```

*(generate your own keypair first with `ssh-keygen -t ed25519`, then paste the `.pub` contents in place of the placeholder above)*

**Result:** direct `ssh root@<target-ip>` access. Retrieved `root.txt`.

---

## Impact

| Aspect | Rating | Notes |
|---|---|---|
| Confidentiality | Critical | Full filesystem read, both flags, all local credentials |
| Integrity | Critical | Arbitrary file write as root, demonstrated via SSH key injection |
| Availability | High | Full root control, service disruption trivially possible |
| Overall | **Critical** | Unauthenticated RCE chained to root with no authentication required at any step |
| CVSS 3.1 (estimated) | **9.8** | `AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`, network-reachable, low complexity, no privileges or user interaction needed, full impact on all three properties |

---

## Remediation

| Finding | Fix |
|---|---|
| Unauthenticated command injection in `index.php` | Never pass user input to `exec`/`shell_exec`/`system`. If shelling out is unavoidable, use parameterized execution (`escapeshellarg`) with a strict allow-list, never a deny-list. |
| Credentials hidden via broken-signature steganography | Don't rely on obscurity for secrets. Use a proper secrets manager or vault with access control and audit logging. |
| `feedback.sh` filter downgraded to exact-match | Remove `eval` entirely. There is almost no safe use case for `eval` on user input. If substitution is genuinely required, use a safe templating approach with no shell interpretation. |
| Sudo misconfiguration (root execution of a free-form-input script) | Apply least privilege: a script that processes arbitrary user input should never run as root via sudo. Treat blacklist filtering as insufficient by design; prefer allow-lists, or better, no shell interpretation at all. |
| `${IFS}` bypass of whitespace filtering | Filtering must account for shell metacharacter equivalents (`${IFS}`, `$IFS$9`, tabs), not just literal spaces. Another argument for eliminating `eval` rather than trying to filter around it. |

---

## Attack Chain Summary

```
Unauthenticated RCE (index.php?cmd=)
        │
        ▼
Reverse shell as web user
        │
        ▼
Recon: hidden dir (b64 passphrase) + oversized JPEG found
        │
        ▼
Fix JPEG magic bytes, steghide extract, creds.txt
        │
        ▼
Lateral move to 'deku', user.txt
        │
        ▼
sudo -l: feedback.sh (eval) runnable as root
        │
        ▼
Blacklist bypass: $(id) does not equal exact-match string "$("
        │
        ▼
${IFS} bypass for whitespace filtering
        │
        ▼
Inject SSH key into /root/.ssh/authorized_keys
        │
        ▼
SSH as root, root.txt
```

---

## Lessons Learned

- A blank 200 response can be more informative than a 404. It usually means a script ran but got no useful input, not that nothing exists at that path.
- Output encoding, like base64 on a response body, is not input validation. The two are frequently conflated under time pressure, but they address unrelated failure modes: one hides a result, the other would have prevented the compromise entirely.
- Filter regressions are often invisible on a quick read of the code. The difference between `==` and `== *...*` is one character with a critical security delta, and it won't jump out unless you're specifically checking what the filter is matching against.
- `${IFS}` and similar shell-metacharacter substitutions belong in a standard filter-bypass checklist, not as an edge case to remember only when stuck.
- Blacklist-based input validation fails predictably. This chain didn't require a novel bypass technique, only the observation that the check's scope had narrowed. Allow-listing, or removing the dangerous sink altogether, is the only fix that doesn't depend on the defender enumerating every future attacker's creativity.

---

*Tags: CTF, TryHackMe, Penetration Testing, Linux Privilege Escalation, Steganography*
