# SuperSpamOld

new room :)

PAS FINI !

# Enum
## Nmap
```
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-11 09:24 EDT
Nmap scan report for 10.10.137.113
Host is up (0.032s latency).
Not shown: 65530 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Home :: Super-Spam
|_http-server-header: Apache/2.4.29 (Ubuntu)
4012/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 86:60:04:c0:a5:36:46:67:f5:c7:24:0f:df:d0:03:14 (RSA)
|   256 ce:d2:f6:ab:69:7f:aa:31:f5:49:70:e5:8f:62:b0:b7 (ECDSA)
|_  256 73:a0:a1:97:c4:33:fb:f4:4a:5c:77:f6:ac:95:76:ac (ED25519)
4019/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x    2 ftp      ftp          4096 Feb 20  2021 IDS_logs
|_-rw-r--r--    1 ftp      ftp           526 Feb 20  2021 note.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.14.59.209
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
5901/tcp open  vnc     VNC (protocol 3.8)
| vnc-info: 
|   Protocol version: 3.8
|   Security types: 
|     VNC Authentication (2)
|     Tight (16)
|   Tight auth subtypes: 
|_    STDV VNCAUTH_ (2)
6001/tcp open  X11     (access denied)
Service Info: OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 63.26 seconds

```
## WPScan
```
```
# FTP Port: 4019
```
┌──(kali㉿kali)-[~/Challenge/TryHackMe/SuperSpamold]
└─$ ftp 10.10.137.113 4019  
Connected to 10.10.137.113.
220 (vsFTPd 3.0.3)
Name (10.10.137.113:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
229 Entering Extended Passive Mode (|||47369|)
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Feb 20  2021 IDS_logs
-rw-r--r--    1 ftp      ftp           526 Feb 20  2021 note.txt
226 Directory send OK.
```
Je récupère le fichier note.txt, il y a aussi dans le dossier IDS_logs pas mal de fichier mais en grande partie vide, je prend les fichiers pcap qui ne sont pas vide.
# .PCAP / .PCAPNG
Plusieurs fichiers, qui corresponde a peu prêt avec les dates des commentaires sur le fichier note.txt.
```
12th January: Note to self. Our IDS seems to be experiencing high volumes of unusual activity.
We need to contact our security consultants as soon as possible. I fear something bad is going
to happen. -adam

13th January: We've included the wireshark files to log all of the unusual activity. It keeps
occuring during midnight. I am not sure why.. This is very odd... -adam

15th January: I could swear I created a new blog just yesterday. For some reason it is gone... -adam

24th January: Of course it is... - super-spam :)
```
Voici les informations trouvés dans ces fichiers .pcap

## 12-01-21.req.pcapng
On y trouve une enumeration des users du groupe admin:
```
Administrator
a-jbrown
a-arobinson
vray
```
## 13-01-21.pcap
On y trouve une connexion vers http[:]//id1[.]cn/rd.s/Btc5n4unOP4UrIfE?url=http[:]//id1[.]cn/
C'est un site en chine.

## 14-01-21.pcapng
Ici, on y vois une connexion sur le partage SMB avec ces informations:
```
Username : lgreen
Hostname : 02694W-WIN10
DNS : 01566s-win16-ir.threebeesco.com
Path : \\172.16.66.36\IPC$
```
## 16-01-21.pcap
Enormement de packet TCP bloquer
