# Level 6 \(practice\)

## Challenge \#1

### Description

Appeler la fonction `politeSuperHero()` avec la chaîne de caractères `"v1hiyw5"` en argument :

![](../../.gitbook/assets/419ae6bae339c90bd9c3389d71848fb4.png)

### Résolution

Un filtrage est effectué sur les caractères `"e"` et `"u"`. Cela va empêcher l'utilisation de l'encodage unicode. Un filtre est aussi présent sur le caractère `" " (espace)` :

![](../../.gitbook/assets/f11f5a752fcfce688cb5c7301d21718a.png)

Difficile de pouvoir appeler la fonction `politeSuperHero()` sans pouvoir utiliser l'encodage unicode. On tente de contourner cela en passant par un label `<svg>` et en ajoutant un `"/"` pour contourner le filtrage du caractère `" " (espace)` . L'utilisation des HTML entities permet de gérer le cas du `"u"` et du `"e"` :

![](../../.gitbook/assets/8b95268af04727ee2ea6cb1aa4c2355b.png)

## Challenge \#2

### Description

Appeler la fonction `prettyHuman()` avec la chaîne de caractères `"xdheb3s"` en argument mais cette fois sur l'attribut `src` dans une balise `<img />` :

![](../../.gitbook/assets/3fe692e3c721273cf700f79efb1d9b95.png)

### Résolution

On tente d'utiliser l'attribut `onerror` de la balise image afin d'exécuter la fonction javascript mais un mécanisme semble filtrer les occurrences `"on"` ainsi que le caractère `"d"` :

![](../../.gitbook/assets/02504d3bb79c4d0a9d75e253ef04139a.png)

On réussi à contourner le filtre sur l'occurrence `"on"` en le répétant une seconde fois. Le caractère `"d"` présent dans la chaîne de caractères passée en paramètre peut, par exemple, être représenté par son équivalent en base64 :

![](../../.gitbook/assets/ee9887a0682224aaa55b8f0674181cf9.png)

## Challenge \#3

### Description

Appeler la fonction `politeFunction()` avec la chaîne de caractères `"237chw3"` en argument mais cette fois sur l'attribut `href` dans une balise `<a>` :

![](../../.gitbook/assets/01ff527592f876043c7fe6d4cba07ac9.png)

### Résolution

Etant donné que nous sommes au sein d'un attribut `href` nous allons tenter d'utiliser la syntaxe `javascript:code`. Le filtre en place supprime le caractère `"t"` ainsi que le caractère spécial `"'" (simple quote)` :

![](../../.gitbook/assets/c50b7f5529ea8cc50dbb558e73077ebd.png)

Il est possible de contourner le premier filtre en utilisant la majuscule, soit `"T"`. Pour les occurrences du caractère `"t"` présents dans le nom de la fonction il est possible d'utiliser l'encodage unicode, puis les HTML entities pour les deux `"'" (simple quote)` entourant la chaîne de caractère en argument :

![](../../.gitbook/assets/33aea2ee5da784bd3dca025acb4e2d45%20%282%29.png)

## Challenge \#4

### Description

Appeler la fonction `tallSuperHero()` avec la chaîne de caractères `"rj7tfb1"` en argument mais cette fois sur la valeur de la propriété `background-color` du sélecteur `banner` :

![](../../.gitbook/assets/69f66f865b50058f5767b89f9fcb1c4d%20%281%29.png)

### Résolution

L'objectif est ici de sortir de la balise `<style></style>` et d'injecter du javascript. Le mot clé `"script"` semble être filtré ici. De même pour le caractère `"f"` nécessaire dans la chaîne de caractères devant être passée en argument :

![](../../.gitbook/assets/6c8463db047d5dd21d0daab656e8cae6.png)

Il est possible de contourner le filtre sur la chaîne `"script"` en mélangeant les majuscule et les minuscules. Pour le caractère `"f"`, j'utilise l'encodage base64 car le caractère `"0"` est en fait également filtré, ce qui empêche l'utilisation de l'encodage unicode :

![](../../.gitbook/assets/c0e473bc849ebfa3b0a4045238fdc5ad.png)

## Challenge \#5

### Description

Appeler la fonction `kindHuman()` avec la chaîne de caractères `"lxu2w3b"` en argument mais cette fois sur l'attribut `value` du tag `<input/>` :

![](../../.gitbook/assets/1be38c35cf6535fd9da7f894b702ebf7.png)

### Résolution

Celui-ci est relativement facile. Nous allons utiliser l'attribut `onmouseover` et tenter de déclencher la payload via la vue DOM. Le seul caractère filtré est le `"H"` présent dans le nom de la fonction :

![](../../.gitbook/assets/5cdd5979489fb409aab554a5bd77ba9b.png)

L'encodage unicode suffit à contourner cette limitation :

![](../../.gitbook/assets/c07963d0ddae101e1d8fdcb8cbba3ffe.png)

## Challenge \#6

### Description

Appeler la fonction `niceSuperHero()` avec la chaîne de caractères `"7451rf5p"` en argument au sein de la valeur d'une variable Javascript :

![](../../.gitbook/assets/e5d02b4f5bb2966d60db62c99ed36582.png)

### Résolution

Une première tentative nous indique un filtrage sur les caractères `""" (double quotes)`, `")"` et `"5"`:

![](../../.gitbook/assets/a809973c4c0e094754be373c52e21c0e.png)

Pour réussir ce challenge, j'ai mis fin à la balise `<script>` et démarrer une nouvelle balise `<svg>` me permettant d'utiliser les HTML entities pour le caractère `")"`. Pas de réelle difficulté sur le contournement du filtre du caractère `""" (double quotes)`, il suffit simplement d'utiliser la version simple `"'" (simple quote)` :

![](../../.gitbook/assets/98b8c2f05325a1e9b8def94ec6b3bb85.png)

## Challenge \#7

### Description

Appeler la fonction `tallSuperHero()` avec la chaîne de caractères `"gt31knj"` en argument mais cette fois en valeur d'une donnée JSON stockée dans la variable `window.appData` :

![](../../.gitbook/assets/62d22163e2fb9bf6bd54b1ddc8482091.png)

### Résolution

Il nous faut ici sortir de la payload JSON. Le plus simple est sans doute de fermer correctement le JSON puis la balise `<script>` et d'ouvrir de nouvelles balises. Cela fonctionne quasiment du premier coup puisque seul le caractère `"g"` présent dans la chaîne passée en argument est filtré :

![](../../.gitbook/assets/74990954317ba72fd1b93e613e7eaf39.png)

Après un nouvel essai, il n'est pas possible d'utiliser l'encodage unicode car le caractère `"0"` est aussi filtré. L'encodage base64 vient à notre rescousse, mais en fait non ... Les simples quotes `"'"` sont également filtrées. Je décide alors de tenter ma change avec la fonction `String.toLowerCase` et d'utiliser le `"G"` au lieu du `"g"` :

![](../../.gitbook/assets/927f18816277f2aed331142c0687e057.png)









