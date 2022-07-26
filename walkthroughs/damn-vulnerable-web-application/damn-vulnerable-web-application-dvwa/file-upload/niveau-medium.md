# Niveau "Medium"

Je tente directement d'uploader à nouveau le précédent fichier `.php` :

![](../../../../.gitbook/assets/c7542f1488e6f62a480b7f1c59e40e30.png)

La nouvelle protection en place m'empêche de réitérer la même attaque. Burp va sans doute m'aider sur ce coup là car je vais tenter de modifier le content-type du fichier `.php` afin de satisfaire au prérequis de l'application.

Voici la requête d'upload de fichiers pour le script PHP :

![](../../../../.gitbook/assets/8b1bf3ed2314c3a2db5325acf8ef5b4c.png)

Je modifie le `Content-Type` afin de lui indiquer qu'il s'agit en fait d'une image :

![](../../../../.gitbook/assets/0a0876accb0ba90f2764565d9289f4dd.png)

Cela fonctionne et mon upload de fichier malicieux fonctionne encore :

![](../../../../.gitbook/assets/cb34324765b97019ee63a98604c52766.png)

Il est possible de faire exécuter le code du script de la même façon que pour le niveau "Low" :

![](../../../../.gitbook/assets/c1126a3bd8f2d2094e6697cb9ff22d5f.png)

{% hint style="info" %}
Cela ne s'arrête pas là mais les possibilités sont vastes : récupération d'un shell plus complet, maintien de l'accès, tentative d'élévation de privilèges, pivotage, ...
{% endhint %}
