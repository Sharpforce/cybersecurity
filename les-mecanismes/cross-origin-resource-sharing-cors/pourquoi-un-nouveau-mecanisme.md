# Quelques rappels sur SOP

CORS permet d'étendre la politique SOP et ainsi donner plus de libertés aux développeurs et consommateurs de ressources, mais pourquoi ce besoin ?

Le Web évolue et les usages aussi, la nécessité de partager de l'information entre plusieurs domaines distincts ou à un plus large public se fait ressentir. Prenons l'exemple d'une nouvelle API qui a pour objectif d'être consommée par d'autres sites web. Cette API est publique et ne contient pas de données sensibles. Mais que se passe t'il actuellement \(c'est à dire en étant soumis à la politique SOP\) si un domaine tente une lecture de cette API ? La lecture de la réponse à la requête **`GET`** sera impossible, mais CORS permet de répondre à ces problématiques.

## Les requêtes simples

Dans un premier temps, un rappel sur ce qu'on nomme parfois les requêtes simples. Les requêtes simples sont les requêtes effectuées avec les verbes suivants :

* **`HEAD`**
* **`GET`**
* **`POST`**

Aucun ajout d'entête custom est accepté et l'entête `Content-Type` ne peut prendre seulement que les valeurs suivantes :

* `application/x-www-form-urlencoded`
* `multipart/form-data`
* `text/plain`

Parmi ces requêtes, certaines ne sont pas autorisées par le navigateur \(pour des raisons de sécurité\), c'est le cas de la lecture de la réponse d'une requête **`GET`** vers un autre domaine.

