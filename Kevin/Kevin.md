# Kevin

IP: 192.168.214.45

Nmap:

![Screenshot 2026-02-03 at 7.31.01 pm.png](Kevin/Screenshot_2026-02-03_at_7.31.01_pm.png)

Port 80 login page: 

Able to login with default creds admin/admin

![Screenshot 2026-02-03 at 8.15.11 pm.png](Kevin/Screenshot_2026-02-03_at_8.15.11_pm.png)

Found that prior to HP Power Manager 4.10 versions were vulnerable to buffer overflow: 

![Screenshot 2026-02-03 at 8.17.47 pm.png](Kevin/Screenshot_2026-02-03_at_8.17.47_pm.png)

Searching the CVE I was able to find an exploit: 

![Screenshot 2026-02-03 at 8.19.13 pm.png](Kevin/Screenshot_2026-02-03_at_8.19.13_pm.png)

I ran the exploit using python2 and obtained a system level shell: 

![Screenshot 2026-02-03 at 8.19.39 pm.png](Kevin/Screenshot_2026-02-03_at_8.19.39_pm.png)

Learning: 

- Test all versions of python (python, python2 and python3)