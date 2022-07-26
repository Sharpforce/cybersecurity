# Niveau "High"

Dans ce dernier niveau de difficulté, il n'est plus possible de renseigner une valeur aléatoire en guise de langue car le serveur répond dans ce cas par une **`302`** redirigeant vers une URL contenant une valeur correcte :

![](../../../../.gitbook/assets/d3501a88d9eb1f97699ae40a8639ff0e.png)

Le code Javascript permettant de renseigner la valeur sélectionnée n'ayant pas été modifiée, j'exploite la vulnérabilité DOM XSS comme il se doit et je contourne les mécanismes de protection en place :

![](../../../../.gitbook/assets/1cf6df179eef29a5ed65da8cd7ca9a70.png)

Et le cookie de la victime est maintenant mien :

![](../../../../.gitbook/assets/b3146bec802b0a365ff2b925e22261db.png)
