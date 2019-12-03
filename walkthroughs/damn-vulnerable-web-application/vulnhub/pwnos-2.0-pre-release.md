# pWnOS 2.0 \(Pre-Release\)

## Détails de la machine

**Nom :** pWnOS 2.0 \(Pre-Release\)  
**Date de sortie :** 4 Juillet 2011  
**Lien de téléchargement :** [http://pwnos.com/files/pWnOS\_v2.0.7z](http://pwnos.com/files/pWnOS_v2.0.7z)  
**Niveau :** Facile  
**Objectif\(s\) :** obtenir un accès "root"  
**Description :**  
`pWnOS v2.0 is a Virutal Machine Image which hosts a server to pratice penetration testing. It will test your ability to exploit the server and contains multiple entry points to reach the goal (root). It was design to be used with WMWare Workstation 7.0, but can also be used with most other virtual machine software.  
  
Server's Network Settings:  
IP: 10.10.10.100  
Netmask: 255.255.255.0  
Gateway: 10.10.10.15`

## Reconnaissance

L'adresse IP de la machine étant statique pas besoin de scan du réseau :

![](../../../.gitbook/assets/1800b61673fdffcc34d1c883cbe67ac2.png)

On commence directement par le scan de services via `nmap` :

![](../../../.gitbook/assets/1f6222a8a24ac1fd029c54628f1ff19a.png)

Un service SSH \(port 22\) et un service Web \(port 80\). On ne peut pas faire plus simple.

### Service SSH

Quelques vulnérabilités sur la version 5.8p1 d'OpenSSH, mais les CVE ont été découvertes après 2011, leurs exploitations n'est donc pas requises pour compromettre la machine :

![](../../../.gitbook/assets/b6bfbeaff127198f4893e32a4a37c31c.png)

### Service Web

Nous sommes donc en face d'un serveur Web Apache 2.2.17 utilisant PHP. Nous commençons par un `dirb` :

![](../../../.gitbook/assets/04c759cc0144b6f984a5bbb5abe98459.png)

Suivi d'un `nikto` :

![](../../../.gitbook/assets/3810de0d0b7b42863fefcbce5cf562fc.png)

Ces deux scans nous apprennent la présence d'une page `info.php` qui effectue un appel à la fonction `phpinfo()` :

![](../../../.gitbook/assets/d24253082253f33f3917010130ea0307.png)

Cela étant fait, allons faire un tour sur le site proposé. La page d'accueil affiche un message de bienvenue et propose 3 liens \(à droite, dont le lien `Home` qui est la page courante\) :

![](../../../.gitbook/assets/515a26491044826578325bc04b416d2b.png)

On peut tenter de s'inscrire via le lien `Register`, cela va peut-être nous permettre de nous authentifier et ainsi avoir accès à d'autres informations :

![](../../../.gitbook/assets/68d009db34acbae94c1c68734c263377.png)

Après une activation du compte via un lien forgé par l'application :

![](../../../.gitbook/assets/4c7e10c440221bdfe2c25751a3dc24fd.png)

Nous sommes authentifié, mais aucun lien supplémentaire ne nous est offert :

![](../../../.gitbook/assets/c7ac4885d84e40aee4e3a7bdabeb2ad9.png)

Aucun lien du site ne l'affiche \(même en étant authentifié\), mais le scan `dirb` remonte également une entrée vers un blog via `/blog` :

![](../../../.gitbook/assets/6cea206a447f388f9a5ff33cefaff289.png)

Après avoir fait le tour de tout ce qui est proposé, je commence à tester quelques injections. Au bout de quelques minutes : "Bingo !". Le champs `login` de la page principale est vulnérable aux injections SQL :

![](../../../.gitbook/assets/b32979ed28dd1ab344aaba2b22f6f538.png)

## Exploitation

On va exploiter l'injection SQL afin de se connecter avec un compte existant et voir ce qu'on peut en tirer :

![](../../../.gitbook/assets/6828d78ef241b25bbee0150c2f0bc71e.png)

Nous sommes connectés avec le premier compte que remonte la requête SQL effectuée par l'application :

![](../../../.gitbook/assets/addd90b3eaae79d95996067a8565a534.png)

Un compte admin, pas mal. En fait, l'accès au compte admin ne donne pas d'information supplémentaire.

{% hint style="info" %}
Je ne l'ai vu seulement après coup mais l'adresse email de l'admin est présente sur la page d'accueil afin de le contacter en cas de questions.
{% endhint %}

On va donc tenter de récupérer les informations de la base de données en exploitant l'injection. Pour cela direction `sqlmap`. On récupère les mots de passe disponibles :

![](../../../.gitbook/assets/b77bffe730d7eb785dee822857ee3936.png)

Le mot de passe est hashé en SHA1 \(40 caractères\), on tente de le cracker :

![](../../../.gitbook/assets/cf0627b44f73e69fb881a86621297b43.png)

Etant déjà connecté en tant qu'admin, ce mot de passe ne va pas nous servir directement, mais je tente de l'utiliser admin de me connecter au blog et également en SSH. Pour cela, j'essaie plusieurs login tels que admin, dan, privett, danprivett, dan.privett mais sans succès.

Etant donné que la seule faille à ma disposition est l'injection SQL, je tente d'obtenir un shell par ce biais. Pour ce faire il est possible d'utiliser `sqlmap`. Il faut pour cela que le compte gérant la base de données possède le droit d'écriture sur le système de fichiers.

Dans un premier temps, on génère notre shell avec `msfvenom` :

![](../../../.gitbook/assets/b3cd3eb65947989d0e23a172abf6965c.png)

On upload notre shell sur le serveur web via `sqlmap` :

![](../../../.gitbook/assets/1f46249ab525fe746cbf035db4a80398.png)

{% hint style="info" %}
Le répertoire cible est indiqué dans la page `info` affichant le `phpinfo()`. Dans le cas contraire il faut tenter les valeurs habituelles voir brute forcer le nom du chemin.
{% endhint %}

On lance un handler en écoute sur le port adéquat, on visite la page contenant le webshell et nous sommes dans la place :

![](../../../.gitbook/assets/92bf2ae7c90fa56f8258829a9c21c338.png)

## Élévation de privilèges

On effectue un peu de reconnaissance :

![](../../../.gitbook/assets/06537bbd3b2156df507c637922492e43.png)

Fichier `/etc/passwd` :

![](../../../.gitbook/assets/c30bea8dbe54d09478e258efc8f9547f.png)

Contenu du seul utilisateur de la machine, dan :

![](../../../.gitbook/assets/c6b6b7f82e2bede5f75d82813eae8ca8.png)

Je vais chercher le mot de passe qu'utilise l'application afin de se connecter à la base de données qui se trouve dans le fichier `/var/www/mysqli_connect.php` :

![](../../../.gitbook/assets/bab38a17bbef8c27589c1d6341c3b18e.png)

Malheureusement, le mot de passe "goodday" ne fonctionne pour aucun compte SSH \(dan ou root\). 

Un `cd ..` \(plutôt chanceux sur le coup\) plus tard on retrouve une copie du fichier `mysqli_connect.php`. Sans doute un backup ou une version antérieure/de test :

![](../../../.gitbook/assets/b994308ce0f2cf9ba03168bc4a5635cd.png)

Ce mot de passe \(indiquant bien qu'il s'agit du mot de passe root\) nous permet de se connecter en root à la machine :

![](../../../.gitbook/assets/27214758cc7845b06d85a1b118a564c0.png)

## Conclusion

Aucune grosse difficulté pour cette machine. J'ai apprécié le fait que l'utilisateur de la base de données mysql possède le droit d'écriture en fichier afin d'uploader le webshell via sqlmap.

Concernant l'élévation j'ai eu un coup de chance de voir le fichier à un niveau plus haut. Mais dans le cas contraire une recherche de mots de passe avec les habituels `find` et `grep` auraient fini par faire le travail.

