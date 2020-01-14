# Hackademic: RTB2

## Détails de la machine

**Nom :** Hackademic: RTB2  
**Date de sortie :** 6 Septembre 2011  
**Lien de téléchargement :** [https://download.vulnhub.com/hackademic/Hackademic.RTB2.zip](https://download.vulnhub.com/hackademic/Hackademic.RTB2.zip)  
**Niveau :** Facile  
**Objectif\(s\) :** obtenir un accès "root" et lire le flag situé dans le fichier `/root/Key.txt`  
**Description :**  
`This is the second realistic hackademic challenge (root this box) by mr.pr0n  
Download the target and get root.  
After all, try to read the contents of the file 'key.txt' in the root directory.  
Enjoy!`

## Reconnaissance

On récupère l'adresse IP de la cible grâce à `netdiscover` :

![](../../../.gitbook/assets/fd957a25c648bee9fd66952e73c6ab43.png)

On continue en scannant notre cible à l'adresse 192.168.56.68 via `nmap` :

![](../../../.gitbook/assets/241914acc60e932ded45530601026b7d.png)

Il y a donc un service web de disponible ainsi qu' service sur le port 666 mais filtré par le pare-feu.

### Serveur Web

Une analyse avec `nikto` va nous permettre d'en savoir un peu plus :

![](../../../.gitbook/assets/7114fd2c1640fef76a969c6d34aaf90e.png)

Le serveur web propose un phpMyAdmin. On tente de connaitre sa version : 

![](../../../.gitbook/assets/ba67bca8a59e203652d76318e0401d73.png)

Tout en lançons un `dirb` :

![](../../../.gitbook/assets/a6ccb00b52107cce6068d4b0f5954d71.png)

Rien de spécifique pour le `dirb`. Il est temps de naviguer sur le site proposé par la machine :

![](../../../.gitbook/assets/f7276170d7f51391968b7999a9eb35f9.png)

Malheureusement aucune des tentatives de weak credentials ne fonctionnent et pas grand chose d'autres d'intéressant à exploiter.

### Port knocking

Etant donné que le serveur web du port 80 ne semble pas très intéressant, on se concentre sur le port 666 qui semble filtré. Ici un peu par chance \(beaucoup de chance en fait\), le port 666 est devenu ouvert :

![](../../../.gitbook/assets/17327db6fdcc4ddee4e48c574cb7f584.png)

Mais qu'elle est cette sorcellerie ? En effet, j'ai dit par chance car j'avais nettoyé mon terminal et donc dû relancer un nmap \(ce qui a ouvert le port protégé par du port knocking\). Le second scan n'indique pas que le port est ouvert \(l'impression écran est le troisième scan\), mais en essayant via le navigateur d'accéder à ce port filtré la page suivante s'est affichée :

![](../../../.gitbook/assets/9c5a4ce47506eaefcf413db928f92ff0.png)

Ce second site utilise le CMS Joomla en 1.5, il faudra sans doute creuser par là. On commence tout d'abord par une petite reconnaissance avec `nikto` : 

![](../../../.gitbook/assets/b37fcc1d43be96964ee63a316c1005c0.png)

Une page d'administration est disponible mais il semble que ce n'est pas la bonne direction :

![](../../../.gitbook/assets/601956672faa2ffc444da10b15258fc5.png)

Et le phpMyAdmin présent sur ce port est également en version 3.3.2.

### Joomla

On attaque la reconnaissance du CMS Joomla qui semble être en version 1.5. `metsaploit` nous donne quelques détails supplémentaires :

![](../../../.gitbook/assets/2055b3827371e34384b2a446f5b90c2f.png)

Il s'agit donc plus précisément de la version 1.5.15. Qu'en est t'il des plugins utilisés :

![](../../../.gitbook/assets/bafaebc1783ff4f639714f9d3992b307.png)

Le plugin com\_abc \(ABC Joomla Extension\) semble être vulnérable aux injections SQL, sans doute notre porte d'entrée.

## Exploitation

### Injection SQL

 `metasploit` nous indique que le paramètre vulnérable est le paramètre "sectionid" \(`http://192.168.56.68:666/index.php?option=com_abc&view=abc&letter=AS&sectionid='`\).

En cherchant un peu on identifie cette vulnérabilité comme étant la CVE-2010-1656 :

![](../../../.gitbook/assets/bc2863fb933df91a8db6b3d47ef3e8e0.png)

Ici on ne s'embête pas, on sort l'artillerie `sqlmap` pour l'exploitation. On récupère deux mots de passe d'utilisateurs lambda mais pas celui de l'administrateur :

![](../../../.gitbook/assets/4db0765c50f081e1a353ab55eadc8f34.png)

Les comptes de JSmith et BTallor ne donnent rien et ne permettent pas d'aller plus loin. Selon les droits que possède le compte se connectant à la base il est peut être possible d'écrire des fichiers et ainsi de récupérer un shell.  

Pour cela, il faut utiliser l'option `--os-shell` :

![](../../../.gitbook/assets/a8e21cfb96625a12e7d30f2ba7c3fc1f.png)

On possède donc maintenant un shell sur la machine mais limité en droits puisque il s'agit du compte www-data.

## Élévation de privilèges

On identifie la version du système d'exploitation :

![](../../../.gitbook/assets/6d41f273d411b9785c03ba2ba2eb09c5.png)

Pour cette version, [Linux-suggester](https://github.com/mzet-/linux-exploit-suggester) nous propose l'exploit suivant :

![](../../../.gitbook/assets/a60387230af9692081d0764a85dadb86.png)

Il s'agit d'un exploit en C à compiler :

![](../../../.gitbook/assets/324351299fa0647a010df20ae6afb008.png)

Pour son exécution, il faut retrouver un shell un peu plus classique, je suis donc passé par l'option `--os-pwn` de `sqlmap` \(le chemin du serveur web fait partie des "common location\(s\)"\) :

![](../../../.gitbook/assets/f8ca36c20faec349a88b06a7e8191a1f.png)

L'exécution de l'exploit nous donne un accès root :

![](../../../.gitbook/assets/933f9c0d57e41f9f3644398a564da510.png)

On récupère le flag présent dans le fichier à l'emplacement /root/Key.txt \(le fichier étant très long, la sortie est tronquée\) :

![](../../../.gitbook/assets/5ae790334d45fd1bb3ddb9f483f0eb73.png)

Le texte est encodé en base64, son décodage nous donne une image :

![](../../../.gitbook/assets/0f1e7b942e08205b6a7f352cb0d1d9a0.png)

## Conclusion

Machine sympathique que j'ai mis un certain temps à finir. La raison était que le shell récupéré était très instable et la connexion se fermait au bout de quelques secondes. Un peu de persévérance a terminé par payer 😉 .

J'ai eu beaucoup de chance concernant le port knocking puisque un double `nmap` suffit à déclencher l'ouverture du port. Par curiosité, je suis allé voir la configuration de `knockd` que voici :

![](../../../.gitbook/assets/5676f7f94dc1c6a3d079e3a3e3c8109a.png)

Il s'agit donc de la séquence par défaut, à savoir 7000, 8000 puis 9000 pour provoquer l'ouverture.

