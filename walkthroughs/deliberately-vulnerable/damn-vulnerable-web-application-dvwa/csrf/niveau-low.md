# Niveau "Low"

Le challenge propose un formulaire permettant de renseigner le nouveau mot de passe ainsi que sa confirmation :

![](../../../../.gitbook/assets/5af2d0ed634cf59f5af9d60cd6df3a0c.png)

Dans un premier temps, j'analyse le fonctionnement de l'application ainsi que les requêtes effectuées lors d'un changement de mot de passe. La mise à jour du mot de passe s'effectue via une requête **`GET`** et aucune protection ne semble présente (pas de jeton anti-CSRF) :

![](../../../../.gitbook/assets/ef490d66bb0105e19c5e034ee5fbae7a.png)

L'exploitation de la vulnérabilité CSRF est donc ici trivial : une simple image pointant vers l'URL d'action est suffisante. Sur un serveur d'attaque, j'héberge la page suivante :

```markup
<html>
  <head>
    <title>Malicious page</title>
  </head>

  <body>
    <img src="http://192.168.56.203:8080/vulnerabilities/csrf?password_new=hacked&password_conf=hacked&Change=Change" />
  </body>
</html>
```

Il est maintenant nécessaire que la victime (l'administrateur de DVWA) soit authentifiée sur l'application vulnérable puis qu'elle visite la page malicieuse. Lors de cette navigation, son navigateur va tenter de récupérer l'image qui va silencieusement modifier son mot de passe :

![](../../../../.gitbook/assets/acb20ffc9390c0d04e0dd7efb4e9fc7f.png)

{% hint style="info" %}
L'URL "http://192.168.56.120/" présent dans l'entête `Referer` représente le serveur malicieux
{% endhint %}



Pour confirmer l'exploitation, l'administrateur tente de se connecter (ou utilise la mire d'authentification du challenge brute force) avec son ancien mot de passe :&#x20;

![](../../../../.gitbook/assets/74c4026702bbb03c147411c0ee08e245.png)

Mais son nouveau mot de passe est bien celui que j'ai renseigné lors de l'attaque :

![](../../../../.gitbook/assets/f341249aaacec4aef550b969ec341d28.png)
