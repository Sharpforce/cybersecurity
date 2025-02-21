# Hacking APIs - Breaking Web Application Programming Interfaces

> Mon avis personnel sur le livre _Hacking APIs - Breaking Web Application Programming Interfaces_ de Corey J. Ball, paru en 2022

## Description du livre

<figure><img src="../../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

**Titre :** Hacking APIs - Breaking Web Application Programming Interfaces\
**Auteur :** Corey J. Ball\
**Dernière parution :** 12 juillet 2022\
**Langue :** Anglaise\
**Prix :** 45,58 $\
**Lien :** [https://nostarch.com/hacking-apis](https://nostarch.com/hacking-apis)

**Résumé / Présentation :** `An Application Programming Interface (API) is a software connection that allows applications to communicate and share services. Hacking APIs will teach you how to test web APIs for security vulnerabilities. You’ll learn how the common API types, REST, SOAP, and GraphQL, work in the wild. Then you’ll set up a streamlined API testing lab and perform common attacks, like those targeting an API’s authentication mechanisms, and the injection vulnerabilities commonly found in web applications.`

**Table des matières :**&#x20;

* Foreword
* Acknowledgments
* Introduction
* PART I: HOW WEB API SECURITY WORKS
  * Chapter 0: Preparing for Your Security Tests
  * Chapter 1: How Web Applications Work
  * Chapter 2: The Anatomy of Web APIs
  * Chapter 3: Common API Vulnerabilities
* PART II: BUILDING AN API TESTING LAB
  * Chapter 4: Your API Hacking System
  * Chapter 5: Setting Up Vulnerable API Targets
* PART III: ATTACKING APIs
  * Chapter 6: Discovery
  * Chapter 7: Endpoint Analysis
  * Chapter 8: Attacking Authentication
  * Chapter 9: Fuzzing
  * Chapter 10: Exploiting Authorization
  * Chapter 11: Mass Assignment
  * Chapter 12: Injection
* PART IV: REAL-WORLD API HACKING
  * Chapter 13: Applying Evasive Techniques and Rate Limit Testing
  * Chapter 14: Attacking GraphQL
  * Chapter 15: Data Breaches and Bug Bounties
* Conclusion
* Appendix A: API Hacking Checklist
* Appendix B: Additional Resources Index

## Mon avis

Il s'agit, selon moi, d'un bon livre qui respecte sa promesse, celle de donner un bon aperçu au lecteur des problématiques de sécurité des API. La première partie, d'environ 70 pages, est une introduction aux fonctionnements des API ainsi qu'une description brève des vulnérabilités les plus fréquentes. Si vous êtes familier avec ce sujet, vous n'apprendrez sans doute rien ici, mais un petit rappel ne fait pas de mal, surtout que cette partie se lit très facilement.

La seconde partie, d'environ 40 pages, est moins agréable à lire. Il s'agit de la mise en place d'un lab de test dédié aux API. On va y installer un Kali avec quelques outils, tels que Postman pour créer des collections d'API vulnérables, Wfuzz, Burp, etc. En plus de l'installation, l'auteur donne quelques instructions pour les utiliser (un peu trop parfois ?). Les machines vulnérables seront installées à travers Docker, et on y trouvera  [Completely ridiculous API (crAPI)](https://github.com/OWASP/crAPI), [OWASP DevSlop's Pixi](https://github.com/DevSlop/Pixi), [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/) ainsi que [Damn Vulnerable GraphQL Application](https://github.com/dolevf/Damn-Vulnerable-GraphQL-Application). Après avoir passé cette étape plutôt pénible, on se dit : "C'est bon, je suis enfin prêt".

La troisième partie est celle qui est tant attendue : on passe à l'attaque (nous sommes presque à la moitié du livre à ce stade). On commence par une partie reconnaissance, qui n'est pas nécessairement liée aux API, car il s'agit de l'utilisation de Nikto, GoBuster, d'Amass, en passant par un peu de blabla sur Shodan, le Google dorking, la recherche sur Github, etc. Suit les vulnérabilités concernant l'authentification, l'autorisation, le fuzzing, le mass assignment, en finissant par les injections. Il n'y a pas vraiment de gros problèmes avec cette partie (à part la reconnaissance que je trouve un peu longue et ennuyeuse à lire et trop développée par rapport aux autres chapitres), mais avec tous les outils et les machines vulnérables installées, je m'attendais à plus qu'un "exercice" par chapitre. Ne vous attendez pas à ce que ce livre vous aide à poncer ces labs dans les moindres recoins, ce n'est absolument pas le cas.

La quatrième et dernière partie est plaisante à lire. Il s'agit de cas réels d'exploitation de faiblesses dans les API ou des remontées de bug bounty. Il n'y a rien de particulier à dire ici, c'est intéressant, même si on oubliera probablement ces exemples dans quelques jours, ou au mieux, dans quelques semaines.

Un livre intéressant, surtout pour les débutants, mais mal amené sur plusieurs points : trop de longueur sur certaines parties et pas assez de pratique.
