# Level 3 (practice)

## Challenge #1

### Description

Appeler la fonction `prettyHuman()` avec la chaîne de caractères `"798"` en argument :

![](../../../.gitbook/assets/4d1c13692f07caad960ba87d19253949.png)

### Résolution

Il s'agit sensiblement du même filtrage que le challenge du niveau 2, à savoir un filtre sur la lettre `"t"` :

![](../../../.gitbook/assets/8ae612e8bd2b21af88b656db03c75a51.png)

De la même manière, je contourne le filtrage en place en utilisant le caractère filtré en majuscule ou son code unicode suivant le contexte :

![](../../../.gitbook/assets/dce6d1f3ab8e38a73a96dbaa11981c61.png)

## Challenge #2

### Description

Appeler la fonction `kindHuman()` avec la chaîne de caractères `"079"` en argument mais cette fois sur l'attribut `src` dans une balise `<img />` :

![](../../../.gitbook/assets/0943bb15cb13da74a962d70a2586d934.png)

### Résolution

Le filtrage s'effectue ici sur le caractère `"  " (espace)` ainsi que sur le caractère `"u"` :

![](../../../.gitbook/assets/ea522911c40233c8212706ee9d337cfb.png)

Le filtrage sur le caractère `"u"` est un peu embêtant car il m'empêche d'utiliser la notation unicode. Je change donc de technique et passe via un `String.fromCharCode()` conjointement avec la fonction `eval()` :

![](../../../.gitbook/assets/ebc88de4ef26d9ecceedffa1701fdec7.png)

## Challenge #3

### Description

Appeler la fonction `kindSuperHero()` avec la chaîne de caractères `"929"` en argument mais cette fois sur l'attribut `href` dans une balise `<a>` :

![](../../../.gitbook/assets/53dc23450b6bf3b13f9bc7ae248f64d0.png)

### Résolution

Au sein d'un attribut `href` il est possible d'utiliser la syntaxe `javascript:code` (par exemple `href="javascript:alert(1)"`) afin d'exploiter une XSS. Dans ce cas, seul le caractère `"9"`, présent dans le paramètre de la fonction, semble être filtré :

![](../../../.gitbook/assets/8d73ff6586ae01746409b2d3e4584e01.png)

Grâce à la concaténation (et à la conversion implicite), il est possible de représenter `"9"` par une autre opération mathématique :

![](../../../.gitbook/assets/6cdb3d277edfa014252f5f2c8ec39248.png)

Ne pas oublier de passer par la vue DOM (View DOM) et de cliquer sur le lien afin d'exécuter la payload :

![](../../../.gitbook/assets/e78f7504ea6a54613e4b48862703cc82.png)

![](../../../.gitbook/assets/c6684d0a7d4336a02c904c93f08d5c80.png)
