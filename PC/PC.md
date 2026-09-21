# PC

IP: 192.168.237.210

Nmap: 

```bash
┌──(kali㉿kali)-[~/OSCP/labs/PC]
└─$ sudo nmap -sV -sC -p- 192.168.237.210 --min-rate=5000                   
[sudo] password for kali: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-04 18:20 +1000
Stats: 0:00:27 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 95.50% done; ETC: 18:20 (0:00:00 remaining)
Nmap scan report for 192.168.237.210
Host is up (0.22s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 62:36:1a:5c:d3:e3:7b:e1:70:f8:a3:b3:1c:4c:24:38 (RSA)
|   256 ee:25:fc:23:66:05:c0:c1:ec:47:c6:bb:00:c7:4f:53 (ECDSA)
|_  256 83:5c:51:ac:32:e5:3a:21:7c:f6:c2:cd:93:68:58:d8 (ED25519)
8000/tcp open  http    ttyd 1.7.3-a2312cb (libwebsockets 3.2.0)
|_http-title: ttyd - Terminal
|_http-server-header: ttyd/1.7.3-a2312cb (libwebsockets/3.2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 60.09 seconds
                                                                                                            

```

![Screenshot 2026-05-04 at 6.21.19 pm.png](PC/Screenshot_2026-05-04_at_6.21.19_pm.png)

After visiting the page on port 8000 I found that it takes us to a virtual shell on the server:

![Screenshot 2026-05-04 at 6.35.54 pm.png](PC/Screenshot_2026-05-04_at_6.35.54_pm.png)

While enumerating I found a port listening on the lookback address:

```bash
user@pc:/opt$ 
user@pc:/opt$ ss -ntlp
State         Recv-Q        Send-Q                Local Address:Port                  Peer Address:Port        Process                                 
LISTEN        0             128                         0.0.0.0:8000                       0.0.0.0:*            users:(("ttyd",pid=1045,fd=12))        
LISTEN        0             4096                  127.0.0.53%lo:53                         0.0.0.0:*                                                   
LISTEN        0             128                         0.0.0.0:22                         0.0.0.0:*                                                   
LISTEN        0             2048                      127.0.0.1:65432                      0.0.0.0:*                                                   
LISTEN        0             128                            [::]:22                            [::]:*                                                   
user@pc:/opt$ 
```

![Screenshot 2026-05-04 at 6.50.00 pm.png](PC/Screenshot_2026-05-04_at_6.50.00_pm.png)

As well as an odd [rpc.py](http://rpc.py) file within the /opt directory: 

```bash
user@pc:/opt$ ls -la
total 16
drwxr-xr-x  3 root root 4096 Aug 25  2023 .
drwxr-xr-x 19 root root 4096 Jun 15  2022 ..
drwx--x--x  4 root root 4096 Jun 28  2023 containerd
-rw-r--r--  1 root root  625 Aug 25  2023 rpc.py
user@pc:/opt$ 

```

![Screenshot 2026-05-04 at 6.50.31 pm.png](PC/Screenshot_2026-05-04_at_6.50.31_pm.png)

So I created a SSH reverse port forward to access the port on my local kali: 

```bash
┌──(kali㉿kali)-[~/OSCP/labs/PC]
└─$ sudo systemctl start ssh  
```

![Screenshot 2026-05-04 at 6.52.45 pm.png](PC/Screenshot_2026-05-04_at_6.52.45_pm.png)

```bash
user@pc:/opt$ ssh -R 65432:127.0.0.1:65432 kali@192.168.45.217
kali@192.168.45.217's password: 
Linux kali 6.19.11+kali-arm64 #1 SMP PREEMPT Kali 6.19.11-1kali1 (2026-04-09) aarch64

The programs included with the Kali GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Kali GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Mon May  4 18:33:16 2026 from 192.168.237.210
┏━(Message from Kali developers)
┃
┃ This is a minimal installation of Kali Linux, you likely
┃ want to install supplementary tools. Learn how:
┃ ⇒ https://www.kali.org/docs/troubleshooting/common-minimum-setup/
┃
┗━(Run: “touch ~/.hushlogin” to hide this message)
┌──(kali㉿kali)-[~]
```

![Screenshot 2026-05-04 at 6.51.02 pm.png](PC/Screenshot_2026-05-04_at_6.51.02_pm.png)

I then tried to scan the port using nmap and noticed it was running uvicorn: 

```bash
┌──(kali㉿kali)-[~/OSCP/labs/PC]
└─$ sudo nmap -sV -sC -p65432 127.0.0.1 --min-rate=5000
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-04 18:26 +1000
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000064s latency).

PORT      STATE SERVICE VERSION
65432/tcp open  http    Uvicorn
|_http-server-header: uvicorn
|_http-title: Site doesn't have a title (text/plain; charset=utf-8).

```

![Screenshot 2026-05-04 at 6.52.00 pm.png](PC/Screenshot_2026-05-04_at_6.52.00_pm.png)

However, any attempts to access the webpage via the browser failed:

![Screenshot 2026-05-04 at 6.52.58 pm.png](PC/Screenshot_2026-05-04_at_6.52.58_pm.png)

While researching uvicorn server online I noticed an exploit for [RPC.py](http://RPC.py) which is what was found on the /opt directory: 

![Screenshot 2026-05-04 at 6.53.34 pm.png](PC/Screenshot_2026-05-04_at_6.53.34_pm.png)

After downloading the exploit and cleaning the file, I executed it with a reverse shell which worked successfully: 

![Screenshot 2026-05-04 at 6.54.57 pm.png](PC/Screenshot_2026-05-04_at_6.54.57_pm.png)

![Screenshot 2026-05-04 at 6.55.07 pm.png](PC/Screenshot_2026-05-04_at_6.55.07_pm.png)

Gaining the root flag: 

![Screenshot 2026-05-04 at 6.55.33 pm.png](PC/Screenshot_2026-05-04_at_6.55.33_pm.png)

Flag: 12783fb524ebc204ea17de4e27b533fa