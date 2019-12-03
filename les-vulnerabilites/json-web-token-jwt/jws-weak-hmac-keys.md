# JWS - Weak HMAC Keys

Dans le cas d'une signature par clé partagé , un  attaquant peut tenter de brute forcer la clé afin de signer de nouveaux jetons. 

Pour contrer cela, la RFC recommande que la taille de la clé partagée soit au moins égale à la taille \(en bits\) de la sortie de la fonction de hachage utilisée par l'algorithme HMAC.

Des outils existent permettant d'automatiser cette tâche :

**C-JWT-cracker**

Outil écrit en C qui permet des attaques par brute force afin de retrouver le secret ayant signé le jeton. 

{% embed url="https://github.com/brendan-rius/c-jwt-cracker" %}

**JWT-tool**

Outil disponible sur GitHub qui permet de forger et de cracker les jetons JWT afin d'identifier des vulnérabilités : 

* Alg none attack
* RS256 as HS 256 attack
* Brute force dictionary attack

{% embed url="https://github.com/ticarpi/jwt\_tool" %}

## 

