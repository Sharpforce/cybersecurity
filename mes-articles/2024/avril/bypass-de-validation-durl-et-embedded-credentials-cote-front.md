---
description: 08 avril 2024
---

# Bypass de validation d'URL et embedded credentials côté front

Lors de la recherche ou de l'exploitation de vulnérabilités SSRF (Server-Side Request Forgery), il n'est pas rare de se confronter à une validation de l'URL par le serveur lors de la soumission de la requête. Les expressions régulières sont fréquemment utilisées pour effectuer cette vérification. Ainsi, l'expression régulière suivante vérifie que l'URL renseignée correspond bien à celle autorisée, soit ici [https://example.com](https://example.com) :&#x20;

```
^https:\/\/example\.com
```

Le contournement d'une telle expression régulière est assez trivial grâce à l'utilisation des embedded credentials :

```
https://example.com@evil.com
```

Lors de l'audit d'une application web, une fonctionnalité permettait l'édition et la consultation de documents HTML accessibles à un ensemble de contributeurs et de lecteurs. Cette fonctionnalité permettait également l'intégration de vidéos Youtube, dont l'utilisateur devait en fournir les URL. Lorsque la soumission de l'URL était acceptée, le contenu était finalement intégré dans une iframe au sein du document. Lors du renseignement de cette URL, une validation était effectuée par le serveur afin de vérifier que l'URL corresponde bien à celle de Youtube (par exemple, https://www.youtube.com).

Un contournement était possible grâce à l'utilisation des embedded credentials, mais étant donné que l'URL était insérée dans un élément iframe, et donc finalement chargée par le navigateur lors de la consultation du document, <img src="../../../.gitbook/assets/image (1).png" alt="" data-size="line">Google Chrome n'autorise pas le chargement d'une telle ressource (et ce depuis la version 59) :

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
Le même comportement a été identifié sur <img src="../../../.gitbook/assets/image (2).png" alt="" data-size="line">Microsoft Edge ainsi que sur <img src="../../../.gitbook/assets/image (3).png" alt="" data-size="line">Chromium.
{% endhint %}

Toutefois, cela reste possible à minima avec <img src="../../../.gitbook/assets/image (4).png" alt="" data-size="line">Mozilla Firefox (testé en 124.0.1) qui autorise bien le chargement de l'iframe. Cela permet donc de profiter du contournement possible afin d'exécuter des actions nécessitant la visite d'une page spécifique de la part de la victime : click-jacking, cross-site request forgery (CSRF) ou plus généralement du phishing.

Sans avoir poussé les recherches plus loin, ce comportement semble être également présent avec les balises `<script></script>` ou `<link>` par exemple, mais une vérification de type est effectuée sur la ressource ainsi chargée.
