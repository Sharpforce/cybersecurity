# Niveau "Low"

Ce challenge nous demande de renseigner le mot "success" pour gagner. Le champ de saisie comporte une valeur par défaut, "ChangeMe" :

![](../../../../.gitbook/assets/faed5e78756c7abffcb07696f5df5793.png)

On lance Burp et on tente notre chance en saisissant le mot demandé :

![](../../../../.gitbook/assets/ed041dd6f38f33b1b6fb174787495a4a.png)

Nous voyons passer un jeton dans la requête, sans doute un champ caché. La réponse du serveur est un peu moins fun puisque nous avons le droit à une erreur "Invalid token" :

![](../../../../.gitbook/assets/5a1736b65834dbaf52f7404dbc105ce8.png)

Pour la suite, j'active l'option de Burp permettant de rendre visible les champs cachés :

![](../../../../.gitbook/assets/f3197ebef6c6835b68e4677de986c571.png)

Il nous faut donc trouver le jeton valide qui correspond à la valeur "success". Pour cela, nous devons comprendre le script qui est présent dans le code de la page :

![](../../../../.gitbook/assets/be176125df7201016a038e9d7657779b.png)

Le script n'est pas difficile à comprendre ici :

1. La méthode `generate_token()` est appelée
2. Cette méthode récupère la valeur présent dans le champ de saisie, effectue un `rot13()` puis calcul son empreinte md5
3. Cette empreinte est finalement placée dans le champ `token`

Afin de récupérer le jeton valide, il faut reproduire les mêmes étapes mais avec la phrase "success". Cela peut se faire directement dans la console du navigateur :

![](../../../../.gitbook/assets/dd8f8ee4841652cbd496adc1e3211ce4.png)

Si nous avons vu juste, il nous suffit de renseigner cette empreinte dans le champ `token` ainsi que la valeur "success" dans le champ `phrase` puis de valider :

![](../../../../.gitbook/assets/9a181712f50027c5441a10ca8be8ed36.png)

Well done !



