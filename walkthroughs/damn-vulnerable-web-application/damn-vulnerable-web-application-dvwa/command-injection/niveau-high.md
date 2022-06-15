# Niveau "High"

Le ping semble fonctionner toujours de la même manière pour ce niveau "High" :

![](<../../../../.gitbook/assets/4a5a279f278e5d0ebb27a28c0a2f4112 (4).png>)

En testant toutes les possibilités, aucun enchaînement de commandes ne semble fonctionner. Mais ici, la petite astuce est d'utiliser un pipe sans ajouter d'espace entre ce pipe et la seconde commande :

![](../../../../.gitbook/assets/f233bbba36d5c89f7a6d0ab2d3af3484.png)

La même attaque mais contenant des espaces ne fonctionne pas :

![](../../../../.gitbook/assets/ff700d3a303d372e3b52b18418b90cf1.png)

Ce fonctionnement provient d'une petite erreur laissée par le développeur lors de la mise en place de sa liste noire. L'occurrence filtrée ici n'est pas `"|"` mais `"| "` (bien noter l'espace) :

![](../../../../.gitbook/assets/7c74130f14e3bba322695cfa6e9e41ae.png)

Une autre solution peut être l'utilisation d'un saut de ligne entre les deux commandes. En effet, lors de l'utilisation d'une séquence de type [**list**](https://linux.die.net/man/1/bash) il est possible d'utiliser un saut de ligne à la place du caractère `";"` :

![](../../../../.gitbook/assets/71d3699455b827c13dd31bde87a4c1f2.png)

Pour exploiter cela, il suffit de transformer notre champ `<input>` en champ `<textarea>` (via la console par exemple) :

![](../../../../.gitbook/assets/0433b8b1411219701c3d400f10c634c7.png)

Puis d'injecter la seconde commande après un saut de ligne :

![](../../../../.gitbook/assets/15748c3634c1cbb35840b410faf5e9e2.png)

Il est tout à fait possible d'utiliser Burp en mode Interception ou Repeater pour insérer ce saut de ligne :

![](../../../../.gitbook/assets/c99b85e1d9a08fb376e4e6ec19790326.png)

{% hint style="info" %}
Cela ne s'arrête pas là mais les possibilités sont vastes : récupération d'un shell plus complet, maintien de l'accès, tentative d'élévation de privilèges, pivotage, ...
{% endhint %}



