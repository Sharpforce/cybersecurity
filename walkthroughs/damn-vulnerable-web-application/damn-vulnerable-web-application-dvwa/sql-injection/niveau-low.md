# Niveau "Low"

Le niveau "Low" présente un simple formulaire permettant de renseigner un id d'utilisateur :

![](../../../../.gitbook/assets/57fe8ea0b980a99aef79795098c5d98a.png)

La première chose à faire est d'étudier le comportement de l'application en renseignant des données valides :

![](../../../../.gitbook/assets/06d6e13cd7524489e12b62fc16b77c8c.png)

La réponse pour un id valide remonte plusieurs informations : le `First name` ainsi que le `Surname`, soit à minima 2 colonnes. Concernant l’`id` il peut soit être  récupéré depuis la réponse à la requête SQL \(donc une troisième colonne\) ou alors être la réflexion de la donnée renseignée en entrée.

Il faut bien sur également tester une requête non valide, c'est-à-dire un `id` inexistant :

![](../../../../.gitbook/assets/9ab8b2fd2ff2b0b0e7fa8513a31db32b.png)

Dans ce cas, aucune donnée n'est remontée et aucun message d'erreur n'est affiché.

Tentons de détecter maintenant si le champ est susceptible d'être vulnérable à une injection SQL. Cela peut être fait de plusieurs manières, mais la plus simple reste sans doute l’insertion d’un caractère spécifique à SQL tel que le signe `"'"`:

![](../../../../.gitbook/assets/e2d71f8f21b72d24b044b313908a9c49.png)

L'erreur SQL retournée ici révèle en effet la possibilité d'une injection SQL ainsi que le fait son type sans doute de type String.

On commence donc par récupérer le nombre de colonnes retourné par la requête :

![](../../../../.gitbook/assets/f968a78f1c7876dc92aae3e6d10f05e7.png)

L'erreur survient lors de la requête sur une troisième colonne :

![](../../../../.gitbook/assets/72df5d3112b5250c63b2812c52ae7589.png)

Cela indique qu'il y a seulement 2 colonnes. On repère maintenant l'emplacement de l'affichage des données remontées :

![](../../../../.gitbook/assets/bb6a2810ddeee823d8faf898bb822a91.png)

L'exploitation de la vulnérabilité peut maintenant réellement commencer. On récupère le nom de la base de données utilisée ainsi que le nom de l'utilisateur s'y connectant :

![](../../../../.gitbook/assets/239580b27e02172e74c281875032a234.png)

On récupère ensuite les noms des tables de la base `dvwa` \(le premier apostrophe est ajouté seulement pour la coloration syntaxique\) :

```sql
'6' UNION SELECT table_name,2 FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = 'dvwa' -- 
```

![](../../../../.gitbook/assets/a60cc14189f8f11d25213e9743077725.png)

La table nommée `users` semble ici la plus intéressante. On utilise la même technique afin de récupérer le nom des colonnes de cette table :

```sql
'6' UNION SELECT column_name,2 FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_SCHEMA = 'dvwa' -- 
```

![](../../../../.gitbook/assets/4c3426e0d20a9b047fb1e6545b25e104.png)

Puis finalement, on récupère les entrées pour les colonnes `user` et `password` de cette table `user` :

```sql
'6' UNION SELECT user,password FROM users -- 
```

![](../../../../.gitbook/assets/0972536dd708ca239a89054f65b5b4c7.png)

Une fois fait, la dernière étape reste de cracker les hash md5 \(32 caractères\) des mots de passe en utilisant par exemple [crackstation.net](https://crackstation.net/) :

![](../../../../.gitbook/assets/80ef0f7a16a8a069f943e801429ef8f7%20%281%29.png)

