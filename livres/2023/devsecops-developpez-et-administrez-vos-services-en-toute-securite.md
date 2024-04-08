# DevSecOps - Développez et administrez vos services en toute sécurité

> Mon retour sur le livre "DevSecOps - Développez et administrez vos services en toute sécurité" de Jordan Assouline, paru en 2023

## Description du livre

<figure><img src="../../.gitbook/assets/image (225).png" alt=""><figcaption></figcaption></figure>

**Titre :** DevSecOps - Développez et administrez vos services en toute sécurité\
**Auteur :** Jordan Assouline\
**Dernière parution :** 22 mai 2023\
**Langue :** Francçaise\
**Prix (minimum) :** 54,00 €\
**Lien :** [https://www.editions-eni.fr/livre/devsecops-developpez-et-administrez-vos-services-en-toute-securite-9782409039188](https://www.editions-eni.fr/livre/devsecops-developpez-et-administrez-vos-services-en-toute-securite-9782409039188)

**Résumé / Présentation :** `Ce livre sur DevSecOps s'adresse à tous les membres des équipes IT ou de développement qui souhaitent intégrer la sécurité dans leur pratique quotidienne et sécuriser leurs développements à chaque étape du cycle de vie des services.`

**Table des matières :**&#x20;

1. Avant-propos
2. Chapitre 1 - Introduction au DevSecOps
3. Chapitre 2 - Intégration et déploiement continus (CI/CD)
4. Chapitre 3 - Utilisation de Docker en DevSecOps
5. Chapitre 4 - Utilisation de Kubernetes en DevSecOps
6. Chapitre 5 - Culture et connaissances en cybersécurité
7. Chapitre 6 - Sécurité du développement et bonnes pratiques
8. Chapitre 7 - Implémenter le DevSecOps en entreprise
9. Chapitre 8 - Analyse de sécurité en DevSecOps

## Mon avis

Le livre a pour objectif de fournir au lecteur suffisamment de connaissances afin de permettre la mise en œuvre du DevSecOps au sein de son entreprise. Le premier chapitre est une introduction au concept de DevSecOps. Pour ce faire, l'auteur commence par la culture DevOps et explique les évolutions qui ont conduit à ce nouveau concept qui vise à y intégrer la sécurité.

Le second chapitre concerne les chaînes d'intégration et de déploiement continus (CI/CD), en commençant le tour d'horizon par l'utilitaire bien connu Git puis la plateforme DevSecOps GitLab.&#x20;

Docker est sans doute la technologie de conteneurisation la plus connue. Elle est très utilisée par les DevOps en entreprise. Le troisième chapitre, qui lui est consacrée, est très bien écrit et le fonctionnement de Docker est très bien expliqué. On passe par un peu de théorie sur la conteneurisation et les machines virtuelles, puis à l'explication des principales commandes de Docker avec quelques étapes pratiques.&#x20;

Le chapitre quatre est une suite logique à celui sur Docker puisqu'il s'agit de Kubernetes. Kubernetes est un orchestrateur de conteneurs qui va permettre de faciliter les déploiements, la mise à l'échelle, etc. Je n'ai pas trouvé ce chapitre très plaisant à lire. Les informations sont là mais je trouve qu'on s'y perd avec tous les termes (pods, clusters, nodes, etc) et l'auteur n'arrive pas à y mettre assez d'ordre pour que la lecture soit fluide.

Le chapitre cinq et six ne sont nécessaires que pour les personnes ayant très peu de connaissances en cybersécurité. Le chapitre cinq propose un petit tour d'horizon sur les principaux concepts liés à la sécurité : la triade CIA, les attaques informatiques (phishing, force brute, reconnaissance, DoS) avec pour finir un peu de cryptographie. On y retrouve ensuite l'application développée par l'OWASP, WebGoat, dont le lecteur est invité à suivre les différentes leçons dans le chapitre six. On termine par une partie sur la journalisation, l'utilisation de Prometheus ainsi que de Grafana.

Le chapitre sept propose quelques sujets intéressants liés au DevSecOps : la mise en place des Security Champions, la sensibilisation à la cybersécurité aux personnes non techniques de l'entreprise, un bref aperçu du secret scanning, ainsi que l'utilisation du Container Registry de GitLab. Une dernière partie concerne la sécurisation des serveurs HTTP Nginx et Apache en mettant en place quelques entêtes de sécurité. Un scan Nikto est ensuite intégré dans la CI afin de valider cela.

Le dernier chapitre est en fait le seul vraiment intéressant selon moi et qui répond véritablement à l'objectif du livre. On y retrouve les sujets de sécurité à mettre en place dans les CI/CD avec le SCA, le SAST, le DAST et l'IAC Scanning. La seconde partie de ce chapitre est une mise en pratique avec un projet de test (mis en place lors des premiers chapitres). Pourquoi pas, mais j'ai l'impression qu'il y a plus de contenu de fichiers de configuration ou de sortie de commandes que d'explications, un peu dommage.

C'est un livre qui traite d'un sujet d'actualité et qui concerne beaucoup d'entreprises de l'IT. Certains chapitres sont très agréables à lire : Docker, l'implémentation et l'analyse de sécurité en DevSecOps. D'autres, en revanche, le sont clairement moins : la culture de la cybersécurité et un énième tour sur WebGoat. J'aurais préféré un chapitre huit beaucoup plus étoffé, abordant par exemple l'aspect communication des résultats des différents outils aux équipes de développement ou opérationnelles, la gestion des faux positifs des outils, ainsi que le reporting possible à l'intention des managers auxquels les équipes doivent généralement rendre des comptes afin de démontrer l'intérêt apporté par le DevSecOps.
