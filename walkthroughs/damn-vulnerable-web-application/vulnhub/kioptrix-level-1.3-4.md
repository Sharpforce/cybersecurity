# Kioptrix: Level 1.3 (#4)

## Détails de la machine

**Nom :** Kioptrix: Level 1.3 (#4)\
**Date de sortie :** 8 Février 2012\
**Lien de téléchargement :** [http://www.kioptrix.com/dlvm/Kioptrix4\_vmware.rar](http://www.kioptrix.com/dlvm/Kioptrix4\_vmware.rar)\
**Niveau :** Facile\
**Objectif(s) :** obtenir un accès "root"\
**Description :**

`Again a long delay between VMs, but that cannot be helped. Work, family must come first. Blogs and hobbies are pushed down the list. These things aren’t as easy to make as one may think. Time and some planning must be put into these challenges, to make sure that:`\
`1. It’s possible to get root remotely [ Edit: sorry not what I meant ]`\
`1a. It’s possible to remotely compromise the machine`\
&#x20; `2. Stays within the target audience of this site`\
&#x20; `3. Must be “realistic” (well kinda…)`\
&#x20; `4. Should serve as a refresher for me. Be it PHP or MySQL usage etc. Stuff I haven’t done in a while.`

## Reconnaissance

La machine cible possède l'adresse IP 192.168.56.69 :

![](../../../.gitbook/assets/c0f17272a479c81e92162ce162e1848e.png)

Plusieurs services sont disponibles sur la machine : un serveur SSH, un serveur Web ou encore un partage de fichiers Samba :

![](../../../.gitbook/assets/894a221970805858bda48459f925aee2.png)

### Service SSH

Pas de CVE intéressante du côté du service SSH.



### Serveur HTTP

Je démarre par un scan `nikto` :

![](../../../.gitbook/assets/1966636ad3ecf456391563f63d60b8d9.png)

Suivi de l'habituel `dirb` :

![](../../../.gitbook/assets/a9ac43d6a90a132c4dc3be80c4aa5033.png)

Cela donne un serveur Apache en 2.2.8 et PHP 5.2.4. Le `dirb` remonte un dossier qu'il faudra analyser à l'adresse `http://192.168.56.69/john/`.

Lors de la navigation sur le site, la page d'accueil présente une mire d'authentification :

![](../../../.gitbook/assets/0881aa2a7e81a81286cedc8363baeaf7.png)

Aucune tentative de compte par défaut ne fonctionnent :

![](../../../.gitbook/assets/5ff9da7fab316d99622a3d07ae88675c.png)

Par contre l'injection d'un simple guillemet dans le champ mot de passe me permet d'identifier une potentielle injection SQL :

![](../../../.gitbook/assets/462413260b8801c9d39046b699e0b4b7.png)

### Samba

Je commence par récupérer la version de Samba grâce au script `Metasploit` :

![](../../../.gitbook/assets/17b6a47806f4dcd143ae2c5442a80648.png)

La version 3.0.28a de Samba semble posséder plusieurs vulnérabilités permettant un contrôle à distance (mais après quelques tests je n'ai pas réussi à faire fonctionner l'exploit) :

![](../../../.gitbook/assets/17d188baa65188a6e377aec4e45306a8.png)

`Metasploit` va me permettre également d'énumérer les utilisateurs de la machine :

![](../../../.gitbook/assets/8920b58a54f7db83a1ce75053f8c75bc.png)

## Exploitation

### Injection SQL

Grâce à l'énumération smb il est possible d'utiliser l'injection SQL afin de se connecter avec un utilisateur donné et de récupérer son mot de passe. Pour cela, l'injection dans le champ mot de passe peut être `' OR 1=1 --`&#x20;

![](../../../.gitbook/assets/45fbf9b8610a465db14bfbb3cb0861eb.png)

Par facilité, j'utilise `sqlmap` afin de dump la totalité de la base :

![](../../../.gitbook/assets/1022555dc8bf17e08d931248a081673a.png)

Rien de très intéressant, je ne récupère seulement que les mots de passe des utilisateurs john et robert. Il est possible de tenter de récupérer un shell via une injection SQL (suivant les droits de l'utilisateur). Tout d'abord il est possible de savoir quel compte s'y connecte :

![](../../../.gitbook/assets/773f46dd5d165df3ddc1082c9be1142d.png)

Le shell a de fortes chances de fonctionner car l'utilisateur s'y connectant est l'utilisateur root. La seconde information nécessaire est le chemin de l'arborescence des pages web ; L'erreur SQL me donne cette information :

![](../../../.gitbook/assets/b17a5ec34f3ffda906edd814362320b8.png)

Quelques secondes plus tard, `sqlmap` me ramène un shell :

![](../../../.gitbook/assets/fcac5fa412efcf0ad40c5b41ce006a56.png)

## Élévation de privilèges

Ici après pas mal de recherches. Il s'avère qu'il est possible de se connecter en SSH avec le compte de john ou robert mais le shell ainsi obtenu est un shell limité :

![](../../../.gitbook/assets/5ff740ad1841382f429835bed8b171fe.png)

Dans le répertoire home de john, visité grâce au shell `sqlmap`, le fichier .lhistory indique une commande intéressante avec `echo` invoquant un bash car la commande `echo` est une commande autorisée :

![](../../../.gitbook/assets/368869da2834fe17bc4251885e0366b7.png)

Il est sans doute possible de s'en servir pour sortir du shell limité :

![](../../../.gitbook/assets/5c83f21a764cbfb0f79e2666ce00e386.png)

Cela fonctionne, le shell n'est plus limité, l'élévation de privilèges peut réellement commencer.

Tout d'abord il est possible de récupérer les identifiants du compte root se connectant à la base de données. Ces informations se situent dans le fichier checklogin.php :

![](../../../.gitbook/assets/d59141abb5390725c2ebfec34c507a2b.png)

`sys_exec` est une UDF qui n'est pas installée par défaut sur MySQL mais dans le cas où elle est présente, elle va me permettre d'exécuter des commandes en root. Je vérifie d'abord sa présence :

![](../../../.gitbook/assets/207a6c748f67d2ac9cc674d203820493.png)

Cela semble le cas. Il me suffit alors d'ajouter l'utilisateur john dans le groupe admin :

![](../../../.gitbook/assets/aad5c605b928b05753c8c16e3d60aa82.png)

Il est dont maintenant possible d'effectuer des commandes grâce à `sudo` :

![](../../../.gitbook/assets/4cd115139980e7a95ed13b1087258961.png)

## Conclusion

Début de VM sympathique mais l'élévation de privilèges était assez difficile (je ne connaissais pas les UDF) j'ai donc dû me rabattre à lire quelques autres writeups pour la terminer.

Sans en être sûr, il est peut être possible de continuer avec le webshell de `sqlmap` et de trouver un exploit existant sur cette version de Linux permettant une lpe.
