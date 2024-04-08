# MyExpense

## Description

* **Difficulté :** Facile
* **Type :** Réaliste
* **Technologies :** PHP / MySQL
* **Réseau :** DHCP

**MyExpense** est une application **volontairement vulnérable** permettant de s’entraîner à détecter et à exploiter différentes vulnérabilités web. Contrairement à une application de type "challenge" plus classique (qui permet de s'entraîner sur une seule vulnérabilité précise), MyExpense contient un ensemble de vulnérabilités.

Le second objectif de cette application est d'immerger l'attaquant dans un environnement professionnel. Pour cela, un effort a été fourni afin que l'application ressemble le plus possible à un outil destiné à une utilisation interne par les employés de l'entreprise "Futura Business Informatique". De plus, il est possible (et nécessaire afin de récupérer le flag/drapeau validant le challenge) de lancer des scripts permettant de simuler l'utilisation de l'application par des employés afin d'exploiter certaines vulnérabilités ayant pour victime les utilisateurs authentifiés.

![](<../.gitbook/assets/image (153).png>)

## Statut du projet

Le développement de l'application est terminée. Elle est disponible directement via [Github](https://github.com/Sharpforce/MyExpense) ou sur le site [Vulnhub](https://www.vulnhub.com/entry/myexpense-1,405/).

## Changelog

**27/03/2024 :** Correction d'un bug lors de l'installation à partir des sources (image Vbox, compatible 7.0 et Debian 12, disponible [ici](https://www.mediafire.com/file/e1hjy5orlpd87au/My\_Expense\_Vulnerable\_Web\_Application\_-\_1.3.ova/file)).

```
Changed
- Based on Debian 12
- Installation from source is nox based on a automatic script

Fixed
- Initialization of the database not possible when installing the application from the sources.
```

**21/05/2023 :** Amélioration des scripts Selenium/Chrome et affichage de l'IP de la machine virtuelle.

```
Added
- IP address is now diplayed at the startup of the virtual machine

Changed
- Improve Selenium/Chrome users scripts
```

**28/03/2023 :** Mise à jour des librairies (image Vbox, compatible 7.0, disponible [ici](https://www.mediafire.com/file/smscycfha2qb3u1/My\_Expense\_Vulnerable\_Web\_Application\_%28Debian11%29.ova/file))

```
Changed
- Based on Debian 11
- Use Python3 and Chrome headless
```
