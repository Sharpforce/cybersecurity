# Niveau "Medium"

A priori, pas de changement dans la fonctionnalité de ping pour ce niveau :

![](<../../../../.gitbook/assets/4a5a279f278e5d0ebb27a28c0a2f4112 (2) (1).png>)

Je vais au plus simple en testant directement le séparateur de commandes `";"` :

![](../../../../.gitbook/assets/fc266b20d0d53f89d2c8f7bd75b4aea5.png)

Cela ne fonctionne pas. Il me faut donc tester toutes les autres possibilités. L'opérateur `"||"` semble fonctionner et me permet ainsi de contourner le filtre en place :

![](../../../../.gitbook/assets/b89228d11bc0daef42bdc31e94d5148e.png)

{% hint style="info" %}
Cela ne s'arrête pas là mais les possibilités sont vastes : récupération d'un shell plus complet, maintien de l'accès, tentative d'élévation de privilèges, pivotage, ...
{% endhint %}

Attention, étant donné qu'il s'agit d'un opérateur OR, il faut impérativement que la première commande (le ping) se termine en erreur. Dans le cas contraire la seconde commande ne sera pas exécutée :

![](../../../../.gitbook/assets/d1408ca550ee85ed0a90cea37298f6e0.png)
