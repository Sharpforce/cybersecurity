# Niveau "High"

## Exploitation

L'affichage du prénom et du nom s'effectue ici de la même manière au sein d'une balise `<div></div>` :&#x20;

![](<../../../../../.gitbook/assets/image (29) (1) (1).png>)

Dont voici la requête :&#x20;

```http
POST /bWAPP/htmli_post.php HTTP/1.1
Host: 192.168.56.116
Content-Length: 76
Origin: http://192.168.56.116
Content-Type: application/x-www-form-urlencoded
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://192.168.56.116/bWAPP/htmli_post.php
Cookie: PHPSESSID=315cfb443aea2634d941d18c55418bb3; security_level=2
Connection: close

firstname=pr%C3%A9nom+%5Blvl+high%5D&lastname=nom+%5Blvl+high%5D&form=submit
```

L'affichage des paramètres d'URL se faire directement au sein de la balise `<div></div>` contenant également le formulaire :&#x20;

```html
<div id="main">
  <h1>HTML Injection - Reflected (POST)</h1>
  <p>Enter your first and last name:</p>
  <form action="/bWAPP/htmli_post.php" method="POST">
    <p><label for="firstname">First name:</label><br />
    <input type="text" id="firstname" name="firstname"></p>
    <p><label for="lastname">Last name:</label><br />
    <input type="text" id="lastname" name="lastname"></p>
    <button type="submit" name="form" value="submit">Go</button>  
  </form>
  <br />
  Welcome prénom [lvl high] nom [lvl high]
</div>
```

Cela oblige donc à passer par l'insertion de nouvelles balises afin d'injecter du contenu. Dans ce niveau de difficulté, aucune balise ne semble être interprétée par l'application, que cela soit le classique `<script></script>` :&#x20;

![](<../../../../../.gitbook/assets/image (7) (1) (1).png>)

L'exécution de Javascript dans un attribut HTML :&#x20;

![](<../../../../../.gitbook/assets/image (20) (1) (1).png>)

Ou également l'utilisation de balises customs :&#x20;

![](<../../../../../.gitbook/assets/image (3).png>)

L'encodage URL (ni même le double encodage) ne fonctionne également pas :&#x20;

![](<../../../../../.gitbook/assets/image (21) (1) (1).png>)

L'application semble être bien protégée contre les injections XSS (et également HTML), aucun contournement possible n'a été identifié.

## Analyse du code source

Après avoir testé que les paramètres sont bien présents, l'affichage s'effectue grâce à un simple `echo` :

{% code title="htmli_post.php" %}
```php
if(isset($_POST["firstname"]) && isset($_POST["lastname"])) {
  $firstname = $_POST["firstname"];
  $lastname = $_POST["lastname"];

  if($firstname == "" or $lastname == "") {
    echo "<font color=\"red\">Please enter both fields...</font>";
  }
  else {
    echo "Welcome " . htmli($firstname) . " " . htmli($lastname);
  }
}
```
{% endcode %}

Les données sont au préalable traitées par la méthode `htmli()` permettant de gérer la difficulté du niveau sélectionné :

{% code title="htmli_post.php" %}
```php
function htmli($data) {

  switch($_COOKIE["security_level"]) {
    case "0" :
      $data = no_check($data);
      break;
    case "1" :
      $data = xss_check_1($data);
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

Le level "High" correspond à la valeur 2, c'est à dire ici à l'exécution de la méthode `xss_check_3()` :&#x20;

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

La fonction `htmlspecialchars()` est une méthode PHP permettant de transformer certains caractères (`&`, `"`, `<` et `>`) dans leur version entités HTML. De plus, la méthode effectue également ce traitement sur le caractère `'` si l'option `ENT_QUOTES` est présente : &#x20;

![](<../../../../../.gitbook/assets/image (12) (1) (1) (1).png>)

Cette protection est donc efficace contre les injections Javascript.

{% hint style="warning" %}
Attention à bien spécifier `ENT_QUOTES comme ici,` car son absence peut introduire un contournement.
{% endhint %}
