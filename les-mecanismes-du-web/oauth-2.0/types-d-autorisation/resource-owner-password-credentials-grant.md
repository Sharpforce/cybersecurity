# Resource Owner Password Credentials Grant

## Description

Ce type d’autorisation convient dans les cas où le détenteur de données possède une relation de confiance avec l’application \(par exemple un employé d’une entreprise et une application interne à l’entreprise ou un système d’exploitation et une application possédant des privilèges élevés\). Le serveur d’autorisation ne doit activer ce type d’autorisation seulement quand aucun autre n’est viable/possible.

Ce type d’autorisation convient pour les clients qui sont en mesures d’obtenir les identifiants \(nom d’utilisateur / mot de passe\) du détenteur de ressources.

## Diagramme de séquence

![](../../../.gitbook/assets/85d51b40584922bd3cd02782ccad9c51.png)

1. Le détenteur de ressources fournit ses identifiants à l’application cliente
2. L’application demande un jeton d’accès au serveur d’autorisation. Elle s’authentifie au serveur d’autorisation \(**client\_id** / **client\_secret**\) et fournit également les identifiants du détenteur de ressources
3. Le serveur d’autorisation authentifie l’application cliente et valide les identifiants du détenteur de ressources. Il fournit ensuite le jeton d’accès
4. L’application cliente effectue une requête au serveur de ressources en incluant le jeton d’accès.
5. Le serveur de ressources répond à l’application cliente avec les ressources demandées

## Synthèse des échanges

### Requête de jeton d’accès \(étape 2\)

Le client effectue une requête au serveur d’autorisation en ajoutant les paramètres suivants :

* **grant\_type** \(OBLIGATOIRE\) : la valeur doit être « password »
* **username** \(OBLIGATOIRE\) : le nom d’utilisateur du détenteur de ressources
* **password** \(OBLIGATOIRE\) : le mot de passe du détenteur de ressources
* **scope** \(OPTIONNEL\) : scope demandé par le client

Si le client est de type **confidential** \(ou si le client fournit une authentification\) il doit alors s’authentifier auprès du serveur d’autorisation, via le header `Authorization` ****par exemple.

Exemple de requête :

```text
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded

grant_type=password&username=johndoe&password=A3ddj3w
```

### Réponse de demande de jeton d’accès \(étape 3\)

Si la demande de jeton d’accès est valide alors le serveur d’autorisation fournit le jeton d’accès et optionnellement le jeton de rafraîchissement. Dans le cas contraire il retourne une erreur.

Exemple de réponse :

```text
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
  "access_token":"2YotnFZFEjr1zCsicMWpAA",
  "token_type":"example",
  "expires_in":3600,
  "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
  "example_parameter":"example_value"
}
```

## 

