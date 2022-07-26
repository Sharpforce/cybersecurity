# Niveau "High"

Le niveau "High" met en place un jeton de session qui semble plus complexe de prime abord :

![](../../../../.gitbook/assets/beb72e331bb44b2b4f78ff4c54ca8e69.png)

Le format du jeton est toutefois reconnaissable et il s'agit ici sans doute d'une empreinte md5 (32 caractères). Deux approches ici :

1. Soit je détermine ce qui a pu donner cette empreinte (mon adresse IP, le timestamp, mon nom d'utilisateur etc)
2. Soit je tente de la reverser et je vois  ce que cela donne

Je suggère la seconde approche et si elle échoue je passe  à la première :yum: :

![](../../../../.gitbook/assets/252a8368d2191db6848c54bde3f79301.png)

L'empreinte est celle du nombre "17". Je génère un second jeton pour confirmer :

![](../../../../.gitbook/assets/7d3dfb347b91166613ef6bf8b0981f2f.png)

Il s'agit de l'empreinte du nombre "18" :

![](../../../../.gitbook/assets/4dba091b77ef23d390b61fe674a66eeb.png)

Il s'agit donc en fait de la même mécanique de génération que le niveau "Low" à laquelle est ensuite appliquée une méthode de hashage.

Pour détourner la session d'un utilisateur il suffit donc de tester le hash d'un nombre inférieur et de croiser les doigts.

Burp va t'il être capable de reconnaître que se cache une suite simple sous ces empreintes :

![](../../../../.gitbook/assets/57ed8689c98fa3fcf2e8f5219971b807.png)

En fait non, Burp analyse l'aléa des jetons comme ayant une entropie de 115 bits. Je propose donc de conclure par : "Rien ne remplacera l'humain :wink: ".
