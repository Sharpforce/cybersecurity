# Silverlight et le fichier clientaccess-policy.xml

Silverlight est une technologie développée par Microsoft qui permet de créer des application web riches \(RIA pour Rich Internet Application\). Il s'agit d'une alternative à Flash et répond aux même problématiques concernant SOP.

## Application Silverlight

L'application Silverlight permettant de tester les requêtes cross-origin utilisée dans cet article est disponible sur [GitHub](https://github.com/nccgroup/CrossSiteContentHijacking). Elle est développée par "nccgroup" :

![](../../.gitbook/assets/e2632be5625fef64d9a1ff52e3e7dd5f.png)

Elle est facilement réutilisable en téléchargeant le dossier `ContentHijacking` et en appelant \(via le navigateur\) le fichier `index.html`.

## Fichier clientaccesspolicy.xml

De la même façon que Flash, le fichier `clientaccesspolicy.xml` est un fichier qui permet de spécifier les domaines autorisés à accéder aux ressources hébergées sur un domaine particulier, par exemple ici pour le domaine `example.com` :

```markup
<access-policy>
  <cross-domain-access>
    <policy>
      <allow-from>      
        <domain uri="http://example.com"/>
      </allow-from>      
      <grant-to>      
        <resource path="/" include-subpaths="true"/>
      </grant-to>      
    </policy>
  </cross-domain-access>
</access-policy>
```

{% hint style="info" %}
Contrairement au fichier utilisé par Flash, le nom de domaine doit s'inscrire comme étant une URI \(schéma + domaine + port\).
{% endhint %}

Il est possible d'autoriser les requêtes provenant de tous les domaines grâce au joker :

```markup
<access-policy>
  <cross-domain-access>
    <policy>
      <allow-from>      
        <domain uri="*"/>
      </allow-from>      
      <grant-to>      
        <resource path="/" include-subpaths="true"/>
      </grant-to>      
    </policy>
  </cross-domain-access>
</access-policy>
```

### Autorisation d'un domaine

Lorsque le site détenteur de la ressource désirée possède un fichier `clientaccesspolicy.xml` bien configuré, par exemple en autorisant seulement le domaine `otherdomain.com` :

```markup
<access-policy>
  <cross-domain-access>
    <policy>
      <allow-from>      
        <domain uri="http://otherdomain.com"/>
      </allow-from>      
      <grant-to>      
        <resource path="/" include-subpaths="true"/>
      </grant-to>      
    </policy>
  </cross-domain-access>
</access-policy>
```

Alors seul le domaine autorisé peut requêter la ressource :

![](../../.gitbook/assets/bcb84a0eeaa443da649b83ce6cf91156.png)

Les requêtes effectuées pendant cet échange indiquent bien la récupération du fichier `clientaccesspolicy.xml` suivi de la requête vers la page de profil :

![](../../.gitbook/assets/98e578ad9116750bc6c940375506bfd5.png)

Si la requête provenait non pas du domaine principal mais d'un sous-domaine, alors la requête vers la ressource serait rejetée. Ici un exemple avec le domaine `poc1.otherdomain.com` :

![](../../.gitbook/assets/9deebba4282a9adf031a5e2ab2a59556.png)

### Autorisation d'un sous-domaine

Il est également possible de n'autoriser seulement qu'un sous-domaine spécifique, par exemple `poc1.otherdomain.com` :

```markup
<access-policy>
  <cross-domain-access>
    <policy>
      <allow-from>      
        <domain uri="http://poc1.otherdomain.com"/>
      </allow-from>      
      <grant-to>      
        <resource path="/" include-subpaths="true"/>
      </grant-to>      
    </policy>
  </cross-domain-access>
</access-policy>
```

Alors seul le sous-domaine spécifié peut accéder à la ressource :

![](../../.gitbook/assets/180eda4e3ca28da74f1bfecc440046c0.png)

Par contre il n'est pas possible pour un autre sous-domaine \(`poc2.otherdomain.com`\) d'accéder à la ressource :

