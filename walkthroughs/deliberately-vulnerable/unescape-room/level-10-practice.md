# Level 10 (practice)

## Challenge #1

### Description

Appeler la fonction `politeRobot()` avec la chaîne de caractères `"XNY7CWWKx1Pd"` en argument mais cette fois sur l'attribut `src` dans une balise `<img />` :

![](../../../.gitbook/assets/304006b652af326a349f6bf405eda1e6.png)

### Résolution

Dans ce challenge, en plus de l'inversion de la chaîne, un filtrage est effectué sur le caractère `"'" (simple quote)` ainsi que sur le caractère `"Y"` :

![](../../../.gitbook/assets/465db7ce0889b4a03ca4c7285acb72a7.png)

Une fois la chaîne inversée, je m’aperçois qu'un filtrage est également réalisé sur l'occurrence `"on"`mais il suffit de répéter le motif pour contourner la mécanique. J'utilise les HTML entities afin de pouvoir entourer de guillemets simples la chaîne à passer en paramètre. Puis pour terminer, j'utilise l'encodage unicode afin de gérer le cas du `"Y"` :

![](../../../.gitbook/assets/7acac40e097a9559b9e00badf954d944.png)

## Challenge #2

### Description

Appeler la fonction `kindFunction()` avec la chaîne de caractères `"JmRBD96amXGY"` en argument mais cette fois sur la valeur de la propriété `background-color` du sélecteur `banner` :

![](../../../.gitbook/assets/9af994876b8b88b42a99f91c09416c20.png)

### Résolution

Le but ici est de fermer la balise `<style>` puis d'injecter le script. Une des deux parenthèses est filtrée ici ainsi que le caractère `"d"`. Reste à voir si aucun autre filtre se déclenche lorsque je vais inverser la payload :

![](../../../.gitbook/assets/d34fa5e26c1fa89dfc61bafb6d5286f3.png)

Aucun autre filtre ne s'active une fois la chaîne inversée. Par contre je dois insérer une balise `<svg>` afin de pouvoir remplacer la parenthèse par son équivalent HTML entities :

![](<../../../.gitbook/assets/bfb82e7af1cdf2cf32d077660c9fb043 (1).png>)

Le caractère `"d"` est filtré mais également le caractère `"0"`. Manque de chance, son équivalent unicode est `\u0046` et son équivelent HTML entities est `&#100;` ...

![](../../../.gitbook/assets/bfb82e7af1cdf2cf32d077660c9fb043.png)

Pour contourner cela, j'utilise en plus la fonction `eval()` ainsi qu'un `toLowerCase()` sur le `"D"` :

![](../../../.gitbook/assets/9a3bc2a01d562f965912ea5ea6ffdbb7.png)

La payload dépassant du champ `<input>` voici sa version complète et non inversée :slight\_smile: :

```
blue;}</style><svg><script>eval('kin'+'D'.toLowerCase(&#41;+'Function("JmRBD96amXGY"&#41;'&#41;</script>
```

## Challenge #3

### Description

Appeler la fonction `kindHuman()` avec la chaîne de caractères `"Eph040Cwo0gt"` en argument au sein de la valeur d'une variable Javascript :

![](<../../../.gitbook/assets/85d348bc01c64534db5473d123e5d5ff (1).png>)

### Résolution

Avec un peu de chance je ne vais rencontrer aucune difficulté sur ce challenge là car seul le caractère `"E"` semble être filtré :

![](../../../.gitbook/assets/cc2d98686cc9c7f075cfea27ce9e7968.png)

En effet, je remplace le caractère par son équivalent unicode et je valide cet autre challenge :

![](../../../.gitbook/assets/23741d75db3a61f30ad5c760153b600f.png)

## Challenge #4

### Description

Appeler la fonction `prettyFunction()` avec la chaîne de caractères `"MRRjzgB2zCHk"` en argument mais cette fois en valeur d'une donnée JSON stockée dans la variable `window.appData` :

![](<../../../.gitbook/assets/74b903ce731295485a4edf96195a257e (1).png>)

### Résolution

Pas mal de filtres pour ce dernier challenge et j'avoue l'avoir passé avec de la chance et surtout, avec beaucoup de tentatives. Le mot clé `"script"` est filtré mais contournable en utilisant un contournement bien connu comme `"scrscriptipt"` auquel j'ajoute des majuscules pour passer quelques autres filtres.&#x20;

L'occurrence `"on"` , présent dans le nom de la fonction, est filtré, mais il est possible de contourner cette limitation en utilisant l'occurrence `"oonn"` . De plus, dans un premier temps j'ai du ajouter des caractères aléatoires au niveau du paramètre de la fonction pour récupérer une sortie lisible :

![](../../../.gitbook/assets/494589d6a008d9027fef7048ad98de77.png)

A noter ici que dans la chaîne passée en argument, les caractères `"a"`, `"b"` et `"\"` sont filtrés. Cela m'a permis de modifier la longueur de la chaîne sans en modifier son contenu et ainsi, de récupérer une sortie lisible afin de valider ce dernier challenge :

![](../../../.gitbook/assets/22d0beaf8915d8ba9c9c8ba937e53583.png)
