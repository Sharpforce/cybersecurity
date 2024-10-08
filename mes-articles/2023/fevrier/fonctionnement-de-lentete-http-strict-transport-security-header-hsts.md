---
description: 12 Février 2023
---

# Fonctionnement de l'entête HTTP Strict Transport Security Header (HSTS)

> Que ce soit lorsqu'on effectue un audit de sécurité d'un site web pour nos clients ou, à l'inverse, lorsqu'un utilisateur nous remonte les résultats d'un outil d'analyse de la configuration de notre application, l'absence de l'en-tête de sécurité HTTP Strict Transport Security Header est généralement relevée. Mais à quoi sert exactement cet en-tête et comment fonctionne-t-il ?

## HTTP contre HTTPS

Sans revenir sur le fonctionnement de HTTPS, sa raison d'être est la sécurité des informations transmises entre le client (l'utilisateur via le navigateur) et le serveur web consulté. Il s'agit d'une couche de sécurité ajoutée au protocole HTTP qui permet au client de vérifier l'identité du site web consulté (grâce au certificat émis par ce dernier) et de garantir la confidentialité des données transmises (grâce au chiffrement) ainsi que leur intégrité (par l'utilisation de méthodes cryptographiques telles que les méthodes de hachage). Cela va, par exemple, protéger l'utilisateur des attaques de type Man-in-the-Middle (MitM), contrairement au HTTP où le trafic sera transmis en clair.

