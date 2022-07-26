# Level 2 (practice)

## Challenge

### Description

Appeler la fonction `niceSuperHero()` avec la chaîne de caractères `"58"` en argument :

![](../../../.gitbook/assets/9d60fd68c40fed43e01dc509e5be5404.png)

### Résolution

Un filtre supprime les occurrences du caractères `"r"`. Ce caractère est présent dans le nom de la fonction mais également dans la balise `<script></script>` :

![](../../../.gitbook/assets/249b03ebfa0af6cdeb599d504adacccd.png)

La balise `<script></script>` peut être écrite indépendamment de la casse utilisée, il est donc également possible d'utiliser le caractère `"R"` mais cette technique ne peut pas fonctionner pour le nom de la fonction car case sensitive :

![](../../../.gitbook/assets/0a9985ee4576b43148a320abb49c3c21.png)

Afin de contourner le filtre, j'utilise le format de caractère unicode, soit ici `\u0072` :

![](../../../.gitbook/assets/417c22c48982f73c7dc7cc56568ea8d3.png)
