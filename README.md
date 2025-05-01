# SkyTower - CTF Writeup

## Introduction

This writeup walks through the exploitation process of the **SkyTower** CTF machine. It involves basic web exploitation, SSH tunneling tricks, manual enumeration, and privilege escalation attempts—all valuable skills for a penetration tester in a simulated environment.

---

## Web Exploitation

Accessing the target via a web browser presents a basic website with a login form.

Testing the input field with a single quote (`'`) triggers an SQL error, indicating a possible **SQL Injection** vulnerability. Since `'OR'` is filtered, we use logical OR with `||` instead:

```sql
' || 1=1 #
```

This bypasses the login successfully.

---

## SSH Enumeration Attempt

We discover an SSH service; however, it’s filtered or inaccessible directly. With some research, we find a tool that allows us to **masquerade as localhost**—essentially tricking the system into thinking we're a local user.

Although we attempt to access SSH via:

```bash
ssh john@127.0.0.1 -p 1234
```

We're still denied access. To investigate, we attempt to read the `.bashrc` file:

```bash
ssh john@127.0.0.1 -p 1234 cat .bashrc
```

The content of `.bashrc` likely includes a script preventing our access. We decide to **delete** the file:

```bash
ssh john@127.0.0.1 -p 1234 rm .bashrc
```

---

## Web Directory Enumeration

After searching through several directories, we locate a `login.php` file under `/var/www`. It contains **MySQL credentials**, which allow further access.

---

## Lateral Movement

With these credentials, we attempt to switch between user accounts and use:

```bash
sudo -l
```

to check for available privileges. However, we face the same `.bashrc` blocking issue as before.

---

## Sensitive File Access

Despite the restrictions, we manage to read both the `/etc/passwd` and `/etc/shadow` files using an existing vulnerability.

Unfortunately, **John the Ripper** proves ineffective in cracking the hashes at this point.

---

## Final Steps and Flag Capture

Despite many privilege escalation attempts using `sudo cat`, no clear path is found. On further research, we locate a writeup of this machine on [VulnHub](https://www.vulnhub.com/), which hints that the flag is located in the `/root` directory.

After several combinations and attempts, we finally succeed in obtaining the password-protected flag file and capture it.

---

## Summary

- ✅ Performed SQL injection login bypass  
- ✅ Explored SSH localhost tunneling and `.bashrc` manipulation  
- ✅ Recovered MySQL credentials from web source files  
- ✅ Accessed sensitive system files  
- ✅ Located the final flag through enumeration and password brute-forcing  

SkyTower offers a great blend of web and system-level challenges, especially in simulating real-world misconfigurations like dangerous `.bashrc` entries and poor credential management.
