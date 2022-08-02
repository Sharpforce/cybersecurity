# Niveau "High"

## Exploitation

La page du challenge permet de charger une `<iframe></iframe>`. L'URL admet 3 paramètres :&#x20;

* le premier, `ParamURL`, est l'URL cible de l'iframe, ayant comme valeur par défaut "robots.txt"
* le second et la troisième, `ParamWidth` et `ParamHeight`, sont la taille en largeur et en hauteur de l'iframe, ayant comme valeur par défaut "250"

Ici, modifier l'URL de l'iframe via le paramètre `ParamURL` ne semble pas fonctionner. Qu'importe sa valeur, le fichier "robots.txt" est affiché :

![](<../../../../../.gitbook/assets/image (8) (1).png>)

Naturellement, l'injection via le paramètre d'URL `ParamUrl` ne fonctionne donc également pas. Concernant les deux paramètres de taille de l'iframe ils sont bien pris en compte, mais aucune injection ne semble possible également :&#x20;

```http
GET /bWAPP/iframei.php?ParamUrl=whatever.txt&ParamWidth=250&ParamHeight=250%22%3E%3C/iframe%3E%3Cscript%3Ealert(1)%3C/script%3E%3Ciframe%20src= HTTP/1.1
Host: 192.168.56.116
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Cookie: PHPSESSID=36ce7aa0711c5215358577749d068476; security_level=2
Connection: close
```

![](<../../../../../.gitbook/assets/image (12).png>)

Il est même visible que les caractères spéciaux sont convertis en entités HTML :&#x20;

```html
<div id="main">
  <h1>iFrame Injection</h1>
  <iframe frameborder="0" src="robots.txt" height="250&quot;&gt;&lt;/iframe&gt;&lt;script&gt;alert(1)&lt;/script&gt;&lt;iframe src=" width="250"></iframe>
</div>
```

## Analyse du code source

L'application spécifie la valeur du `src` de l'iframe directement à la valeur "robots.txt" et ne tient pas compte de la valeur présente dans le paramètre d'URL `ParamUrl`. Ce n'est pas le cas des paramètres concernant la taille de l'iframe :&#x20;

{% code title="iframei.php" %}
```php
<?php
  if($_COOKIE["security_level"] == "1" || $_COOKIE["security_level"] == "2") {
?>
    <iframe frameborder="0" src="robots.txt" height="<?php echo xss($_GET["ParamHeight"])?>" width="<?php echo xss($_GET["ParamWidth"])?>"></iframe>
<?php
  }
?>
```
{% endcode %}

La méthode `xss()` est un `switch` exécutant une certaine méthode selon le niveau de sécurité actuel (2 dans le cas du niveau "High") : &#x20;

{% code title="iframei.php" %}
```php
function xss($data) {
  switch($_COOKIE["security_level"]) {
    case "0" :
      $data = no_check($data);      
      break;
    case "1" :
      $data = xss_check_4($data);
      break;
    case "2" :
      $data = xss_check_3($data);
      break;
    default :
      $data = no_check($data);
      break;   
  }

  return $data;
}
```
{% endcode %}

La méthode `xss_check_3()`assainie les paramètres renseignés par l'utilisateur via un appel à la méthode `htmlspecialchars()` :&#x20;

{% code title="functions_external.php" %}
```php
function xss_check_3($data) {
  // htmlspecialchars - converts special characters to HTML entities
  // '&' (ampersand) becomes '&amp;'
  // '"' (double quote) becomes '&quot;' when ENT_NOQUOTES is not set
  // "'" (single quote) becomes '&#039;' (or &apos;) only when ENT_QUOTES is set
  // '<' (less than) becomes '&lt;'
  // '>' (greater than) becomes '&gt;'

  return htmlspecialchars($data, ENT_QUOTES, $encoding);
}
```
{% endcode %}

La méthode d'assainissement est appliquée avec le bon paramètre, à savoir `ENT_QUOTES`. L'injection n'est donc ici pas possible.
