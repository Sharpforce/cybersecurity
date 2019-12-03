---
description: 'Walkthrough de la machine Metasploit: 1'
---

# Metasploitable: 1

## Détails de la machine

**Nom :** Metasploitable: 1  
**Date de sortie :** 19 Mai 2010  
**Lien de téléchargement :** [https://download.vulnhub.com/metasploitable/Metasploitable.zip](https://download.vulnhub.com/metasploitable/Metasploitable.zip)  
**Niveau :** Facile  
**Objectif\(s\) :** obtenir un accès "root"  
**Description :**`One of the questions that we often hear is "What systems can i use to test against?" Based on this, we thought it would be a good idea throw together an exploitable VM that you can use for testing purposes.`  
****`Metasploitable is an Ubuntu 8.04 server install on a VMWare 6.5 image. A number of vulnerable packages are included, including an install of tomcat 5.5 (with weak credentials), distcc, tikiwiki, twiki, and an older mysql.  
You can use most VMware products to run it, and you'll want to make sure it's configured for Host-only networking unless it's in your lab - no need to throw another vulnerable machine on the corporate network. It's configured in non-persistent-disk mode, so you can simply reset it if you accidentally 'rm -rf' it.  
Source:` [`http://web.archive.org/web/20100525233058/http://blog.metasploit.com/2010/05/introducing-metasploitable.html`](http://web.archive.org/web/20100525233058/http://blog.metasploit.com/2010/05/introducing-metasploitable.html)

## Reconnaissance

Un `netdiscover` suivi d'un `nmap` afin d'en savoir plus sur notre cible :

![](../../../.gitbook/assets/4481aa8181ab746ad9f568403bf4a46a.png)

La machine "Metasploit: 1" possède donc l'adresse IP 192.168.1.12. Voici le résultat du `nmap` :

![](../../../.gitbook/assets/7d354cef77ea1547541819b70228e8c4.png)

Beaucoup de services disponibles sur cette machine, la reconnaissance va sans doute être un peu plus long que d'habitude 🙂 

### Service FTP

La machine dispose d'un service FTP sur le port 21 et `nmap` nous indique qu'il s'agit d'un ProFTPd 1.3.1. Il semble qu'une exécution de code distant soit possible pour les version &lt; 1.3.3g, mais l'exploit nécessite un compte valide \(que nous n'avons pas à l'heure actuelle\), et surtout aucun exploit semble disponible.

![](../../../.gitbook/assets/335733064ff6797d0880893b9c402c19.png)

### Service SSH

Le service SSH est un OpenSSH 4.7p1. Il sera possible d'énumérer les utilisateurs du système en fournissant au préalable un dictionnaire de noms d'utilisateur. Je n'ai rien trouvé d'autre de particulier sur cette version.

### Service Telnet

Un service Telnet ouvert, un login/mot de passe est demandé à la connexion :

![](../../../.gitbook/assets/ecec563e2b5af5f75afffb10c3101e96.png)

Au passage on note la version du système d'exploitation de la cible : Ubuntu 8.04.

### Service HTTP \(port 80\)

Ici on commence avec un `nikto` et un `dirb` qu'on laisse tourner pendant notre reconnaissance manuelle. Rien à noter puisqu'une seule page est disponible, celle par défaut du serveur Apache :

![](../../../.gitbook/assets/d182a076c56fec9385ac8dde09aab9a2.png)

On passe à l'analyse du résultat de `niko` et `dirb` :

![](../../../.gitbook/assets/6e44100616c727dfd95ac9577a0e5b33.png)

`nikto` nous remonte les version du serveur Apache \(soit 2.2.8\) et la version de PHP \(5.2.4\). Il identifie également une page nommée "phpinfo.php" qui exécute la méthode PHP `phpinfo()` :

![](../../../.gitbook/assets/4a486f027d980f7660f4e85bebb03af4.png)

#### Tikiwiki

Une URL "/tikiwiki" est également détectée, il s'agit d'une application web de type CMS et de travail collaboratif : 

![](../../../.gitbook/assets/255e2a11128e839d8561be10397cab24.png)

L'outil installé est en version 1.9.5. Cette version semble vulnérable à une fuite d'information avec la CVE-2006-5702 \(la CVE-2006-5703 concerne aussi cet outil pour cette version, mais il s'agit d'une vulnérabilité de type XSS\) :

![](../../../.gitbook/assets/a9de5e0f5e6b537bb957a7c6c6967917.png)

Une petite recherche sur la documentation nous apprend aussi que les credentials par défaut sont admin/admin. En effet, cela fonctionne :

![](../../../.gitbook/assets/d8abc36041fa91fa51c0b1f6032c7fbc.png)

Et pour terminer, la version 1.9.5 de ce logiciel est également vulnérable à une injection de code avec la CVE-2007-5423 que voici :

![](../../../.gitbook/assets/f324a527273c5c602f589066cea070ae.png)

Passons aux résultats de `dirb` :

![](../../../.gitbook/assets/f9e36fda75b12f89f7b9468ea7ca5f93.png)

#### Twiki

Une autre URL intéressante apparaît ici, celle d'un wiki d'entreprise. Ce wiki, nommé "Twiki" est disponible à l'URL "/twiki" :

![](../../../.gitbook/assets/a908f971cf73ccc6f52432573a3cb663.png)

En bas de la page du wiki, nous pouvons voir la version installée :

![](../../../.gitbook/assets/2c730ac5eb30dc6c2d21dd498a34f882.png)

Cette version est vulnérable à exécution de code à distant avec la CVE-2005-2877 :

![](../../../.gitbook/assets/6819bf2e2d8148755c22775db47e352d.png)

Ou encore à la CVE-2004-1037 :

![](../../../.gitbook/assets/94e7498938b0171635a500e81ff4445c.png)

### Samba

Une authentification anonyme est possible et certains répertoires partagés ne nécessite pas de droits spécifiques :

![](../../../.gitbook/assets/aec8554820c5b38a5a66292c38d5cd6a.png)

Il est également possible d'énumérer les comptes présents sur la cible :

![](../../../.gitbook/assets/4343b92116812e9dd19e24245052274a.png)

La version 3.0.20 est vulnérable à une exécution de code distant. Cette vulnérabilité est décrite par la CVE-2007-2447 :

![](../../../.gitbook/assets/7736cf24296386d04fbf57e29ab65ded.png)

### Compilateur \(Distccd\)

Il s'agit du serveur du compilateur Distccd en version 4.2.4 qui est vulnérable à une exécution de code distant. Le petit nom de la CVE en question est la CVE-2004-2687 :

![](../../../.gitbook/assets/7fa6d91408980fb6109f72988caa468c.png)

### PostgreSQL

`nmap` indique une version de PostgreSQL comprise entre 8.3.0 et 8.3.7. Metasploit nous permet d'affiner le numéro de version :

![](../../../.gitbook/assets/b6a5c56d09dd613c3fbfa5e86eee54f0.png)

Il s'agit donc de la version 8.3.1. On voit également l'info "Post-Auth" indiquant que Metasploit a réussi à se connecter avec les credentials présents dans les options \(cela lui permet d'affiner la détection de version\), à savoir postgres/postgres. On peut tenter de s'y connecter afin de confirmer :

![](../../../.gitbook/assets/e7f915f221f375059cd6e4a5258b07aa.png)

### Serveur HTTP \(8180\)

Il s'agit ici d'un serveur Apache Tomcat en version 5.5. La page d'accueil est la page par défaut, ce qui peut laisser penser que le serveur est encore en cours de configuration :

![](../../../.gitbook/assets/b6d17eefcd54d1ee5931876856f8eb8f.png)

Il faut donc penser ici à tester les credentials par défaut \(tomcat/tomcat\) :

![](../../../.gitbook/assets/7f343a60811f4cdaadc1626d025a0122.png)

Nous avons accès à l'interface d'administration de Tomcat. Cela peut nous être utile afin d'uploader un .war file malicieux.

## Exploitation

### Tikiwiki \(CVE-2006-5702\)

Un module Metasploit existe afin d'exploiter la vulnérabilité de Tikiwiki :

![](../../../.gitbook/assets/5937f039a0b7f06fae4dea3c0f3e449c.png)

Connaissant les credentials de la base de données `mysql` il est possible maintenant de s'y connecter \(port 3306\) :

![](../../../.gitbook/assets/e161cb79d25d07063818e2d032c52cbb.png)

### Tikiwiki \(CVE-2007-5423\)

Il existe également un module Metasploit pour cette autre vulnérabilité de Tikiwiki :

![](../../../.gitbook/assets/8549cfd60cfde07d1b51e9d15597fb7d.png)

Nous possédons actuellement un compte à privilèges limités.

### Samba \(CVE-2007-2447\)

![](../../../.gitbook/assets/e9e955b57b4263ef445d972a428a1a82.png)

Cette vulnérabilité de Samba est plus intéressante dans le sens ou aucune élévation de privilèges n'est nécessaire : nous sommes déjà "root".

### Distcc \(CVE-2004-2687\)

![](../../../.gitbook/assets/c19f07c30b4a95b2360e5000c187f5e1.png)

Un compte limité "daemon" pour cette vulnérabilité sur Distcc.

### Autres exploitations

Il y a encore d'autres exploitations à effectuer, que cela soit sur Twiki, PostgreSQL ou encore le serveur Apache Tomcat \(chargement d'une application malicieuse, par exemple un reverse shell généré avec `msfvenom`\) : tout cela peut se faire avec Metasploit.

## Élévation de privilèges

Dans le cas d'un accès avec un compte limité il nous faut trouver un moyen de devenir calife à la place du calife. Un `uname -a` nous donne :

![](../../../.gitbook/assets/14e8b0e4d9ac8cb076677236e6712130.png)

Les versions 2.6 de Linux sont vulnérables à la faille "udev". Nous allons faire le tout grâce à Metasploit en exploitation la CVE-2007-5423 déjà vu pour obtenir notre shell limité :

![](../../../.gitbook/assets/947bf365afbf2eec332e0619124fc3ab.png)

La vulnérabilité de Tikiwiki est exploitée, nous avons un shell limité. Afin d'effectuer l'élévation de privilèges il nous faut connaitre le PID du process "udev" :

![](../../../.gitbook/assets/00bcb0a4512ee99a15b04305201ec7e7.png)

Avec la technique du "ps" il ne faut pas oublier d'ôter "1" au PID affiché pour le processus "udev". On indique à Metasploit d'utiliser l'exploit Udev :

![](../../../.gitbook/assets/85e56c3abb86f99f84d718c21639b864.png)

Travail terminé, nous sommes "root".

## Conclusion

La machine en soi n'est pas difficile, mais il y a tellement de services et de vulnérabilités à exploiter que cela en devient long. 

J'ai identifié d'autres petites choses comme des weak credentials avec l'utilisateur user/user \(donc disponible pour du telnet, SSH etc\). Une vulnérabilité sur le PostgreSQL qui a les droits en écritures sur le "/tmp" et qui permet de récupérer un shell \(module Metasploit disponible\). Il doit y avoir encore pas mal de choses encore mais je pense avoir fait un bon petit bout déjà.

J'ai également tenté d'exploiter les vulnérabilités sur le wiki d'entreprise Twiki grâce aux modules de Metasploit mais cela n'a pas fonctionné, je n'ai pas cherché plus loin que ça mais il me semble bien que le wiki soit vulnérable pourtant.

