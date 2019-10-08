# Considérations de sécurité

## HTTPS

Le cookie pouvant contenir des données sensibles \(jeton de session par exemple\), il est impératif qu'il soit envoyé seulement via des URLs en HTTPS.

L'attribut **Secure** permet d'obliger l'envoi de cookie seulement si l'URL est de type HTTPS.

## Cross-Site Scripting \(XSS\)

La vulnérabilité XSS permet de récupérer le contenu des cookies de la victime. La payload la plus simple permettant de prouver cela est sans doute :

```markup
<script>alert(document.cookie)</script>
```

Si le cookie récupéré par l'attaquant est le cookie de session il pourra alors l'utiliser afin d'avoir accès à la session de la victime \(il possédera les mêmes droits que sa victime jusqu'à invalidation ou expiration de la session\).

L'attribut **HttpOnly** permet de mitiger ces risques en rendant illisible le contenu des cookies par un autre protocole que HTTP.

## Cross-Site Request Forgery \(CSRF\)

Le fait que les cookies soient envoyés automatiquement par le navigateur lors des requêtes offre la possibilité à l'exploitation de la faille CSRF. Cette faille permet de faire effectuer une action à la victime et cela à son insu.

En plus d'implémenter les bonnes pratiques de développement concernant cette vulnérabilité, l'attribut **SameSite** peut aider à mitiger cette vulnérabilité en imposant une politique quant à l'envoi de cookies en ce qui concerne les requêtes Cross-Origin.

## Cross-Site Script Inclusion \(XSSI\)

La vulnérabilité XSSI repose sur le même principe que la vulnérabilité CSRF. L'attaquant va se servir du fait que les cookies de la victime sont envoyés lors de l'utilisation de requêtes Cross-Origin. 

A la place de faire effectuer une action à la victime, l'attaquant va récupérer des données potentiellement sensibles disponibles dans certains types de fichiers et cela grâce à l'utilisation de la balise `script` :

```markup
<script src="http://sitevulnerable/fichier-sensible.hs></script>
```

## Identifiants de session faible

Lorsque le cookie véhicule le jeton de session permettant de reconnaître le visiteur, une attention particulière doit être portée à la génération de cet identifiant et au fait qu'il ne doit pas être prédictible et cryptographiquement robuste. 

Un mauvais exemple peut être un jeton incrémentant une valeur pour chaque nouvelle session. 

## Fixation de session

La vulnérabilité Fixation de session survient quand l'attaquant est capable de fixer \(spécifier\) lui-même la valeur de l'id de session qui sera utilisé. 

Dans un environnement sécurisé, cette valeur doit être généré aléatoirement, posséder une longueur minimale et être non prédictible. De plus, cette valeur doit être déterminée \(fixée\) par le serveur et non par le client.

## Dépendance avec le système DNS

Afin de déterminer si le cookie doit être joint à la requête ou non, le navigateur se base sur le système DNS. Il doit en effet déterminer la correspondance entre le domaine requêté et le domaine spécifié dans le cookie.

Cela signifie donc que si le serveur DNS utilisé pour cette résolution est partiellement ou complètement compromis alors la sécurité du cookie \(et donc des visiteurs et du site\)  n'est plus assurée.

