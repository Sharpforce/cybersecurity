# Level 4 \(practice\)

## Challenge \#1

### Description

Appeler la fonction `elegantRobot()` avec la chaîne de caractères `"47295"` en argument :

![](../../.gitbook/assets/cfd7d0f6f3d21f1e2f77d252a75cfe20.png)

### Résolution

Un filtrage plutôt simple ici puisqu'il s'agit d'un filtre sur le caractère `"g"` ainsi que sur le caractère `"5"` :

![](../../.gitbook/assets/2d5d89832f3e13b78125fcc8a9b7fed5.png)

La notation unicode ainsi que l'opération mathématique permet de s'en passer outre aisément :

![](../../.gitbook/assets/c6029c256ff2a2c4ea0a458f0f77db46.png)

## Challenge \#2

### Description

Appeler la fonction `braveRobot()` avec la chaîne de caractères `"52294"` en argument mais cette fois sur l'attribut `src` dans une balise `<img />` :

![](../../.gitbook/assets/1fa813704b5b74d0ebeed1aa8db18475.png)

### Résolution

Un filtrage est effectué sur le caractère `"e"` ainsi que sur le caractère `"4"` :

![](../../.gitbook/assets/8cb6683a392640434e4ac82f88a5803a.png)

Il est possible de le contourner en utilisant les mêmes techniques que précédemment à savoir l'encodage unicode ainsi que l'opération mathématique :

![](../../.gitbook/assets/e5647e33c683d7e2b313528feb910ecb.png)

## Challenge \#3

### Description

Appeler la fonction `fancySuperHero()` avec la chaîne de caractères `"28323"` en argument mais cette fois sur l'attribut `href` dans une balise `<a>` :

![](../../.gitbook/assets/24f89eb8ee33533147db74ce62c4936c.png)

### Résolution

Un mécanisme de filtrage est présent mais exclusivement sur le caractère `"e"` :

![](../../.gitbook/assets/c3c4a7719e90601642cd54e63a5fc83c.png)

Soit, en utilisant l'encodage unicode :

![](../../.gitbook/assets/3c380c51b35637e2205767de44b3cb5f.png)

Etant donné que notre payload se situe dans un `href`, il ne faut pas oublier de cliquer sur le lien pour déclencher l'injection \(View DOM\) :

![](../../.gitbook/assets/ffcc5eb2241bd36aa306ea0d3bdcad70.png)



