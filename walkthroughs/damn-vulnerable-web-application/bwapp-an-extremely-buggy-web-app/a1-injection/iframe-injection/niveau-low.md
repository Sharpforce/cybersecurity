# Niveau "Low"

## Exploitation

La page du challenge permet de charger une `<iframe></iframe>`. L'URL admet 3 paramètres :&#x20;

* le premier, `ParamURL`, est l'URL cible de l'iframe, ayant comme valeur par défaut "robots.txt"
* le second et la troisième, `ParamWidth` et `ParamHeight`, sont la taille en largeur et en hauteur de l'iframe, ayant comme valeur par défaut "250"

![](<../../../../../.gitbook/assets/image (8) (1).png>)

Avant de tenter de contourner l'iframe (via ses attributs) afin d'injecter du code Javascript, il est possible d'analyser l'assainissement des paramètres d'entrée. Est ce que l'application assainie correctement les caractères tels que `"`, `'` ou encore `<` et `>` :&#x20;

```http
GET /bWAPP/iframei.php?ParamUrl=robots.txt%22%3E%3C/iframe%3E%3Cscript%3Ealert(1)%3C/script%3E%3Ciframe%20src=%22&ParamWidth=250&ParamHeight=250 HTTP/1.1
Host: 192.168.56.116
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Cookie: security_level=0; PHPSESSID=94d5a1179643bdb8e87d3d0039b6cac5
Connection: close
```

![](<../../../../../.gitbook/assets/image (33).png>)

L'injection est donc ici réussi facilement.&#x20;

## Analyse du code source

L'application récupère les variables et affiche le retour de la méthode `xss()` en tant que valeur des attributs HTML de l'iframe (le niveau de sécurité est ici 0 pour le niveau "Low" exécutant le code situé dans le `else`) :&#x20;

{% code title="iframei.php" %}
```php
<?php
  if($_COOKIE["security_level"] == "1" || $_COOKIE["security_level"] == "2") {
?>
    <iframe frameborder="0" src="robots.txt" height="<?php echo xss($_GET["ParamHeight"])?>" width="<?php echo xss($_GET["ParamWidth"])?>"></iframe>
<?php
  }
  else {
?>
    <iframe frameborder="0" src="<?php echo xss($_GET["ParamUrl"])?>" height="<?php echo xss($_GET["ParamHeight"])?>" width="<?php echo xss($_GET["ParamWidth"])?>"></iframe>
<?php
  }
?>
```
{% endcode %}

La méthode `xss()` est un `switch` exécutant une certaine méthode selon le niveau de sécurité actuel : &#x20;

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

Dans le cas du niveau "Low", la méthode `no_check()` est exécutée, mais comme son nom l'indique, elle ne fais aucun assainissement et retourne seulement le paramètre passée en paramètre :&#x20;

{% code title="functions_external.php" %}
```php
function no_check($data) {
  return $data;
}
```
{% endcode %}
