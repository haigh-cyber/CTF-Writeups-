
![[Screenshot 2026-10-12 at 8.54.23 pm.png]]
Nmap: 
```
──(kali㉿ldinsle123)-[~]
└─$ nmap -sV -sC -p- 192.168.154.22 --min-rate=5000 -o nmap
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-12 20:53 +1100
Nmap scan report for 192.168.154.22
Host is up (0.19s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b9:bc:8f:01:3f:85:5d:f9:5c:d9:fb:b6:15:a0:1e:74 (ECDSA)
|_  256 53:d9:7f:3d:22:8a:fd:57:98:fe:6b:1a:4c:ac:79:67 (ED25519)
3000/tcp open  http    WEBrick httpd 1.7.0 (Ruby 3.0.2 (2021-07-07))
|_http-server-header: WEBrick/1.7.0 (Ruby/3.0.2/2021-07-07)
|_http-title: RubyDome HTML to PDF
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.92 seconds

```


WeBirck http 1.7.0

![[Screenshot 2026-10-12 at 8.54.50 pm.png]]

The fucking page didn't load and I spent 20 minutes trying to find an exploit or vulnerbility 

Dickheads

Now it load:

![[Screenshot 2026-10-12 at 9.08.15 pm.png]]

When attempting to retrieve a file from the python webserver on my local host, im met with the following error: **ImproperWkhtmltopdfExitStatus**

![[Screenshot 2026-10-12 at 9.25.28 pm.png]]


A quick google search and multiple exploits are given: 
![[Screenshot 2026-10-12 at 9.25.04 pm.png]]

https://www.exploit-db.com/exploits/51293

Downloaded the exploit from exploitdb and ran it against port 3000 on the webserver to gain a shell (The exploit had a shell functionality): ![[Screenshot 2026-10-12 at 9.39.30 pm.png]]



And now im andrew: 
![[Screenshot 2026-10-12 at 9.39.52 pm.png]]

Stablised the shell and got the flag: 
![[Screenshot 2026-10-12 at 9.42.34 pm.png]]

FLAG: fd3032755186b09bc5647a6bc658a923



Sudo can be used without a password when running ruby against app.rb: 

And andrew can edit the file lol;

![[Screenshot 2026-10-12 at 9.43.34 pm.png]]


Created a reverse shell on my local and retrieved it on the target: 

![[Screenshot 2026-10-12 at 9.56.03 pm.png]]

After running the file with sudo: 
![[Screenshot 2026-10-12 at 9.56.37 pm.png]]


We got a webshell as root: 
![[Screenshot 2026-10-12 at 9.57.01 pm.png]]![[Screenshot 2026-10-12 at 9.57.22 pm.png]]


FLAG:  a9495ccfbfb4c65391ccc77c0ceb1132