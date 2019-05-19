# Notions avancées

## Utiliser un jeton de rafraîchissement

Si le serveur d’autorisation fournit un jeton de rafraîchissement \(**refresh token**\) alors le client peut effectuer une requête au token endpoint afin d’obtenir un nouveau jeton d’accès. Voici les paramètres à renseigner dans une telle requête : 

* **grant\_type** \(OBLIGATOIRE\) : la valeur doit être « refresh\_token »
* **refresh\_token** \(OBLIGATOIRE\) : le jeton de rafraîchissement
* **scope** \(OPTIONNEL\) : scope demandé par le client

Si le client est de type **confidential** \(ou si le client fournit une authentification\) il doit alors s’authentifier auprès du serveur d’autorisation, via le header `Authorization` ****par exemple.

Exemple de requête :

```text
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&refresh_token=tGzv3JOkF0XG5Qx2TlKWIA
```

Si la requête est valide, alors le serveur d’autorisation doit fourni un nouveau jeton d’accès mais également un nouveau jeton de rafraîchissement \(l’ancien doit être révoqué par le serveur d’autorisation\). Le nouveau jeton de rafraîchissement possédera le même **scope** que le précédent.

## Le type de jeton d'accès

Il existe plusieurs types de jeton d’accès : 

* **Bearer token** : il doit être utilisé en étant renseigné dans un entête HTTP nommé `Authorization` et ayant la valeur `Bearer` :

```text
GET /resource/1 HTTP/1.1
Host: example.com
Authorization: Bearer mF_9.B5f-4.1JqM
```

* **Mac token** : il doit être utilisé en étant renseigné dans un entête HTTP nommé `Authorization` ****et ayant la valeur `Mac`. De plus, le message MAC \(Message Authentication Code\) doit être inclut :

```text
GET /resource/1 HTTP/1.1
Host: example.com
Authorization: MAC id="h480djs93hd8", nonce="274312:dj83hs9s", mac="kDZvddkndxvhGRXZhvuDjEWhGeE="
```

## Introspection

Avec OAuth 2.0, le contenu d’un jeton d’accès est opaque pour le client, c'est-à-dire qu’un client ne connait rien à propos du jeton lui-même ou de sa structure. Pourtant, un certain nombre d’informations sont rattachées à un jeton d’accès, par exemple sa validité \(est-il valide, expiré ou encore révoqué\), les **scopes** associés ainsi que d’autres informations sur le contexte pour lequel il a été fourni \(qui est le serveur d’autorisation et pour quel client\). Ce mécanisme a été mis en place pour les **serveurs de ressources** et les **ressources protégées**.

### Introspection endpoint

Une requête vers l’endpoint d’introspection comprend un paramètre contenant le jeton d’accès et la réponse sera au format JSON contenant les méta-informations du jeton. Il est nécessaire que ce endpoint soit protégé grâce à TLS.

### Requête d’introspection

Le serveur de ressource effectue une requête d’introspection en utilisant le verbe `HTTP POST` :

* **token** \(OBLIGATOIRE\) : le jeton d’accès.
* **username\_type\_hint** \(OPTIONNEL\) : un paramètre optionnel pouvant indiquer le type de jeton afin d’aider et d’accélérer la recherche effectuée par le serveur d’autorisation.

Il est nécessaire à des fins de sécurité que le serveur de ressource s’authentifie \(en utilisant des credentials ou un second jeton d’accès\) auprès du serveur d’autorisation.

Exemple de requête \(en utilisant un second jeton d’accès\) :

```text
POST /introspect HTTP/1.1
Host: server.example.com
Accept: application/json
Content-Type: application/x-www-form-urlencoded
Authorization: Bearer 23410913-abewfq.123483

token=2YotnFZFEjr1zCsicMWpAA
```

#### Réponse d’introspection

Le serveur répond par un objet JSON contenant les informations suivantes :

* **active** \(OBLIGATOIRE\) : booléen indiquant l’état du jeton
* **scope** \(OPTIONNEL\) : un paramètre optionnel indiquant la liste des scopes associés au jeton
* **client\_id** \(OPTIONNEL\) : l’identifiant du client associé au jeton d’accès
* **username** \(OPTIONNEL\) : identifiant du détenteur de ressources associé à ce jeton d’accès
* **token-type** \(OPTIONNEL\) : type de jeton \(`bearer` ou `mac`\)
* **exp** \(OPTIONNEL\) : date à laquelle le jeton expire
* **iat** \(OPTIONNEL\) : date à laquelle le jeton a été fourni
* **nbf** \(OPTIONNEL\) : date à laquelle le jeton peut être utilisé
* **sub** \(OPTIONNEL\) : sujet du jeton comme défini dans la norme JWT \(Il s’agit d’un identifiant du détenteur de ressources associé à ce jeton\)
* **aud** \(OPTIONNEL\) : identifiant du destinataire comme défini dans la norme JWT
* **iss** \(OPTIONNEL\) : identifiant du fournisseur du jeton comme défini dans la norme JWT
* **jti** \(OPTIONNEL\) : chaîne de caractère identifiant le jeton comme défini dans la norme JWT

Exemple de réponse :

