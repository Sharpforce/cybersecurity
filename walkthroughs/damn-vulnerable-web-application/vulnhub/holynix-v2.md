---
description: 'Walkthrough de la machine Holynix: v2'
---

# Holynix: v2

## Détails de la machine

**Nom :** Holynix: v2\
**Date de sortie :** 8 Décembre 2010\
**Lien de téléchargement :** [https://download.vulnhub.com/holynix/holynix-v2.tar.bz2](https://download.vulnhub.com/holynix/holynix-v2.tar.bz2)\
**Niveau :** N/A\
**Objectif(s) :** obtenir un accès "root"\
**Description :** \
`Holynix is an Linux distribution that was deliberately built to have security holes for the purposes of penetration testing. The object of the challenge v1 is just to root the box. Register on the forums to receive an email update when a new challenge is released.`\
``\
`Network Configuration Holynix v2 is set with static ip and requires some network configuration in order to run.`\
`Network: 192.168.1.0/24` \
`Pool Starting Addr: 192.168.1.2` \
`Gateway Addr: 192.168.1.1` \
`Subnet Mask: 255.255.255.0`

## Reconnaissance

Comme d'habitude on commence par identifier notre cible grâce à `netdiscover` :

![](../../../.gitbook/assets/c7201406205e5baf75debf55023bb39e.png)

Et le scan `nmap` de la machine "192.168.1.88" :

![](../../../.gitbook/assets/74f0ae8b88293db530f6056fdd9f1cfa.png)

Nous avons donc un service FTP Pure-FTPd mais `nmap` ne nous remonte pas sa version. Un service SSH de type OpenSSH 4.7p1, un service DNS avec un serveur ISC BIND en version 9.4.2-P2.1 ainsi qu'un serveur web sur le port 80 (Apache 2.2.8 et PHP 5.2.4).

### Service FTP

J'ai tenté ici d'en savoir plus sur le service Pure-FTPd et notamment sur son possible fingerprinting afin de pouvoir rechercher les vulnérabilités associées à sa version. Tout d'abord grâce au module Metasploit :

![](../../../.gitbook/assets/0e89d97387b33ece71f82c447cc43089.png)

Il est possible de récupérer la même chose sans utiliser Metasploit mais `telnet` :

![](../../../.gitbook/assets/f92bbff16958794f72632fe61e63b472.png)

Il est indiqué que l'authentification anonyme n'est pas possible. C'est en effet sans doute le cas, sinon `nmap` l'aurait remonté pendant son scan (grâce à l'option activant certains scripts `-sC`).

### Service SSH

Il s'agit d'une version 4.7 donc vulnérable à la CVE-2018-15473, mais celle-ci datant de 2018 je ne pense pas que l'auteur de la machine en avait conscience  :wink: . On la garde sous le coude on ne sait jamais.

### Service DNS

Le service DNS est géré par un ISC BIND en version 9.4.2-P2.1, mais je n'ai trouvé aucune vulnérabilité qui peut nous aider ici.

### Serveur HTTP

La page d'accueil nous indique qu'il faut être un utilisateur de "ZincFTP" afin de pouvoir accéder aux services offerts part la plateforme. Deux champs permettent de demander l'accès après confirmation par un administrateur :

![](../../../.gitbook/assets/7181d962124851a4646556c4ed5f9a28.png)

Pas d'injection SQL identifiée sur ces deux champs, on continue l'analyse en utilisant `dirb` et `nikto` :

![](../../../.gitbook/assets/3d5bf36342276eb0d5ef8084cf4e2046.png)

Un phpMyAdmin est disponible mais la réponse est un 403 (Forbidden), de même pour le "/server-status". Le "/register" est la page qui reçoit la requête de la page d'accueil :

![](../../../.gitbook/assets/33bfaeb07a1cfe824eaeb8f954944dad.png)

![](../../../.gitbook/assets/db06c628cfcd8719902f360aeec9e23e.png)

Rien de spécial en sortie du `nikto`.

Pour réussir à avancer, il faut revenir sur la page d'accueil. Elle nous indique que les serveurs de noms sont disponibles sur "ns1.zincftp.com" ainsi que sur "ns2.zincftp.com". De plus, il est possible d'accéder à un répertoire web propre à l'utilisateur en accédant à l'URL "http://username.zincftp.com", le problème c'est que nous ne connaissons aucun nom d'utilisateur.

Si nous pouvons récupérer les sous-domaines existants on obtiendra donc la liste des utilisateurs de la  plateforme et peut être même accéder aux différents répertoires web. Il faut donc sans doute se concentrer sur le serveur DNS présent sur la machine. A partir de là, vient l'idée de tenter un transfert de zone.

## Exploitation

### Transfert de zone DNS

Avant de tenter un transfert de zone, nous allons commencer par en savoir un peu plus sur le nom de domaine "zincftp.com" en interrogeant le DNS de la machine grâce à la commande `dig` :

![](../../../.gitbook/assets/a82667f4f755ffc81fd3107ac8987269.png)

Un second serveur DNS est disponible à l'adresse 192.168.1.89 mais étant donné que la machine ne contient qu'une seule interface réseau cette adresse ne donne rien. Avant de passer au transfert de zone quelques rappels à son sujet :

{% hint style="info" %}
Il ne s'agit pas d'une attaque mais d'un mécanisme de duplication des bases de données des serveurs DNS. Dans cette base se trouvent la liste des domaines gérées par le serveur en question. Dans notre cas il doit contenir par exemple les sous-domaines propres aux utilisateurs de la plateforme sous la forme "username.zincftp.com".&#x20;

La réplication de cette base sert dans le cas par exemple où deux (ou plus) serveurs DNS sont en place. Le premier est le serveur primaire, il répond aux requêtes des utilisateurs permettant ainsi de traduire une adresse IP en nom de domaine (c'est quand même plus facile à retenir pour nous). Mais que se passe t'il si ce serveur tombe en panne ? Le serveur secondaire prend le relais. Mais pour qu'il puisse prendre le relais il faut qu'il possède la même base de données que le serveur primaire, d'où la nécessité de se maintenir à jour en demandant régulièrement au serveur primaire une copie de sa base de données.
{% endhint %}

L'attaque consiste donc à utiliser ce mécanisme afin de récupérer sa base de données et ainsi connaître les domaines qu'il gère :

![](../../../.gitbook/assets/0aabe3d225bde217e14e4a8d0390db80.png)

Ici "AXFR" signifie simplement : "donne moi toute les entrées que contient ta base". Mais dans notre cas, le transfert de zone ne fonctionne pas : le transfert a échoué. Après un certain temps de réflexion, il m'est venu l'idée d'utiliser l'IP du second serveur DNS, soit 192.168.1.89, et de réitérer la demande. En effet, il se peut qu'une sécurité soit mise en place afin que seul le second serveur DNS puisse effectuer cette requête, mais vu qu'il ne semble pas en ligne ici pas de problème pour se faire passer pour lui :

![](../../../.gitbook/assets/7b9b82ef5655cb5390f4ca60bcc4cdac.png)

Notre machine d'attaque possède maintenant l'adresse 192.168.1.89 qui correspond au serveur DNS secondaire. On tente à nouveau le transfert de zone :

![](../../../.gitbook/assets/add9a35fb4907dd4883e7417f0070800.png)

Cela fonctionne et la réponse est plus intéressante maintenant, nous possédons donc les sous-domaines valides et donc également une liste de noms d'utilisateur.

### Découverte des sous-domaines

Afin de pouvoir visiter les différents sous-domaines il est nécessaire de configurer l'adresse DNS de notre  machine pour qu'il pointe vers 192.168.1.88 pour la résolution des noms de domaine. Pour ma part, mon Kali est en fait une VM VirtualBox qui ne possède pas d'environnement graphique (je m'y connecte en SSH) depuis l'hôte qui est un Windows. Faire du browsing web à coup de `cURL` ou de Lynx ne m’intéressait pas vraiment donc j'ai configuré ma machine hôte directement. Pour que cela fonctionne j'ai du désactiver IPV6 (je ne sais pas si c'était réellement nécessaire, mais vu que cela ne fonctionnait pas dans le cas contraire ...) :

![](../../../.gitbook/assets/0997227a3480c99713f71d0d8b6f0a82.png)

Malheureusement, après avoir visité tous les sous-domaines possibles, aucune information réellement intéressante (à part quelques photos ou vidéos légèrement drôle).&#x20;

### PhpMyAdmin (CVE-2005-3299)

Nous savons qu'il y a un phpMyAdmin de disponible, mais l'accès nous est interdit (403 Forbidden). Après quelques minutes de recherche, j'ai tenté de changer d'adresse IP pour une autre, présente en réponse du transfert de zone (bon en fait je mens un peu, il y a deux bonnes heures de recherche entre temps :yum:).

La seule adresse qui est sous le même sous-réseau est l'adresse 192.168.1.34 qui correspond au sous-domaine "trusted.zincftp.com". Et là, si on tente d'accéder au phpMyAdmin avec notre nouvelle adresse IP (en se faisant donc passer pour "trusted.zincftp.com") :

![](../../../.gitbook/assets/4c44f6b53b180e114ce01820f2bb8584.png)

Pour info la configuration de ma machine hôte devient donc :

![](../../../.gitbook/assets/9673cf2a0583eac286937510ee9e6654.png)

On récupère ce que contient la base de données (c'est-à-dire pas grand chose ici) :

![](../../../.gitbook/assets/0bfa97114b00cc99a9f79ad2d651f649.png)

{% hint style="info" %}
Les trois dernières entrées sont des tests que j'avais effectués en phase de reconnaissance
{% endhint %}

Pas de mot de passe pour nous aider à avancer :neutral\_face: . On peut tenter de connaître la version de phpMyAdmin grâce à son ficher "README" :

![](../../../.gitbook/assets/1d471816486293db84ffcc15884dfd54.png)

Il s'agit donc de la version 2.6.4-pl1 qui contient une vulnérabilité de type LFI (CVE-2005-3299) :

![](../../../.gitbook/assets/64602b6b9da68084f6325d04cc8ee193.png)

Un `searchsploit` nous indique qu'un exploit en Perl existe mais un Burp suffit amplement ici. Voici la requête à exécuter pour récupérer le fichier "/etc/passwd" :

![](../../../.gitbook/assets/1abf0ad835859bcca8dc2b506661a683.png)

La réponse avec le fichier désiré :

![](../../../.gitbook/assets/da21167123f11fc36250e5f198ef910b.png)

Il y a un sacré paquet de comptes. Mais nous pouvons distinguer les comptes qui ont un accès à un shell ("/bin/bash"), des autres comptes de service ou de ceux indiqués avec "/bin/false" (comparable à un "/bin/nologin").&#x20;

Après un peu de recherche, on apprend que Pure-FTPd stocke les mots de passe des utilisateurs dans un fichier spécifique "/etc/pureftpd.passwd" :

![](../../../.gitbook/assets/e47a6592c3b534ff19c6b31a2e07f77c.png)

Grâce à la vulnérabilité de phpMyAdmin, nous récupérons ce fichier :

![](../../../.gitbook/assets/593e74488a36116dc5f7875ce6feeb02.png)

Hmm, ce fichier ne semble pas exister sur la machine dommage. En creusant un peu plus il apparaît que le chemin peut également être quelque chose comme "/etc/pure-ftpd/pureftpd.passwd" ou "/etc/pure-ftpd/conf/pureftpd.passwd" :

![](../../../.gitbook/assets/8e5a42a7f41610f21a955920841e48e3.png)

On tente à  nouveau :

![](../../../.gitbook/assets/fa0449555bf0bf11432d8a67817f3b40.png)

Bingo, il s'agit donc non pas des credentials d'un accès SSH mais plutôt des comptes FTP, c'est toujours ça. Rien ne nous empêche de faire du credentials stuffing afin de voir si les mêmes mots de passe ne sont pas réutilisés.&#x20;

### Reverse Shell

Dans un premier temps, nous crackons les mots de passe avec `john` :

![](../../../.gitbook/assets/44e343fc2e895996ed8645e682f830f1.png)

Pour info le dictionnaire utilisé était le fichier "rockyou" de Kali. Seul le dossier FTP de "tmartin" possède un fichier de type .rar (les autres sont vides), mais il est protégé par un mot de passe :

![](../../../.gitbook/assets/f02d32df306027446b9947ea6106a2da.png)

J'ai tenté de casser le mot de passe avec `john` (précédé d'un `rar2john`) mais c'était beaucoup trop long (j'ai bien sur essayé son mot de passe FTP mais sans succès) :

