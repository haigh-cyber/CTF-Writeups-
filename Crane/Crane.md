# Crane

IP: 192.168.160.146

Nmap:

![Screenshot 2026-02-14 at 12.59.32 pm.png](Crane/Screenshot_2026-02-14_at_12.59.32_pm.png)

Wrappalyzer: 

![Screenshot 2026-02-14 at 1.00.58 pm.png](Crane/Screenshot_2026-02-14_at_1.00.58_pm.png)

Login page: 

![Screenshot 2026-02-14 at 1.01.46 pm.png](Crane/Screenshot_2026-02-14_at_1.01.46_pm.png)

Admin:admin credentials worked: 

![Screenshot 2026-02-14 at 1.02.04 pm.png](Crane/Screenshot_2026-02-14_at_1.02.04_pm.png)

Found version numbers: 

![Screenshot 2026-02-14 at 1.03.41 pm.png](Crane/Screenshot_2026-02-14_at_1.03.41_pm.png)

Found an RCE vulnerability for SuiteCRM Versions prior to 7.12.4

![Screenshot 2026-02-14 at 1.05.15 pm.png](Crane/Screenshot_2026-02-14_at_1.05.15_pm.png)

The medium post points us to a POC on github 

![Screenshot 2026-02-14 at 1.06.43 pm.png](Crane/Screenshot_2026-02-14_at_1.06.43_pm.png)

POC was successful: 

![Screenshot 2026-02-14 at 1.12.24 pm.png](Crane/Screenshot_2026-02-14_at_1.12.24_pm.png)

Found that sudo would work with the /usr/sbin/service binary:

![Screenshot 2026-02-14 at 1.13.54 pm.png](Crane/Screenshot_2026-02-14_at_1.13.54_pm.png)

Used GTFObins to find a its vulnerability: 

![Screenshot 2026-02-14 at 1.14.26 pm.png](Crane/Screenshot_2026-02-14_at_1.14.26_pm.png)

Gained root privileges: 

![Screenshot 2026-02-14 at 1.14.52 pm.png](Crane/Screenshot_2026-02-14_at_1.14.52_pm.png)

Didn’t know there were two flags because there were no uses in the /home directory

But used find to find the local user flag: 

![Screenshot 2026-02-14 at 1.16.02 pm.png](Crane/Screenshot_2026-02-14_at_1.16.02_pm.png)