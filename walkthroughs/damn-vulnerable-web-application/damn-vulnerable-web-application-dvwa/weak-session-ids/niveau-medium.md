# Niveau "Medium"

Le niveau "Medium" renvoi comme valeur de jeton une suite de chiffres :

![](../../../../.gitbook/assets/db617544b1cb08c5cf12ff0969ed0c96.png)

Cette suite est reconnaissable à son format, il s'agit d'un timestamp UNIX :

![](../../../../.gitbook/assets/d1b226b62a4010844515e1791563aca9.png)

Afin d'usurper l'identité d'une personne il suffira donc de tester les sessions passées avec, une certaine chance que les sessions les plus récentes soient encore actives.

Voyons ce qu'en dit le Sequencer de Burp :

![](../../../../.gitbook/assets/a5a367d82180205beb7abd7ff1395e8d.png)

En effet, étant donné qu'il n'y ait toujours pas d'aléa dans ce jeton, Burp le qualifie également d' "extremely poor" avec une entropie de 0 bits.



