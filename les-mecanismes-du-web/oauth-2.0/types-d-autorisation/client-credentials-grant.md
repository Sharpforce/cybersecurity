# Client Credentials Grant

## Description

Le client peut demander via une requête un jeton d’accès en utilisant seulement ses identifiants clients \(**client\_id** et **client\_secret**\) si les données à protéger sont sous son contrôle.

## Diagramme de séquence

![](../../../.gitbook/assets/cff0557e1ad44e701547da26a53368e6.png)

1. Le client s’authentifie auprès du serveur d’autorisation en fournissant ses identifiants \(**client\_id** et **client\_secret**\) et demande un jeton d’accès
2. Le serveur d’autorisation authentifie le client et fournit le jeton d’accès
3. L’application cliente effectue une requête au serveur de ressources en incluant le jeton d’accès
4. Le serveur de ressources répond à l’application cliente avec les ressources demandées

## Synthèse des échanges

### Requête de jeton d’accès \(étape 1\)

Le client effectue une requête au serveur d’autorisation en ajoutant les paramètres suivants :

* **grant\_type** \(OBLIGATOIRE\) : la valeur doit être « client\_credentials »
* **scope** \(OPTIONNEL\) : scope demandé par le client

Le client doit s’authentifier auprès du serveur d’autorisation, via le header `Authorization` par exemple.

Exemple de requête :

```text
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
```

### Réponse de demande de jeton d’accès \(étape 3\)

Si la demande de jeton d’accès est valide alors le serveur d’autorisation fournit le jeton d’accès. Le jeton de rafraîchissement ne devrait pas être inclus. Si la demande n’est pas valide alors le serveur d’autorisation doit retourner une erreur. 

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
  "example_parameter":"example_value"
}
```

