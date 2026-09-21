# DVR4

IP: 192.168.160.179

Nmap:

![Screenshot 2026-02-15 at 10.20.57 am.png](DVR4/Screenshot_2026-02-15_at_10.20.57_am.png)

Found ‘Argus Surveillance’ http web proxy on port 8080:

![Screenshot 2026-02-15 at 10.24.05 am.png](DVR4/Screenshot_2026-02-15_at_10.24.05_am.png)

Looks like they’re running Argus Version 4.0

Exploit DB has multiple vulnerabilities for this version: 

![Screenshot 2026-02-15 at 10.25.02 am.png](DVR4/Screenshot_2026-02-15_at_10.25.02_am.png)

Following the Directory Traversal vulnerability we found it successful according to the POC:  

![Screenshot 2026-02-15 at 10.27.37 am.png](DVR4/Screenshot_2026-02-15_at_10.27.37_am.png)

Attempting to curl the **`C:\ProgramData\PY_Software\Argus Surveillance DVR\DVRParams.ini`** file:

![Screenshot 2026-02-15 at 12.08.15 pm.png](DVR4/Screenshot_2026-02-15_at_12.08.15_pm.png)

The file is meant to contain password hashes however non were within the file 

Proxy port scan returned nothing 

![Screenshot 2026-02-15 at 1.00.14 pm.png](DVR4/Screenshot_2026-02-15_at_1.00.14_pm.png)

Found a link to the user manual on the help page: 

![Screenshot 2026-02-15 at 1.07.00 pm.png](DVR4/Screenshot_2026-02-15_at_1.07.00_pm.png)

Wasn’t accesable via the web browser unless using adobe acrobat: 

![Screenshot 2026-02-15 at 1.07.03 pm.png](DVR4/Screenshot_2026-02-15_at_1.07.03_pm.png)

I attempted to curl the webpage and found the location for the manual: 

![Screenshot 2026-02-15 at 1.07.06 pm.png](DVR4/Screenshot_2026-02-15_at_1.07.06_pm.png)

We found the manual location: 

![Screenshot 2026-02-15 at 1.06.28 pm.png](DVR4/Screenshot_2026-02-15_at_1.06.28_pm.png)

Nothing notable within the user manual 

Potential users: 

![Screenshot 2026-02-15 at 1.44.56 pm.png](DVR4/Screenshot_2026-02-15_at_1.44.56_pm.png)

Able to view the viewer user’s private key: 

![Screenshot 2026-02-15 at 1.56.51 pm.png](DVR4/Screenshot_2026-02-15_at_1.56.51_pm.png)

Created an id_rsa file with the private key of the viewer user: 

![Screenshot 2026-02-15 at 1.59.11 pm.png](DVR4/Screenshot_2026-02-15_at_1.59.11_pm.png)

Then attempted to connect to the user via SSH: 

![Screenshot 2026-02-15 at 1.59.38 pm.png](DVR4/Screenshot_2026-02-15_at_1.59.38_pm.png)

SUCCESS MF: 

![Screenshot 2026-02-15 at 1.59.58 pm.png](DVR4/Screenshot_2026-02-15_at_1.59.58_pm.png)

## Priv Esc

Found a program in the program file directory that doesn’t look default (RUXIM):

![Screenshot 2026-02-15 at 3.21.27 pm.png](DVR4/Screenshot_2026-02-15_at_3.21.27_pm.png)

After a quick google search found that it’s vulnerable to priv esc via 

[https://www.synacktiv.com/en/advisories/windows-10-plugscheduler-elevation-of-privilege](https://www.synacktiv.com/en/advisories/windows-10-plugscheduler-elevation-of-privilege)

There is currently two scheduled tasks:

![Screenshot 2026-02-15 at 3.18.25 pm.png](DVR4/Screenshot_2026-02-15_at_3.18.25_pm.png)

![Screenshot 2026-02-15 at 3.19.40 pm.png](DVR4/Screenshot_2026-02-15_at_3.19.40_pm.png)

- At system start up

However, when trying to restart host I get an ‘access denied’ error even though I have the correct privileges: 

![Screenshot 2026-02-15 at 3.32.43 pm.png](DVR4/Screenshot_2026-02-15_at_3.32.43_pm.png)

I recalled there was a config file for Argus DVR that contained the username and password for users: 

![Screenshot 2026-02-15 at 5.42.28 pm.png](DVR4/Screenshot_2026-02-15_at_5.42.28_pm.png)

It contained the Administrator password for Argus DVR

![Screenshot 2026-02-15 at 5.43.16 pm.png](DVR4/Screenshot_2026-02-15_at_5.43.16_pm.png)

When initially enumerating the version of Argus I found a priv esc vuln in the way the passwords are encrypted. They are apparently weak: 

![Screenshot 2026-02-15 at 5.44.12 pm.png](DVR4/Screenshot_2026-02-15_at_5.44.12_pm.png)

With the help from a script we found that the password is 14WatchD0G$:

![Screenshot 2026-02-15 at 5.44.54 pm.png](DVR4/Screenshot_2026-02-15_at_5.44.54_pm.png)

We weren’t able to SSH into the host using this password 

However, we could use the runas command to attempt to execute command with administrator

This worked and we were able to add the `viewer` user in the administrators group: 

![Screenshot 2026-02-15 at 5.46.35 pm.png](DVR4/Screenshot_2026-02-15_at_5.46.35_pm.png)

We were then able to reconnect via SSH to the Viewers group and have administrator privileges: 

![Screenshot 2026-02-15 at 5.42.02 pm.png](DVR4/Screenshot_2026-02-15_at_5.42.02_pm.png)