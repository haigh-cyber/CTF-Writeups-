# Image

IP: 192.168.175.178

Nmap: 

![Screenshot 2026-04-24 at 3.21.12 pm.png](Image/Screenshot_2026-04-24_at_3.21.12_pm.png)

Port 80: 

![Screenshot 2026-04-24 at 3.21.01 pm.png](Image/Screenshot_2026-04-24_at_3.21.01_pm.png)

Searched up ImageMagick along with the version number Version: 6.9.6-4 and found an RCE vulnerability: 

![Screenshot 2026-04-24 at 3.20.46 pm.png](Image/Screenshot_2026-04-24_at_3.20.46_pm.png)

Vulnerability: [https://sploitus.com/exploit?id=68275CD5-0557-5690-9297-92D06AC0CBFA](https://sploitus.com/exploit?id=68275CD5-0557-5690-9297-92D06AC0CBFA)

Exploit: [http://github.com/SudoIndividual/CVE-2023-34152](http://github.com/SudoIndividual/CVE-2023-34152)

Downloaded the exploit and ran it with my local IP and listening port of my choice.

This created a NC listener and a payload for me to upload: 

![Screenshot 2026-04-24 at 3.22.01 pm.png](Image/Screenshot_2026-04-24_at_3.22.01_pm.png)

Once I uploaded the payload and pressed ‘Upload and Identify’, I received a webshell on my listener: 

![Screenshot 2026-04-24 at 3.22.37 pm.png](Image/Screenshot_2026-04-24_at_3.22.37_pm.png)

And from there I was able to retrieve the flag: 

![Screenshot 2026-04-24 at 3.23.45 pm.png](Image/Screenshot_2026-04-24_at_3.23.45_pm.png)

I imported [lineas.s](http://lineas.sh)h onto the host and ran it.

I found an odd SUID binary that could be exploitable:

![Screenshot 2026-04-24 at 3.26.53 pm.png](Image/Screenshot_2026-04-24_at_3.26.53_pm.png)

After a quick check on GTFObins I found that this binary is exploitable: 

![Screenshot 2026-04-24 at 3.27.37 pm.png](Image/Screenshot_2026-04-24_at_3.27.37_pm.png)

Using the command shown the in screenshot above I was able to elevate my privileges to root: 

![Screenshot 2026-04-24 at 3.28.12 pm.png](Image/Screenshot_2026-04-24_at_3.28.12_pm.png)