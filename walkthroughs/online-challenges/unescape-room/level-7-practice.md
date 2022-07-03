# Level 7 \(practice\)

## Challenge \#1

### Description

Appeler la fonction `tallHuman()` avec la chaîne de caractères `"9ultl0bs"` en argument :

![](../../../.gitbook/assets/82d4e2408e9b064e7739cfa70ecbd12a.png)

### Résolution

Dans cet exercice, la chaîne semble être inversée et un filtrage est effectué sur le caractère `"t"` : 

![](../../../.gitbook/assets/b805221344cd2a67c1ae3246154e9f3c.png)

L'idée va donc être de gérer dans un premier temps le filtrage sur le caractère `"t"` puis d'effectuer un inversement sur notre chaîne. Concernant le filtrage on utilise le `"T"` à la place du `"t"` pour pouvoir écrire le mot clé `"script"`. Le filtre du caractère dans le nom de la fonction ainsi que dans la chaîne passée en paramètre est contournable grâce à l'encodage unicode :

![](../../../.gitbook/assets/1b90a6e251c390791592eb6af289684d.png)

## Challenge \#2

### Description

Appeler la fonction `prettySuperHero()` avec la chaîne de caractères `"86zk0a0i"` en argument mais cette fois sur l'attribut `src` dans une balise `<img />` :

![](../../../.gitbook/assets/b8057761808b64d32583d6c062c90c1a.png)

### Résolution

Seuls les caractères `" " (espace)` , `"6"` et `"0"` sont filtrés, on ne rencontre donc pas de réel problème ici :

![](../../../.gitbook/assets/559adf43a8d5753f3d0b6c1cb99800ed.png)

Cela ne sert à rien de tenter de contourner le filtre sur le caractère `" " (espace)`, il sera ajouté automatiquement. On utilise les opérations mathématiques afin de retrouver notre `"6"` et nos `"0"` :

![](../../../.gitbook/assets/56e99beff016ef220387065c9b635b26.png)

## Challenge \#3

### Description

Appeler la fonction `kindSuperHero()` avec la chaîne de caractères `"zccb5yih"` en argument mais cette fois sur l'attribut `href` dans une balise `<a>` :

![](../../../.gitbook/assets/42e88e844493d571d95c53b5277b6593.png)

### Résolution

Plutôt facile pour celle-ci : la chaîne est inversée et seul le caractère `"i"` est filtré :

![](../../../.gitbook/assets/d483be9f4ba36a4daabfb79bf96bd3c9.png)

On utilise donc le contournement par le jeu de majuscule/minuscule et l'encodage unicode avant d'inverser la payload :

![](../../../.gitbook/assets/e3792001b34cd2659f6d9edd850aa27a.png)

## Challenge \#4

### Description

Appeler la fonction `fancySuperHero()` avec la chaîne de caractères `"1vose8qg"` en argument mais cette fois sur la valeur de la propriété `background-color` du sélecteur `banner` :

![](../../../.gitbook/assets/e93fe5a0dd53be23f9e3aac07fc8115e.png)

### Résolution

Dans cet exercice, le caractère `"p"` ainsi que le caractère `"\"` sont filtrés :

![](../../../.gitbook/assets/75b495b9963e0d44d852be4d6ee9a166.png)

On contourne le filtrage sur `"p"` dans les balises en utilisant `"P"`. Pour le caractère `"p"` de la chaîne passée en argument, il n'est pas possible d'utiliser l'encodage unicode car le caractère `"\"` est filtré. Je passe alors par un `toLowerCase()` :

![](../../../.gitbook/assets/c298dabe10e09a4d1c650317967ff21b.png)

## Challenge \#5

### Description

Appeler la fonction `braveFunction()` avec la chaîne de caractères `"kacwpkiy"` en argument mais cette fois sur l'attribut `value` du tag `<input/>` :

![](../../../.gitbook/assets/5b56b4b54c0f8ad020eb797a20604470.png)

### Résolution

Ici la chaîne est inversée. De plus, les caractères `"F"`, `"("` et `" "(espace)` sont filtrés :

![](../../../.gitbook/assets/c3231fbbebbc76f0bce06810b65517f4.png)

On remplace les caractères par la version HTML entities ou unicode selon le cas, puis, on inverse la chaîne :

![](../../../.gitbook/assets/46dd72f5d4a364332f26a894c982df39.png)

## Challenge \#6

### Description

Appeler la fonction `niceSuperHero()` avec la chaîne de caractères `"3hbw9yod"` en argument au sein de la valeur d'une variable Javascript :

![](../../../.gitbook/assets/aeb887b351e71f5ea595a2f09ca49215.png)

### Résolution

Un filtrage est effectué sur le caractère `"c"` \(ainsi que sur `"C"`\) ainsi que sur le caractère `"3"` . Finalement la chaîne est également inversée :

![](../../../.gitbook/assets/519b02a07133e6d9d06ae8f4b37d2d22.png)

On est tenté d'appeler directement la fonction `niceSuperHero()` , mais étant donné que le `"c"` est filtré et son code unicode est `\u0063`et qu'il contient un `"3"` ...  On passe alors par un encodage base64 et un `toLowerCase()` :

![](../../../.gitbook/assets/474bb0091cd6a1cfbb5066eb05cad264.png)

## Challenge \#7

### Description

Appeler la fonction `braveHuman()` avec la chaîne de caractères `"we9o3qci"` en arugment mais cette fois en valeur d'une donnée JSON stockée dans la variable `window.appData` :

![](../../../.gitbook/assets/adf5369ffd772eb2237974c3e786aac0.png)

### Résolution

Pas de difficulté ici puisque seul le caractère `"9"` semble être filtré :

![](../../../.gitbook/assets/58c19d1523411bbf4cc7037e35a36b07.png)

Soit en utilisant une opération mathématique :

![](../../../.gitbook/assets/f4103b5cbd88d24a4edc8ad0ce53b79e.png)



