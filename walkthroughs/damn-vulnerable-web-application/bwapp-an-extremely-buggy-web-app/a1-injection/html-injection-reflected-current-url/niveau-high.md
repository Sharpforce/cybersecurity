# Niveau "High"

## Exploitation

L'URL est ici réfléchie au sein de la page en utilisant la méthode HTTP `GET` :&#x20;

```http
GET /bWAPP/htmli_current_url.php?parametre=valeur HTTP/1.1
Host: 192.168.56.116
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Cookie: security_level=2; PHPSESSID=2f4611482fb97564af7dbe4243dca828
Connection: close
```

![](<../../../../../.gitbook/assets/image (23) (1) (1) (1) (1).png>)

En PHP, la récupération de l'URL peut s'effectuer grâce aux variables `$_SERVER["REQUEST_URI"]` ou `$_SERVER["QUERYSTRING"]` par exemple (le fonctionnement est différent pour les variables types `$_GET` ou `$_POST` et autres équivalents). Dans le premier cas, les navigateurs récents encodent l'URL avant de la transmettre à l'application qui rend les tentatives d'injections HTML et XSS inefficaces :&#x20;

```http
GET /bWAPP/htmli_current_url.php?parametre=%3Cscript%3Ealert(1)%3C/script%3E HTTP/1.1
Host: 192.168.56.116
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Cookie: security_level=2; PHPSESSID=2f4611482fb97564af7dbe4243dca828
Connection: close
```

![](<../../../../../.gitbook/assets/image (25) (1).png>)

Aucune injection ne semble possible ici, même en testant le contournement sous IE11 :&#x20;

![](<../../../../../.gitbook/assets/image (16).png>)

## Analyse du code source

L'application affiche le contenu de la variable `$url` au d'une balise `<i></i>` :

{% code title="htmli_current_url.php" %}
```php
<div id="man">
  <h1>HTML Injection - Reflected (URL)</h1>
  <?php echo "<p align=\"left\">Your current URL: <i>" . $url . "</i></p>";?>
</div>
```
{% endcode %}

La variable `$url` est construite de la manière suivante :&#x20;

{% code title="htmli_current_url.php" %}
```php
$url= "";

switch($_COOKIE["security_level"]) {
  case "0" :
    // $url = "http://" . $_SERVER["HTTP_HOST"] . urldecode($_SERVER["REQUEST_URI"]);
    $url = "http://" . $_SERVER["HTTP_HOST"] . $_SERVER["REQUEST_URI"];
    break;
  case "1" :
    $url = "<script>document.write(document.URL)</script>";
    break;
  case "2" :
    $url = "http://" . $_SERVER["HTTP_HOST"] . xss_check_3($_SERVER["REQUEST_URI"]);
    break;
  default :
    // $url = "http://" . $_SERVER["HTTP_HOST"] . urldecode($_SERVER["REQUEST_URI"]);
    $url = "http://" . $_SERVER["HTTP_HOST"] . $_SERVER["REQUEST_URI"];
    break;
}
```
{% endcode %}

Le level "High" correspond à la valeur 2, c'est à dire qu'ici, la variable `$url` est construite de la façon suivante :

```php
$url = "http://" . $_SERVER["HTTP_HOST"] . xss_check_3($_SERVER["REQUEST_URI"]);
```

La méthode `xxs_check_3()` assainit correctement la variable `$_SERVER["REQUEST_URI"]` grâce à la méthode PHP `xss_check_3()` :&#x20;

{% code title="functions_external.php" %}
```php
function xss_check_3($data, $encoding = "UTF-8") {
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

Il y a ici potentiellement une injection possible via l'entête HTTP `Host` mais cela nécessite de passer par une interception par un proxy :&#x20;

![](<../../../../../.gitbook/assets/image (18) (2).png>)

![](<../../../../../.gitbook/assets/image (28) (1).png>)

L'exploitation n'est pas réalisable sur les navigateurs récents à cause de l'encodage effectué sur l'URL et Internet Explorer  11 ne support pas la syntaxe `https(s)://username:password@example.com` ([https://docs.microsoft.com/en-us/troubleshoot/developer/browsers/security-privacy/name-and-password-not-supported-in-website-address](https://docs.microsoft.com/en-us/troubleshoot/developer/browsers/security-privacy/name-and-password-not-supported-in-website-address)).
