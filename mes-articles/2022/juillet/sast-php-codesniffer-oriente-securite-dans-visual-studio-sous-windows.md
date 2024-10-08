---
description: 12 Juillet 2022
---

# SAST - PHP CodeSniffer orienté sécurité dans Visual Studio (sous Windows)

> **PHP Code Sniffer** est un outil qui permet d'analyser le code source PHP d'une application pour détecter des mauvaises pratiques de développement en se basant sur un ensemble de règles. Ces règles concernent principalement la qualité du code, mais il est également possible d'en ajouter pour détecter des failles de sécurité.

## Installation

1. La première étape est de récupérer l'archive PHP PHAR disponible à l'URL [https://squizlabs.github.io/PHP\_CodeSniffer/phpcs.phar](https://squizlabs.github.io/PHP\_CodeSniffer/phpcs.phar).&#x20;
2. La déposer dans le dossier désiré (ici `C:\Program Files\PHP_CodeSniffer`).
3. Dans le même dossier, créer le fichier suivant :&#x20;

{% code title="phpcs.bat" %}
```basic
@echo off
REM PHP_CodeSniffer detects violations of a defined coding standard.
REM 
REM @author    Greg Sherwood <gsherwood@squiz.net>
REM @copyright 2006-2015 Squiz Pty Ltd (ABN 77 084 670 600)
REM @license   https://github.com/squizlabs/PHP_CodeSniffer/blob/master/licence.txt BSD Licence

set PHPBIN=php
"%PHPBIN%" "%~dp0phpcs.phar" %*
```
{% endcode %}

4\. Récupérer le jeu de règles (ruleset) orienté sécurité présent dans le dossier "Security" sur le dépôt Github suivant : [https://github.com/FloeDesignTechnologies/phpcs-security-audit](https://github.com/FloeDesignTechnologies/phpcs-security-audit) et le déposer dans le même dossier que PHPCS.

5\. Le contenu du dossier doit maintenant être le suivant :&#x20;

![](<../../../.gitbook/assets/image (180).png>)

{% hint style="warning" %}
Il est nécessaire que le chemin de l'exécutable PHP soit renseigné dans le PATH de Windows.
{% endhint %}

6\. Dans Visual Studio Code, installer l'extension **phpcs** :&#x20;

![](<../../../.gitbook/assets/image (178).png>)

7\. Configurer l'extension de la façon suivante (éditer le fichier `settings.json` pour plus de facilité) :&#x20;

![](<../../../.gitbook/assets/image (16).png>)

L'extension est maintenant bien configurée et fonctionnelle. Par exemple ici la détection de l'utilisation de `shell_exec()` dans l'application bWAPP menant à une injection de commande :&#x20;

![](<../../../.gitbook/assets/image (113).png>)

## Références

* [https://pear.php.net/package/PHP\_CodeSniffer/](https://pear.php.net/package/PHP\_CodeSniffer/)
* [https://github.com/FloeDesignTechnologies/phpcs-security-audit](https://github.com/FloeDesignTechnologies/phpcs-security-audit)
