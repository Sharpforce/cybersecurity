---
description: 17 Septembre 2022
---

# Cross-Site Scripting (XSS) et schéma d'URI javascript

> Dans cet article, nous allons examiner quelques exemples d'injections XSS possibles lorsque les données entrées par l'utilisateur sont réfléchies au sein d'un attribut HTML représentant une URL, tel que `href`, `src`, ou `url`.

L'accès à une URL par un navigateur est conditionné par son schéma (_scheme_). Le schéma est le premier niveau de la structure de nommage d'un URI. Plus concrètement, il s'agit de la partie la plus à gauche, avant le caractère ":". Il ne faut pas confondre le schéma et le protocole : le schéma représente la sémantique alors que le protocole est la manière de communiquer avec le service cible.

Le schéma d'URI [Javascript](https://datatracker.ietf.org/doc/html/draft-hoehrmann-javascript-scheme-03) fait partie des schémas communs qui fonctionne sur les navigateurs récents mais n'est pas un schéma [officiel](https://en.wikipedia.org/wiki/List\_of\_URI\_schemes). Ce schéma peut permettre d'exécuter du code Javascript comme ceci :&#x20;

```javascript
javascript:doSomething()
```

ou plusieurs instructions à la suite : &#x20;

```javascript
javascript:doSomething();doSomethingElse()
```

Il peut être intéressant, lors des tests d'intrusions ou des séances de bug bounty, de connaitre son utilisation afin de détecter des vulnérabilités de type Cross-Site Scripting. Il ne sera pas question dans cet article de lister tous les éléments HTML sensibles à cette attaque mais plutôt d'analyser les cas les plus fréquents.

## L'élément HTML \<a>

Le premier cas analysé est sans doute celui le plus rencontré lors des audits. Il s'agit de l'utilisation d'une donnée non fiable en tant que valeur de l'attribut `href` de la balise `<a>` : &#x20;

```html
<a href="{user input}">a link</a>
```

Si aucun assainissement est effectué, il est facile de s'échapper de l'attribut `href` afin d'écrire le code d'exploitation en ouvrant une balise `<script>`. Mais qu'en est t'il si un assainissement est présent empêchant cela par un filtrage des guillemets simples et/ou doubles ?&#x20;

Dans l'exemple suivant, le développeur, qui a été sensibilisé aux vulnérabilités Web, pense avoir correctement assaini l'entrée utilisateur grâce à l'appel à la méthode `htmlspecialchars()` :

```php
<!DOCTYPE html>
<html>
  <head lang="en">
    <meta charset="UTF-8">
  </head>
  <body>
    <?php
      $var = $_GET['userInput'];
      echo '<a href="' . htmlspecialchars($var, ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5, "UTF-8") . '">Lien</a>';
    ?>
  </body>
</html>
```

Bien qu'il ne soit plus possible d'effectuer l'injection du code malicieux en s'échappant de l'attribut HTML `href` : &#x20;

<figure><img src="../../../.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>

l'utilisation du schéma Javascript peut tout de même permettre l'exécution de code lors du clique sur le lien par la victime :

<figure><img src="../../../.gitbook/assets/image (149).png" alt=""><figcaption></figcaption></figure>

## L'élément HTML \<iframe>

Il n'est pas rare qu'une application web permette à l'utilisateur de renseigner la source d'une `<iframe>` qui sera intégrée dans une des pages du site. L'attribut `src` de la balise `<iframe>` permet également l'exécution de code en utilisant le schéma Javascript. Soit le code suivant :

```php
<!DOCTYPE html>
<html>
  <head lang="en">
    <meta charset="UTF-8">
  </head>
  <body>
    <?php
      $var = $_GET['userInput'];
      echo '<iframe src="' . $var . '">';
    ?>
  </body>
</html>
```

et l'exécution du code qui ne nécessite pas d'action utilisateur : &#x20;

<figure><img src="../../../.gitbook/assets/image (190).png" alt=""><figcaption></figcaption></figure>

## Autres éléments HTML

Il existe d'autres éléments HTML sensibles à cette attaque dont voici quelques exemples : &#x20;

```html
<svg><a xlink:href="javascript:alert(1)"><text x="20" y="20">XSS</text></a>
```

```html
<object data="javascript:alert(1)"> // Seulement sur Firefox
```

```html
<embed src="javascript:alert(1)"> // Seulement sur Firefox
```

Pour plus d'informations, la [cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#protocols) de PortSwigger mérite le coup d'œil.
