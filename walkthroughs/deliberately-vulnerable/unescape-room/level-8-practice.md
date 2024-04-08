# Level 8 (practice)

## Challenge #1

### Description

Appeler la fonction `tallSuperHero()` avec la chaîne de caractères `"1mzslvt3gc"` en argument :

![](../../../.gitbook/assets/b2865ecfe2b83a53838f4292a7f98636.png)

### Résolution

En plus de la chaîne renversée, les caractères `"'" (simple quote)`, `"S"` ainsi que `"e"` sont filtrés :

![](../../../.gitbook/assets/7f414979a8120e4b67398743f80293c3.png)

En essayant plusieurs payloads je me rends compte que le caractère `"5"` est également filtré. C'est assez gênant car le caractère `"S"` est représente par `"\u0053"` en unicode et le caractère `"e"` par `"\u0065"`empêchant cette technique de contournement.&#x20;

La technique est ici alors d'utiliser la balise `<svg>`. Cette balise va permettre d'utiliser les HTML entities au sein de la balise `<script></script>` car la balise `<svg>` fait travailler dans un contexte XML :

![](../../../.gitbook/assets/0fbca9cf35939074ad5b55ebb715f6a3.png)

## Challenge #2

### Description

Appeler la fonction `tallRobot()` avec la chaîne de caractères `"ocxzd5fkum"` en argument mais cette fois sur l'attribut `src` dans une balise `<img />` :

![](../../../.gitbook/assets/1f352e162ebde616f6e4a223547dc40f.png)

### Résolution

Une inversion de la chaîne (pour changer :stuck\_out\_tongue: ) , un filtrage sur le caractère `"5"` ainsi que sur le caractère `"m"` :

![](../../../.gitbook/assets/0773121ef3103c64035ba2a43cbfb325.png)

Une concaténation avec une opération mathématique ainsi qu'un `toLowerCase()` suffit à valider ce challenge :

![](../../../.gitbook/assets/38af25f44accec0b9cb31bf6d40328e3.png)

## Challenge #3

### Description

Appeler la fonction `braveHuman()` avec la chaîne de caractères `"jbtczli6o0"` en argument mais cette fois sur l'attribut `href` dans une balise `<a>` :

![](../../../.gitbook/assets/bd3eeaf4d2442dea6cfb507ef1cfad78.png)

### Résolution

Challenge très facile puisque le seul caractère filtré est le `"0"` qui est présent dans la chaîne de caractères passée en paramètre :

![](../../../.gitbook/assets/1b86761e7d1097f7df55e08033011bc7.png)

Une opération mathématique suffit à contourner ce filtre (ne pas oublier de cliquer sur le lien dans la vue DOM pour activer la payload) :

![](../../../.gitbook/assets/0545edadf0c7e3108616ce7056d9712e.png)

## Challenge #4

### Description

Appeler la fonction `elegantFunction()` avec la chaîne de caractères `"cqin5qnyfv"` en argument mais cette fois sur la valeur de la propriété `background-color` du sélecteur `banner` :

![](../../../.gitbook/assets/aa5796f9a1644a0e049f9f923d3b1b21.png)

### Résolution

Ce challenge semble proposer une inversion de la chaîne de caractères et un filtre sur les caractères `"y"` et `"F"` :

![](../../../.gitbook/assets/ec355bed09f8c3f5657dfcd7b4ef0622.png)

Concernant le caractère `"y"` il suffit de le répéter dans la balise `<style>` car seule la première occurrence semble être supprimée. L'encodage unicode va me permettre d'appeler correctement la fonction malgré le filtrage du caractère `"F"`, puis, j'inverse le tout :

![](../../../.gitbook/assets/bac1e101560c4bc2855d16872fff8755.png)

## Challenge #5

### Description

Appeler la fonction `kindSuperHero()` avec la chaîne de caractères `"p89folqp99"` en argument mais cette fois sur l'attribut `value` du tag `<input/>` :

![](../../../.gitbook/assets/b92cb37dea9976150e36a5fda0521cde.png)

### Résolution

On s’aperçoit que le caractère `" " (espace)` est filtré et que la chaîne est inversée :

![](../../../.gitbook/assets/69b88b8eb5dc648a8dffe8cc268fa517.png)

Le filtrage sur le caractère `" " (espace)` n'est pas gênant car il n'est pas nécessaire dans la balise. Par contre, si j'inverse ma payload, un second filtrage (voir même un brouillage :laughing: ) s'active :

![](../../../.gitbook/assets/f19587312b2c89ddb8ec5af75d1e6190.png)

Après quelques recherches, le fautif semble être trouvé. Il s'agit non pas d'un caractère spécifique mais sans doute de l'emplacement de ce caractère voir peut être de la longueur de la chaîne. En effet, il semblerait que la chaîne s'affiche correctement seulement lorsque la transformation base64 se termine par un signe `"="`. Un encodage unicode du `"S"` va permettre de faire passer ma payload et de valider ce challenge :

![](../../../.gitbook/assets/f77ae7b912dcbf71b26c70c824e84075.png)

## Challenge #6

### Description

Appeler la fonction `kindSuperHero()` avec la chaîne de caractères `"z569avwq74"` en argument au sein de la valeur d'une variable Javascript :

![](../../../.gitbook/assets/4aa89dd86e7f58b334cd1d7f086ce397.png)

### Résolution

Le filtrage s'effectue seulement sur les caractères `"4"` et `"5"` présent dans la chaîne passée en argument :

![](../../../.gitbook/assets/cb0102cec4d8160f3819f9827c502126.png)

Le contournement est assez facile ici puisqu'une concaténation avec des opérations mathématiques suffit :

![](../../../.gitbook/assets/f2e80b461b7b79fe992dcdbd960624ad.png)

## Challenge #7

### Description

Appeler la fonction `fancyFunction()` avec la chaîne de caractères `"fdk8qhry90"` en argument mais cette fois en valeur d'une donnée JSON stockée dans la variable `window.appData` :

![](../../../.gitbook/assets/937d867cd6a293f918a7aa941164b1d9.png)

### Résolution

Premièrement la chaîne est inversée, et, plus embêtant, le caractère `"<"` est filtré ainsi que le caractère `"d"` :

![](../../../.gitbook/assets/9082b09b1725e21d56e5acdfe9b56e24.png)

Il me faut donc trouver une payload qui n'a pas besoin de fermer la balise `<script>` ouverte et également gérer la fin de la variable JSON, à savoir `"};"` , afin de ne pas avoir d'erreur de syntaxe.&#x20;

En essayant plusieurs payload, je me rends compte du filtrage de l'occurrence `"on"` ainsi que celui du  caractère `"4"`. Pour contourner cela, il suffira de doubler le `"on"` et d'utiliser la méthode `toLowerCase()`pour le caractère `"d"` (son encodage unicode n'était pas possible car il contient le caractère `"4"` ) :

![](../../../.gitbook/assets/04f2aa0daca599196576492f7470ab2c.png)
