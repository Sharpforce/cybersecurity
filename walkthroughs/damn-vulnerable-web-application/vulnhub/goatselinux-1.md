# GoatseLinux: 1

## Détails de la machine

**Nom :** GoatseLinux: 1\
**Date de sortie :** 27 Juin 2009\
**Lien de téléchargement :** [http://neutronstar.org/tmp/GoatseLinux\_1.0\_VM.rar](http://neutronstar.org/tmp/GoatseLinux\_1.0\_VM.rar)\
**Niveau :** Facile\
**Objectif(s) :** obtenir un accès "root"\
**Description :**`GoatseLinux v1.0 pentest lab Virtual MachineSteve Pordon`\
`2009.06.27`\
`Feel free to distribute this far and wide under the gnu license.`\
`This is specifically built for VMware 6.5 compatibility.`\
`WARNING: GoatseLinux is intentionally unsecure. It was designed as a laboratory box to practice penetration testing on. Due to the wide open nature of nearly every program installed on it, I would strongly advise against setting your VM network to anything other than "host-based," unless you enjoy your VMs being used as zombie spamboxes.`\
`Notes:`\
`Built on the Slax 5.0.7 distro.`\
`Source: readme.txt`\


## Reconnaissance

Pour scanner une cible on doit tout d'abord connaitre son adresse IP. Cela peut se faire grâce à l'outil `netdiscover` :

![](../../../.gitbook/assets/dda4d9317564af4bac9cdb698758b3b5.png)

Le listing des services via `nmap` pour l'adresse IP 192.168.1.28 :

![](../../../.gitbook/assets/af71b11b2f198a81e0bb32cf2572458f.png)

### Service HTTP

Rien de très intéressant sur le service web (qui est en cours de construction) disponible sur le port 80 malgré un `nikto` et un `dirb`.

Résultat du `nikto` :

![](../../../.gitbook/assets/890e2275dc4d88860063aa3bdbfd5976.png)

Résultat du `dirb` :

![](../../../.gitbook/assets/1cd9693cd6eb8444ed133049e00abc09.png)

La page "goatse.html" qui est disponible quand on clique sur l'image d'accueil permet de récupérer trois adresses emails ainsi que la fonction de chacun des utilisateurs :

![](../../../.gitbook/assets/0a4688e8850df7f64a34a74e23a0f214.png)

L'analyse s'arrête ici pour le serveur web, on continue avec le prochain service : Webmin.

### Webmin

Une recherche sur le webmin permet d'identifier une vulnérabilité de type Arbitrary File Disclosure :

![](../../../.gitbook/assets/cab2a936f9146e332f0b86164a81f9a4.png)

Toutes les version inférieures à 1.290 sont vulnérables à un Arbitrary File Disclosure

## Exploitation

### Webmin (CVE-2006-3392)

L'exploitation reste très simple avec un script PHP existant. On récupère un premier fichier qui est "/etc/passwd" :

![](../../../.gitbook/assets/176bb3cdc4ca3d61b6fb564baa73a371.png)

On continue avec le fichier "/etc/shadow" :

![](../../../.gitbook/assets/892cb970140ba41f582b89cbfc324dc9.png)

On va tenter de cracker les mots de passe grâce à John The Ripper. Pour cela tout d'abord un `unshadow` :

![](../../../.gitbook/assets/c243fdf5bb5a8cc8be6ce0f395f12e71.png)

Puis on lance `john` :

![](../../../.gitbook/assets/3562a697fcc39cb70d567ed1ccf80bbd.png)

On récupère quatre des cinq mots de passe possibles

## Élévation de privilèges

On se connecte avec chacun des comptes en SSH (à noter l'utilisation de quelques options SSH car la machine cible commence à se faire vieille) :

![](../../../.gitbook/assets/442a348df6358e910d38cfc44fa83133.png)

Un de ces comptes possède les droits `sudo` :

![](../../../.gitbook/assets/8e25dcb12d7ae8952614b739aa551605.png)

Travail terminé

## Conclusion

Aucune difficulté ici car le Webmin était déjà connu grâce à d'autres machines et l'élévation de privilèges ne requiert aucune recherche particulière (sauf se connecter sur tous les comptes pour connaitre celui qui possède les droits `sudo`).
