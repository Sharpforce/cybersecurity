# Nucular Power Plant

## Détails du challenge

"Nucular Power Plant!" est un challenge Web de niveau très facile (baby plus exactement). Son accès se fait via une adresse indiquée dans son énoncé :

![](../../../.gitbook/assets/18d934334db3d49e9381bea4119aa36e.png)

Il a été résolu 238 fois pour 973 équipes :

![](../../../.gitbook/assets/94c1aa3d2fb65254ea0bc397bf71ab38.png)

## Reconnaissance

Une seule page est à priori disponible. Cette page permet de visualiser des informations concernant des centrales électriques :

![](../../../.gitbook/assets/eb26144873cd4496e53df9fa0eea37bb.png)

La page ne se recharge pas lorsqu'on sélectionne une centrale mais des requêtes websocket permettent de récupérer les infos :

![](../../../.gitbook/assets/c50905e23ed8994ea0195fa45cc2c660.png)

![](../../../.gitbook/assets/c0e23e78ddfbceae388702f5f1b4072a.png)

On est tenté assez rapidement de tester l'injection SQL. L'insertion d'un caractère `'` remonte une erreur indiquant qu'aucune ligne n'est remontée :

![](../../../.gitbook/assets/5c43a430070a040fa3242b9ee2d20469.png)

On tente le caractère `"` :

![](../../../.gitbook/assets/5aa5aac1c63a952416dcf9533cf80179.png)

L'erreur retournée renseigne sur le type de base de données utilisé tout en renfonçant notre confiance quant à la présence d'une injection. Pour confirmer, on effectue un test de tautologie :

![](../../../.gitbook/assets/258eb1c118996ed1d1510028f9d4db08.png)

## Exploitation

Il s'agit donc d'exploiter une injection SQL mais sur une base SQLite, ce qui va faire légèrement varier les requêtes par rapport à une habituelle base MySQL. Dans un premier temps, on confirme la présence de seulement 7 centrales électriques et qu'il n'existe pas de centrale secrète :stuck\_out\_tongue: :

![](../../../.gitbook/assets/5706489061982e3fade549ad8774eee4.png)

On retrouve le nombre de colonnes de la table :

![](../../../.gitbook/assets/44d073aa522511cb5b95997e4cdb2277.png)

La table possède donc 6 colonnes. Reste maintenant à régler le problème de typage :

![](../../../.gitbook/assets/0fd5fcda7e288bae7649a510128f2ffe.png)

Après quelques essais, notre requête ne remonte plus d'erreur :

![](../../../.gitbook/assets/f1c42761f57149566247cd8678ff5e69.png)

On récupère quelques informations, par exemple le numéro de version :

![](../../../.gitbook/assets/48e5dbcb83658b20b4b9fdbf7f98a439.png)

On passe maintenant à la récupération des informations sur les tables existantes :

![](../../../.gitbook/assets/4efa49c13839dc9da436dfa65186c4d7.png)

![](../../../.gitbook/assets/bde17f75168d7ac63e9f4b78d443fa8b.png)

La table "secret" contient le flag :

![](../../../.gitbook/assets/4ff28b85c711c663e6dd552d387fe061.png)

Le flag permettant de valider le challenge est donc `flag{sqli_as_a_socket}`.



