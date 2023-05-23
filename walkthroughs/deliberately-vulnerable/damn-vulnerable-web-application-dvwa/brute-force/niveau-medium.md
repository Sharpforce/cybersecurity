# Niveau "Medium"

La mire d'authentification présentée ne semble pas différente pour ce niveau :

![](../../../../.gitbook/assets/754b2c241667e45c33ef2f42f51e09f2.png)

Après quelques essais j'identifie un délai présent pour chaque tentative d’authentification. La console du navigateur me permet de déterminer facilement ce délai, qui est ici de 2 secondes :

![](../../../../.gitbook/assets/9be207dc9120a7252140e51e845ce6eb.png)

Aucune autre mesure de protection ne semble être présente.

L’attaque précédente, effectuée avec `hydra` , fonctionnera toujours mais sera juste un peu plus lente. L’attaque par dictionnaire prendra alors, dans ce cas de figure, un peu moins d'une minute au lieu des quelques secondes du niveau "Low" :

![](../../../../.gitbook/assets/928b85a9cda4634ad830fec3d4d9d5f3.png)

L'attaque est donc certes ralentie mais toujours faisable.
