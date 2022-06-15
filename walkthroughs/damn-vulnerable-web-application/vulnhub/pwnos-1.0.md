---
description: Walkthrough de la machine pWnOS 1.0
---

# pWnOS 1.0

## Détails de la machine

**Nom :** pWnOS 1.0\
**Date de sortie :** 27 Juin 2008\
**Lien de téléchargement :** [http://pwnos.com/files/pWnOS\_v1.0.zip](http://pwnos.com/files/pWnOS\_v1.0.zip)\
**Niveau :** Facile\
**Objectif(s) :** obtenir un accès "root"\
**Description :** \
`Some of you may have noticed this new pWnOS forum section. I created pWnOS as a virtual machine and Grendel was nice enough to let me post about it here. Here's a bit of information on pWnOS.`\
`It's a linux virtual machine intentionally configured with exploitable services to provide you with a path to r00t. :) Currently, the virtual machine NIC is configured in bridged networking, so it will obtain a normal IP address on the network you are connected to. You can easily change this to NAT or Host Only if you desire. A quick ping sweep will show the IP address of the virtual machine.`\
`Sorry...no scenario/storyline with this one. I wasn't really planning to release it like this, so maybe for version 2.0 I'll be more creative. :) I'm anxious to get feedback so let me know how it goes or if you have questions. Thanks and good luck!`

## Reconnaissance

On commence par identifier notre cible grâce à l'outil `netdiscover` :&#x20;

![](../../../.gitbook/assets/f6a1dbcdd88ab149949cae9ffe6bfd89.png)

L'outil `nmap` va nous permettre de connaitre les services disponibles s'exécutant sur la machine :

![](../../../.gitbook/assets/9dc25fa2442de4044eb45751e7cd17fb.png)

Nous avons donc un service SSH (port 22), un serveur HTTP (port 80), des services NetBIOS/SMB (port 139 et 445) ainsi qu'un Webmin (port 10000).

### Service SSH

Le service SSH est un OpenSSH en version 4.6p1, `searchsploit` donne un exploit compatible qui pourrait nous aider pour la suite :

![](../../../.gitbook/assets/442d8a3758d7a05dc868fb15ef5cbdce.png)

En effet, il est possible d'effectuer une énumération de comptes grâce à la vulnérabilité CVE-2018-15473 et à l'exploit "OpenSSH < 7.7 - User Enumeration (2)".

### Serveur HTTP

Intéressons nous au serveur HTTP disponible sur le port 80 dont voici la page d'accueil :

![](../../../.gitbook/assets/7d015eef878d9b201f7d23a633491d90.png)

Rien d'intéressant sur la première page, on continue l'exploration par le bouton "Next" :

![](../../../.gitbook/assets/10dd09b23dc6cd72ae9f89eb8b3cdb44.png)

Deux paramètres sont ici présents, "help" et "connect" qui ont pour valeur "true"_._ On commence par les modifier afin de voir le comportement du serveur. Ce qui nous permet d'identifier une possible LFI/RFI :

![](../../../.gitbook/assets/5e27b1cfd9ee11f85304a96eadf7c628.png)

On note cela dans un coin et on continue l'exploration du site web :

![](../../../.gitbook/assets/664faf4b57f285ddfde6d20b73cd8a71.png)

Même si cela nous aide pas, on remarque ici que les deux paramètres de l'URL "name" et "level_"_ sont vulnérables à une faille XSS. Etant donné qu'il s'agit d'une XSS réfléchie, Chrome va bloquer son exploitation, pensez donc à utiliser Firefox à la place :

![](../../../.gitbook/assets/47328ddf30811e23f179916c334b59e8.png)

En parallèle de la navigation, j'ai également lancé un `nikto` ainsi qu'un `dirb` dont voici les résultats :

![](../../../.gitbook/assets/cb4eed5e344919d4081177a3a59c4d46.png)

`nikto` nous indique des versions d'Apache et de PHP obsolètes, la présence d'un répertoire "/php" ainsi que l'éventuel présence d'une LFI/RFI.

Quand à `dirb` :

![](../../../.gitbook/assets/86a7db8a6178b48873aea57f45b95f95.png)

Il nous indique également la présence de répertoire "/php" et nous avertit que le __ listing directory __ y est possible. Allons voir ce que cela donne :

![](../../../.gitbook/assets/5c61da7887b99d38cc38b627ed85a923.png)

Il s'agit d'un PhpMyAdmin protégé par une authentification HTTP Basic_._ Le test d'un premier couple de login/mot de passe nous conduit à une erreur mais l'on obtient ainsi sa version :

![](../../../.gitbook/assets/87c0c42a614b4a18306e24ace3f28117.png)

C'en est fini de la reconnaissance du service web, passons aux autres services disponibles.

### Samba

On commence par un `nbtscan` afin de connaître un peu mieux notre cible :

![](../../../.gitbook/assets/af17bbd9117f3421930bf6e7cf19d9b4.png)

La troisième colonne indique que la machine partage (ou est en mesure de le faire) certains dossiers. On tente de se connecter grâce à `rpcclient` en tant qu'utilisateur anonyme :

![](../../../.gitbook/assets/58cbc1523720809d72788c48cbeba39c.png)

Ce qui nous amène à l'énumération des utilisateurs Samba __ et des répertoires partagés :

![](../../../.gitbook/assets/fec6fe5bb71c5807d914c1221641f43c.png)

On tente ensuite d'accéder à ces répertoires partagés :

![](../../../.gitbook/assets/9ebbeb981e705b0490e5dff087c91a10.png)

Aucun droit qui nous permet d'aller plus loin, mais l'énumération nous a donné le nom d'un utilisateur de la machine : "vmware".

### Webmin

La navigation sur le port 10000 de la machine nous conduit à une page web demandent un login / mot de passe pour pouvoir accéder au service Webmin __ :

![](../../../.gitbook/assets/d387cbf387aa3cf0679fdf5db2cb46b5.png)

Un `searchsploit` nous indique une vulnérabilité qui peut être intéressante (Arbitrary File Disclosure) :

![](../../../.gitbook/assets/5af2cf2fd5bd367333f7025aac42ea6d.png)

La phase de reconnaissance est maintenant terminée, passons à l'exploitation.

## Exploitation

### Webmin (CVE-2006-3392)

On commence par exploiter la vulnérabilité la plus simple qui est celle de Webmin, en effet un module Metasploit existe pour nous faciliter la vie :p, mais elle est tellement simple que nous allons le faire manuellement. Afin d'accéder à un fichier présent sur le serveur il suffit d’accéder à cette URL (le pattern est une répétition de "..%01") :

`http://target:port/unauthenticated/pattern/filename`

On récupère donc le fichier "/etc/passwd" afin de connaitre la liste complète des utilisateurs :

![](../../../.gitbook/assets/089db0fbd89c2207ae41acc3561a05d4.png)

Il y a donc bien l'utilisateur "vmware" ainsi que "obama", "osama" et "yomama". On tente avec un fichier un peu plus sensible, le fichier "/etc/shadow" :

![](../../../.gitbook/assets/ac2b9ea53eb6d6341ecdc4fe13461f34.png)

Bingo ! Le service Webmin doit sans doute tourner avec les droits root pour pouvoir accéder en lecture à ce fichier. Tentons maintenant de casser ces mots de passe avec `john` (ne pas oublier de faire un `unshadow` avant) :

![](../../../.gitbook/assets/ac997be395e5d51b7d408c5d5fe2add6.png)

Un seul mot de passe est cassé, celui de "vmware". On peut donc maintenant se connecter en SSH en utilisant le nom d'utilisateur "vmware" et le mot de passe "h4ckm3" :

![](../../../.gitbook/assets/ec4d68b09a2ccf2dfc8bdc85db197e72.png)

## Élévation de privilèges

Un `uname -a` nous donne rapidement le numéro de version du noyau du système :

![](../../../.gitbook/assets/0261b62fab03bd903b13759676e2a2f9.png)

Le script `Linux_Exploit_Suggester` nous permet de connaitre les potentiels exploits d'élévation de privilèges :

![](<../../../.gitbook/assets/f9144b5d0faaed3200da27ab1ede8368 (1).png>)

Un exploit qui a fonctionné est le "vmsplice1", soit la CVE-2008-0600. La machine n'ayant pas accès à internet dans mon cas, on copie l'exploit grâce à `scp` :

![](../../../.gitbook/assets/0fc680479f0129f33f14c4172c12e706.png)

Sur la cible, on compile puis exécute l'exploit :

![](../../../.gitbook/assets/8dfd3d4087b9d6c9a0a2480d1b7456a9.png)

Congratz ! Nous sommes "root" **** (qui n'a d'ailleurs en fait pas de nom ;) )!

## Conclusion

Machine plutôt facile grâce à l'accès à n'importe quel fichier via la vulnérabilité sur le Webmin __ qui tourne en "root" et qui permet de récupérer le fichier "/etc/shadow" :

![](../../../.gitbook/assets/86b7c88a10dbc3a7240a03da6c9c6c68.png)

L'élévation de privilèges à partir du compte "vmware" n'a pas posée de problème, l'exploit utilisé fonctionnant du premier coup.&#x20;

Je me demande si en passant un peu plus de temps sur le service smb il n'y a pas moyen d'obtenir plus d'information. De mémoire il n'y a pas de vulnérabilité intéressante concernant le phpmyadmin mais j'avoue ne pas avoir creuser beaucoup cette piste. A noter qu'il y a également une vulnérabilité d'OpenSSL (voir [https://en.wikipedia.org/wiki/OpenSSL#Predictable\_private\_keys\_(Debian-specific)](https://en.wikipedia.org/wiki/OpenSSL#Predictable\_private\_keys\_\(Debian-specific\))) qui permet de se connecter en SSH en retrouvant certaines clés privées, mais je suis passé à côté de celle-ci.

