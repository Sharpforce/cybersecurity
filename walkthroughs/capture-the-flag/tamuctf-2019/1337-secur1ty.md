# 1337 Secur1ty

## Détails du challenge

"1337 Secur1ty" est un challenge Web de niveau difficile (hard). Son accès se fait grâce à une URL indiquée dans une modale :

![](../../../.gitbook/assets/930d4313c8678e573f5b78af7a9712d2.png)

## Reconnaissance

L'application permet de s'authentifier ainsi que de créer un compte. Je commence donc par m'enregistrer afin d'aller voir ce qu'il se cache derrière cette mire d'authentification :

![](../../../.gitbook/assets/70d116abcfc8e28e45508091270c79e8.png)

Une fois le compte créé, je suis automatiquement authentifié et redirigé vers la page de profil :

![](../../../.gitbook/assets/8f1059ef897bc301724b0b1dd09ccb3c.png)

Je possède une adresse email "sforce@1337secur1ty.hak" qui a été automatiquement créée par l'application. Avant d'aller plus loin, je m'intéresse au mécanisme de session. L'application fournit deux cookies. Le premier nommé "secret", semble contenir une valeur aléatoire, le second est nommé "userid" et a pour valeur 3, qui représente sans aucun doute l'id en base de données de la ligne correspondant à notre compte. Je tente tout d'abord de changer cet id, l'objectif étant de voir s'il n'est pas possible d'accéder à un compte d'une autre personne. Malheureusement cela me ramène à la mire d'authentification.

Je continue donc l'exploration des fonctionnalités avec l'onglet "Messages" et "Employees", en commençant par le dernier, soit "Employees" :

![](../../../.gitbook/assets/d28b82029915e4438eeb715bfba48096.png)

Le premier compte est intéressant, il s'agit d'un compte de type administrateur. L'identifiant "ID" correspond également à l'id stocké dans le cookie, soit "3" dans mon cas.

Je passe à la fonctionnalité "Messages" et j'ai déjà un message en attente de lecture provenant de l'administrateur du site :

![](../../../.gitbook/assets/47f8040228fdbc756bb400b60518b7c1.png)

En cliquant sur le numéro du message (# 1) il est possible de lire le message complet :

![](../../../.gitbook/assets/a4f699af8baec38d1d25722d04cd815c.png)

Le message possède l'id 2 présent en paramètre de l'URL. Le champ des possibles est assez grand ici, je commence par tenter d'accéder à d'autres messages en itérant l'id. L'itération est effectuée via Burp et son intruder pour pouvoir itérer sur un grand ensemble de valeur dans le cas ou un message serait "caché" avec un id très grand par exemple :

![](../../../.gitbook/assets/28697db240d76923610b0d07f0bb3c7f.png)

La taille de la réponse permet de voir que l'id 1 est un message valide :

![](../../../.gitbook/assets/2c6c761d73a196479ee4943efd991971.png)

Intéressant, les cookies ne doivent pas être si sécurisés que cela. Je tente maintenant de détecter une injection SQL. Après plusieurs essais, j'identifie l'injection grâce à un `sleep(10)`. De plus, il faut faire attention au paramètre qui ne semble pas être un entier :

![](../../../.gitbook/assets/d4be1c819151cfdd6a9b167564aecd66.png)

## Exploitation

Je sors l'artillerie lourde pour gagner du temps, merci `sqlmap` . Une seule base de données est disponible (en plus de "information\_schema") nommée "1337\_Secur1ty"  :

![](../../../.gitbook/assets/5146224b790ea51d4d86f4d5f696c0e1.png)

Je dump la base qui m'intéresse, et plus particulièrement la table nommé "Users" :

![](../../../.gitbook/assets/0672cebbf0b3194b2d7291e933b03f5a.png)

`sqlmap` n'arrive pas à cracker le mot de passe de l'administrateur. Ce n'est pas grave, connaissant son id (id 1) et possédant la valeur de son jeton secret (transmis par le cookie) il me suffit d'un petit changement et me voilà connecté en tant qu'administrateur. Le flag s'affiche alors :

![](../../../.gitbook/assets/78f21d6c405c189c260414201afd63ec.png)

Il est tout de même possible de récupérer le mot de passe grâce à [https://hashtoolkit.com](https://hashtoolkit.com) :

![](../../../.gitbook/assets/7b9c8d20d95cfd410663f1a6cb6e229a.png)

Mais la connexion au compte administrateur n'est en fait pas possible, car je ne connaissais pas son OTP (One Time Password) demandé à l'authentification.
