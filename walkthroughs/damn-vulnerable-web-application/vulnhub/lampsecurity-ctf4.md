---
description: 'Walkthrough de la machine LAMPSecurity: CTF4'
---

# LAMPSecurity: CTF4

## Détails de la machine

**Nom :** LAMPSecurity: CTF4  
**Date de sortie :** 10 Mars 2009  
**Lien de téléchargement :** [http://sourceforge.net/projects/lampsecurity/files/CaptureTheFlag/CTF4/ctf4.zip/download](http://sourceforge.net/projects/lampsecurity/files/CaptureTheFlag/CTF4/ctf4.zip/download)  
**Niveau :** Facile  
**Objectif\(s\) :** obtenir un accès "root"  
**Description :**   
`Updated to set default runlevel to 3 (no X windows) and fixed DHCP.  
  
This is the fourth capture the flag exercise. It includes the target virtual virutal machine image as well as a PDF of instructions. The username and password for the targer are deliberately not provided! The idea of the exercise is to compromise the target WITHOUT knowing the username and password. Note that there are other capture the flag exercises. If you like this one, download and try out the others. If you have any questions e-mail me at justin AT madirish DOT net.  
  
The LAMPSecurity project is an effort to produce training and benchmarking tools that can be used to educate information security professionals and test products. Please note there are other capture the flag exercises (not just the latest one). Check the SourceForge site to find other exercises available (http://sourceforge.net/projects/lampsecurity/files/CaptureTheFlag/).  
  
These exercises can be used for training purposes by following this documentation. Alternatively you may wish to test new tools, using the CTF virtual machines as targets. This is especially helpful in evaluating the effectiveness of vulnerability discovery or penetration testing tools.`

## Reconnaissance

On récupère l'adresse IP de la machine cible via `netdiscover` :

![](../../../.gitbook/assets/e6cefd01d0ee0373005316e0815632a2.png)

Comme d'habitude, on scanne les services disponibles grâce à `nmap` :

![](../../../.gitbook/assets/9693a17ebe056adbb0d8f19c7ad5bc14.png)

Nous avons donc un service OpenSSH en version 4.3 sur le port 22, un service SMTP sur le port 25 avec un Sendmail en version 8.13.5 et pour terminer un Apache 2.2.0 sur le port 80. `nmap` nous révèle aussi la présence d'un fichier "robots.txt" ainsi que son contenu, soit les URLs suivantes :

* /mail
* /restricted
* /conf
* /sql
* /admin

### Service SSH

![](../../../.gitbook/assets/442d8a3758d7a05dc868fb15ef5cbdce.png)

Il nous sera possible d’énumérer les comptes valides grâce à l'exploit "45939.py" \(CVE-2018-15473\) si besoin, mais cela nécessite de fournir une liste d'utilisateurs en entrée.

### Service SMTP

`searchsploit` va nous permettre de savoir si un exploit existe pour Sendmail en version 8.13.5 :

![](../../../.gitbook/assets/56c1e964f0838614c49a8051485bb890.png)

Il semble qu'un exploit nommé "Remote Signal Handling \(POC\)", soit en fait la CVE-2006-0058, existe pour cette version en question et permet une exécution de code arbitraire :

![](../../../.gitbook/assets/e51eaeeb7d5d3835190a8da19c7e2414.png)

L'outil `smtp-user-enum` peut nous permettre d’énumérer les utilisateurs existants mais il nécessite au préalable une liste d'utilisateurs en entrée. En effet, le service SMTP va répondre si oui non l'utilisateur spécifié existe. 

### Serveur HTTP

Le serveur HTTP sert une première d'accueil ainsi qu'un blog et des travaux de recherche :

![](../../../.gitbook/assets/6f01ebdc5dc900e4c1f3943da82866aa.png)

On navigue sur le site, puis en modifiant le paramètre "page" on identifie une LFI :

![](../../../.gitbook/assets/dfbcee0aef6d0b25c52531047b4e584e.png)

La modification du paramètre "page" n'engendre pas une erreur PHP mais seulement une page vide. Pour confirmer la vulnérabilité il a donc ici fallu faire quelques tests et tenter par exemple de récupérer un fichier \(ici le fichier "/etc/passwd"\). De plus, il faut ici utiliser une seconde vulnérabilité qui est un Null byte injection, soit l'injection du caractère "NULL" représenté par "%00" permettant d'indiquer une fin de chaîne. En effet, le développeur a ici concaténé le nom du fichier \(par exemple "blog"\) à la chaîne de caractères ".php". Si nous tentons d'inclure tout simplement "/etc/passwd", la fonction `include()` recevra en fait :

```php
include($file . ".php"); // $file sera égale ici à /etc/passwd
```

Il y a sans doute également la concaténation d'un \(ou plusieurs\) répertoire avant l'inclusion du fichier, c'est pour cette raison qu'il faut effectuer une attaque de type path transversal pour récupérer le fichier désiré. En conclusion, le développeur doit avoir développé quelque chose comme ceci :  

```php
include("/chemin/vers/" . $file . ".php");
```

La lecture d'un article du blog nous permet de voir un paramètre intéressant nommé ici "id"_._ Ce type de paramètre est propice aux injections SQL, ce qui est ici bien le cas. Pour confirmer la vulnérabilité une attaque de type boolean-based en deux étapes suffit :

![](../../../.gitbook/assets/f9c854833b15357f0f7fa41ff17bd0c5.png)

Pour confirmer la vulnérabilité il suffit de faire en sorte que la partie après le "AND" soit fausse \(en logique  `vrai ET faux` donne `faux`\) :

![](../../../.gitbook/assets/776372ba1d5ba0bbf32f459c38658b70.png)

Nous avons donc une injection SQL sous le coude pour la phase d'exploitation. Continuons l'exploration avec le contenu fichier "robots.txt" :

![](../../../.gitbook/assets/fb6d3fffda475fa91dc11c807aff65cb.png)

Le répertoire "/mail" amène à un webmail SquirrelMail en version 1.4.17 :

![](../../../.gitbook/assets/b4683639729e9c71194c10843bf0c92b.png)

Un `searchsploit` indique une possibilité de RCE \(Remote Code Execution, CVE-2017-7692\) grâce à l'exploit "41910.sh". En regardant de plus près cette exploit, on s'aperçoit qu'il faut un login / mot de passe valide pour l'exploiter, mais sait-on jamais :

![](../../../.gitbook/assets/1339e3c8d68df8ab9c413e070311f290.png)

Le "/restricted" est sécurisé par une authentification de type HTTP Basic. J'ai bien tenté un tampering du verbe HTTP mais sans succès, :

![](../../../.gitbook/assets/48d9cd9cfe93dc3d82be69ee3947479f.png)

Le répertoire "/conf" renvoie une erreur HTTP 500, on ira pas plus loin par là pour l'instant :

![](../../../.gitbook/assets/896db44fcecdb8d05c76395f3a597bc6.png)

Le répertoire "/sql" quant à lui nous permet de récupérer un fichier ".sql" sans doute utilisé pour mettre en place la base de données du blog. La base de données porte le nom de "ehks" et possède 3 tables : "user", "blog" et "comment". Quelle idée de laisser traîner ce genre de choses 😒 :

![](../../../.gitbook/assets/731c580cf725e3cf694627542d8e161b.png)

Le dernier répertoire présent est le "/admin", qui met souvent le sourire aux lèvres en pensant à quoi cela peut donne accès si on trouve le sésame :

![](../../../.gitbook/assets/cc4699e98994900d39c144790c582655.png)

Quelques tests rapides sur la mire d'authentification conduit à une possible injection SQL \(attention au javascript qui supprime certains caractères lors de l'envoi\) :

