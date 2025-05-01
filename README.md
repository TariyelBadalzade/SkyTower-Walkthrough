# SkyTower - CTF Writeup

## Introduction

This writeup walks through the exploitation process of the **SkyTower** CTF machine. It involves basic web exploitation, SSH tunneling tricks, manual enumeration, and privilege escalation attempts—all valuable skills for a penetration tester in a simulated environment.

---

## Web Exploitation

We start by identifying the IP address of the target machine using `netdiscover`.
```bash
netdiscover
```
Once we identify the IP, we perform a comprehensive scan using `nmap` to find open ports and running services:

![image](https://github.com/user-attachments/assets/52cd1b3e-c487-46fd-9e49-629fc2455acc)


Accessing the target via a web browser presents a basic website with a login form.

![image](https://github.com/user-attachments/assets/4576f5ee-f5ec-4033-8514-5b4927b0883d)


Testing the input field with a single quote (`'`) triggers an SQL error, indicating a possible **SQL Injection** vulnerability.

![image](https://github.com/user-attachments/assets/1e11a046-9572-4c97-87e3-fa61c6d88417)


```sql
' || 1=1 #
```
Since `'OR'` is filtered, we use logical OR with `||` instead:

![image](https://github.com/user-attachments/assets/5fc4114f-7e73-4699-ba59-5d726d3d1edd)

This bypasses the login successfully.

![image](https://github.com/user-attachments/assets/4d63cea0-08b4-4cf3-a028-079096742b13)

---

## SSH Enumeration Attempt

We discover an SSH service; however, it’s filtered or inaccessible directly.

![image](https://github.com/user-attachments/assets/c9a26ce6-249e-4931-ad17-e29e0930e938)

With some research, we find a tool named [**Proxytunnel**](https://github.com/proxytunnel/proxytunnel) that allows us to **masquerade as localhost**—essentially tricking the system into thinking we're a local user.

![image](https://github.com/user-attachments/assets/c0f70934-7be0-4cb2-afb6-44220fcf7067)

![image](https://github.com/user-attachments/assets/2b000bb2-5a15-4987-a859-b49d80aea6b4)

Although we attempt to access SSH via:

```bash
ssh john@127.0.0.1 -p 1234
```

We're still denied access. To investigate, we attempt to read the `.bashrc` file:

```bash
ssh john@127.0.0.1 -p 1234 cat .bashrc
```

![image](https://github.com/user-attachments/assets/f59f98fb-97ec-42cb-9092-a118467487ce)

The content of `.bashrc` likely includes a script preventing our access. We decide to **delete** the file:

```bash
ssh john@127.0.0.1 -p 1234 rm .bashrc
```

![image](https://github.com/user-attachments/assets/b7ef8976-23b7-42ce-9a06-bcae9109a06a)

---

## Web Directory Enumeration

After searching through several directories, we locate a `login.php` file under `/var/www`. It contains **MySQL credentials**, which allow further access.

![image](https://github.com/user-attachments/assets/baa8265f-8a71-473c-b5de-e3d0510a6a3a)

```bash
mysql -uroot -proot
```

![image](https://github.com/user-attachments/assets/a069e5b4-9de0-4f21-827f-0261de2b9f89)

```sql
show databases
```

![image](https://github.com/user-attachments/assets/205d2ad3-c17f-45ae-99f8-2333ad97d81f)

```sql
use SktTech
```

![image](https://github.com/user-attachments/assets/ccd6c80c-047b-47dd-bbde-b1e4336c8313)

```sql
select * from login
```

![image](https://github.com/user-attachments/assets/4d3ba700-4b5a-47f7-9e94-d8868362d558)

---

## Lateral Movement

With these credentials, we attempt to switch between user accounts and use:

![image](https://github.com/user-attachments/assets/36689308-150b-4e54-bcd9-ae6ebe229142)


```bash
sudo -l
```

to check for available privileges. However, we face the same `.bashrc` blocking issue as before.

![image](https://github.com/user-attachments/assets/75880421-1815-407c-b76e-2be370f6ee19)

---

## Sensitive File Access

Despite the restrictions, we manage to read both the `/etc/passwd` and `/etc/shadow` files using an existing vulnerability.

![image](https://github.com/user-attachments/assets/e2af086a-4253-4814-9e7b-7476077ca3a3)

Unfortunately, **John the Ripper** proves ineffective in cracking the hashes at this point.

---

## Final Steps and Flag Capture

Despite many privilege escalation attempts using `sudo cat`, no clear path is found. On further research, we locate a writeup of this machine on [VulnHub](https://www.vulnhub.com/entry/skytower-1,96/), which hints that the flag is located in the `/root` directory.

![image](https://github.com/user-attachments/assets/51936a6f-1bd9-4971-897e-3ddc9031b74c)

After several combinations and attempts, we finally succeed in obtaining the password-protected flag file and capture it.

![image](https://github.com/user-attachments/assets/6c2e4bf4-87a1-4504-b3e4-05735fba6845)

---

## Summary

- ✅ Performed SQL injection login bypass  
- ✅ Explored SSH localhost tunneling and `.bashrc` manipulation  
- ✅ Recovered MySQL credentials from web source files  
- ✅ Accessed sensitive system files  
- ✅ Located the final flag through enumeration and password brute-forcing  

SkyTower offers a great blend of web and system-level challenges, especially in simulating real-world misconfigurations like dangerous `.bashrc` entries and poor credential management.
