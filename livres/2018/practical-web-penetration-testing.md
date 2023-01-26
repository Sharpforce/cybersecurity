# Practical Web Penetration Testing

> Ma critique du livre "Practical Web Penetration Testing" de Gus Khawaja, paru en 2018.

## Description du livre

![](<../../.gitbook/assets/image (17).png>)

**Titre :** Practical Web Penetration Testing\
**Auteur :** Gus Khawaja\
**Dernière parution :** 22 juin 2018\
**Langue :** Anglaise\
**Prix (ebook) :** 24,99 $\
**Lien :** [https://www.packtpub.com/product/practical-web-penetration-testing/9781788624039](https://www.packtpub.com/product/practical-web-penetration-testing/9781788624039)

**Résumé / Présentation :** `Companies all over the world want to hire professionals dedicated to application security. Practical Web Penetration Testing focuses on this very trend, teaching you how to conduct application security testing using real-life scenarios.`&#x20;

`To start with, you’ll set up an environment to perform web application penetration testing. You will then explore different penetration testing concepts such as threat modeling, intrusion test, infrastructure security threat, and more, in combination with advanced concepts such as Python scripting for automation. Once you are done learning the basics, you will discover end-to-end implementation of tools such as Metasploit, Burp Suite, and Kali Linux. Many companies deliver projects into production by using either Agile or Waterfall methodology. This book shows you how to assist any company with their SDLC approach and helps you on your journey to becoming an application security specialist.`

`By the end of this book, you will have hands-on knowledge of using different tools for penetration testing.`



**Table des matières :**

1. Building a Vulnerable Web Application Lab
2. Kali Linux Installation
3. Delving Deep into the Usage of Kali Linux
4. All About Using Burp Suite
5. Understanding Web Application Vulnerabilities
6. Application Security Pre-Engagement
7. Application Threat Modeling
8. Source Code Review
9. Network Penetration Testing
10. Web Intrusion Tests
11. Pentest Automation Using Python

## Mon avis

**Pratical Web Penetration Testing** est un livre que je ne trouve pas assez utile, même pour un débutant. La première partie, soit les chapitres 1 à 4, concerne la mise en place d'un laboratoire de test avec l'installation d'une Kali en machine virtuelle, de l'application vulnérable de l'OWASP [Mutillidae](https://github.com/webpwnized/mutillidae) ainsi que le proxy Burp. Cette partie s'établie sur 120 pages (pour un total d'un peu plus de 400). Je trouve cela un peu trop, surtout que toutes les explications sont très facilement accessibles sur Internet. Cela peut toutefois être compréhensible si l'auteur souhaite toucher un public très débutant.

Vient la seconde partie qui concerne les vulnérabilités Web, qui s'établie sur seulement 47 pages. Sont décrites très succinctement les vulnérabilités LFI/RFI, XSS, CSRF, SQLi ainsi que les Command Injection. Un petit aperçu du [Top Ten OWASP 2017](https://owasp.org/www-project-top-ten/2017/) est également présent avec pour chaque item une description de quelques lignes. Je trouve cela très très léger et vraiment trop basique. A moins que le lecteur n'ai jamais entendu parler de sécurité Web, il n'apprendra pas grand chose en si peu de lignes.

La troisième partie, une trentaine de pages, est pour moi incompréhensible. Elle cible les freelanceurs (ou chefs d'équipes) qui veulent se lancer dans les tests d'intrusions pour ses clients. L'auteur explique comment rédiger un contrat, définir le périmètre de la prestation, diriger les réunions avec les clients, etc. Je ne vois pas l'intérêt pour un débutant ne connaissant que le contenu de ce livre de se lancer dans ce genre de chantiers. Pour les chefs d'équipes, autant demander un coup de main aux membres de son équipe, ça sera toujours plus direct et plus contextuel.

La quatrième partie concerne le Threat Modeling. C'est un peu le même raisonnement que je tiens ici. Cela peut être intéressant pour un débutant d'avoir un aperçu de ce domaine, mais cette partie concerne 52 pages du livre soit plus que celle concernant la compréhension des vulnérabilités Web.

La dernière partie est un mélange d'un peu de tout. On y retrouve des guidelines de revue de code ou de pentest, sans trop d'explications ou de mises en situation, de l'audit réseau/services, l'introduction du scoring CVSS puis l'automatisation de pentest en utilisant Python. J'ai passé très rapidement sur cette partie, trop brouillonne selon moi. Certaines informations sont pourtant intéressantes et pertinentes, mais noyées dans la masse.

En conclusion, ce n'est donc pas un livre que je recommande, même pour les débutants. L'auteur ne creuse pas assez la partie sur les vulnérabilités Web, et passe trop de temps sur des sujets pas encore intéressants pour l'auditoire ciblé (Threat Modeling, contrat avec les entreprises).
