# Extplorer

IP: 192.168.175.16

Nmap scan: 

![Screenshot 2026-04-24 at 9.09.30 am.png](Extplorer/Screenshot_2026-04-24_at_9.09.30_am.png)

Expanding my nmap scan search for all scripts including http, I found an interesting directory: 

![Screenshot 2026-04-24 at 10.32.46 am.png](Extplorer/Screenshot_2026-04-24_at_10.32.46_am.png)

Which takes me to eXtplorer file manager: 

![Screenshot 2026-04-24 at 10.33.21 am.png](Extplorer/Screenshot_2026-04-24_at_10.33.21_am.png)

Tried admin:admin as the username and password, and successfully logged in: 

![Screenshot 2026-04-24 at 10.39.51 am.png](Extplorer/Screenshot_2026-04-24_at_10.39.51_am.png)

Because I had full access to the directory including uploading files, I uploaded a php reverse shell onto the system:

![Screenshot 2026-04-24 at 10.44.38 am.png](Extplorer/Screenshot_2026-04-24_at_10.44.38_am.png)

![Screenshot 2026-04-24 at 10.43.52 am.png](Extplorer/Screenshot_2026-04-24_at_10.43.52_am.png)

I then started a listener on my local kali and opened the shell.php file in a new tab to catch the reverse shell: 

![Screenshot 2026-04-24 at 10.45.00 am.png](Extplorer/Screenshot_2026-04-24_at_10.45.00_am.png)

![Screenshot 2026-04-24 at 10.45.25 am.png](Extplorer/Screenshot_2026-04-24_at_10.45.25_am.png)

I then upgraded the shell using python3 pty module and export xterm:

![Screenshot 2026-04-24 at 10.46.02 am.png](Extplorer/Screenshot_2026-04-24_at_10.46.02_am.png)

Looking at files within the filemanager directory, we found hashed credentials for the user dora:

![Screenshot 2026-04-24 at 10.56.11 am.png](Extplorer/Screenshot_2026-04-24_at_10.56.11_am.png)

Using hashcat and the found hash I was able to crack the Dora user password:

![Screenshot 2026-04-24 at 12.44.21 pm.png](Extplorer/Screenshot_2026-04-24_at_12.44.21_pm.png)

![Screenshot 2026-04-24 at 12.45.29 pm.png](Extplorer/Screenshot_2026-04-24_at_12.45.29_pm.png)

Since SSH authentication using a password is disabled, I used the su command to change to the dora user: 

![Screenshot 2026-04-24 at 12.46.32 pm.png](Extplorer/Screenshot_2026-04-24_at_12.46.32_pm.png)

And gained the flag

![Screenshot 2026-04-24 at 12.46.46 pm.png](Extplorer/Screenshot_2026-04-24_at_12.46.46_pm.png)

While looking at the [linpeas.sh](http://linpeas.sh) output, I noticed that the dora user has 6(disk) privileges:

![Screenshot 2026-04-24 at 1.26.34 pm.png](Extplorer/Screenshot_2026-04-24_at_1.26.34_pm.png)

Disk group misconfiguration is a Linux privilege escalation technique because it grants a low-privileged user raw read/write access to physical storage devices 

https://www.hackingarticles.in/disk-group-privilege-escalation/

As such I listed the current drives and looked for the one mounted to the root directory / 

![Screenshot 2026-04-24 at 1.27.59 pm.png](Extplorer/Screenshot_2026-04-24_at_1.27.59_pm.png)

Then using the debugfs command I was able to open an instance to that mounted file system. I then wanted to test it worked so I used cat to view the root flag:

![Screenshot 2026-04-24 at 1.28.40 pm.png](Extplorer/Screenshot_2026-04-24_at_1.28.40_pm.png)

My initial thought was to retrieve the rsa_key within the /etc/ssh directory but this didnt work. There was also no id_rsa file within the /root directory either.  

![Screenshot 2026-04-24 at 1.29.33 pm.png](Extplorer/Screenshot_2026-04-24_at_1.29.33_pm.png)

So I read the /etc/passwd and /etc/shadow file and exported them to my local kali: 

![Screenshot 2026-04-24 at 1.30.50 pm.png](Extplorer/Screenshot_2026-04-24_at_1.30.50_pm.png)

I then used unshadow to combine the two files before using johntheripper to attempt to crack the password hash of root:

![Screenshot 2026-04-24 at 1.31.02 pm.png](Extplorer/Screenshot_2026-04-24_at_1.31.02_pm.png)

after some time the password was cracked root:explorer:

![Screenshot 2026-04-24 at 1.31.44 pm.png](Extplorer/Screenshot_2026-04-24_at_1.31.44_pm.png)

Because SSH using password and rsa key is disabled, I used the su command on the shell I already had to change users to root and retrieve the flag:

![Screenshot 2026-04-24 at 1.32.18 pm.png](Extplorer/Screenshot_2026-04-24_at_1.32.18_pm.png)

![Screenshot 2026-04-24 at 1.38.27 pm.png](Extplorer/Screenshot_2026-04-24_at_1.38.27_pm.png)