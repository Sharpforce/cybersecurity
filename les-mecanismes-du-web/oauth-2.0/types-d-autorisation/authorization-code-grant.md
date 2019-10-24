# Authorization Code Grant

## Description

Ce type d’autorisation est utilisé pour obtenir un jeton d’accès ainsi qu’un jeton de rafraîchissement \(optionnel\) et est optimisé lorsque le client peut garder des informations confidentielles \(code côté serveur en général\). Cela signifie que l’utilisateur final n’aura jamais accès aux différents jetons d’accès et de rafraîchissement. 

Pour que ce type d’autorisation fonctionne il faut que le client soit capable d’interagir avec le détenteur des données \(l’utilisateur final via le navigateur web par exemple\) et doit également être en mesure de recevoir des requêtes de la part du serveur d’autorisation \(via redirection du navigateur\). 

Il est tout de même possible \(selon la RFC\) de récupérer un code d’autorisation sans envoyer le secret **client\_secret**, mais au final, rare sont les solutions implémentant cela.

## Diagramme de séquence

![](../../../.gitbook/assets/3a94c169a526f72c549437e4ee90ceb6.png)

1. Le client initie la demande d’autorisation en redirigeant le détenteur de données \(via le navigateur\) vers le endpoint du serveur d’autorisation. Dans cette **redirection**, le client inclut son **client\_id**, le type d’autorisation demandé, ici **code**, le **scope** demandé, un paramètre **state** ainsi que l’**URL de redirection**
2. Le serveur d’autorisation authentifie le détenteur de données et lui demande s’il autorise l’application cliente à accéder aux ressources demandées
3. En supposant que le détenteur de données a accepté la demande, le serveur d’autorisation redirige le détenteur de données \(via le navigateur\) vers l’application cliente grâce à l’URL de redirection fournit à la première étape ou lors de la phase d’enregistrement du client. La requête de redirection inclut le **code d’autorisation** ainsi que le paramètre **state** si fournit par le client
4. L’application cliente demande un jeton d’accès au serveur d’autorisation en incluant le **code d’autorisation** précédemment reçut. En effectuant cette requête, le client s’authentifie au serveur d’autorisation en envoyant également son **client\_id** ainsi que son **client\_secret**. Il doit également inclure l’**URL de redirection** qui a permis d’obtenir le code d’autorisation, le serveur d’autorisation vérifie alors la correspondance entre ces deux valeurs. Le client doit aussi inclure le type d’autorisation utilisé \(ici **authorization\_code**\)
5. Le serveur d’autorisation authentifie le client et valide le code d’autorisation. Si tout est valide, alors le serveur d’autorisation répond au client avec un jeton d’accès et optionnellement un jeton de rafraîchissement. Il se peut également que le serveur d’autorisation envoi d’autres informations comme par exemple la durée de vie du jeton d’accès
6. L’application cliente peut donc maintenant appeler le serveur de ressources en incluant dans sa requête le jeton d’accès
7. Le serveur de ressources répond en retournant les données demandées par le client \(si le jeton d’accès est valide\)

## Synthèse des échanges

### Demande d’autorisation \(étape 1\)

Le client construit l’URL de requête en ajoutant les paramètres suivants :

* **response\_type** \(OBLIGATOIRE\) : ici « code »
* **client\_id** \(OBLIGATOIRE\) : l’identifiant du client comme déclaré au niveau du serveur d’autorisation
* **redirect\_uri** \(OPTIONNEL\) : URL où sera redirigé le détenteur de données. Il est optionnel mais tout de même recommandé à des fins de sécurité \(dépend du serveur d’autorisation\)
* **scope** \(OPTIONNEL\) : scope demandé par le client
* **state** \(RECOMMANDE\) : un nonce à des fins de sécurité \(anti-CSRF\) ou variable d'état

Exemple de requête :

```text
GET /authorize?response_type=code&client_id=s6BhdRkqt3&state=xyz&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
Host: server.example.com
```

