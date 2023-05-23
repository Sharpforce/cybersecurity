# Niveau "High"

En "High", je constate que la technique utilisée précédemment pour uploader le script PHP ne fonctionne plus et cela même en modifiant le `Content-Type` :

![](../../../../.gitbook/assets/1e0ab4cb53726442207bcb4d14e9e251.png)

Le serveur renvoi une erreur indiquant un mauvais format d'image :

![](../../../../.gitbook/assets/40e5e73ad8a5e2c8e2412981c9dd5a2c.png)

De plus l'extension est également vérifiée par l'application car il n'est pas possible d'uploader une image valide mais dont l'extension n'est pas une extension de type image.

La solution ici est en fait de passer par les métadonnées de l'image afin d'y injecter la payload PHP. Pour cela j'utilise le logiciel "Exif Pilot", mais n'importe quel logiciel pouvant éditer les métadonnées doit pouvoir faire l'affaire :

![](../../../../.gitbook/assets/9bdecaa28670f077fa7fdb730201d8be.png)

Rien ne se passe si je visualise l'image directement :

![](../../../../.gitbook/assets/abdc1b531e346bccc83116d5a1703892.png)

Par contre, si j'exploite en plus une vulnérabilité de type File Inclusion (disponible sur DVWA et exploitée ici en "Low") :

![](<../../../../.gitbook/assets/85009f332def02ee08861fe6963c1500 (1).png>)

Alors le code PHP est bien exécuté par le serveur.

{% hint style="info" %}
Cela ne s'arrête pas là mais les possibilités sont vastes : récupération d'un shell plus complet, maintien de l'accès, tentative d'élévation de privilèges, pivotage, ...
{% endhint %}
