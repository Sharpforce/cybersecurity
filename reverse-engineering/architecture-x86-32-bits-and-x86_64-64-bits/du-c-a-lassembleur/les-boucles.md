# Les boucles

Les boucles permettent de réitérer un certain nombre de fois une action ou un traitement. L'objectif de ce chapitre est d'identifier les motifs assembleur représentant les boucles _for_, _while_ et _do ... while_.

## La boucle _for_

Par exemple, une boucle allant de "0" à "10" \(non-inclus\) qui incrémente de "10" une variable nommée _c_ à chaque itération :

```c
int main(int argc, char **argv) {
  int compteur = 10;
  int c = 0;

  for(int i = 0 ; i < compteur ; i++) {
    c += 10;
  }

  return 0;
}
```

Son code assembleur :

```text
01: 80483db:       55                      push   ebp
02: 80483dc:       89 e5                   mov    ebp,esp
03: 80483de:       83 ec 10                sub    esp,0x10
04: 80483e1:       c7 45 f4 0a 00 00 00    mov    DWORD PTR [ebp-0xc],0xa
05: 80483e8:       c7 45 fc 00 00 00 00    mov    DWORD PTR [ebp-0x4],0x0
06: 80483ef:       c7 45 f8 00 00 00 00    mov    DWORD PTR [ebp-0x8],0x0
07: 80483f6:       eb 08                   jmp    8048400 <main+0x25>
08: 80483f8:       83 45 fc 0a             add    DWORD PTR [ebp-0x4],0xa
09: 80483fc:       83 45 f8 01             add    DWORD PTR [ebp-0x8],0x1
10: 8048400:       8b 45 f8                mov    eax,DWORD PTR [ebp-0x8]
11: 8048403:       3b 45 f4                cmp    eax,DWORD PTR [ebp-0xc]
12: 8048406:       7c f0                   jl     80483f8 <main+0x1d>
13: 8048408:       b8 00 00 00 00          mov    eax,0x0
14: 804840d:       c9                      leave
15: 804840e:       c3                      ret
```

Ligne 4, "0xa", qui correspond à la valeur 10 \(autrement dit la variable _compteur_\) est stockée dans _\[ebp-0xc\]_. Ligne 5, la valeur 0 est stockée dans _\[ebp-0x4\]_ \(variable "c"\) et également dans _\[ebp-0x8\]_ \("variable "i"\), ligne 6. La prochaine instruction est un saut inconditionnel à l'adresse _0x08048400_, soit à la ligne 10. Cette ligne va assigner le contenu de la variable "i" au registre _EAX_. La ligne 11 compare le registre _EAX_ à la variable "compteur". Si _EAX_ est strictement inférieur à "compteur" \(ligne 12\) alors l'exécution se poursuivra à la ligne 8. Dans le cas contraire \(_EAX_ supérieur ou égale à "compteur\), l'exécution du programme se termine \(exécution de la ligne 13, 14 puis 15\). La ligne 8 ajoute la valeur 10 à la variable "c", tandis que la ligne 9 incrémente la variable "i". L'exécution continue à la ligne 10 déjà explicité précédemment.

## La boucle _while_

Même exemple que précédemment mais, cette fois, en utilisant une boucle _while_ :

```c
int main(int argc, char **argv) {
  int compteur = 10;
  int c = 0;
  int i = 0;

  while(i < compteur) {
    c += 10;
    i++;
  }

  return 0;
}
```

Son code assembleur :

```text
01: 80483db:       55                      push   ebp
02: 80483dc:       89 e5                   mov    ebp,esp
03: 80483de:       83 ec 10                sub    esp,0x10
04: 80483e1:       c7 45 f4 0a 00 00 00    mov    DWORD PTR [ebp-0xc],0xa
05: 80483e8:       c7 45 fc 00 00 00 00    mov    DWORD PTR [ebp-0x4],0x0
06: 80483ef:       c7 45 f8 00 00 00 00    mov    DWORD PTR [ebp-0x8],0x0
07: 80483f6:       eb 08                   jmp    8048400 <main+0x25>
08: 80483f8:       83 45 fc 0a             add    DWORD PTR [ebp-0x4],0xa
09: 80483fc:       83 45 f8 01             add    DWORD PTR [ebp-0x8],0x1
10: 8048400:       8b 45 f8                mov    eax,DWORD PTR [ebp-0x8]
11: 8048403:       3b 45 f4                cmp    eax,DWORD PTR [ebp-0xc]
12: 8048406:       7c f0                   jl     80483f8 <main+0x1d>
13: 8048408:       b8 00 00 00 00          mov    eax,0x0
14: 804840d:       c9                      leave
15: 804840e:       c3                      ret
```

Il n'y a pas beaucoup de changements ici. La boucle _while_ fait la même chose que la boucle _for_ de l'exemple précédent : le code assembleur généré est alors le même.

## La boucle do ... while

La boucle _do ... while_ possède une différence majeure. Les instructions présentes au sein de la boucle sont obligatoirement exécutées au moins une fois. Le code assembleur généré va donc forcément être différent cette fois-ci :

```c
int main(int argc, char **argv) {
  int compteur = 10;
  int c = 0;
  int i = 0;

  do {
    c += 10;
    i++;
  } while(i < compteur);

  return 0;
}
```

Son code assembleur :

```text
01: 80483db:       55                      push   ebp
02: 80483dc:       89 e5                   mov    ebp,esp
03: 80483de:       83 ec 10                sub    esp,0x10
04: 80483e1:       c7 45 f4 0a 00 00 00    mov    DWORD PTR [ebp-0xc],0xa
05: 80483e8:       c7 45 fc 00 00 00 00    mov    DWORD PTR [ebp-0x4],0x0
06: 80483ef:       c7 45 f8 00 00 00 00    mov    DWORD PTR [ebp-0x8],0x0
07: 80483f6:       83 45 fc 0a             add    DWORD PTR [ebp-0x4],0xa
08: 80483fa:       83 45 f8 01             add    DWORD PTR [ebp-0x8],0x1
09: 80483fe:       8b 45 f8                mov    eax,DWORD PTR [ebp-0x8]
10: 8048401:       3b 45 f4                cmp    eax,DWORD PTR [ebp-0xc]
11: 8048404:       7c f0                   jl     80483f6 <main+0x1b>
12: 8048406:       b8 00 00 00 00          mov    eax,0x0
13: 804840b:       c9                      leave
14: 804840c:       c3                      ret
```

La différence ici est bien sûr l'absence du saut inconditionnel, qui, habituellement permet de tester la condition avant d'exécuter les instructions présentes dans la boucle \(ou de ne pas exécuter les instructions, si la condition de sortie est vérifiée\). Le programme exécute donc une première fois les instructions de la boucle _do ... while_ \(incrément de 10 de la variable "c", puis, incrément de 1 de la variable "i", ligne 7 et 8\), puis, teste si la variable _i_ est inférieure à la variable _compteur_ à la ligne 10 et 11. Si c'est le cas, alors les instructions contenues dans la boucle sont à nouveau exécutées \(retour à la ligne 7\). Dans le cas contraire, le programme se termine par l'exécution des lignes 12, 13 et 14.

