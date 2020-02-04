---
description: 'Walkthrough de la machine Kioptrix: Level 3'
---

# Kioptrix: Level 1.2 \(\#3\)

## Détails de la machine

**Nom :** Kioptrix: Level 1.2 \(\#3\)  
**Date de sortie :** 11 Février 2011  
**Lien de téléchargement :** [http://www.kioptrix.com/dlvm/KVM3.rar](http://www.kioptrix.com/dlvm/KVM3.rar)  
**Niveau :** Facile  
**Objectif\(s\) :** obtenir un accès "root"  
**Description :**`It's been a while since the last Kioptrix VM challenge. Life keeps getting the way of these things you know.`  
****`After the seeing the number of downloads for the last two, and the numerous videos showing ways to beat these challenges. I felt that 1.2 (or just level 3) needed to come out. Thank you to all that downloaded and played the first two. And thank you to the ones that took the time to produce video solutions of them. Greatly appreciated.  
  
As with the other two, this challenge is geared towards the beginner. It is however different. Added a few more steps and a new skill set is required. Still being the realm of the beginner I must add. The same as the others, there’s more then one way to “pwn” this one. There’s easy and not so easy. Remember… the sense of “easy” or “difficult” is always relative to ones own skill level. I never said these things were exceptionally hard or difficult, but we all need to start somewhere. And let me tell you, making these vulnerable VMs is not as easy as it looks…`  
****

## Reconnaissance

La machine étant en DHCP il nous faut tout d'abord récupérer son adresse IP :

![](../../../.gitbook/assets/2deb7203ebaf8282be37f5b5c566da05.png)

Comme indiqué dans la consigne du challenge, la machine a besoin d'une résolution DNS avec le nom `kioptrix3.com`. Direction donc le fichier `/etc/hosts` :

```text
192.168.56.53	kioptrix3.com
```

Le `nmap` habituel, afin de déterminer les services exposés par la machine :

![](../../../.gitbook/assets/205a03d8a77e3eda16e25c1447f7583e.png)

Peu de services puisque seul un service SSH et un serveur HTTP sont exposés.

### Service SSH

Pas de vulnérabilité notable pour cette version de SSH, ça ne sera sans doute pas notre porte d'entrée :

![](../../../.gitbook/assets/4aeb07eae3f4a5743e45a741ab7e1fe8.png)

### Serveur HTTP

Le `nmap` nous apprend que le serveur HTTP est un Apache 2.2.8 utilisant PHP en version 5.2.4. Ajoutons à cela les résultats d'un `nikto` :

![](../../../.gitbook/assets/acba11cde47bf0cc7aa3a4b96d66634b.png)

Le scan nous apprend la présence d'un phpMyAdmin. La consultation du fichier `changelog.php` nous indique son numéro de version :

![](../../../.gitbook/assets/2545c9cd63d702ea68f6cc394345308a.png)

Je ne mets pas la sortie du `dirb` car cela ne donne pas grand chose.

Passons maintenant à l'analyse des pages web hébergées par le service. Tout d'abord la page d'accueil :

![](../../../.gitbook/assets/e3f3136cda2c6480eebe477ad90a07c9.png)

La mire de login est intéressante car elle nous informe du CMS utilisé :

![](../../../.gitbook/assets/7173a10a17abc6ab810f7da6a3bebd85.png)

LotusCMS était un CMS open-source écrit en 2007/2008. Le code source est encore disponible sur [GitHub](https://github.com/kevinbluett/LotusCMS-Content-Management-System) \(hébergé à l'origine sur [sourceforge](http://sourceforge.net/projects/arboroiancms/%20)\).

Mon premier but était d'identifier la version utilisée, j'ai donc récupéré le code source afin de repérer les différents fichiers pouvant contenir cette information. Le titre du fichier `install.html` présent dans le répertoire `/style/comps/admin/` nous rend ce service :

![](../../../.gitbook/assets/0a7d7315353be9fb05d96c32190f0256.png)

LotusCMS en version 3.0 possède une vulnérabilité de type LFI pouvant amener à une RCE \([CVE-2011-0518](https://www.cvedetails.com/cve/CVE-2011-0518/)\). Pour faire court, le système de post de commentaires sur le blog permet en fait l'écriture de fichiers sur le disque, puis la faille LFI permet de les charger.

## Exploitation

### CVE-2011-0518

Un module `metasploit` existe concernant cette vulnérabilité :

![](../../../.gitbook/assets/a1890c95c8818d0bf8cc7f3a7bc535ff.png)

L'exploitation sera donc plutôt aisée :

![](../../../.gitbook/assets/2988561f2435318f7427a3c3768b174c.png)

## Élévation de privilèges

La phase de reconnaissance permet de récupérer la liste des utilisateurs de la machine :

![](../../../.gitbook/assets/2c3f30bda9d912122fe4e1957260e240.png)

Deux comptes sont des utilisateurs de la machine : **dreg** et **loneferret**. Après avoir fouillé dans le `/home` de **loneferret** :

![](../../../.gitbook/assets/c5b0eda6c53681648b49e1c8f970edf3.png)

Le fichier `CompanyPolicy.README` \(ainsi que le `.bash_history`\) indique l'utilisation d'un programme nommé `ht`. Après quelques recherches, il s'agit d'un éditeur de texte. Chose intéressante, le binaire possède le **suid** à 1 :

![](../../../.gitbook/assets/c5e239b32f7331685e26d9e96aa25bc0.png)

Je tente donc de lancer le binaire :

![](../../../.gitbook/assets/07676563dbfc28b8a2447d27aa1f7039.png)

Je suis pas un grand connaisseur des erreurs XTERM et compagnie. Je tente un peu à l'aveugle quelques commandes trouvées ici et là, par exemple :

![](../../../.gitbook/assets/00f9b4c7c13114c658f7c0f472698099.png)

Cela améliore sensiblement les choses, à savoir que je peux maintenant lancer le binaire mais ce n'est toujours pas utilisable \(la saisie de caractères écrit tout simplement sur l'interface\) :

![](../../../.gitbook/assets/6b0c60d0c0bc18120c7c034dc721d701.png)

Après un certain temps à tourner en rond, j'ai décidé de rétropédaler et d'approfondir la reconnaissance du système.

Mon premier objectif est d'avoir accès à la partie administration de LotusCMS. Etant donné qu'il s'agit d'un CMS qui utilise seulement des fichiers plats, le mot de passe doit se situer dans un de ses fichiers de configuration.

Après quelques recherches il se trouve que le mot de passe est présent dans un fichier distinct pour chaque utilisateur. Dans notre cas il s'agit de l'administrateur :

![](../../../.gitbook/assets/380aae7f14e17ac297e227f92c0780fe.png)

Après une lecture du code source, il se trouve que le mot de passe, avant d'être haché en SHA1, est tout d'abord concaténé avec un sel statique. Ce sel est présent dans le fichier /data/config/salt.dat :

![](../../../.gitbook/assets/5dd5d4f39ca30c4520b36b8e5b4b4381.png)

John va nous aider à cracker ce mot de passe, mais il faut bien prendre en compte le sel utilisé. J'ajoute donc le hash dans un fichier sous le format `username:hash$sel` :

![](../../../.gitbook/assets/abdc00f93c7dbe14060e4aa7c4f3a9d4.png)

Il ne faut pas oublier d'indiquer le format \(ici `dynamic_24`\) dans notre ligne de commande :

![](../../../.gitbook/assets/a7299ae089a9a30c4ea1c1f906a66d21.png)

Tout cela nous donne accès à la partie administration de LotusCMS :

![](../../../.gitbook/assets/e4c88163fa456c0f2fd3e190a8acc3fe.png)

Malheureusement, impossible d'aller plus loin dans la compromission de la machine par ce vecteur, je me penche alors sur l'accès shell.

La recherche de mot de passe en plaintext porte ses fruits au niveau du répertoire `/gallery` :

![](../../../.gitbook/assets/522a3d5f14bea3929868e3676a3f0935.png)

Cela nous permet d'avoir accès à la base MySQL avec le compte root/fuckeyou. Pour une analyse plus facile du contenu de la base de données, je passe par le phpMyAdmin.

Nous avons donc un premier mot de passe dans la table `gallarific_users` :

![](../../../.gitbook/assets/395cef4fc251ed7ef313d36062b0da48.png)

Et deux autres mots de passe \(cette fois hashés\) pour les utilisateurs **dreg** et **loneferret** dans la table `dev_accounts` :

![](../../../.gitbook/assets/8b492cfedeb8f61b28c6978d024e7bda.png)

On tente de récupérer les mots de passe en clair :

![](../../../.gitbook/assets/c1fce2e38ed6841b010536035e7fa937.png)

Cela nous permet de se connecter en SSH avec le compte de **loneferret** :

![](../../../.gitbook/assets/6ac743790d4e460ef50b7b0661ceb8d5.png)

Etant donné que `ht` possède le bit suid à 1, il est possible d'éditer des fichiers avec les droits root. On modifie alors le fichier `/etc/sudoers` :

![](../../../.gitbook/assets/0d7ea31987d0a4a1e312a93a5b4a5ca4.png)

afin d'ajouter le droit su à notre utilisateur **loneferret** :

![](../../../.gitbook/assets/743a0c067215825e0d3a98017dd1b66b%20%282%29.png)

Un petit `sudo` su plut tard, nous voilà root :

![](../../../.gitbook/assets/fa187d718ea2ce9be51246ea929a9e6b.png)

## Conclusion

Très facile jusqu’à la récupération d'un shell, la machine se corse en ce qui concerne l'élévation de privilèges. J'ai passé un sacré bout de temps à tenter de faire fonctionner `ht` avec le compte **www-data** avant d'explorer d'autres solutions.

J'ai apprécié de pouvoir/devoir étudier le code source du CMS afin de connaitre les emplacements des données intéressantes et également son fonctionnement \(notamment sur la façon de stocker ses mots de passe\).

La leçon à retenir est de ne pas oublier d'explorer les autres pistes quand on est bloqué à une étape afin de collecter un maximum de données. Je m’aperçois que je dois fortement m'améliorer en ce qui concerne la phase de reconnaissance post compromission.

