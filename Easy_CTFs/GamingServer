# 🕹️ GamingServer — TryHackMe

## 🎯 Objective

The objective of this room was to gain access to the target machine and retrieve both flags:

- `user.txt`
- `root.txt`

---

## 🔍 Enumeration

### Tools Used
- `nmap`
- `gobuster`
- `feroxbuster`

I started with an Nmap scan to identify exposed services running on the target machine.

- **The scan revealed two open ports:

### Key Findings
- Open ports: **22, 80**
- Interesting services:
  - SSH service running on port 22
  - Apache web server hosted on port 80

**Nmap Scan Result:**

![Nmap_scan](https://imgur.com/zw9uxpn)

---

After checking port **80**, I found a website hosted by the server.

**Website Preview:**

![web_page](https://imgur.com/skp9gV8)

I manually explored the available pages, but nothing particularly interesting stood out.

As part of basic web enumeration, I inspected the website source code and discovered an interesting note.

This note contained a **username**, which became useful later during authentication attempts.

**Source Code Finding:**

![interest note](https://imgur.com/p5k9YQC)

---

To continue enumeration, I performed directory discovery using:

- `gobuster`
- `feroxbuster`

This revealed two interesting directories:

- `/uploads`
- `/secret`

**Directory Enumeration Result:**

![gobuster_scan](https://imgur.com/YYq2s0l)
![feroxbuster_scan](https://imgur.com/QZjtxxb)

### Interesting Findings

#### `/uploads`

This directory contained a file named:

```text
dict.lst
```

The file contained a list of common passwords, which later proved useful during credential attacks.

![uploads](https://imgur.com/jh5kRI0)

download it :

![dict.lst](https://imgur.com/JFLnYEs)

---

#### `/secret`

This directory contained a file named:

```text
secretKey
```

Upon inspection, it appeared to be an **SSH private key**.

![secret](https://imgur.com/04iphl9)

---

## Initial Access

### Vulnerability

Exposed sensitive files through improper web directory access.

### Exploit Method

At this stage, I had:

- A valid username
- An SSH private key

My first thought was to attempt SSH authentication using the discovered credentials.

However, during login, SSH requested a **passphrase** for the private key.

![ssh attempt](https://imgur.com/yJhZbUy)

At that moment, the passphrase was unknown.

**SSH Login Attempt:**

To recover it, I converted the SSH key into a crackable format using:

```bash
ssh2john secretKey > hash.txt
```

Then, I used the password list discovered earlier (`dict.lst`) to crack the pase.

The attack succeeded, revealing the correct passphrase.

![Cracked](https://imgur.com/Jt3MUr1)

After obtaining it, I attempted SSH login again , and successfully gained access to the target machine.

**Successful SSH Access:**

After gaining access, I navigated the system and successfully retrieved the first flag:

**User Flag:**

![ssh login](https://imgur.com/i7LL8KX)

---

## 🔼 Privilege Escalation

After performing internal enumeration, I discovered an interesting privilege-related finding:

The current user was a member of the **LXD group**.

### What is LXD?

LXD is a Linux container management system that can allow privileged container abuse when improperly configured.

**LXD Group Membership:**

![internal enum](https://imgur.com/ZPaFmTw)

After researching possible exploitation methods, I found a useful project:

The Alpine Builder script, which allows building a lightweight Linux image that can later be mounted through LXD to access the host filesystem.

The script can be found here:

https://github.com/saghul/lxd-alpine-builder

download it on your local machin , then transfer it to the target machin .

![alpine](https://imgur.com/LYFQcgE)
### Exploitation Steps
Source : https://gtfobins.org/gtfobins/lxd/#shell

![Commands](https://imgur.com/Nwt3Yrn)
#### 1. Import the Alpine image

```bash
lxc image import ./alpine*.tar.gz --alias image_name
```

#### 2. Creates a privileged container using the imported image.

```bash
lxc init image_name container_name -c security.privileged=true
```

#### 3. Mounts the host root filesystem inside the container.

```bash
lxc config device add container_name host-root disk source=/ path=/mnt recursive=true
```

#### 4. Starts the newly created container.

```bash
lxc start container_name
```

#### 5. Spawn a shell inside the container

```bash
lxc exec container_name /bin/sh
```

Because the host filesystem was mounted under `/mnt`, it became possible to access sensitive files directly from the host machine.

This ultimately led to retrieving:

**Root Flag:**

![Root Flage](https://imgur.com/cEKsV7f)

---

## ✅ Conclusion

This room demonstrated the importance of proper web enumeration and secure file exposure practices.

The attack chain involved:

1. Web enumeration
2. Sensitive file discovery
3. SSH key passphrase cracking
4. Initial SSH access
5. Privilege escalation through LXD group abuse

A strong reminder that small misconfigurations can lead to full system compromise.assphr