```text
HTTP/1.1 200 OK
Content-Type: application/json

{
  "active": true,
  "client_id": "l238j323ds-23ij4",
  "username": "jdoe",
  "scope": "read write dolphin",
  "sub": "Z5O3upPC88QrAjx00dis",
  "aud": "https://protected.example.net/resource",
  "iss": "https://server.example.com/",
  "exp": 1419356238,
  "iat": 1419350238,
  "extension_field": "twenty-seven"
}
```

## Enregistrement dynamique du client

### Diagramme de séquence

![](../../../.gitbook/assets/be54b847ac468b1cff7525ccc15e6708.png)

1. \(OPTIONNEL\) : Le client récupère un jeton d’accès permettant d’accéder au endpoint d’enregistrement dynamique
2. \(OPTIONNEL\) : Le client récupère une assertion \(JWT par exemple\) signé  permettant d’accéder au endpoint d’enregistrement dynamique
3. Le client effectue un appel au endpoint d’enregistrement du serveur d’autorisation. Cette requête contient les métadonnées du client ainsi qu’un jeton d’accès si nécessaires \(1 ou 2\)
4. Le serveur d’autorisation enregistre le client et retourne une réponse contenant les informations suivantes : - Les métadonnées du client - Un identifiant client **client\_id** - ****Un ensemble d’information comme par exemple le **client\_secret** si le type de client le permet

### Les métadonnées clientes

Les clients possèdent un certain nombre de métadonnées associées à leur identifiant client. Voici le format de ces métadonnées :

* **redirects\_uris** : la ou les URLs de redirection du client
* **token\_endpoint\_auth\_method** : chaîne de caractères indiquant le type d’authentification pour effectuer une requête vers le endpoint d’enregistrement :
  * **none** : le client est public et ne possède pas de donnée d’authentification
  * **client\_secret\_post** : le client utilise la méthode `HTTP POST` pour fournir les données d’authentification
  * **client\_secret\_basic** : le client utilise l’authentification `HTTP Basic`
* **grant\_type** : type d’autorisation désiré \(si non renseigné alors code d’autorisation par défaut\) :

  * **authorization\_code**
  * **Implicit**
  * **password**
  * **client\_credentials**
  * **refresh\_token**

En réponse, le serveur d’autorisation retourne les informations suivantes \(fournit par le client\) :

* Réponse de type « code » si le type d’autorisation est code d’autorisation, « token » s’il s’agit du type implicite et « none » pour les autres types
* **client\_name** : nom du client
* **client\_uri** : url d’une page permettant d’obtenir des informations sur le client
* **logo\_uri** : logo du client
* **scope** : liste des scopes que le client peut utiliser
* **contacts** : information de contact du client
* **tos\_uri** : URL indiquant les conditions d’utilisation
* **policy\_uri** : URL indiquant la politique en vigueur \(collecte et stockage des informations personnelles etc\)
* **jwks\_uri** : URI indiquant du document contenant la clé publique JWK \(JSON Web Key\)
* **jwks** : informations concernant JWK et qui contient également la clé publique
* **sofwtare\_id** : identifiant unique fournit par le client permettant d’identifier le client et de l’enregistrer de façon dynamique
* **sofwtare\_version** : numéro de version du client qui doit être mis à jour après chaque modification du client

Le serveur d’autorisation ajoute également les informations suivantes :

* **client\_id** \(OBLIGATOIRE\) : identifiant du client
* **client\_secret** \(OPTIONNEL\) : secret du client \(dépend du type d’autorisation\)
* **client\_id\_issued\_at** \(OPTIONNEL\) : date à laquelle le client\_id a été fournit
* **client\_secret\_expires\_at** \(OPTIONNEL\) : date à laquelle le client\_secret expire \(0 si n’expire jamais\)

Exemple de requête \(sans jeton d’accès initial\) :

```text
POST /register HTTP/1.1
Content-Type: application/json
Accept: application/json
Host: server.example.com

 {
   "redirect_uris": [
   "https://client.example.org/callback",
   "https://client.example.org/callback2"],
   "client_name": "My Example Client",
   "client_name#ja-Jpan-JP":
   "\u30AF\u30E9\u30A4\u30A2\u30F3\u30C8\u540D",
   "token_endpoint_auth_method": "client_secret_basic",
   "logo_uri": "https://client.example.org/logo.png",
   "jwks_uri": "https://client.example.org/my_public_keys.jwks",
   "example_extension_parameter": "example_value"
}
```

Exemple de réponse :

```text
HTTP/1.1 201 Created
Content-Type: application/json
Cache-Control: no-store
Pragma: no-cache

{
  "client_id": "s6BhdRkqt3",
  "client_secret": "cf136dc3c1fc93f31185e5885805d",
  "client_id_issued_at": 2893256800,
  "client_secret_expires_at": 2893276800,
  "redirect_uris": [
  "https://client.example.org/callback",
  "https://client.example.org/callback2"],
  "grant_types": ["authorization_code", "refresh_token"],
  "client_name": "My Example Client",
  "client_name#ja-Jpan-JP":
  "\u30AF\u30E9\u30A4\u30A2\u30F3\u30C8\u540D",
  "token_endpoint_auth_method": "client_secret_basic",
  "logo_uri": "https://client.example.org/logo.png",
  "jwks_uri": "https://client.example.org/my_public_keys.jwks",
  "example_extension_parameter": "example_value"
}
```

## 

