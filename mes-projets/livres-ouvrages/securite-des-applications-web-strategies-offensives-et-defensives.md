---
description: >-
  Mon retour d'expérience concernant la rédaction du livre "Sécurité des
  applications web - Stratégies offensives et défensives" paru aux Editions ENI.
---

# Sécurité des applications web - Stratégies offensives et défensives

## Description du livre

<figure><img src="../../.gitbook/assets/image (338).png" alt=""><figcaption></figcaption></figure>

**Titre :** Sécurité des applications web - Stratégies offensives et défensives

**Dernière parution :** juin 2024

**Langue :** Française

**Prix (minimum) :** 54,00 €

**Lien :** [https://www.editions-eni.fr/livre/securite-des-applications-web-strategies-offensives-et-defensives-9782409045127](https://www.editions-eni.fr/livre/securite-des-applications-web-strategies-offensives-et-defensives-9782409045127)

**Résumé / Présentation :** `Cet ouvrage explore en profondeur la sécurité des applications web, offrant une expertise complète aux développeurs, professionnels de la cybersécurité ou passionnés du numérique qui désirent comprendre et maîtriser les techniques d’attaque et de défense. À travers des études de cas, des exemples, des conseils, des astuces et des exercices pratiques, ce livre offre une expérience d’apprentissage immersive et enrichis­sante et dresse un panorama complet des vulnérabilités web les plus courantes...`

**Table des matières :**

1. Avant-propos
2. Introduction à la sécurité applicative
3. Définition du concept de vulnérabilité
4. Fonctionnement d'une application web
5. L'évolution du Web
6. Mise en place du laboratoire
7. Les principales vulnérabilités web
8. Autres vulnérabilités applicatives
9. Les principaux en-têtes HTTP de sécurité
10. Introduction à la cryptographie
11. Fonctionnement de TLS
12. Introduction au DevSecOps
13. Les prestations en entreprise

## Contexte et Motivation

Au fil du temps, j'ai réalisé que la sécurité des applications web est un domaine à la fois vaste et complexe. Le désir de partager mes connaissances a été ma principale motivation. Selon moi, il existe encore trop peu de ressources francophones sur le sujet, et j'ai voulu contribuer à combler ce manque.

J'ai également constaté que beaucoup de développeurs n'acquièrent pas de solides bases en sécurité des applications web, que ce soit durant leurs études ou plus tard dans leur carrière professionnelle. C'est pourquoi j'ai voulu que ce livre s'adresse aussi à eux. En parallèle, je souhaitais également améliorer la compréhension défensive des pentesters, afin qu'ils puissent mieux conseiller les équipes clientes lors des restitutions des rapports d'audits, notamment en ce qui concerne les remédiations. Pour y parvenir, j'ai intégré un volet "défensif" de la sécurité, en expliquant comment développer des applications de manière sécurisée, avec une approche pratique en corrigeant des codes vulnérables.

Pour approfondir cet aspect "défensif", j'ai également voulu inclure une section sur les en-têtes HTTP de sécurité, qui sont mentionnés dans d'autres ouvrages mais, selon moi, rarement expliquées de manière suffisamment détaillée. En complément, et toujours dans l'optique de renforcer le côté "défensif", j'estime qu'il est aujourd'hui indispensable de comprendre le fonctionnement de la chaîne d'intégration continue ainsi que les différents outils orientés sécurité pouvant s'y intégrer.

## Code source de l'application

**Pourquoi avoir développé une nouvelle application ?**

Je lis de nombreux livres sur la sécurité des applications web, en français comme en anglais, et je remarque souvent que les mêmes applications de test reviennent : bWAPP, DVWA ou encore WebGoat. Pour me démarquer, j'ai voulu proposer une application entièrement nouvelle, spécifiquement conçue pour cet ouvrage. De plus, comme mon objectif était que le lecteur puisse corriger le code source vulnérable, il m’a semblé plus simple de développer une application sur mesure plutôt que d’adapter le code d’un projet déjà existant. J'ai également voulu que le code reste simple, afin que les lecteurs puissent le comprendre, le modifier et l’adapter à leurs besoins sans être contraints par l'utilisation de frameworks de développement spécifiques.

**Pourquoi une application en PHP ?**

Principalement par commodité. C'est le langage avec lequel je peux développer le plus rapidement, et étant donné le temps limité pour la rédaction de l'ouvrage, c'était la solution la plus pratique pour moi. La plupart des exploitations des vulnérabilités décrites dans le livre ne dépendent pas de la technologie utilisée. Cependant, j'aurais souhaité inclure certains exercices en NodeJS pour diversifier le contenu et les corrections à apporter, mais cela aurait été trop chronophage dans le délai imparti.

**Projet GitHub**

L'application accompagnant le livre est disponible sur le dépôt des Éditions ENI :[https://github.com/EditionsENI/Livre-Securite-applications-web-Strategies-offensives-defensives](https://github.com/EditionsENI/Livre-Securite-applications-web-Strategies-offensives-defensives).

Sur mon GitHub personnel, je mets à disposition l'application sous forme de conteneur Docker au lieu d'une machine virtuelle type Kali Linux :[https://github.com/Sharpforce/Livre-Securite-applications-web-Strategies-offensives-defensives](https://github.com/Sharpforce/Livre-Securite-applications-web-Strategies-offensives-defensives).

## Errata et Corrections

Malheureusement, il est difficile d’écrire un livre sans laisser quelques erreurs derrière soi. Même avec toute l'attention du monde, certaines fautes peuvent passer inaperçues lors du processus d'écriture et de relecture. Que ce soit des coquilles, des petites incohérences ou des maladresses de style, ces imperfections font partie du parcours de tout auteur.

Dans cette section, je vais lister les erreurs qui se sont discrètement glissées dans mon livre et que j'ai découvertes par la suite.

**Les wrappers PHP (page 242)**

&#x20;Le wrapper `php://filter` ne nécessite pas de de données au niveau du corps de la requête :&#x20;

```
GET /wrappers.php?file=php://filter/read=/resource=file:///etc/passwd HTTP/1.1
Host: 192.168.56.122
Connection: close
```

Au lieu de :&#x20;

```
GET /wrappers.php?file=php://filter/read=/resource=file:///etc/passwd HTTP/1.1
Host: 192.168.56.122
Connection: close
Content-Length: 22

<?php system('whoami'); ?>
```

**Object manipulation (page 350)**

L'étape 4 indique de supprimer le profil "PHPSESSID" alors qu'il s'agit plutôt de la suppression du cookie "PHPSESSID".

**Regular expression denial of service (page 360)**

L'expression régulière contient un signe `+` afin de traiter les sous-domaines, mais l'explication affiche un caractère `*` (la définition est par contre correcte) :&#x20;

– \* : indique que le motif précédent peut se répéter une fois ou plus, autorisant ainsi les URL possédant un, ou plusieurs sous-domaines.

Devient :&#x20;

– + : indique que le motif précédent peut se répéter une fois ou plus, autorisant ainsi les URL possédant un, ou plusieurs sous-domaines.

**Déni de service (page 381)**

L'attaque XXE de type lol flow est correcte mais l'inclusion est effectuée par l'entité `lol6` alors qu'elle devrait être faite par l'entité `lol14` :&#x20;

```
<!-- Inclusion malicieuse de l'entité lol6 -->
&lol6;
```

Devient :&#x20;

```
<!-- Inclusion malicieuse de l'entité lol14 -->
&lol14;
```
