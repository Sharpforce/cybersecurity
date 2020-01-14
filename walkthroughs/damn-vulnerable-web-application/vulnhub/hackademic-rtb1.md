# Hackademic: RTB1

## Détails de la machine

**Nom :** Hackademic: RTB1  
**Date de sortie :** 6 Septembre 2011  
**Lien de téléchargement :** [https://download.vulnhub.com/hackademic/Hackademic.RTB1.zip](https://download.vulnhub.com/hackademic/Hackademic.RTB1.zip)  
**Niveau :** Facile  
**Objectif\(s\) :** obtenir un accès "root" et lire le flag situé dans le fichier `/root/Key.txt`  
**Description :**  
`This is the first realistic hackademic challenge (root this box) by mr.pr0n  
Download the target and get root.  
After all, try to read the contents of the file 'key.txt' in the root directory.  
Enjoy!`

## Reconnaissance

La machine étant en DHCP il nous faut tout d'abord récupérer son adresse IP :

![](../../../.gitbook/assets/1de092adf19aba548408904500ec6b76.png)

On garde les mêmes habitudes avec un scan de services grâce à `nmap` :

![](../../../.gitbook/assets/7f43db7f25b63493ad897e7f93a6c111.png)

Le port 22 \(service SSH\) est clos, il nous reste donc seulement le serveur HTTP sur le port 80.

### Serveur Web

On utilise les habituels outils, `nikto` :

![](../../../.gitbook/assets/a94a30131e61354e02ef83efefd7b38d.png)

qui nous indique l'installation d'un CMS Wordpress. Suivi de `dirb` :

![](../../../.gitbook/assets/3877f4f2389732ec04bc30be6da13c1d.png)

qui lui, remonte l'URL d'un `/phpMyAdmin` retournant un code 403 Forbidden. On continue la reconnaissance en accédant à la page d'accueil du site :

![](../../../.gitbook/assets/581211021923cfc3598f0f06a4f7c2e3.png)

Le lien "target" nous dirige vers une seconde page `/Hackacdemic_RTB1` :

![](../../../.gitbook/assets/1110476b13592225a5110d1134eb1706.png)

Je retente un `dirb` à parti de cette nouvelle URL afin de récupérer un maximum d'informations :

![](../../../.gitbook/assets/fdd61ce62ddbdeec16e73154cda0cd9e.png)

Il nous permet la découverte des URL liées au Wordpress déjà détecté par `nikto` . On identifie rapidement la mire d'authentification du CMS à l'URL `/Hackademic_RTB1/wp-login.php` :

![](../../../.gitbook/assets/832e45afd0b7edbc13cc7bab6270ee79.png)

Après quelques tests basiques d'injections, on repère facilement l'injection SQL sur la paramètre `?cat=`de la page d'accueil `index.php` :

![](../../../.gitbook/assets/5731466f855da622f28fee578a59e50e.png)

## Exploitation

L'exploitation de l'injection SQL peut être nous permettre de récupérer des identifiants permettant de s'authentifier sur le Wordpress :

![](../../../.gitbook/assets/3a8f02fdca30c632a865c0c53fa1ef8a.png)

{% hint style="info" %}
Je ne sais pas si cela venait d'une instabilité de ma VM mais les codes HTTP renvoyés par l'application étaient des codes 500. Le contenu HTML étant bien renvoyé, l'injection reste donc possible mais j'ai dû ajouter l'option `--ignore-code=500` à `sqlmap`
{% endhint %}

L'exploitation de l'injection permet de récupérer certains logins/mots de passe \(table wp\_users\) :

![](../../../.gitbook/assets/bd27d8eba5b7571ed8c1ba5d07d7e624.png)

La suite de l'attaque se poursuivra sans doute avec le compte GeorgeMiller car il semble posséder le plus de permissions :

![](../../../.gitbook/assets/6703c5ecf1e52cb1f65c34e8e24115be.png)

Le compte possède les droits d'édition sur certaines pages `.php` \(par exemple ici `wp-content/plugins/markdown.php`\) :

![](../../../.gitbook/assets/df8ca090d7ddd83fbcfa634b830de656.png)

L'exploitation de cette fonctionnalité semble assez évidente, on va remplacer le contenu de la page par le code d'un reverse shell. Tout d'abord on génère le code avec `msfvenom` :

![](../../../.gitbook/assets/f5cd140454b0bd378039257745e506d0.png)

On modifie la page `markdown.php` :

![](../../../.gitbook/assets/f4dab86e60aeefa39e27f2525fa4eb67.png)

On sauvegarde les modifications puis met en place le handler sur la machine d'attaque :

![](../../../.gitbook/assets/3bd9b91184cfe1f0e1d724a1858b7560.png)

On active la payload en visitant la page malicieuse :

![](../../../.gitbook/assets/d1dcbc382b9505c50f821534b36bb121.png)

La connexion est établie, nous sommes dans la place :

![](../../../.gitbook/assets/66625727be0a50597c141037ae907fd9.png)

## Élévation de privilèges

Un petit tour de reconnaissance avec nos nouveaux droits :

![](../../../.gitbook/assets/a24222d7cc5795b5ddff4d09b810511c.png)

On liste les utilisateurs présents sur la machine :

![](../../../.gitbook/assets/d1ed2ad0ce8f162190ce9f6980c58cd7.png)

On récupère le mot de passe de connexion à la base mySQL :

![](../../../.gitbook/assets/8de69eb5bdf00a492b2523a39cae714b.png)

On termine par une recherche de mots de passe hardcodé autre que celui-ci ou encore la recherche de binaire suid mais rien. Je me tourne vers les exploits possibles pour la version 2.6.31 du noyau Linux. Pour cela je me sers de ce [GitHub](https://github.com/lucyoa/kernel-exploits) qui liste les différentes vulnérabilités et versions vulnérables :

![](../../../.gitbook/assets/6678b60fcb8e6e699f21c41ba4ad913b.png)

Ici j'ai posé mon cerveau et j'ai simplement testé un à un tous les exploits possibles pour la version 2.6.31 en version x86 \(je ne dis pas que c'est la meilleure solution 🙃 \). Au bout d'un moment il s'avère que l'exploit RDS passe :

![](../../../.gitbook/assets/983b98b1ac80673d879d646bdec360dd.png)

Afin de l'exécuter rien de plus simple, on met en place un serveur HTTP python côté attaque et on récupère l'exploit avec un `wget` :

![](../../../.gitbook/assets/ebbf097099ab5c7faeb1c19e587a4701.png)

On le compile :

![](../../../.gitbook/assets/cecf0e907f02089e7c4a99eff6d6ade6.png)

Puis on l'exécute :

![](../../../.gitbook/assets/d50cf2738935fc5ea6723602878318c3.png)

Afin de satisfaire la demande du créateur de la machine, on récupère le flag contenu dans le fichier `/root/key.txt` :

![](../../../.gitbook/assets/e5503701c90771ac4905a7458e17cc1a.png)

Travail terminé.

## Conclusion

La machine n'est pas difficile. L'injection SQL se trouve facilement et permet d'aller à la prochaine étape en possédant des comptes Wordpress valides. Récupérer un shell ne présente pas de soucis non plus et la technique d'exécution de code via une page php forgée pour l'occasion est maintenant une habitude.

Là où je suis un peu moins fier de moi c'est l'élévation de privilèges ou j'ai bêtement testé tous les exploits existants pour la version appropriée. L'excuse était que la fatigue commençait à m'envahir et je voulais terminer la machine au plus vite 😋 . Je pense qu'il est possible de faire une première analyse afin de savoir quel exploit peut convenir le mieux sans avoir à tous les exécuter mais je suis pas un grand connaisseur dans ce domaine.

