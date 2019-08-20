# Erreurs d'implémentation

Il arrive régulièrement que des standards de sécurité puisse être contournés à cause de leurs mauvaises implémentations lors du développement d'un produit. Ci-dessous quelques exemples de telles vulnérabilités \(liste loin d'être exhaustive\) :

## Edge bypass \([CVE-2017-0002](https://www.cvedetails.com/cve/CVE-2017-0002/)\)

_**Versions affectées**_

Microsoft Edge

_**Description**_

Une mauvaise implémentation rend possible un contournement de SOP sur Edge lors de l'utilisation de `about:blank` et les URLs de type `data:`.

## Firefox bypass \([CVE-2015-7188](https://www.cvedetails.com/cve/CVE-2015-7188/)\)

_**Versions affectées**_

Mozilla Firefox &lt; 42.0 et Firefox ESR 38.x avant 38.4

_**Description**_

Ces version de Firefox permettent de contourner la politique de même origine lorsque l'origine est représenté grâce à une adresse IP

## Java bypass \([CVE-2010-3573](https://www.cvedetails.com/cve/CVE-2010-3573/)\)

_**Versions affectées**_

Java 1.6u45 et Java 1.7u17

_**Description**_

Les applets Java exécutés avec les versions vulnérables permettent un contournement de la politique de sécurité dans le cas ou les origines sont résolus par la même adresse IP \(dans le cadre par exemple d'ub hébergement mutualisé\).



## 



