# Niveau "Medium"

A première vue, aucune différence au niveau du formulaire de changement de mot de passe :

![](../../../../.gitbook/assets/f25e26c4fec0b697b8ff774f7ea91ab4.png)

Ni au niveau de la requête effectuée lors de la soumission du formulaire :

![](../../../../.gitbook/assets/41f83d7a7ebc0b31f89865fb6b795d38.png)

L'attaque précédente, effectuée pour le niveau "Low", doit donc toujours fonctionner. Mais ce n'est malheureusement pas aussi simple :

![](../../../../.gitbook/assets/bfe2487eaf3948bcc51c89015ab3c758.png)



J'analyse alors la différence entre la requête légitime et celle générée par ma précédente (tentative d') attaque :

![](../../../../.gitbook/assets/57e93dd720248cb62751c3e65862f417.png)

La seule différence réside dans le contenu de l'entête HTTP `Referer`. L'application doit sans doute vérifier que cet entête possède la même valeur que l'hôte d'où elle est hébergée. Une technique permettant de contourner cette protection est d'identifier et exploiter une vulnérabilité XSS sur l'application cible.&#x20;

Cela tombe bien puisque DVWA contient également une telle vulnérabilité (ne pas oublier de contourner la limitation de la longueur du champ, voir le challenge XSS si besoin) :

![](../../../../.gitbook/assets/8b1b89d32a7c2ffb168a59b9cc2aa285.png)

{% hint style="info" %}
Il est possible de repasser en niveau "Low" le temps de forger la payload XSS car seule l'exploitation de la faille CSRF a besoin de se faire en niveau "Medium"
{% endhint %}



Soit lors de la navigation de la victime :

![](../../../../.gitbook/assets/d0da1fda181ffba0dc7774a0bf695fd4.png)

Lors de sa visite de la page du Guestbook contenant l'image malveillante, la requête déclenchée sera effectuée avec un `Referer` valide :

![](../../../../.gitbook/assets/052890166f4826c302b676545f6c067d.png)

Et son mot de passe sera ainsi changé, contournant ainsi, la protection en place :

![](../../../../.gitbook/assets/ed9212a48569e45bc67a2b0667b49889.png)
