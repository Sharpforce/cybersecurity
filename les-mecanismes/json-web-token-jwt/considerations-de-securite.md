# Considérations de sécurité

## Stockage du jeton & informations sensibles

Le jeton est en général stocké soit dans le local/session storage soit dans un cookie. Cette dernière option est préférable en protégeant correctement le cookie avec les options habituels de sécurité \(HttpOnly, Secure ou encore SameSite\).

De plus, étant donné que les données sont seulement encodées, il est déconseillé de faire porter des informations sensibles par le jeton \(ou alors il faut veiller à utiliser un jeton chiffré JWE\).

## Cross-Site Request Forgery \(CSRF\)

Si le jeton JWT est stocké en cookie, alors une attaque CSRF peut être possible si l'application consommatrice n'implémente pas de protection adéquate. Il faut donc, dans ce cas, veiller à bien implémenter les mesures de protections anti-CSRF. 

Si le jeton JWT n'est pas stocké en tant que cookie alors les attaques CSRF ne sont plus possible, mais cela amène d'autres problématiques comme XSS. 

## Cross-Site Scripting \(XSS\)

Si le jeton JWT n'est pas stocké dans un cookie, il est alors stocké dans le local storage ou le session storage. Cette technique est utilisée afin de pouvoir ajouter le jeton dans l'entête HTTP `Authorization` afin d'effectuer des appels \(vers une API par exemple\) :

`Authorization: Bearer <token>`

Cela protège des failles CSRF mais il devient alors possible pour un attaquant de récupérer le contenu du jeton si l'application consommatrice est vulnérable à la faille XSS \(ou un script malicieux/rendu malicieux provenant d'une tierce partie\). Exit donc la protection offerte par l'option **HttpOnly** qui protège contre un vol de session sur une mécanique de session par cookie. 

L'application qui nécessite le jeton doit donc faire attention à être protégée contre ce type de failles.

## Révocation d'un jeton JWT

Par défaut, un jeton JWT ne peut pas être révoqué. Si un attaquant réussi à récupérer un jeton JWT d'un utilisateur légitime, alors il aura accès aux ressources que permet ce jeton pendant toute sa durée de vie; c'est pour cette raison qu'un jeton JWT doit avoir une durée de vie très courte.

Implémenter une possibilité de révocation d'un jeton à l'inconvénient de perdre le coté stateless de JWT. En effet, il faudra d'une manière ou d'une autre maintenir à jour une liste de jetons révoqués, et vérifier que le jeton présenté par l'utilisateur ne soit pas présent dans cette liste \(auquel cas il se verra refuser l'accès\).



