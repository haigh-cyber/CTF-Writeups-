# Codo

IP: 192.168.153.23

Nmap: 

![Screenshot 2026-02-13 at 1.24.40 pm.png](Codo/Screenshot_2026-02-13_at_1.24.40_pm.png)

HTTP web page running CodoForum: 

![Screenshot 2026-02-13 at 1.58.12 pm.png](Codo/Screenshot_2026-02-13_at_1.58.12_pm.png)

Standard login page take Admin:Admin as the credentials: 

![Screenshot 2026-02-13 at 1.58.25 pm.png](Codo/Screenshot_2026-02-13_at_1.58.25_pm.png)

![Screenshot 2026-02-13 at 1.58.36 pm.png](Codo/Screenshot_2026-02-13_at_1.58.36_pm.png)

Looking around and I found an admin login page

![Screenshot 2026-02-13 at 2.00.14 pm.png](Codo/Screenshot_2026-02-13_at_2.00.14_pm.png)

Same credentials Admin:admin work here: 

![Screenshot 2026-02-13 at 2.01.16 pm.png](Codo/Screenshot_2026-02-13_at_2.01.16_pm.png)

Found a Abritrary File Upload Vuln: https://nvd.nist.gov/vuln/detail/CVE-2022-31854

![Screenshot 2026-02-13 at 1.56.18 pm.png](Codo/Screenshot_2026-02-13_at_1.56.18_pm.png)

Navigated to the Global Configurations section and uploaded a PHP reverse shell 

![Screenshot 2026-02-13 at 2.02.55 pm.png](Codo/Screenshot_2026-02-13_at_2.02.55_pm.png)

Then I was able to execute it by navigating to the http://192.168.153.23/sites/default/assets/img/attachments/php-reverse-shell.php:

![Screenshot 2026-02-13 at 2.01.44 pm.png](Codo/Screenshot_2026-02-13_at_2.01.44_pm.png)

Ran linpeas: 

Found a possible priv esc vuln: 

![Screenshot 2026-02-13 at 2.12.27 pm.png](Codo/Screenshot_2026-02-13_at_2.12.27_pm.png)

https://nvd.nist.gov/vuln/detail/cve-2021-3560

It was found that polkit could be tricked into bypassing the credential checks for D-Bus requests, elevating the privileges of the requestor to the root user. This flaw could be used by an unprivileged local attacker to, for example, create a new local administrator. The highest threat from this vulnerability is to data confidentiality and integrity as well as system availability.

![Screenshot 2026-02-13 at 2.17.30 pm.png](Codo/Screenshot_2026-02-13_at_2.17.30_pm.png)

Found credentials to their MySQL server: 

![Screenshot 2026-02-13 at 2.18.37 pm.png](Codo/Screenshot_2026-02-13_at_2.18.37_pm.png)

Credentials worked: 

![Screenshot 2026-02-13 at 2.19.39 pm.png](Codo/Screenshot_2026-02-13_at_2.19.39_pm.png)

No useful information with the database 

However, I tried the password FatPanda123 on root and it worked

![Screenshot 2026-02-13 at 2.24.42 pm.png](Codo/Screenshot_2026-02-13_at_2.24.42_pm.png)