### Réponse d’autorisation \(étape 3\)

Le serveur d’autorisation fournit le code d’autorisation au client grâce à une redirection en ajoutant les paramètres suivants à l’URL :

* **code** \(OBLIGATOIRE\) : le code d’autorisation généré par le serveur d’autorisation. Ce code doit expirer rapidement après son envoi au client. Une durée de vie de 10 minutes est recommandée. De plus, le client doit utiliser le code une seule fois. Si le serveur d’autorisation reçoit plusieurs fois le même code, il doit refuser les demandes et révoquer tous les jetons d’accès émis pour ce code. Le code est lié à l’identifiant du client \(**client\_id**\) ainsi que l’URL de redirection
* **state** \(OBLIGATOIRE si présent lors de la requête d’autorisation\) : si le client fournit ce paramètre alors la même valeur doit également être présente dans la réponse

Exemple de requête :

```text
HTTP/1.1 302 Found
Location: https://client.example.com/cb?code=SplxlOBeZQQYbYS6WxSbIA&state=xyz
```

### Requête de jeton d’accès \(étape 4\)

Le client effectue une demande de jetons au **token endpoint** du serveur d’autorisation en fournissant les données suivantes :

* **grant\_type** \(OBLIGATOIRE\) : la valeur doit être « authorization\_code »
* **code** \(OBLIGATOIRE\) : le code d’autorisation reçu précédemment
* **redirect\_uri** \(OBLIGATOIRE si présent lors de la requête d’autorisation\) : permet de vérifier la bonne correspondance entre cette valeur et l’URL fournit lors de la requête d’autorisation
* **authorization** \(OBLIGATOIRE si le client n’est pas encore authentifié\) : si le client est déclaré comme confidentiel ou qu’il possède un **client\_id** et un **client\_secret**. La valeur du header est `Base64(client_id:client_secret)` ou alors dans le corps de la requête

Exemple de requête :

```text
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&code=SplxlOBeZQQYbYS6WxSbIA&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb
```

### Réponse de demande de jeton d’accès \(étape 5\)

Si la demande de jeton d’accès est valide alors le serveur d’autorisation fournit le jeton d’accès et optionnellement le jeton de rafraîchissement.

Exemple de requête :

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

Réponse en cas d’erreur : Il est possible que le serveur d’autorisation retourne une erreur si les requêtes sont invalides. A noter qu’en cas d’erreur, le serveur d’autorisation peut ne pas rediriger le détenteur de données comme prévu par le flux normal. Voici les paramètres pouvant être présents dans une réponse d’erreur :

* **error** \(OBLIGATOIRE\) : un code erreur en ASCII. Voici à quoi peuvent correspondre les codes erreur :
  * **invalid\_request** : requête invalide car mal-formée \(paramètre manquant, invalide ou présent plusieurs fois\)
  * **unauthorized\_client** : le client n’est pas autorisé à demander un code d’autorisation en utilisant cette méthode
  * **access\_denied** : le détenteur de ressources n’a pas donné son consentement
  * **unspported\_response\_type** : le serveur d’autorisation ne permet pas d’obtenir un code d’autorisation en utilisant cette méthode
  * **invalid\_scope** : le scope fournit est invalide, inexistant ou mal-formé
  * **server\_error** : une erreur inattendue est survenue \(erreur HTTP 500\)
  * **temporary\_unavailable** : le serveur d’autorisation est occupé \(surcharge ou maintenance, erreur HTTP 503\)
* **error\_description** \(OPTIONNEL\) : description fournissant des informations complémentaires sur l’erreur
* **error\_uri** \(OPTIONNEL\) : URI qui permet de fournir de plus amples détails sur l’erreur
* **state** \(OBLIGATOIRE si présent lors de la requête d’autorisation\) : si le client fournit ce paramètre alors la même valeur doit également être présent dans la réponse

Exemple de requête :

```text
HTTP/1.1 302 Found
Location: https://client.example.com/cb?error=access_denied&state=xyz
```

## 

