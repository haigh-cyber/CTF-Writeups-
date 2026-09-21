# Jacko

IP: 192.168.132.66

nmap: 

![Screenshot 2026-02-02 at 12.31.10 pm.png](Jacko/Screenshot_2026-02-02_at_12.31.10_pm.png)

![Screenshot 2026-02-02 at 12.32.10 pm.png](Jacko/Screenshot_2026-02-02_at_12.32.10_pm.png)

Masscan: 

Port 80:

![Screenshot 2026-02-02 at 12.32.53 pm.png](Jacko/Screenshot_2026-02-02_at_12.32.53_pm.png)

Port 8082:

![Screenshot 2026-02-02 at 12.33.32 pm.png](Jacko/Screenshot_2026-02-02_at_12.33.32_pm.png)

On port 8082 I found a version number of H2 Console (1.4.199):

![Screenshot 2026-02-02 at 12.41.34 pm.png](Jacko/Screenshot_2026-02-02_at_12.41.34_pm.png)

After a quick search on google I was able to find an exploit: 

![Screenshot 2026-02-02 at 12.42.19 pm.png](Jacko/Screenshot_2026-02-02_at_12.42.19_pm.png)

Following the exploit steps I was able to gain code execution on the host with the POC being the execution of whoami and the return output being jacko\tony:

![Screenshot 2026-02-02 at 12.42.37 pm.png](Jacko/Screenshot_2026-02-02_at_12.42.37_pm.png)

The windows commands that were executable are very limited on the host 

There was no python, php, perl, netcat, powershell

I attempted to retrieve nc.exe from my own host however due to the OS of the host being 64x I had to download nc64.exe from github and then retrieve the file from my host 

In doing so, we were able to gain a reverse shell on the host :

![Screenshot 2026-02-02 at 2.38.42 pm.png](Jacko/Screenshot_2026-02-02_at_2.38.42_pm.png)

Local user Flag:

![Screenshot 2026-02-02 at 2.40.17 pm.png](Jacko/Screenshot_2026-02-02_at_2.40.17_pm.png)

After gaining the shell, I looked at the available privileges for the user tony and noticed SeImpersonatePrivilege was available so I tried using SigmaPotato.exe to gain System level command execution: 

![Screenshot 2026-02-02 at 3.10.13 pm.png](Jacko/Screenshot_2026-02-02_at_3.10.13_pm.png)

I first tried adding tony as an administrator and it was successful but I wasn’t able to view the Administrator directory: 

![Screenshot 2026-02-02 at 3.11.32 pm.png](Jacko/Screenshot_2026-02-02_at_3.11.32_pm.png)

I also tried to change the administrator password to then able to use runas and gain a shell as the administrator user:

![Screenshot 2026-02-02 at 3.12.28 pm.png](Jacko/Screenshot_2026-02-02_at_3.12.28_pm.png)

However, runas would not allow me to input a password:

![Screenshot 2026-02-02 at 3.13.21 pm.png](Jacko/Screenshot_2026-02-02_at_3.13.21_pm.png)

My final attempt was to create another reverse shell but this time since sigmapotato executes with SYSTEM level commands I hoped it would give me a SYSTEM level shell:

![Screenshot 2026-02-02 at 3.14.27 pm.png](Jacko/Screenshot_2026-02-02_at_3.14.27_pm.png)

I used the same nc64 file I downloaded earlier and was able to gain a SYSTEM level shell on the host and view the proof.txt file 

PWNED!!!

Additional research: 

Some other writeups used printerSpool.exe for priv esc: 

```html
https://github.com/itm4n/PrintSpoofer/releases 
```