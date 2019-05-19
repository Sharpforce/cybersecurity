# Un jeton JWT c'est quoi ?

## Composition d'un jeton JWT

Un jeton JWT est composé de 3 parties distinctes concaténées avec le caractère "." sur le format suivant :

`HEADER.PAYLOAD.SIGNATURE`

Chacune de ces parties est encodée en Base64URL. Base64URL est une variante de l'encodage Base64 adapté pour être transmis par URL et pour être compatible avec le format des noms de fichiers. La seule différence avec l'encodage Base64 concerne les caractères "+" et "/" qui sont remplacés respectivement par "-" et par "\_". De plus, le padding \(le ou les signes "=" à la fin de la chaîne encodée\) est \(généralement\) ignoré. Le format du jeton devient donc :

`Base64URL(HEADER).Base64URL(PAYLOAD).Base64(SIGNATURE)`

### L'entête JWT \(Header\)

L'entête JWT est un objet au format JSON qui décrit avec quelle méthode parser \(lire\) le jeton ainsi que la méthode de signature ou de chiffrement utilisée le cas échéant. Certains champs \(le terme 'claims' est souvent utilisé\) sont obligatoires.

L'exemple le plus simple de jeton JWT est un jeton non signé et non chiffré, qui possède alors un seul champ obligatoire, le champ `alg` qui doit avoir pour valeur `none` :

`{  
  "alg": "none"  
}`

Un champ non obligatoire mais parfois rencontré est le champ `typ` qui permet de connaître le type du jeton, la valeur rencontrée généralement est `jwt` :

`{  
  "alg": "none",  
  "typ": "jwt"  
}`

### La charge utile \(Payload\)

La charge utile est également un objet au format JSON qui représente le contenu du jeton \(l'information à véhiculer\). Par exemple, si les informations à transmettre sont le nom ainsi que le rôle de l'utilisateur l'objet peut alors être :

`{  
  "username": "Anonyme",  
  "role": "User"  
}`

Bien qu'aucun champ soit obligatoire dans la charge utile, certains champs standards \(définis à la section [10.1 de la RFC](https://tools.ietf.org/html/rfc7519#section-10.1)\) sont souvent utilisés \(et recommandés\) :

* **iss** \(issuer\) : nom de l'entité qui a fournit le jeton JWT
* **sub** \(subject\) : nom de l'entité concerné par le jeton JWT \(par exemple pour l'utilisateur "Anonyme"\)
* **aud** \(audience\):  nom de l'entité \(ou du service par exemple\) pour laquelle le jeton JWT a été créé
* **exp** \(expiration \(time\)\): date d'expiration du jeton JWT
* **iat** \(issued at \(time\)\): date de création du jeton JWT

### Jeton non signé/non chiffré

Au final, une fois les deux parties \(header et payload\) du jeton encodées en Base64URL et concaténées avec le signe "." le jeton ressemble à ceci :

`eyJhbGciOiJub25lIn0.eyJ1c2VybmFtZSI6IkFub255bWUiLCJyb2xlIjoiVXNlciJ9.`

Il s'agit ici d'un jeton JWT non signé et non chiffré. Il est reconnaissable car la troisième partie \(signature\) est manquante mais le dernier signe "." de concaténation est tout de même présent.

## Utilisation d'un jeton JWT

### Jwt pour l'autorisation

Un jeton JWT peut être délivré en réponse à une authentification réussie d'un utilisateur. Ce jeton est sensible puisqu'il représente une preuve d'authentification \(équivalent à un jeton de session par exemple ou au couple login/mot de passe pendant sa validité\). Le jeton doit donc avoir une durée de vie aussi courte que possible.

En général, le jeton est envoyé soit en en-tête de la requête HTTP `Authorization` de type `Bearer` :

`Authorization: Bearer <token>`

Soit via un cookie.

###  JWT pour transporter de l'information

Il est également possible d'utiliser les jetons JWTs pour véhiculer de l'information \(autre que concernant l'autorisation ou l'authentification\). Par exemple, il est possible de stocker dans un jeton le contenu du panier de l'utilisateur d'un site d'e-commerce. Afin de s'assurer de l'émetteur et de l'intégrité du jeton on utilisera les mécanismes de signatures \(JWS pour JSON Web Signature\) voir de chiffrement\(JWE pour JSON Web Encryption \) que permet JWT.

