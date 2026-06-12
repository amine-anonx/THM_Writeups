# Mr. Robot [THM]
![img](https://i.imgur.com/cpF4sM4.png)

**Difficulty:** Medium

---

## Objective

The objective of this room is to retrieve the following keys:

* `key-1-of-3.txt`
* `key-2-of-3.txt`
* `key-3-of-3.txt`

---

## Reconnaissance

### Tools Used

* `nmap`
* `feroxbuster`

---

An Nmap scan was performed to identify open services on the target machine.

The scan revealed the following ports:

* **22 (SSH)**
* **80 (HTTP)**
* **443 (HTTPS)**

This indicated that a web server was running on the target.

**Nmap Scan Result:**

![nmap_scan](https://i.imgur.com/2UmtNUd.png) 

**Feroxbuster Scan Result:**

![img](https://i.imgur.com/PmTSoQI.png)

---

### Web Application Discovery

After accessing the web server, I discovered an interactive terminal interface.

The application only accepted a limited set of commands:

```text id="mr1"
prepare
fsociety
question
inform
wakeup
join
```

**Web Interface:**

![web_page](https://i.imgur.com/X4jJ5o6.png)
---

### Directory Enumeration

Further enumeration revealed sensitive files in `robots.txt`:

* `key-1-of-3.txt`
* `fsocity.dic`

The first file contained **Key 1 of the challenge**.
![img](https://i.imgur.com/YBddDDR.png)

The second file was a wordlist that would later be used for credential attacks.

**robots.txt Findings:**
![img](https://i.imgur.com/A2SaHnM.png)
---

---
* `Downloading the wordlist`
![img](https://i.imgur.com/5QhCYdL.png)

### WordPress Discovery

Directory enumeration revealed a WordPress login page:

```text id="mr2"
/wp-login.php
```

The login page confirmed a WordPress installation running on the target.

**WordPress Login Page:**

![img](https://i.imgur.com/sTNziG6.png)

---

### Username Enumeration

During authentication testing, the system returned verbose error messages.

When testing invalid credentials:

```text id="mr3"
admin:admin
```

The response:

* `Invalid username` → indicates username does not exist

After a cracking operation that revealed a valid username 
![img](https://i.imgur.com/NlcPuPJ.png)
the response changed to:

* `Incorrect password for username Elliot`

This confirmed that the username **Elliot** was valid.

![img](https://i.imgur.com/xwbcGri.png)
---

## Initial Access

### Vulnerability

WordPress theme editor allows direct modification of PHP files leading to remote code execution.

---

### Exploit Method

After gaining access to the WordPress admin panel, I navigated to:

```text id="mr4"
Appearance → Theme Editor → archive.php
```

Alternatively:

```text id="mr5"
http://mrrobot.thm/wp-admin/theme-editor.php?file=archive.php&theme=twentyfifteen
```

I replaced the contents of `archive.php` with a **PHP reverse shell payload**.

![img](https://i.imgur.com/MnxdjQ6.png)
---

### Shell Execution

To trigger the payload, I accessed:

```text id="mr6"
http://mrrobot.thm/wp-content/themes/twentyfifteen/archive.php
```

This successfully executed the reverse shell and provided initial system access.

**Reverse Shell Access:**

![img](https://i.imgur.com/Szsb8mt.png)

---

## Privilege Escalation

### Internal Enumeration

During system enumeration, a file was discovered:

```text id="mr7"
password.raw-md5
```

The file contained:

```text id="mr8"
robot:c3fcd3d76192e4007dfb496cca67e13b
```

This indicated:

* Username: `robot`
* Password: MD5 hash

---

### Credential Cracking

The MD5 hash was successfully cracked, revealing the plaintext password.

![img](https://i.imgur.com/KpVHCeu.png)

This allowed SSH login as the user **robot**.

```bash id="mr9"
ssh robot@<TARGET_IP>
```
![img](https://i.imgur.com/sIZNbeS.png)

here we go **Key 2 of the challenge** has been found .

---

### Privilege Escalation Vector

Further enumeration revealed an interesting SUID binary:

```text id="mr10"
/usr/local/bin/nmap
```

This binary had the SUID bit set, allowing elevated execution.

---

### Exploitation Method

The following command was used:

```bash id="mr11"
nmap --interactive
```

Inside interactive mode:

```text id="mr12"
!sh
```

This spawned a root shell.

---

## Root Access

A root shell was successfully obtained, allowing full system compromise and retrieval of:

* `key-3-of-3`

---

## Summary

Attack chain:


1. Reconnaissance : Services scanning
2. Web enumeration : dir fuzzing, robots.txt discovery, WordPress login page identification
3. Credentials enumeration : usename:password cracking
4. WordPress theme editor RCE
5. Reverse shell execution 
6. Initial access : Credential discovery (MD5 hash)
7. Lateral movement through SSH login as robot
8. Internal enumeration
9. SUID nmap privilege escalation
10. Root shell via interactive mode
