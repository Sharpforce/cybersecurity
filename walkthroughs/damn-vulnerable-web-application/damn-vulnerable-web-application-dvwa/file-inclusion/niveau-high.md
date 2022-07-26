# Niveau "High"

Les réponses du serveur face aux attaques précédentes varient légèrement ici puisque je récupère l'erreur suivante pour chacun de mes essais :

![](../../../../.gitbook/assets/f67415de8b00b3d38436adf029c583a7.png)

La seule chose facilement réalisable est l'accès à la page cachée :

![](../../../../.gitbook/assets/fe24d097dc3d31bc633ce1d0c82663bd.png)

Il est sans doute possible qu'un filtre vérifie la présence du mot "file" dans le paramètre d'inclusion. L'idée du développeur était sans doute ici de ne pouvoir inclure seulement les fichiers `file*.php` .&#x20;

Je tente un premier contournement mais sans succès :

![](../../../../.gitbook/assets/eca544497d1118465bb8fb73ca0ba739.png)

Après quelques essais, j'identifie qu'il est impératif que la valeur du paramètre commence par la chaîne "file" :

![](../../../../.gitbook/assets/f658ec962579ec04fe396494372eb65a.png)

Ou alors, en utilisant le schéma `file://` qui permet d'accéder facilement au système de fichiers local :

![](../../../../.gitbook/assets/5d1fa13aa88a23e7dbed07e60fdd8642.png)

&#x20;Je n'ai pas réussi à effectuer une RFI ici, mais cela ne veut pas dire que ce n'est pas possible :upside\_down:&#x20;
