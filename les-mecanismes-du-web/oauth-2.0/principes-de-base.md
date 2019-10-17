# Principes de base

## Les rôles

Le protocole OAuth 2.0 définit 4 rôles :

* **Resource Owner** \(Détenteur de données\) : Entité capable de donner l’accès à des ressources protégées. Quand le détenteur de données est une personne et également l’utilisateur de l’application cliente on le nomme alors utilisateur final \(end-user\)
* **Resource Server** \(Serveur de ressources\) : Serveur qui héberge les données dont l’accès est protégé et qui est capable d’accepter et de répondre aux requêtes demandant l’accès aux ressources protégées grâce à un jeton d’accès
* **Client Application** \(Consommateur\) : Application/client qui demande l’accès aux informations protégées. Le client peut très bien s’exécuter sur un serveur, une machine de bureau ou tout autre dispositif
* **Authorization Server** \(Serveur d’autorisation\) : Serveur qui délivre les différents jetons \(tokens\) au consommateur après authentification et autorisation du détenteur de données

## Les jetons

Les jetons sont des chaines de caractères qui sont envoyés dans la requête par le client afin d’accéder aux ressources. Il existe deux jetons différents : 

* **Access Token** \(Jeton d’accès\) : Ce jeton est récupéré auprès du serveur d’autorisation et doit être joint aux requêtes effectuées par le client afin d'accéder aux ressources. Pour simplifier, il est possible de voir un jeton d’accès comme étant un mot de passe. Si le jeton est valide, alors le client peut accéder à la ressource. Il a une durée de vie limitée qui est définit par le serveur d’autorisation
* **Refresh Token** \(Jeton de renouvellement\) : Ce jeton est optionnel et est délivré en même temps que le jeton d’accès. Il n’a pas besoin d’être joint à chaque requête effectué par l’application car il sert simplement à demander un nouveau jeton d’accès quand le premier arrive à expiration

## Le périmètre \(scope\)

Le jeton d’accès n’est pas forcément valable pour toutes les ressources de l'utilisateur, les scopes permettent de définir la liste de ces ressources. Par exemple un jeton d’accès pour accéder à Facebook peut permettre seulement d’accéder aux informations du profil \(nom, prénom, etc\) mais pas aux photos de l’utilisateur. Les scopes sont définit par le serveur d’autorisation et il doit également être renseigné dans la requête effectuée par le client.

## Le paramètre d'état \(state\)

Le paramètre d’état est présent dans les requêtes et réponses effectuées ou reçues par le client. Le client est en mesure de fixer sa valeur lors de la requête et cette même valeur sera retournée par le serveur d’autorisation lors de la redirection. Il peut donc être utilisé par le client pour « garder en mémoire » un état spécifique de l’application ou de l’utilisateur final. 

Une autre utilisation est de s’en servir comme jeton anti-CSRF \(voir partie Recommandations de sécurité\).

## Le client

### S'enregistrer en tant que client

Pour pouvoir accéder aux ressources protégées en utilisant OAuth, il faut que le client s’inscrive auprès du serveur d’autorisation. Lors de l’enregistrement le client doit fournir les informations suivantes : 

* Spécifier le type du client : **confidential** ou **public**
* Spécifier les URLs de redirection \(peut prendre une ou plusieurs valeurs\) :
  * Plusieurs URLs de redirections peuvent être utiles afin d’utiliser un seul **client\_id** pour une application web et une application mobile par exemple, ou alors pour utiliser plusieurs environnements \(sandbox & production par exemple\).
  * Certains serveurs d’autorisation effectue une vérification par égalité \(entre l’URL spécifié dans la requête et celle enregistrée\) ce qui engendre que l’url [https://example.com/auth](https://example.com/auth) ne correspondra pas à l’url [https://example.com/auth?destination=account](https://example.com/auth?destination=account). Pour éviter cela, il est préférable d’éviter les paramètres dans les URLs de redirection
* Fournir des informations complémentaires \(non obligatoire\) : description de l’application, logo, etc 

En échange, le serveur d’autorisation retournera un **client\_id** ainsi qu’un **client\_secret** \(optionnel\).

### Les types de client

OAuth 2.0 définit deux types de client basés sur le niveau de confidentialité possible entre le client et le serveur d’autorisation :

* **Confidential** : le client est capable de garder secret ses informations d’authentification \(par exemple, application côté serveur et contrôle d'accès permettant l’accès à ses informations\)
* **Public** : le client est incapable de garder secret ses informations d’authentification \(par exemple une application full-Javascript ou encore application lourde sur la machine de l’utilisateur final\). Ce type d’applications ne possède donc pas de **client\_secret**

### Identifiant du client

Le serveur d’autorisation fournit à la suite de l’enregistrement du client un identifiant client unique \(nommé **client\_id**\). Cet identifiant n’est pas secret et peut donc être exposé à l’utilisateur final. Par contre, il ne doit pas \(en tout cas, ne devrait pas\) permettre seul d'authentifier un client auprès du serveur d’autorisation.

### Authentification du client

Si le type du client est **confidential** alors le client et le serveur d’autorisation établissent une méthode d’authentification satisfaisant les deux parties. Par exemple il est possible d’utiliser une authentification par mot de passe, par une paire de clé publique/privée, etc.

#### Authentification HTTP Basic

Si le client est en possession d’un mot de passe \(nommé **client\_secret**\) il est alors fortement recommandé d’utiliser le schéma d’authentification `HTTP Basic`. La valeur du header est le résultat de la fonction `Base64(client_id:client_secret)` :

```text
Authorization: Basic czZCaGRSa3F0Mzo3RmpmcDBaQnIxS3REUmJuZlZkbUl3
```

#### Authentification dans le corps de la requête

Si une des deux parties ne supportent pas cette méthode d’authentification il est alors possible de transmettre le **client\_id** et le **client\_secret** dans le corps de la requête. Cette méthode n’est toutefois pas recommandée pour des raisons de sécurité.

{% hint style="warning" %}
De plus, il est clairement recommandé de ne pas inclure ces informations en tant que paramètres de l’URL \(méthode GET\).
{% endhint %}

Exemple d’authentification :

```text
POST /token HTTP/1.1
Host: server.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&refresh_token=tGzv3JOkF0XG5Qx2TlKWIA&client_id=s6BhdRkqt3&client_secret=7Fjfp0ZBr1KtDRbnfVdmIw
```

## Les endpoints

Le processus d’autorisation utilise au moins deux endpoints \(ressources HTTP\) différents :

* **Authorization endpoint** : utilisé par le client pour obtenir l’autorisation de la part du détenteur de données \(authentification de l’utilisateur final\) en utilisant une redirection HTTP. Il est utilisé pour les types d’autorisation **authorization code grant** ainsi que **implicit grant**
* **Token enpoint** : utilisé par le client pour récupérer le jeton d’accès permettant d’accéder aux ressources protégées. Ne sert pas pour le type **implicit grant**

Le client possède également un endpoint : 

* **Redirection endpoint** : utilisé par le serveur d’autorisation pour envoyer les réponses au client en utilisant une redirection HTTP