![](../../../.gitbook/assets/3a52de21fe9acb33db4e6f31da19157a.png)

Je me suis un peu renseigné sur comment fonctionne la protection de WinRAR et en effet il semble que c'est assez consommateur en temps/puissance à cause des nombreuses itérations sur SHA-1 :

![source : https://www.tomshardware.com/reviews/password-recovery-gpu,2945-8.html](../../../.gitbook/assets/1c7c16853817b9e9e234b31ab3f4efbe.png)

La solution était en fait plus simple, nul besoin de l'archive WinRar. Il faut uploader un web shell puis l'appeler directement en accédant au sous-domaine qui correspond à l'utilisateur :

![](../../../.gitbook/assets/6f7d270952772bde14d0a2f8c5b39e2e.png)

La paylaod a été générée avec `msfvenom`. Côté Metasploit on récupère donc une session `meterpreter` en tant que "www-data" :

![](../../../.gitbook/assets/1c2f4a312f5b6c481cf0785c8926452a.png)

Après une recherche de droits sudo, etc, on récupère un fichier qui mentionne une réinitialisation de mot de passe pour l'utilisateur "amckinley" :

![](../../../.gitbook/assets/6779b3030e0dea49c3fabb695a9bc366.png)

Voici le contenu entier de l'échange par mail :

![](../../../.gitbook/assets/1dda93ff17e03d78d3cbf998cb6fe0a2.png)

Le mot de passe de l'utilisateur "amckinley" a été réinitialisé à la valeur "agustinmckinley2ba9" (son prénom est présent dans le fichier "/etc/passwd" récupéré plus tôt).

Nous avons donc un accès SSH plus stable et peut être un compte plus permissif :

![](../../../.gitbook/assets/33efa852445206b376f716b34c10e836.png)

## Élévation de privilèges

Je n'ai pas trouvé ici d'autre solution que l'exploitation d'une vulnérabilité du kernel. La version du noyau est une 2.6.22 :

![](../../../.gitbook/assets/64796c71e8fc24aead124db9f23a067b.png)

Ce noyau est vulnérable à l'exploitation "vmsplice" ([https://downloads.securityfocus.com/vulnerabilities/exploits/27704.c](https://downloads.securityfocus.com/vulnerabilities/exploits/27704.c)). Une fois compilé et exécuté, nous obtenons un accès "root" :

![](../../../.gitbook/assets/9aab694a463e072e660351d28c16d88f.png)

## Conclusion

J'ai vraiment apprécié cette machine ! Le transfert de zone sort de ce que j'ai précédemment fait sur les Vulnhub et la suite est du même acabit (des fichiers pour nous mettre sur une fausse piste, pas mal de recherche à effectuer etc).

J'ai par contre passé plusieurs soirs dessus et je ne l'ai pas trouvé si simple que cela (pour mon niveau bien sûr). Le challenge était là et selon moi, il vaut le détour.