![](../../../.gitbook/assets/7cf62581105a7f90be9765ca5b7e380e.png)

Je pourrais bien sur ici directement remplacer mon `SLEEP(10)` par un `1=1` et qui permettra sans doute de contourner l'authentification mais j'essai de bien séparer la phase de détection et celle de l’exploitation des vulnérabilités.

Pendant tout ce temps tournait bien sur un `nikto` ainsi qu'un `dirb` :

![](../../../.gitbook/assets/9bf8dc69d8470fde3e1e1b3088723444.png)

`nikto` __remonte une nouvelle URL à tester, le répertoire "/pages" que voici :

![](../../../.gitbook/assets/440c1d257ddcd63d5cd27ebe7db9db26.png)

Et voici le résultat de `dirb` __\(la sortie est tronquée, je n'ai mis que ce qui me semblait intéressant\) :

![](../../../.gitbook/assets/9468037c1d7ca9d3745d13227be71d2d%20%281%29.png)

`dirb` indique la présence d'un "/calendar" ainsi qu'un fichier "README". Sa consultation nous permet de connaitre la version du logiciel, soit la version 0.10 :

![Le fichier README contient la version de calendar utilis&#xE9;e](../../../.gitbook/assets/acf2332fe48be2c09c5f90388c6aa0ef.png)

Un `searchsploit`  indique une vulnérabilité de type Arbitrary File inclusion pour la version de php-calendar &lt; 0.10.1 \(CVE-2004-1423\) :

![](../../../.gitbook/assets/a524cc0aa8994dc8e69f02ec7a6853e0.png)

La phase de reconnaissance se termine ici, pas mal de choses à se mettre sous la dent pour la phase d'exploitation. 

## Exploitation

Je laisse de côté l'énumération côté OpenSSH et SMTP, étant donné qu'on possède déjà le fichier "/etc/passwd". Dans un premier temps j'ai tenté d'exploiter la faille RCE du Sendmail mais sans succès \(l'exploit ne semble plus être fonctionnel en l'état\).

### Injection SQL

Côté du serveur web on peut exploiter l'injection SQL afin de dumper la base et voir si elle contient des information intéressantes \(le mot de passe administrateur n'est plus vraiment un prérequis à cause de l'injection SQL sur la mire d'authentification\). Sqlmap est la pour nous aider et l'on tente de récupérer dans un premier temps le nom des bases de données existantes :

* calendar
* ehks
* roundcubemail

On commence par récupérer les noms et mots de passe de la table "phpc\_users" de la base "calendar" :

![](../../../.gitbook/assets/56aa16e56b5e002afcf1c7a8fa56de3d.png)

Bon la connexion à tous ces comptes est plus pour le fun qu'autre chose car cela ne change pas grand chose pour la suite des opérations. On continue avec la base qui correspond au webmail. Malheureusement pas de mot de passe présent dans les tables. Il faut savoir que le webmail permet de gérer les mots de passe à partir de différentes sources : LDAP, SQL ou encore dans les fichiers "/etc/passwd" et "/etc/shadow", ce n'est donc pas étonnant que les mots de passe ne se retrouvent pas forcément dans la base. On tente de se connecter avec les mots de passe déjà retrouvé pour le service calendar et cela fonctionne pour "dstevens". En lisant ses mails on tombent sur quelques informations intéressantes :

![](../../../.gitbook/assets/74033055d9afb452d585baec3bd70995.png)

On sait donc qu'il est administrateur de la machine, et, étant donné qu'on possède son mot de passe du webmail et que ce webmail se base sans doute sur les utilisateurs systèmes ... . On sait également que "Andrew Chen" est lui aussi un administrateur du système \(et on possède aussi son mot de passe\). C'est du joli tout ça ! Si besoin de compiler un certain exploit, le système possède aussi un `gcc`.

On termine avec la base "ehks" :

![](../../../.gitbook/assets/3132d343921a8f2d752d993623999d8b.png)

Bravo les admins, utiliser les mêmes mots de passe sur tous les services, quelle bonne idée !

Il est temps d'accéder à la machine en SSH grâce au compte de "dstevens" ou de "achen" :

![](../../../.gitbook/assets/d58f1c90d8b133f80f156e956030b6bc.png)

## Élévation de privilèges

L’élévation est ici très facile puisque nous savons que "dstevens" est administrateur du système :

![](../../../.gitbook/assets/e326c8adcfb0f0db2edf472ba5277689.png)

Travail terminé.

## Conclusion

CTF facile puisque à la finale, une seule injection SQL suffit à devenir "root" \(plusieurs comptes sont en fait administrateur du système\).

Pour ne pas alourdir le writeup je n'ai pas mis les essais que j'ai effectué sur le blog \(en se connectant grâce à l'injection SQL ou avec un compte récupéré via l'injection SQL\) mais la partie "/admin" ne semble pas apporter quelque chose \(à par des XSS\).

Je met quand même ci-dessous l'exploitation des deux vulnérabilités CVE-2004-1423 et CVE-2017-7692 \(respectivement concernant le php-calendar ainsi que le webmail\).

### PHP-Calendar \(CVE-2004-1423\)

Cette vulnérabilité permet d'exécuter un script distant, dans notre cas un shell php généré par `msfvenom`. Le shell est hébergé sur notre machine d'attaque comme ceci : "/include/html.php"

![](../../../.gitbook/assets/400947470315986662efd88e178f406e.png)

Il ne reste maintenant que l'élévation de privilèges \(puisque nous sommes connecté en tant que "apache"\). Premièrement un `uname -a` nous donne la version du noyau :

![](../../../.gitbook/assets/f149ab679a9b1b3cae21d5eae73befb2.png)

Ce noyau est vulnérable à la CVE-2009-2698, son exploit est ici : [https://www.exploit-db.com/exploits/9542](https://www.exploit-db.com/exploits/9542), et un petit `gcc` plus tard :

![](../../../.gitbook/assets/af9ee8e9571ad2f8f428dab9afda23cc.png)

### SquirrelMail \(CVE-2017-7692\)

Une erreur sur l'exploit ne permet pas la RCE, je n'ai pas cherché plus loin étant donné que j'avais déjà rooté la machine.

