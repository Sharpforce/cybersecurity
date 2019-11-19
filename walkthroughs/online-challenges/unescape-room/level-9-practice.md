# Level 9 \(practice\)

## Challenge \#1

### Description

Appeler la fonction `braveSuperHero()` avec la chaîne de caractères `"viaq7ed38yz"` en argument mais cette fois sur l'attribut `src` dans une balise `<img />` :

![](../../../.gitbook/assets/7e69aeadd2e01ec86731348991afd9b7.png)

### Résolution

Le caractère `"8"` est ici filtré et la chaîne est inversée :

![](../../../.gitbook/assets/204881d35f6f224dc6f3542f0667e36b.png)

Si nous tentons seulement d'inverser la chaîne et de gérer le caractère filtré avec une opération mathématique, plus rien ne s'affiche :

![](../../../.gitbook/assets/cf7fc79bf3009af39d9187166e80619a.png)

C'est plutôt difficile de trouver exactement les filtres en place ici, mais la longueur de la payload semble jouer un rôle. De plus, en allant pas à pas j'ai du utiliser la méthode `toLowerCase()` pour les caractères `"z"` et `"a"` ce qui m'a permis de valider ce challenge :

![](../../../.gitbook/assets/609e1e26d092facf4fb4f78887fc679c.png)

## Challenge \#2

### Description

Appeler la fonction `fancyHuman()` avec la chaîne de caractères `"rm2zma4i1v6"` en argument mais cette fois sur la valeur de la propriété `background-color` du sélecteur `banner` :

![](../../../.gitbook/assets/697180b33c4fcf472a63986fba7162cd.png)

### Résolution

Nous avons à priori de la chance car seul le caractère `"4"` semble être filtré. La chaîne est inversée, mais nous commençons à avoir l'habitude de cela 😛 :

![](../../../.gitbook/assets/752d0273870fa3d3f4a42af34733419b.png)

Une opération mathématique et un reverse après \(hmm plutôt light pour un level 9 non ? 😁 \) :

![](../../../.gitbook/assets/3ece5e347fb610bd3c03238a4187d489.png)

## Challenge \#3

### Description

Appeler la fonction `fancyHuman()` avec la chaîne de caractères `"hhjot6krndy"` en argument au sein de la valeur d'une variable Javascript :

![](../../../.gitbook/assets/9de12f0005e65399a89d871c238c9ae5.png)

### Résolution

J'ai bien tenté ma chance en renseignant ma payload sans tenir compte des filtres et de faire un état des lieux après mais c'est difficile quand rien ne s'affiche en retour :

![](../../../.gitbook/assets/f31b764522eee691edd796134683b3dc.png)

Alors ici j'ai expérimenté pas mal de choses ici. Dans un premier temps, le `"H"` ou `"h"` semblent être filtrés mais pas forcément au même endroit. Puis vient le tour de s'apercevoir du filtre sur l'occurrence `"//"` \(je m'en servais pour commenter la fin de ligne\). Il y a sans doute plus simple et je suis loin d'être sûr de moi quant aux filtres réellement en place, mais le challenge est tout de même validé 😁 :

![](../../../.gitbook/assets/7addae512fff6918b65ea57e85b32391.png)

## Challenge \#4

### Description

Appeler la fonction `tallFunction()` avec la chaîne de caractères `"e8ocf2bxv6h"` en argument mais cette fois en valeur d'une donnée JSON stockée dans la variable `window.appData` :

![](../../../.gitbook/assets/5836e7423d43a02b70d17586f37f49bf.png)

### Résolution

Je tente ici de fermer la première balise `<script></script>` afin de ne pas avoir à m'occuper du dernier `"};"`. La parenthèse ouvrante `"("` est filtré ainsi que le caractère `"8"` et `"h"` de la chaîne en argument :

![](../../../.gitbook/assets/565dcaf7a99216e30362b85a1aa5e9f8.png)

 Etant donné qu'il nous faut contourner le filtre sur la parenthèse ouvrante nous allons utiliser la balise `<svg>` afin de pouvoir utiliser son équivalent HTML entities. Le caractère `"h"`, qui est également filtré, possède un `"8"` dans son unicode, nous passons alors également par son équivalent HTML entities :

![](../../../.gitbook/assets/44464d254ed2b28c5fca9d0c016ae9f6.png)





