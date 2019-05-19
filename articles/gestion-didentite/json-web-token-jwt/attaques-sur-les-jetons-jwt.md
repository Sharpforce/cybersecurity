# Attaques sur les jetons JWT

## **JWS - Alg "none" attack**

Cette attaque consiste à faire passer un jeton signé pour un jeton non signé afin de pouvoir en modifier la payload. Par exemple, voici un jeton légitime signé reçu par l'attaquant :

```text
eyJhbGciOiJSUzI1NiIsICJ0eXAiOiJKV1QifQ.eyJuYW1lIjoiU2hhcnBmb3JjZSIsInJvbGUiOiJVc2VyIn0.qo2Pr1Fo5c_HeItw85G8Tu8qWdQeIiB-HfPaWs-s2Wk
```

Son entête est le suivant \(après décodage Base64URL\) :

`{  
  "alg":"RS256",  
  "typ":"JWT"  
}`

Ainsi que la payload :

`{  
  "name":"Sharpforce",  
  "role":"User"  
}`

Ce jeton peut être modifié afin, par exemple, de s'octroyer les droits administrateur. Pour cela il suffit de modifier la payload comme ceci :

`{  
  "name":"Sharpforce",  
  "role":"Admin"  
}`

Le problème est que le service consommateur n'acceptera jamais ce jeton, car la signature n'est pas valide. Alors l'attaquant va simplement indiquer que le jeton n'est pas signé et supprimer la signature :

`{  
  "alg":"none",  
  "typ":"JWT"  
}`

`{  
  "name":"Sharpforce",  
  "role":"Admin"  
}`

Une fois encodé le jeton falsifié qui sera envoyé au service consommateur devient :

```text
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJuYW1lIjoiU2hhcnBmb3JjZSIsInJvbGUiOiJBZG1pbiJ9.
```

Le jeton pourra ainsi être considéré comme légitime et donc accepté par le service.

Cette attaque se retrouve par exemple dans la librairie `prime-jwt` dans les version &lt;= 1.30. La CVE correspondante est détaillée ici :

{% embed url="https://nvd.nist.gov/vuln/detail/CVE-2018-1000531" %}

## **JWS - RS256 Public-Key as HS256 Secret Attack**

En cryptographie asymétrique, la clé publique est en général récupérable par tout le monde. La clé publique sert à chiffrer une donnée à destination du possesseur de la clé privée ou alors à  vérifier la signature d'une donnée signée par le possesseur de la clé privée. En admettant que l'attaquant ai accès à cette clé, il lui est alors possible de modifier le jeton afin de tromper le service consommateur en lui faisant croire qu'il s'agit d'un jeton signé utilisant un secret partagé \(par exemple HS256\).

 Admettons le jeton suivant :

`{  
  "alg":"RS256",  
  "typ":"JWT"  
}  
{  
  "name":"Sharpforce",  
  "role":"User"  
}`

Etant donné que le jeton est signé en utilisant un algorithme asymétrique, le service doit normalement vérifier la signature en utilisant la clé publique à sa disposition. Mais l'attaquant peut tenter de forcer le service à vérifier le jeton en lui faisant croire qu'il s'agit d'une signature à secret partagé \(en modifiant l'algorithme dans l'entête du jeton\) :

`{  
  "alg":"HS256",  
  "typ":"JWT"  
}  
{  
  "name":"Sharpforce",  
  "role":"User"  
}`

Cette fois, contrairement à l'attaque précédente, l'attaquant _connait_ la "clé partagée" utilisée pour la signature. Cette clé partagé est en fait la clé publique qui doit servir à vérifier le jeton signé par l'entité émettrice. Pour résumer, cette attaque consiste à faire passer une signature à clés symétriques à une signature à secret partagé. La clé publique devenant ainsi le secret partagé, l'attaquant peut alors modifier puis signer le jeton illégitime \(ou alors en forger un nouveau\).

Cette vulnérabilité est présente dans la libraire Node.js `jwt-simple` pour les version &lt;= 0.3.0 sous la dénomination CVE-2016-10555 : 

{% embed url="https://nvd.nist.gov/vuln/detail/CVE-2016-10555" %}

## **JWS - Weak HMAC Keys** 

Dans le cas d'une signature par clé partagé , un  attaquant peut tenter de brute forcer la clé afin de signer de nouveaux jetons. 

Pour contrer cela, la RFC recommande que la taille de la clé partagée soit au moins égale à la taille \(en bits\) de la sortie de la fonction de hachage utilisée par l'algorithme HMAC.

## **Substitution Attacks**

Admettons que dans un réseau d'entreprise, plusieurs services sont disponibles grâce aux jetons JWT et que ce jeton JWT soit délivré par une entité unique pour l'entreprise. Un utilisateur peut par exemple \(après authentification\) accéder au service consommateur A :

![](../../../.gitbook/assets/0e1e42f046827532f946c3234e53ecc4.png)

Cette requête est légitime, l’utilisateur possède bien les droits pour accéder à ce service. Admettons maintenant qu'un service consommateur B existe, mais que l'utilisateur ne possède pas les droits d'accès pour ce service. Par exemple, lorsqu'il tente d'y accéder, il est redirigé vers le fournisseur de jeton pour authentification et celui-ci indique une erreur \(accès non-autorisé\) après sa saisie de son login/mot de passe.

Un utilisateur malicieux peut alors tenter d'injecter le jeton valide utilisé pour accéder au service consommateur A mais dans une requête à destination du service consommateur B \(en interceptant la requête avec un proxy puis en injectant le header d'autorisation avec le jeton\) :

![](../../../.gitbook/assets/5ce3d29e62039239cf4f242e98c642af.png)

Pour contrer cela, un champ spécifique \(**aud**\) peut être ajouté dans l'entête du jeton permettant d'indiquer le service pour lequel le jeton a été créé.

