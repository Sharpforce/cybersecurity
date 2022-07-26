# Niveau "High"

Le formulaire du challenge se voit ajouter un champ contenant un jeton anti-CSRF de type per-request (c'est à dire qu'il s'agit d'un jeton différent pour chaque requête) :

![](../../../../.gitbook/assets/366f3f72d838c81409f2e182f76acc53.png)

De plus, le délai entre deux tentatives est devenu aléatoire pour une valeur comprise plus ou moins dans l'intervalle \[1 ; 3] secondes (en fait \[0 ; 3] d'après les sources):

![](../../../../.gitbook/assets/05c200ef17b4c900634756113303cf58.png)

Afin de contourner la cette nouvelle protection, il me faut dans un premier temps extraire le jeton de la réponse HTTP précédente (présent suite à la requête **`GET`** permettant d'afficher le formulaire) puis l'injecter dans la prochaine tentative de brute force. Cela se fait bien en utilisant Burp et son Intruder :

![](../../../../.gitbook/assets/120fcd787545648cba72980540bc09a2.png)

La payload 1 est le dictionnaire de mots de passe et la payload 2 représente le jeton anti-CSRF en mode recursive grep :

![](../../../../.gitbook/assets/6239fb57635d4bb5b13a37fed63140f7.png)

En plus de l'extract, il ne faut pas oublier de faire suivre les redirections à Burp (et de limiter le nombre de thread à 1) :

![](../../../../.gitbook/assets/d02dba4ae42405189dfb8217b9f13f46.png)

Puis j'inclue le premier jeton (celui présent dans la requête qui a permis de basculer en mode Intruder) :

![](../../../../.gitbook/assets/2f56ffa0b2b2065a7b0ec16d2e27c13b.png)

Concernant le délai il n'y a pas de solution mis à part prendre son mal en patience, mais je retrouve tout de même assez rapidement le mot de passe de l'administrateur. La requête contenant le bon mot de passe est identifiée grâce à sa taille qui est différente des autres requêtes (mais il est possible de se baser sur d'autres informations si nécessaire) :

![](../../../../.gitbook/assets/aa0ec3cdb6e518c711c5b0828e9c6cd4.png)
