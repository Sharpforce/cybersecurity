# Level 5 (practice)

## Challenge #1

### Description

Appeler la fonction `prettyHuman()` avec la chaîne de caractères `"m5lccw"` en argument :

![](../../../.gitbook/assets/fb47e14cc50b2150eefeb8f5a70473f7.png)

### Résolution

Ici, deux filtres sont en place : les occurrences du caractère `"c"` sont supprimées et le caractère `" " (espace)` n'est également pas disponible. Le filtrage du caractère `"c"` est gênant car la balise `<script></script>` n'est pas utilisable :

![](../../../.gitbook/assets/99533f26bcb0f0dc30be7ecd5468b8b3.png)

Je contourne le filtrage du caractère `"c"` en passant par un tag `<svg>`ainsi que de l'encodage unicode suivant le contexte. Le `"/"` me permet de contourner la suppression du caractère `" " (espace)` :

![](../../../.gitbook/assets/f57fe3417e199719c0c63b030528c50f.png)

## Challenge #2

### Description

Appeler la fonction `prettyRobot()` avec la chaîne de caractères `"elcmf4"` en argument mais cette fois sur l'attribut `src` dans une balise `<img />` :

![](../../../.gitbook/assets/1ea837cb58255119c63c0062f48f2a4b.png)

### Résolution

Il me faut plusieurs essais avant de déterminer les quelques filtres en place ici. Il semble que l’occurrence `"on"` ne soit pas permise ainsi que l’occurrence `"rr"` qui est transformée en `"r"`. De plus le caractère `"t"` ne semble pas être permis non plus :

![](../../../.gitbook/assets/c7fa03cbe2d3c98581db70fa31c5ba2e.png)

Concernant le `"on"` de `onerror` je peux passer par le balise `<script></<script>`. Pour avoir une seule occurrence de `"r"` j'en renseigne un second, puis, finalement, j'utilise l'encodage unicode pour contourner le filtre sur le caractère `"t"` :

![](../../../.gitbook/assets/be28df2a6a27f4a6ac51cc82d483c447.png)

## Challenge #3

### Description

Appeler la fonction `politeSuperHero()` avec la chaîne de caractères `"twxdfu"` en argument mais cette fois sur l'attribut `href` dans une balise `<a>` :

![](../../../.gitbook/assets/5aa17e12154645053e5b191a6197345d.png)

### Résolution

Dans ce challenge, le caractère `"u"` ainsi que le caractère `"o"` sont filtrés :

![](../../../.gitbook/assets/92f35925b895f58e15fb170dc43f5ab7.png)

Etant donné que le `"u"` est filtré, il n'est pas possible d'utiliser l'encodage unicode. J'utilise donc les HTML entities pour le filtrage des caractères `"o"` et `"u"`. Concernant le `"u"` présent dans la chaîne passée en argument on utilise la fonction `String.fromCharCode()` . J'utilise également les HTML entities car je me rends compte que la chaîne `"String"` est également filtrée :

![](../../../.gitbook/assets/3b161555d0facb03956540bec8f16573.png)

Je me suis rendu compte seulement à posteriori , mais le filtrage sur le `"o"` est contournable en le doublant (par exemple `"oo"`).