Par défaut, le protocole HTTP fonctionne sur le port 80 et sa version sécurisée sur le port 443. Ils peuvent être atteints en spécifiant le protocole comme dans les exemples suivants : [http://example.com](http://example.com/) (non sécurisé) ou [https://example.com](https://example.com/) (sécurisé). Sous Google Chrome, par exemple, une version non sécurisée est visible également dans la barre d'adresse par l'indication "Non sécurisé" accompagnée d'un pictogramme d'exclamation, tandis que la version sécurisée se voit dotée d'un cadenas :

| ![](<../../../.gitbook/assets/image (95).png>) | ![](<../../../.gitbook/assets/image (60).png>) |
| :--------------------------------------------: | :--------------------------------------------: |

{% hint style="info" %}
Mais pourquoi un utilisateur renseignerait une URL avec le protocole HTTP ?&#x20;

Il existe plusieurs cas où cela peut arriver (et sans doute d'autres) :&#x20;

* L'utilisateur n'est pas sensibilisé à la sécurité et renseigne `http://` dans la barre d'adresse.
* L'utilisateur suit un tel lien sur un site (forum, etc.) ou qu'une personne lui a fourni par messagerie instantanée, mail, etc.
* L'utilisateur a enregistré en favori une telle URL afin d'y accéder plus facilement
{% endhint %}

## Redirection des utilisateurs vers HTTPS

Afin de protéger les utilisateurs, les applications web effectuent généralement une redirection du port 80 vers le port 443. De cette façon, un utilisateur tentant d'accéder à l'URL via le protocole HTTP sera redirigé, par la réponse du serveur, vers une connexion HTTPS.&#x20;

<figure><img src="../../../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

Par exemple, la configuration suivante est celle d'un serveur Nginx et permet une telle redirection :&#x20;

```nginx
server {
  listen 80;
  server_name example.com;
  return 301 https://exemple.com$request_uri;
}
```

Lorsqu'un utilisateur accédera à l'application en renseignant [http://example.com](http://example.com) :&#x20;

{% code overflow="wrap" %}
```http
GET / HTTP/1.1
Host: example.com
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Connection: close
```
{% endcode %}

Le serveur le redirigera vers la version sécurisée :&#x20;

{% code overflow="wrap" %}
```http
HTTP/1.1 301 Moved Permanently
Content-Type: text/html
Content-Length: 178
Connection: close
Location: https://example.com/

<html>
  <head>
    <title>
      301 Moved Permanently
    </title>
  </head>
  <body bgcolor="white">
    <center>
      <h1>
        301 Moved Permanently
      </h1>
    </center>
    <hr>
      <center>
        nginx
      </center>
  </body>
</html>
```
{% endcode %}

Générant ainsi une seconde requête afin d'accéder, cette fois, à la ressource de façon sécurisée :&#x20;

{% code overflow="wrap" %}
```http
GET / HTTP/1.1
Host: example.com
Cache-Control: max-age=0
Sec-Ch-Ua: "Not_A Brand";v="99", "Google Chrome";v="109", "Chromium";v="109"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate
Accept-Language: fr-FR,fr;q=0.9,en-US;q=0.8,en;q=0.7
Connection: closehttp
```
{% endcode %}

Malheureusement, cette technique n'est pas infaillible et est sensible à plusieurs attaques telles que l'ARP/NDP spoofing ou la connexion à un hotspot Wi-Fi malveillant (DNS spoofing ou SSL stripping, par exemple). Dans de tels cas, l'attaquant s'intercale entre l'utilisateur et le site Web légitime pour forcer ce premier à rester sur une connexion non sécurisée (en HTTP), "annulant" ainsi la redirection prévue vers HTTPS et le redirigeant vers un site malveillant (qui peut ressembler au site légitime, par exemple).

## HTTP Strict Transport Security Header (HSTS)

HTTP Strict Transport Security (HSTS) est un mécanisme de sécurité qui permet de protéger les communications sur un site web en s'assurant que les données transitent toujours via une connexion sécurisée (HTTPS). Lorsqu'un navigateur web reçoit une directive HSTS, il mémorise la directive et s'assure que toutes les communications ultérieures avec l'application se font uniquement via HTTPS, même si l'utilisateur a saisi l'adresse ou a cliqué sur un lien en spécifiant l'utilisation du protocole HTTP. Il est important de noter que HSTS ne peut être utilisé que si le site web utilise déjà HTTPS, cet entête ne peut donc pas être présent en réponse à une requête si l'URL est de type `http://`.

HTTP Strict Transport Security Header (HSTS) est supporté par tous les navigateurs même ceux relativement anciens :&#x20;

<figure><img src="../../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

Avec ce nouveau fonctionnement, la problématique du premier échange lors d'une connexion à un site web en utilisant HTTP semble disparaitre. Enfin pas tout à fait, comme va le montrer la présence de la directive `preload`.

### Syntaxe

La syntaxe de l'entête est la suivante : &#x20;

```http
Strict-Transport-Security: max-age=<expire-time>
Strict-Transport-Security: max-age=<expire-time>; includeSubDomains
Strict-Transport-Security: max-age=<expire-time>; includeSubDomains; preload
```

### Directives

`max-age=<expire-time>`

Le temps pendant lequel un navigateur se souviendra de l'utilisation d'HTTPS pour accéder à un site est spécifié en secondes. La recommandation minimale est d'un an (31536000 secondes) mais il est préférable d'utiliser une durée de deux ans (63072000 secondes).

A noter que le navigateur n'attendra pas la fin de la durée d'expiration pour oublier la configuration HSTS. Le compteur du navigateur sera actualisé à chaque réponse HTTP du serveur web qui contient l'en-tête HSTS.

`includeSubDomains` _(optionnel)_

Si présent, la règle s'applique pour tous les sous-domaines de l'application.

`preload` _(optionnel) (non standard)_

Que se passe-t-il si un utilisateur visite le site de sa banque alors qu'il est connecté à un hotspot malveillant à l'aéroport ? Si cet utilisateur a récemment consulté ses comptes (et si le site implémente correctement l'entête de sécurité HSTS), le navigateur se rappellera qu'il est obligatoire d'utiliser HTTPS même lorsque le protocole spécifié est HTTP. L'attaquant ne sera pas en mesure d'utiliser les méthodes d'attaque décrites ci-dessus pour détourner la connexion de la victime.

Cependant, que se passera-t-il si c'est la première fois que l'utilisateur visite le site de sa banque (ou si cela fait plus de temps que la durée définie par la directive `max-age`) ? Le navigateur n'aura pas en mémoire l'utilisation exclusive de HTTPS et donc la victime verra sa connexion compromise par l'attaquant. C'est pour contrer cela, qu'existe la directive `preload`.

Google maintient une liste blanche qui indique que les sites inscrits ne doivent utiliser que le protocole HTTPS. Cette "liste de préchargement" est codée en dur dans Chrome et est également disponible pour d'autres navigateurs. Avant de se connecter à un site, le navigateur y vérifie la présence du site. Si tel est le cas, le navigateur utilisera HTTPS dès la première requête. Grâce à cela, l'utilisateur est maintenant protégé même dans le cas de la première connexion.

**Soumission du formulaire**

La soumission du formulaire s'effectue ici : [https://hstspreload.org/](https://hstspreload.org/). La validation est soumise à plusieurs prérequis :&#x20;

* Avoir un certificat valide
* Avoir une redirection HTTP vers HTTPS sur le même hôte (si écoute sur le port 80)
* Tous les sous-domaines, y compris _www_,  doivent être en HTTPS
* Inclure l'entête HSTS avec la configuration suivante : &#x20;
  * un `max-age` d'au moins un an (31536000 secondes)
  * présence de la directive `includeSubDomains`
  * présence de la directive `preload`
* Les redirections effectuées en HTTPS doivent également contenir l'entête HSTS

<figure><img src="../../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

La présence de la directive `preload` peut ne pas être évidente. En réalité, elle permet de vérifier que l'ajout du domaine à la "liste de préchargement" est bien souhaité par son propriétaire. Ainsi, une autre personne peut soumettre le formulaire, mais ne sera pas en mesure de modifier la configuration du serveur web pour y inclure cette directive. Dans ce cas, l'ajout du domaine à la "liste de préchargement" ne sera pas effectué.

### Votre connexion n'est pas privée

Il peut arriver en naviguant sur certains sites de rencontrer des erreurs liées au certificat TLS. L'erreur rencontrée peut varier mais ressemble à ceci :

<figure><img src="../../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

Le site web https://poclab1.com ne possède pas l'entête de sécurité HSTS, il est donc possible pour l'utilisateur de forcer la navigation malgré le message d'avertissement (et donc d'accepter les risques encourus). Par contre, comme indiqué dans le message d'avertissement, cette possibilité n'est pas présente pour les sites mettant en place HSTS :&#x20;

<figure><img src="../../../.gitbook/assets/image (90).png" alt=""><figcaption></figcaption></figure>

### Cas particulier de la redirection vers un sous-domaine

Il est important de noter que le mécanisme HSTS ne s'applique qu'au domaine qui répond à la demande ou, si la directive `includeSubDomains` est présente, à tous ses sous-domaines. En revanche, la politique ne s'appliquera en aucun cas au domaine parent.

En admettant le scénario suivant :

Pour certaines raisons, l'entreprise possédant le domaine [example.com](http://exemple.com) souhaite plutôt rediriger ses utilisateurs vers son sous-domaine [www.exemple.com](http://www.example.com). Afin d'appliquer cette politique, l'entreprise va donc mettre en place la redirection adéquate :&#x20;

<figure><img src="../../../.gitbook/assets/image (294).png" alt=""><figcaption></figcaption></figure>

L'entreprise met correctement en place l'entête HSTS en réponse, que cela soit sur une réponse HTTPS pour [example.com](https://example.com) :&#x20;

```http
HTTP/1.1 200 OK
Content-Type: text/html
Strict-Transport-Security: max-age=63072000; includeSubDomains
Connection: close
```

ou pour [www.example.com](https://www.example.com) :&#x20;

```http
HTTP/1.1 200 OK
Content-Type: text/html
Strict-Transport-Security: max-age=63072000; includeSubDomains
Connection: close
```

Bien que ce soit toujours mieux que rien, cette implémentation n'est pas parfaite. Lorsqu'un utilisateur visite le site [http://example.com](http://example.com), il est redirigé vers [https://www.example.com](https://www.example.com) qui indiquera sa politique HSTS au navigateur. Si, plus tard, l'utilisateur visite le lien [http://www.example.com](http://www.example.com), la requête sera automatiquement transformée en HTTPS. Cependant, cela ne s'applique seulement qu'à [www.example.com](https://www.example.com) et ses sous domaines. Cela signifie que si l'utilisateur visite à nouveau [http://example.com](http://example.com) HSTS ne sera pas actif pour cette requête et la politique HSTS pour [https://example.com](https://example.com) ne sera jamais connu par le navigateur.

{% hint style="info" %}
Ne pas oublier que la politique HSTS est appliquée seulement dans le cadre d'une réponse HTTPS et non pour une réponse HTTP.
{% endhint %}

En général, il est alors conseillé de toujours rediriger vers le même domaine (afin d'appliquer HSTS), quitte à effectuer une seconde redirection pour rediriger l'utilisateur vers le sous-domaine "officiel" de l'entreprise :&#x20;

<figure><img src="../../../.gitbook/assets/image (293).png" alt=""><figcaption></figcaption></figure>

## Références

* [https://fr.wikipedia.org/wiki/HTTP\_Strict\_Transport\_Security](https://fr.wikipedia.org/wiki/HTTP\_Strict\_Transport\_Security)
* [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)