![](../../.gitbook/assets/f9451a9264127d27c46876c1ef597716.png)

Ni même au domaine principal `otherdomain.com` :

![](../../.gitbook/assets/f0d37a38f688fb419cb434a7dd245191.png)

{% hint style="info" %}
Contrairement à Flash, la requête du domaine principal serait également rejetée si le fichier de politique autorisait tous les sous domaines :

```markup
<domain uri="http://*.otherdomain.com"/>
```
{% endhint %}

### Absence du fichier

**Requête cross-domain**

Lorsqu'un domaine \(`otherdomain.com`\) tente d'accéder à une ressource \(hébergée sur `cybersecurity.com`\) via Silverlight mais que le fichier `clientaccesspolicy.xml` est absent, alors l'application va tenter de récupérer le fichier `crossdomain.xml` \(il s'agit du fichier utilisé par Flash\). En effet, Silverlight peut s'appuyer également sur ce fichier pour fonctionner \(cela a pour avantage de ne pas avoir à maintenir deux fichiers distincts lorsque le site accepte Flash et Silverlight\) :

![](../../.gitbook/assets/32f58a86670209518d6c41ef2aa40b09.png)

{% hint style="info" %}
Cette technique semble avoir quelques limites car Silverlight acceptera la requête seulement si le fichier `crossdomain.xml` autorise tous les domaines \(grâce au symbole "\*"\)
{% endhint %}

**Requête même domaine mais de port différent**

Une origine se calcul à en se basant sur le domaine, le port ainsi que le schéma. Lorsque la requête s'effectue sur un port différent, alors la requête est considérée comme étant d'origine différente \(respect du principe de SOP\) :

![](../../.gitbook/assets/4a97ce9e0e1d46afce74e0f37514964f.png)

Cela implique donc la recherche du fichier `clientaccesspolicy.xml` :

![](../../.gitbook/assets/e23cdfdc654f9a6ec81dc552c608d9af.png)

**Requête même domaine mais de schéma différent**

Lorsque le schéma est différent \(que cela soit `HTTP` vers `HTTPS` ou inversement\) la ressource n'est pas récupérée auprès de l'hébergeur **:**

![](../../.gitbook/assets/5cdbacd8e8945171180b8b4b94c894a5.png)

La raison est que le fichier de politique est d'abord recherché ce qui signifie que Silverlight considère la requête comme étant une requête cross-origin :

![](../../.gitbook/assets/4cd308c95218543ad5cdcc7fdcb56af5.png)

## Entête Origin

A l'instar de Flash, les requêtes cross-origin provenant d'une application Silverlight ne contiennent pas l'entête `Origin` :

![](../../.gitbook/assets/b28f81f10901419dfc914dceaceb99b3.png)

## Problématiques de sécurité

La problématique est ici similaire à celle de Flash. En effet, il est possible d'autoriser tous les domaines à effectuer des requêtes vers la ressource :

```markup
<access-policy>
  <cross-domain-access>
    <policy>
      <allow-from>      
        <domain uri="*"/>
      </allow-from>      
      <grant-to>      
        <resource path="/" include-subpaths="true"/>
      </grant-to>      
    </policy>
  </cross-domain-access>
</access-policy>
```

Cela peut poser les mêmes problèmes de fuite de données privées en s'appuyant sur des vulnérabilités de type CSRF.

Il est également possible de n'autoriser seulement les sous-domaines d'un domaine spécifique. Cela réduit le risque d'exploitation d'une vulnérabilité CSRF mais ne l'annule pas :

```markup
<access-policy>
  <cross-domain-access>
    <policy>
      <allow-from>      
        <domain uri="http://*.otherdomain.com"/>
      </allow-from>      
      <grant-to>      
        <resource path="/" include-subpaths="true"/>
      </grant-to>      
    </policy>
  </cross-domain-access>
</access-policy>
```

Concernant le respect de SOP, Silverlight améliore un peu les choses en considérant les requêtes d'un même domaine et même schéma mais de port différent comme étant une requête cross-origin \(contrairement à Flash\).



