# Boolean

IP: 192.168.189.231

Nmap: 

![Screenshot 2026-02-02 at 9.01.21 pm.png](Boolean/Screenshot_2026-02-02_at_9.01.21_pm.png)

Masscan: 

Port 80:

![Screenshot 2026-02-02 at 9.33.49 pm.png](Boolean/Screenshot_2026-02-02_at_9.33.49_pm.png)

The web page allows you to register a user: 

![Screenshot 2026-02-02 at 9.44.25 pm.png](Boolean/Screenshot_2026-02-02_at_9.44.25_pm.png)

I registered a test user with username: test and password: test 

After registering a user and signing into the account, it asks you to confirm the account via email address: 

![Screenshot 2026-02-02 at 9.45.49 pm.png](Boolean/Screenshot_2026-02-02_at_9.45.49_pm.png)

The user login page limits access however it allows you to change the users email address: 

![Screenshot 2026-02-02 at 9.46.24 pm.png](Boolean/Screenshot_2026-02-02_at_9.46.24_pm.png)

When intercepting the request in burp we see a PATCH request is sent to the server and what’s returned is a JSON containing the account details:

![Screenshot 2026-02-02 at 9.47.13 pm.png](Boolean/Screenshot_2026-02-02_at_9.47.13_pm.png)

I was curious how the web server was able to retrieve, identify and change our desired user based on the patch request. 

After decoding our patch request from URL encoding and comparing it to the HTTP Response body, I noticed something interesting:

```html
Patch request: 
_method=patch
authenticity_token=eKI-XNYSmHnfOoauUEAzhb21EZFiVAkpso5w43W6IjBq-IzAOJABqHlxgaCG7jCam7S9yZu6rFj1t62q4nLBmw
user[email]=test1@test.com
commit=Confirm email

Response body: 
{
"email":"test1@test.com",
"id":1,
"username":"test",
"confirmed":false,
"created_at":"2026-02-02T10:02:30.573Z", 
"updated_at":"2026-02-02T10:17:04.043Z"
}
```

From the above - and after analysing multiple responses - the authenticity token remained the same 

But how the webserver changed the email field is through user user[email]

So I attempted to change the “confirmed” column of the table by using user[confirmed]

And it worked: 

![Screenshot 2026-02-02 at 9.53.22 pm.png](Boolean/Screenshot_2026-02-02_at_9.53.22_pm.png)

The webpage is a file manager: 

![Screenshot 2026-02-02 at 10.41.16 pm.png](Boolean/Screenshot_2026-02-02_at_10.41.16_pm.png)

Trying to upload and execute file on the system however all files re-download after being uploaded and clicked

When attempting to upload a file to the system the following request is sent including 3 parameters: 

![Screenshot 2026-02-03 at 12.45.54 pm.png](Boolean/Screenshot_2026-02-03_at_12.45.54_pm.png)

Params: 

```html
cwd
file
download
```

From these I attempted to using directory traversal on the first param cwd (current working directory) and was successful:

![Screenshot 2026-02-03 at 12.47.28 pm.png](Boolean/Screenshot_2026-02-03_at_12.47.28_pm.png)

Was unable to find a way through. 

Tried every avenue to gain a webshell

Found that I was able to upload files to the .ssh directory for remi 

So I created an SSH key pair on my local and uploaded the public to the .ssh directory for remi:

![Screenshot 2026-02-03 at 4.21.55 pm.png](Boolean/Screenshot_2026-02-03_at_4.21.55_pm.png)

![Screenshot 2026-02-03 at 4.22.14 pm.png](Boolean/Screenshot_2026-02-03_at_4.22.14_pm.png)

I was then able to gain an ssh shell: 

![Screenshot 2026-02-03 at 4.22.29 pm.png](Boolean/Screenshot_2026-02-03_at_4.22.29_pm.png)

## Priv esc

On the users home directory there were several files that I inspected. 

One of the files created an alias for bash named root and it looks like a clear priv esc vector 

![Screenshot 2026-02-03 at 4.23.09 pm.png](Boolean/Screenshot_2026-02-03_at_4.23.09_pm.png)

Executing root, I successful escalated my privileges to root LOL

![Screenshot 2026-02-03 at 4.24.04 pm.png](Boolean/Screenshot_2026-02-03_at_4.24.04_pm.png)

**Learning - how to connect via SSH if the .ssh directory is editable:** 

1. Use this within the /kali/.ssh directory `ssh-keygen -t ed25519 -C kali@kali` 
    1. Ensure the private file is within the /kali/.ssh directory 
2. This creates two files `id_ed25519` and `id_ed25519.pub` 
3. Upload the pub file to the .ssh directory of the target user and change the name of the file to `authorized_keys` -VERY IMPORTANT 
4. Then connect to the host normally with ssh `ssh user[@](mailto:remi@192.168.214.231)IP`