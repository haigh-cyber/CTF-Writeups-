Nmap:

```
┌──(kali㉿ldinsle123)-[~/OSCP/labs/Windows/Algernon]
└─$ sudo nmap -sV -sC -p- 192.168.103.65 --min-rate=5000
sudo: unable to resolve host ldinsle123: Name or service not known
[sudo] password for kali: 
Starting Nmap 7.99 ( <https://nmap.org> ) at 2026-09-21 15:45 +1000
Stats: 0:01:37 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 92.86% done; ETC: 15:47 (0:00:06 remaining)
Stats: 0:02:23 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 92.86% done; ETC: 15:48 (0:00:09 remaining)
Stats: 0:03:01 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 92.86% done; ETC: 15:49 (0:00:12 remaining)
Nmap scan report for 192.168.103.65
Host is up (0.35s latency).
Not shown: 65521 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 04-29-20  10:31PM       <DIR>          ImapRetrieval
| 09-20-26  10:42PM       <DIR>          Logs
| 04-29-20  10:31PM       <DIR>          PopRetrieval
|_04-29-20  10:32PM       <DIR>          Spool
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5040/tcp  open  unknown
9998/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-IIS/10.0
| http-title: Site doesn't have a title (text/html; charset=utf-8).
|_Requested resource was /interface/root
| uptime-agent-info: HTTP/1.1 400 Bad Request\x0D
| Content-Type: text/html; charset=us-ascii\x0D
| Server: Microsoft-HTTPAPI/2.0\x0D
| Date: Mon, 21 Sep 2026 05:49:00 GMT\x0D
| Connection: close\x0D
| Content-Length: 326\x0D
| \x0D
| <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN""<http://www.w3.org/TR/html4/strict.dtd>">\x0D
| <HTML><HEAD><TITLE>Bad Request</TITLE>\x0D
| <META HTTP-EQUIV="Content-Type" Content="text/html; charset=us-ascii"></HEAD>\x0D
| <BODY><h2>Bad Request - Invalid Verb</h2>\x0D
| <hr><p>HTTP Error 400. The request verb is invalid.</p>\x0D
|_</BODY></HTML>\x0D
17001/tcp open  remoting      MS .NET Remoting services
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-09-21T05:49:09
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required

Service detection performed. Please report any incorrect results at <https://nmap.org/submit/> .
Nmap done: 1 IP address (1 host up) scanned in 221.48 seconds

```


## Port 21 

FTP services allows anonymous logins

Directories:
```
04-29-20  10:31PM       <DIR>          ImapRetrieval
09-20-26  10:42PM       <DIR>          Logs
04-29-20  10:31PM       <DIR>          PopRetrieval
04-29-20  10:32PM       <DIR>          Spool
```



## Port 9998

HTTP service on Port 9998: ![[Screenshot 2026-09-21 at 4.01.01 pm.png]]



## Port 17001 (Intresting)

Since this port is non-standard, I thought i'd start looking into this one first. 

MS .NET Remoting services

![[Screenshot 2026-09-21 at 4.03.49 pm.png]]

https://www.pentestpad.com/port-exploit/port-17001-alt-service-alternative-service

![[Screenshot 2026-09-21 at 4.05.36 pm.png]]


Looks like theres a metasploit exploit for this: 
![[Screenshot 2026-09-21 at 4.09.30 pm.png]]

Using Metasploit I looked up any available exploits for the smartermail application:
![[Screenshot 2026-09-21 at 4.13.26 pm.png]]


Looks like I have a root shell lol: 
![[Screenshot 2026-09-21 at 4.14.58 pm.png]]


And just like that we were able to get the flag within the Administrators Desktop directory: 
![[Screenshot 2026-09-21 at 4.16.12 pm.png]]