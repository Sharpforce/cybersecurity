# Niveau "Low"

Le challenge se présence ici sous la forme d'un guestbook nous permettant de laisser un message visible par tous les utilisateurs :

![](../../../../.gitbook/assets/6ab2ac37de99102a462805cac7b612ee.png)

Deux champs nous sont proposés ici. Un premier test va nous permettre de mieux cerner le fonctionnement de l'application :

![](../../../../.gitbook/assets/95c1c1c95928f8fd821ca02b139e5e9e.png)

Les deux champs sont présents au niveau de l'affichage du message. De plus, les longueurs des champs sont respectivement limitées à 15 et 50 caractères, mais un second test nous indique que cette limitation n'est pas renforcée côté back \(la console de dév ou Burp peut nous aider à supprimer cette limitation\) :

![](../../../../.gitbook/assets/a6a5dc14827e194802bb07ca976d740f.png)

On effectue un essai d'injection de scripts :

![](../../../../.gitbook/assets/482e22a915c4851cd87a87f74ba0ebfb.png)

Les deux champs sont vulnérables à une injection XSS. Une payload simple à base d'image va nous permettre de récupérer le jeton de notre victime :

![](../../../../.gitbook/assets/0f21d4f70df502302de84738f70e8df9.png)

Etant donné que la faille XSS est de type stockée, alors chaque utilisateur visitant la page va déclencher la payload \(dont nous même 🤣\) :

![](../../../../.gitbook/assets/7c0ebe3e5065021fb7e0bb496625a85a.png)

