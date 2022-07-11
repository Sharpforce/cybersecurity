# Niveau "High"

## Exploitation

Le message est transmit en utilisant la méthode HTTP `POST` afin d'être affiché à l'utilisateur à la manière d'un forum :&#x20;

```http
POST /bWAPP/htmli_stored.php HTTP/1.1
Host: 192.168.56.116
Content-Length: 33
Origin: http://192.168.56.116
Content-Type: application/x-www-form-urlencoded
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://192.168.56.116/bWAPP/htmli_stored.php
Cookie: PHPSESSID=dd072f586873ddbc6e0a10ae3229b270; security_level=2
Connection: close

entry=Test&blog=submit&entry_add=
```

![](<../../../../../.gitbook/assets/image (26).png>)

Le message est donc ici stockée sur le serveur, sans doute en base de données, afin que tous les utilisateurs puissent y accéder (fonctionnalité Show all). L'injection XSS ne semble pas réalisable ici, que cela soit par les balises `<script></script>` :&#x20;

```http
POST /bWAPP/htmli_stored.php HTTP/1.1
Host: 192.168.56.116
Content-Length: 68
Origin: http://192.168.56.116
Content-Type: application/x-www-form-urlencoded
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://192.168.56.116/bWAPP/htmli_stored.php
Cookie: PHPSESSID=dd072f586873ddbc6e0a10ae3229b270; security_level=2
Connection: close

entry=%3Cscript%3Ealert%281%29%3C%2Fscript%3E&blog=submit&entry_add=
```

![](<../../../../../.gitbook/assets/image (12).png>)

Par des attributs HTML :&#x20;

```http
POST /bWAPP/htmli_stored.php HTTP/1.1
Host: 192.168.56.116
Content-Length: 69
Origin: http://192.168.56.116
Content-Type: application/x-www-form-urlencoded
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://192.168.56.116/bWAPP/htmli_stored.php
Cookie: PHPSESSID=dd072f586873ddbc6e0a10ae3229b270; security_level=2
Connection: close

entry=%3Cimg+src%3Dx+onerror%3Dalert%281%29%3E&blog=submit&entry_add=
```

![](<../../../../../.gitbook/assets/image (15).png>)

Par quelques contournements comme de l'encodage URL :&#x20;

```http
POST /bWAPP/htmli_stored.php HTTP/1.1
Host: 192.168.56.116
Content-Length: 154
Origin: http://192.168.56.116
Content-Type: application/x-www-form-urlencodeAccept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://192.168.56.116/bWAPP/htmli_stored.php
Cookie: PHPSESSID=dd072f586873ddbc6e0a10ae3229b270; security_level=2
Connection: close

entry=%253c%2573%2563%2572%2569%2570%2574%253e%2561%256c%2565%2572%2574%2528%2531%2529%253c%252f%2573%2563%2572%2569%2570%2574%253e&blog=submit&entry_add=
```

![](<../../../../../.gitbook/assets/image (30).png>)

L'injection HTML/XSS ne semble donc pas réalisable.

## Analyse du code source

L'application récupère dans la variable `$entry` les messages stockés en base de données avant d' afficher le retour de la méthode `xss_check_3()` (ligne 35, correspondant au niveau "High" ou 2) :&#x20;

{% code title="htmli_stored.php" %}
```php
<table id="table_yellow">
  <tr height="30" bgcolor="#ffb717" align="center">
    <td width="20">#</td>
    <td width="100"><b>Owner</b></td>
    <td width="100"><b>Date</b></td>
    <td width="445"><b>Entry</b></td>
  </tr>

<?php
  // Selects all the records
  $entry_all = isset($_POST["entry_all"]) ? 1 : 0;
  if($entry_all == false) {
    $sql = "SELECT * FROM blog WHERE owner = '" . $_SESSION["login"] . "'";
  }
  else {
    $sql = "SELECT * FROM blog";
  }
  
  $recordset = $link->query($sql);
  if(!$recordset) {
    // die("Error: " . $link->connect_error . "<br /><br />");
?>
  <tr height="50">
    <td colspan="4" width="665"><?php die("Error: " . $link->error);?></td>
  </tr>
<?php
  }
  while($row = $recordset->fetch_object()) {
    if($_COOKIE["security_level"] == "1" or $_COOKIE["security_level"] == "2") {
?>
  <tr height="40">
    <td align="center"><?php echo $row->id; ?></td>
    <td><?php echo $row->owner; ?></td>
    <td><?php echo $row->date; ?></td>
    <td><?php echo xss_check_3($row->entry); ?></td>
  </tr>
<?php
    }
    else {
?>
  <tr height="40">
    <td align="center"><?php echo $row->id; ?></td>
    <td><?php echo $row->owner; ?></td>
    <td><?php echo $row->date; ?></td>
    <td><?php echo $row->entry; ?></td>
  </tr>
<?php
    }
  }
  
  $recordset->close();
  $link->close();
?>
</table>
```
{% endcode %}

La méthode `xss_check_3()` effectue en fait un assainissement en appelant à sont tour la méthode PHP `htmlspecialchars()` :&#x20;

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

L'injection HTML/XSS n'est donc pas possible. Un traitement est toutefois effectué lors de l'ajout du message en base (`entry_add`) par la méthode `htmli()` :&#x20;

{% code title="htmli_stored.php" %}
```php
if(isset($_POST["entry_add"])) {
  $entry = htmli($_POST["entry"]);
  $owner = $_SESSION["login"];

  if($entry == "") {
    $message =  "<font color=\"red\">Please enter some text...</font>";
  }
  else {
    $sql = "INSERT INTO blog (date, entry, owner) VALUES (now(),'" . $entry . "','" . $owner . "')";
    $recordset = $link->query($sql);
    if(!$recordset) {
      die("Error: " . $link->error . "<br /><br />");
    }

    $message = "<font color=\"green\">Your entry was added to our blog!</font>";
  }
}
```
{% endcode %}

Qui effectue un assainissement via la méthode `sqli_check_3()` :&#x20;

{% code title="htmli_stored.php" %}
```php
function htmli($data) {
  include("connect_i.php");

  switch($_COOKIE["security_level"]) {
    case "0" :
      $data = sqli_check_3($link, $data);
      break;
    case "1" :
      $data = sqli_check_3($link, $data);
      break;
    case "2" :
      $data = sqli_check_3($link, $data);
      break;
    default :
      $data = sqli_check_3($link, $data);
      break;
  }

  return $data;
}
```
{% endcode %}

Cette fonction effectue un assainissement protégeant contre les injections SQL (mais pas HTML/XSS) grâce à la méthode PHP `mysqli_real_scape_string()` :&#x20;

{% code title="functions_external.php" %}
```php
function sqli_check_3($link, $data) {
  return mysqli_real_escape_string($link, $data);
}
```
{% endcode %}

