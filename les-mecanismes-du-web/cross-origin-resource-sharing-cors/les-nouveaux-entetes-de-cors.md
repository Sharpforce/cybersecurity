# Les nouveaux entêtes de CORS

## Entêtes de réponse

CORS spécifie plusieurs nouveaux entêtes de réponse renvoyés par le serveur hébergeant la ressource. Ces entêtes permettent d'effectuer un contrôle d'accès sur les ressources protégées.

### Access-Control-Allow-Origin

L'entête principal se nomme `Access-Control-Allow-Origin` et permet de dresser la liste des domaines autorisés à accéder à la ressource protégée. Il peut prendre plusieurs valeurs :

* `*` : le symbole  `*` , ou joker, permet d'autoriser tous les domaines à requêter la ressource

  ```text
  Access-Control-Allow-Origin: *
  ```

  > Note : si l'entête contient le signe `*` alors le navigateur interdira l'envoi des informations d'authentification même si cela est explicitement demandé

* `<origin>` : il est également possible de spécifier un domaine autorisé, dans ce cas, le serveur doit également inclure un entête `Vary` à la valeur `Origin` :

  ```text
  Access-Control-Allow-Origin: https://example.com
  Vary: Origin
  ```

* `null` : même si cela n'est pas recommandé, il est possible de renseigner la valeur `null` :

  ```text
  Access-Control-Allow-Origin: null
  ```

### Access-Control-Allow-Methods

Indique quel méthodes HTTP sont autorisées à être utilisées lors de l'accès à la ressource. Il peut prendre plusieurs valeurs :

* `*` : le symbole `*`, ou joker, permet d'autoriser tous les verbes HTTP

  ```text
  Access-Control-Allow-Methods: *
  ```

  > Note : si l'entête contient le signe `*` et que les informations de connexion sont envoyées, alors le caractère `*` ne sera plus considéré comme un joker mais spécifiquement par un verbe HTTP ayant pour nom `*`.

* `<method>` : permet de spécifier une liste blanche de méthodes HTTP autorisées :

  ```text
  Access-Control-Allow-Methods: GET, POST, OPTIONS
  ```

### Access-Control-Allow-Headers

Indique quel entêtes HTTP sont autorisés à être utilisés lors de l'accès à la ressource. Il peut prendre plusieurs valeurs :

* `*` : le symbole `*`, ou joker, permet d'autoriser tous les entêtes HTTP

  ```text
  Access-Control-Allow-Headers: *
  ```

  > Note :
  >
  > * si l'entête contient le signe `*` et que les informations d'authentification sont envoyées, alors le caractère `*` ne sera plus considéré comme un joker mais spécifiquement par un verbe HTTP ayant pour nom `*`.
  > * l'entête HTTP `Authorization` ne peut pas être _wildcardé_ et doit être explicitement déclaré
  > * les entêtes `Accept`, `Accept-Language`, `Content-Language` et `Content-Type` sont toujours autorisés et ne nécessitent pas d'être ajoutés.

* `<header>` : permet de spécifier une liste blanche d'entêtes HTTP autorisés :

  ```text
  Access-Control-Allow-Headers: Custom-Header1, Custom-Header2
  ```

### Access-Control-Allow-Credentials

Autorise ou non le front \(code Javascript\) à accéder à la réponse lorsque les informations d'authentification sont incluses.

* `<directive> (boolean)` : porte la valeur true ou false :

  ```text
  Access-Control-Allow-Credentials: true
  ```

### Access-Control-Max-Age

Cet entête permet d'indiquer au navigateur la durée dont le résultat de la requête de contrôle \(preflight request\) peut être mis en cache. Une fois ce temps passé, le navigateur devra effectuer à nouveau ce contrôle. Chaque navigateur possède une valeur seuil maximale évitant ainsi les abus des serveurs ayant renseigné une valeur démesurée.

* `<time> (en secondes)` : un cache d'une durée de 1 heure

  ```text
  Access-Control-Max-Age: 3600
  ```

  > Note :
  >
  > * Une valeur de  "- 1" va désactiver le cache, obligeant le navigateur à effectuer une preflight request pour toute requêtes cross-origin
  > * Firefox admet une valeur maximale à 24 heures \(86 400 secondes\)
  > * Chromium \(avant la v76\) admet une valeur maximale à 10 minutes \(600 secondes\)
  > * Chromium \(depuis la v76\) admet une valeur maximale à 2 heures \(7 200 secondes\)
  > * Chromium admet une valeur par défaut à 5 secondes

### Access-Control-Expose-Headers

Indique quel entêtes HTTP sont autorisés à être exposés dans la réponse HTTP. Il peut prendre plusieurs valeurs :

* `*` : le symbole `*`, ou joker, permet d'autoriser tous les entêtes HTTP

  ```text
  Access-Control-Expose-Headers: *
  ```

  > Note : si l'entête contient le signe `*` et que les informations de connexion sont envoyées, alors le caractère `*` ne sera plus considéré comme un joker mais spécifiquement par un verbe HTTP ayant pour nom `*`.

* les entêtes sont séparés par des virgules :

  ```text
  Access-Control-Expose-Headers: Custom-Header1, Custom-Header2
  ```

  > Note : Par défaut, les 6 entêtes suivants sont exposés :
  >
  > * `Cache-Control`
  > * `Content-Language`
  > * `Content-Type`
  > * `Expires`
  > * `Last-Modified`
  > * `Pragma`

## Entêtes de requête

CORS spécifie trois nouveaux entêtes de requête envoyé par le navigateur. Cela peut être lors de l'accès à la ressource \(dans le cas d'une requête simple\) ou dans une preflight request.

### Origin

Cet entête contient le domaine \(l'origine\) effectuant la requête. Il peut prendre plusieurs valeurs :

* `<origin>` : il s'agit de l'URI qui a demandé la requête

  ```text
  Origin: http://example.com
  ```

* `null` : dans certains cas, l'origine d'une requête peut être `null`

  ```text
  Origin : null
  ```

### Access-Control-Request-Method

Cet entête est présent seulement dans les preflight request et a pour objectif d'interroger le serveur afin de savoir si la méthode demandée est autorisée

* `<method>` : nom de la méthode HTTP dont le navigateur demande l'autorisation

  ```text
  Access-Control-Request-Method: POST
  ```

### Access-Control-Request-Headers

Cet entête est présent seulement dans les preflight request et a pour objectif d'interroger le serveur afin de savoir si les entêtes demandés sont autorisés

* `<headers>` : liste d'entêtes dont le navigateur demande l'autorisation

  ```text
  Access-Control-Request-Headers: Custom-Header1, Custom-Header2
  ```

