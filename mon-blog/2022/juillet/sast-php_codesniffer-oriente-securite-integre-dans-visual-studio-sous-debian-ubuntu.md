---
description: 18 Juillet 2022
---

# SAST - PHP\_CodeSniffer orienté sécurité intégré dans Visual Studio (sous Debian/Ubuntu)

{% hint style="info" %}
Un article concernant l'installation de PHP\_CodeSniffer sous Windows est également disponible [ici](https://sharpforce.gitbook.io/cybersecurity/mon-blog/2022/juillet/sast-php\_codesniffer-oriente-securite-integre-dans-visual-studio-sous-windows).
{% endhint %}

**PHP\_Code Sniffer** est un outil permettant d'analyser le code source PHP d'une application afin de détecter des mauvaises pratiques de développement en se basant sur un ensemble de règles. Ces règles concernent principalement la qualité du code, mais nous allons voir qu'il est possible d'en ajouter afin de détecter des failles de sécurité.

1. La première étape est d'installer l'outil grâce au gestionnaire de paquets `apt` :&#x20;

```shell-session
$ sudo apt-get install php-codesniffer
Lecture des listes de paquets... Fait
Dépaquetage de php-codesniffer (3.6.2-1)
```

L'exécutable est présent dans le répertoire `/usr/bin/` :&#x20;

```shell-session
$ which phpcs
/usr/bin/phpcs
```

et les règles sont dans le dossier `/usr/share/php/PHP/CodeSniffer/src/` :&#x20;

```shell-session
$ ls -l /usr/share/php/PHP/CodeSniffer/src
total 228
-rw-r--r--  1 root root 66054 Jun 18 16:57 Config.php
drwxr-xr-x  2 root root  4096 Jul 19 00:42 Exceptions
drwxr-xr-x  2 root root  4096 Jul 19 00:42 Files
drwxr-xr-x  2 root root  4096 Jul 19 00:42 Filters
-rw-r--r--  1 root root 23868 Jun 18 16:57 Fixer.php
drwxr-xr-x  2 root root  4096 Jul 19 00:42 Generators
-rw-r--r--  1 root root 13445 Jun 18 16:57 Reporter.php
drwxr-xr-x  2 root root  4096 Jul 19 00:42 Reports
-rw-r--r--  1 root root 50807 Jun 18 16:57 Ruleset.php
-rw-r--r--  1 root root 31532 Jun 18 16:57 Runner.php
drwxr-xr-x  2 root root  4096 Jul 19 00:42 Sniffs
drwxr-xr-x 10 root root  4096 Jul 19 00:42 Standards
drwxr-xr-x  2 root root  4096 Jul 19 00:42 Tokenizers
drwxr-xr-x  2 root root  4096 Jul 19 00:42 Util
```

2\. Récupérer le jeu de règles (ruleset) orienté sécurité présent dans le dossier "Security" sur le dépôt Github suivant : [https://github.com/FloeDesignTechnologies/phpcs-security-audit](https://github.com/FloeDesignTechnologies/phpcs-security-audit) et le déposer dans le dossier `/src/` :&#x20;

```shell-session
$ sudo cp -R /tmp/Security /usr/share/php/PHP/CodeSniffer/src/
```

```shell-session
$ ls -l
total 232
-rw-r--r--  1 root root 66054 Jun 18 16:57 Config.php
drwxr-xr-x  2 root root  4096 Jul 19 00:42 Exceptions
drwxr-xr-x  2 root root  4096 Jul 19 00:42 Files
drwxr-xr-x  2 root root  4096 Jul 19 00:42 Filters
-rw-r--r--  1 root root 23868 Jun 18 16:57 Fixer.php
drwxr-xr-x  2 root root  4096 Jul 19 00:42 Generators
-rw-r--r--  1 root root 13445 Jun 18 16:57 Reporter.php
drwxr-xr-x  2 root root  4096 Jul 19 00:42 Reports
-rw-r--r--  1 root root 50807 Jun 18 16:57 Ruleset.php
-rw-r--r--  1 root root 31532 Jun 18 16:57 Runner.php
drwxr-xr-x  3 root root  4096 Jul 19 00:48 Security
drwxr-xr-x  2 root root  4096 Jul 19 00:42 Sniffs
drwxr-xr-x 10 root root  4096 Jul 19 00:42 Standards
drwxr-xr-x  2 root root  4096 Jul 19 00:42 Tokenizers
drwxr-xr-x  2 root root  4096 Jul 19 00:42 Util
```

3\. Dans Visual Studio Code, installer l'extension **phpcs** :&#x20;

![](<../../../.gitbook/assets/image (27) (1).png>)

4\. Configurer l'extension de la façon suivante (éditer le fichier `settings.json` pour plus de facilité) :&#x20;

![](<../../../.gitbook/assets/image (27).png>)

L'extension est maintenant bien configurée et fonctionnelle. Par exemple, ici la détection de l'utilisation de `shell_exec()` dans l'application bWAPP menant à une injection de commande :&#x20;

![](<../../../.gitbook/assets/image (23) (1) (1).png>)
