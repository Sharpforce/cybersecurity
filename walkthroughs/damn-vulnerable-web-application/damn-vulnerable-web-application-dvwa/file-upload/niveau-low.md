# Niveau "Low"

Premièrement, analysons comment fonctionne l’application et quels types de fichiers sont acceptés. Appuyons simplement sur le bouton « Upload » sans préalablement sélectionner une image :

![](../../../../.gitbook/assets/24f3922bcae7c785abe756017df059de.png)

Puis, chargeons une image valide. Par exemple Homer mangeant un délicieux donuts :

![](../../../../.gitbook/assets/24e1b21a348343de9fa06cdc4e6f5f26.png)

L'accès à l'image se fait par le chemin indiqué en réponse :

![](../../../../.gitbook/assets/59677938ca6eac6f878bf137748b63b6.png)

L'upload de fichiers étant plutôt destinée à des images, voyons s'il est possible d'uploader du code PHP :

```php
<?php
  echo 'Hacked';
?>
```

L'application semble accepter notre fichier sans broncher :

![](../../../../.gitbook/assets/dda6ca0d46af060803593830fc5474ae.png)

Et l'accès à notre fichier `.php` permet donc une exécution de code :

![](../../../../.gitbook/assets/058409ffe57b950db5d5ecf37b90db29.png)

{% hint style="info" %}
Cela ne s'arrête pas là mais les possibilités sont vastes : récupération d'un shell plus complet, maintien de l'accès, tentative d'élévation de privilèges, pivotage, ...
{% endhint %}

