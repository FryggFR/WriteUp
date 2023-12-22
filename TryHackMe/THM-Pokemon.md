# Pokemon 
IP : 10.10.131.239

Une VM facile et plutôt cool ^^ pas de compétence particulière de requise, a part pas mal de fouilles !
# Enum
## NMAP:
```nmap
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-24 08:39 EDT
Nmap scan report for 10.10.131.239
Host is up (0.034s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 58:14:75:69:1e:a9:59:5f:b2:3a:69:1c:6c:78:5c:27 (RSA)
|   256 23:f5:fb:e7:57:c2:a5:3e:c2:26:29:0e:74:db:37:c2 (ECDSA)
|_  256 f1:9b:b5:8a:b9:29:aa:b6:aa:a2:52:4a:6e:65:95:c5 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Can You Find Them All?
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.84 seconds
```
## Page web :
On y trouve une fonction :
```
   <script type="text/javascript">
    	const randomPokemon = [
    		'Bulbasaur', 'Charmander', 'Squirtle',
    		'Snorlax',
    		'Zapdos',
    		'Mew',
    		'Charizard',
    		'Grimer',
    		'Metapod',
    		'Magikarp'
    	];
    	const original = randomPokemon.sort((pokemonName) => {
    		const [aLast] = pokemonName.split(', ');
    	});

    	console.log(original);
```
```
        <pokemon>:<***************>
        	<!--(Check console for extra surprise!)-->
```
Dans la console on y trouve le résultat de la fonction dans un array:
```
Array(10) [ "Bulbasaur", "Charmander", "Squirtle", "Snorlax", "Zapdos", "Mew", "Charizard", "Grimer", "Metapod", "Magikarp" ]
0: "Bulbasaur"
1: "Charmander"
2: "Squirtle"
3: "Snorlax"
4: "Zapdos"
5: "Mew"
6: "Charizard"
7: "Grimer"
8: "Metapod"
9: "Magikarp"
length: 10
```
# SSH
Enfaite...
```
<pokemon>:<*********************>
```
Ce sont des credentials pour se connecter en SSH sur le serveur..

# Grass-Type
On arrive sur le serveur avec la session **pokemon**. 

Dans le dossier **/Desktop/** ce trouve une archive **P0kEmOn.zip**

Dedans ce trouve le fichier **grass-type.txt**
```
50 6f 4b 65 4d 6f 4e 7b 42 75 6c 62 61 73 61 75 72 7d
```
C'est de l'hexa :
```
506f4b654d6f4e7b42756c6261736175727d:PoKeMoN{**********}
```
Nous avons le 1er flag.

# Water-Type 
Dans le dossier **/var/www/html** ce trouve le fichier **water-type.txt**

Il contient :
```
Ecgudfxq_EcGmP{Ecgudfxq}
```
Il est encoder en caesar cipher :
```
Squirtle_SqUaD{***********}
```
Nous avons le second flag !
# Fire-Type
Avec la commande find, on retrouve le dernier :
```
pokemon@root:/var/www/html$ find / -name fire-type.txt 2</dev/null
/etc/why_am_i_here?/fire-type.txt
```
Il contient :
```
UDBrM20wbntDaGFybWFuZGVyfQ==
```
C'est du base64.
```
pokemon@root:/var/www/html$ echo "UDBrM20wbntDaGFybWFuZGVyfQ==" | base64 -d
P0k3m0n{***********}
```
Et de 3 ! Plus que le root ;)

# Privesc to root !
Après avoir fait tout les tests possible (SUID/GUID, crontab, linpeas, lse, etc..) je trouve toujours rien de bien probant. Je continue mes fouilles et je tombe la dessus :
```
pokemon@root:~/Videos/Gotta/Catch/Them/ALL!$ cat Could_this_be_what_Im_looking_for\?.cplusplus 
# include <iostream>

int main() {
        std::cout << "ash : pik*********"
        return 0;
```
Il s'agit du compte de **ash**
```
pokemon@root:~/Videos/Gotta/Catch/Them/ALL!$ su - ash
Password: 
No directory, logging in with HOME=/
$ id
uid=1001(ash) gid=1001(ash) groups=1001(ash),27(sudo)
$ sudo -l
[sudo] password for ash: 
Matching Defaults entries for ash on root:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User ash may run the following commands on root:
    (ALL : ALL) ALL
$ sudo /bin/bash
root@root:/#
```
Nous voila donc root.
Le flag ce trouve dans **/home**
```
root@root:/home# cat roots-pokemon.txt 
**********!
```
