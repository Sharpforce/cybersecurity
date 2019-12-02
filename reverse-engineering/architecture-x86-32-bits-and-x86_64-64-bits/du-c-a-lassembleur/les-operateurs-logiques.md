# Les opérateurs logiques

Les opérateurs logiques, au nombre de 6, permettent d'effectuer des opérations bit à bit.

## Le ET \(AND\) bit à bit

Le **ET** bit à bit est utilisé en général afin de mettre certains bits à zéro en fonction d'un masque. Par exemple un **ET** entre la valeur 5 \(soit en binaire "0101"\) et 12 \(soit en binaire "1100"\) donne 4 \(soit "0100" en binaire\) :

```c
int main(int argc, char **argv) {
  int a = 5;
  int b = 12;
  int c;

  c = a & b;

  return 0;
}
```

Et en assembleur :

```text
01: 80483db:       55                      push   ebp
02: 80483dc:       89 e5                   mov    ebp,esp
03: 80483de:       83 ec 10                sub    esp,0x10
04: 80483e1:       c7 45 fc 05 00 00 00    mov    DWORD PTR [ebp-0x4],0x5
05: 80483e8:       c7 45 f8 0c 00 00 00    mov    DWORD PTR [ebp-0x8],0xc
06: 80483ef:       8b 45 fc                mov    eax,DWORD PTR [ebp-0x4]
07: 80483f2:       23 45 f8                and    eax,DWORD PTR [ebp-0x8]
08: 80483f5:       89 45 f4                mov    DWORD PTR [ebp-0xc],eax
09: 80483f8:       b8 00 00 00 00          mov    eax,0x0
10: 80483fd:       c9                      leave
11: 80483fe:       c3                      ret
```

L'instruction _and_, à la ligne 7, effectue un **ET \(&\)** bit à bit entre _\[ebp-0x8\]_ et _\[ebp-0x4\]_ soit respectivement 12 et 5. Le résultat est stocké dans la variable "c" à _\[ebp-0xc\]_ à la ligne 8.

## Le OU \(OR\) bit à bit

Le **OU** est très souvent utilisé pour activer des drapeaux ou options à une fonction par exemple. Cet opérateur a pour effet de mettre à '1' certains bits. Par exemple un **OU** entre la valeur 5 \(soit en binaire "0101"\) et 12 \(soit en binaire "1100"\) donne 13 \(soit "1101" en binaire\) :

```c
int main(int argc, char **argv) {
  int a = 5;
  int b = 12;
  int c;

  c = a | b;

  return 0;
}
```

Et en assembleur :

```text
01: 80483db:       55                      push   ebp
02: 80483dc:       89 e5                   mov    ebp,esp
03: 80483de:       83 ec 10                sub    esp,0x10
04: 80483e1:       c7 45 fc 05 00 00 00    mov    DWORD PTR [ebp-0x4],0x5
05: 80483e8:       c7 45 f8 0c 00 00 00    mov    DWORD PTR [ebp-0x8],0xc
06: 80483ef:       8b 45 fc                mov    eax,DWORD PTR [ebp-0x4]
07: 80483f2:       0b 45 f8                or     eax,DWORD PTR [ebp-0x8]
08: 80483f5:       89 45 f4                mov    DWORD PTR [ebp-0xc],eax
09: 80483f8:       b8 00 00 00 00          mov    eax,0x0
10: 80483fd:       c9                      leave
11: 80483fe:       c3                      ret
```

L'instruction _or_, à la ligne 7, effectue un **OU \(\|\)** bit à bit entre _\[ebp-0x8\]_ et _\[ebp-0x4\]_ soit respectivement 12 et 5. Ligne 8, le résultat est stocké dans la variable "c" à _\[ebp-0xc\]_.

## Le OU exclusif \(XOR\) bit à bit

Il permet de ne garder seulement les bits déjà à '1' dans une des deux valeurs. Par exemple un **XOR** entre la valeur 5 \(soit en binaire "0101"\) et 12 \(soit en binaire "1100"\) donne 9 \(soit "1001" en binaire\) :

```c
int main(int argc, char **argv) {
  int a = 3;
  int b = 2;
  int c;

  c = a ^ b;

  return 0;
}
```

En assembleur :

```text
01: 80483db:       55                      push   ebp
02: 80483dc:       89 e5                   mov    ebp,esp
03: 80483de:       83 ec 10                sub    esp,0x10
04: 80483e1:       c7 45 fc 05 00 00 00    mov    DWORD PTR [ebp-0x4],0x5
05: 80483e8:       c7 45 f8 0c 00 00 00    mov    DWORD PTR [ebp-0x8],0xc
06: 80483ef:       8b 45 fc                mov    eax,DWORD PTR [ebp-0x4]
07: 80483f2:       33 45 f8                xor    eax,DWORD PTR [ebp-0x8]
08: 80483f5:       89 45 f4                mov    DWORD PTR [ebp-0xc],eax
09: 80483f8:       b8 00 00 00 00          mov    eax,0x0
10: 80483fd:       c9                      leave
11: 80483fe:       c3                      ret
```

L'instruction _xor_, à la ligne 7, effectue un **OU EXCLUSIF \(^\)** bit à bit entre _\[ebp-0x8\]_ et _\[ebp-0x4\]_ soit respectivement 12 et 5. Le résultat est stocké à la ligne 8, dans la variable "c" à _\[ebp-0xc\]_.

## Le complément à un

Le complément à un est un opérateur unaire \(il prend un seul paramètre\). Il a pour objectif d'inverser les bits. Tous les bits à un deviennent zéro et inversement. Par exemple la valeur 9 \(soit "1001" en binaire\) donne 6 \(soit "0110" en binaire\) :

```c
int main(int argc, char **argv) {
  int a = 9;
  int c;

  c = 15 & ~a;
  return 0;
}
```

En assembleur :

```text
01: 80483db:       55                      push   ebp
02: 80483dc:       89 e5                   mov    ebp,esp
03: 80483de:       83 ec 10                sub    esp,0x10
04: 80483e1:       c7 45 fc 09 00 00 00    mov    DWORD PTR [ebp-0x4],0x9
05: 80483e8:       8b 45 fc                mov    eax,DWORD PTR [ebp-0x4]
06: 80483eb:       f7 d0                   not    eax
07: 80483ed:       83 e0 0f                and    eax,0xf
08: 80483f0:       89 45 f8                mov    DWORD PTR [ebp-0x8],eax
09: 80483f3:       b8 00 00 00 00          mov    eax,0x0
10: 80483f8:       c9                      leave
11: 80483f9:       c3                      ret
```

L'instruction _not_ à la ligne 6, permet de récupérer la négation de la variable "a" \(qui vaut 9\). Ligne 7, un **ET** logique est effectué entre _EAX_ \(résultat de l'instruction précédente\) et la valeur 15. ce **ET** est nécessaire afin d'effectuer un masque sur les quatre premiers bits de la variable "a" \(15 vaut "1111" en binaire soit les 4 premiers bits à 1\), si cela était omit alors des effets de bords peuvent apparaître à cause du bit de signe\). Ligne 8, le résultat est stocké dans la variable "c" à l'adresse _\[ebp-0x8\]_.

## Le décalage à gauche "&lt;&lt;"

Le décalage à gauche permet de décaler d'un certain nombre de bits la valeur affectée. Par exemple la valeur 1 \(soit en binaire "0001"\) vaudra 2 \(soit en binaire "0010"\) pour un décalage de "1", mais vaudra 4 \(soit en binaire "0100"\) pour un décalage de "2" etc :

```c
int main(int argc, char **argv) {
  int a = 1;
  int c;

  c = a << 2;
  return 0;
}
```

En assembleur :

```text
01: 80483db:       55                      push   ebp
02: 80483dc:       89 e5                   mov    ebp,esp
03: 80483de:       83 ec 10                sub    esp,0x10
04: 80483e1:       c7 45 fc 01 00 00 00    mov    DWORD PTR [ebp-0x4],0x1
05: 80483e8:       8b 45 fc                mov    eax,DWORD PTR [ebp-0x4]
06: 80483eb:       c1 e0 02                shl    eax,0x2
07: 80483ee:       89 45 f8                mov    DWORD PTR [ebp-0x8],eax
08: 80483f1:       b8 00 00 00 00          mov    eax,0x0
09: 80483f6:       c9                      leave
10: 80483f7:       c3                      ret
```

L'instruction _shl_ \(pour _shifl left_\), ligne 6, permet de décaler de "2" la valeur de la variable stockée à _\[ebp-0x4\]_. Dans le cas de l'exemple la valeur 1 est décalée de "2". Le résultat est stocké dans la variable "c" à la ligne 7.

## Le décalage à droite "&gt;&gt;"

Le décalage à droite permet de décaler d'un certain nombre de bits la valeur affectée. Par exemple la valeur 2 \(soit en binaire "0010"\) vaudra 1 \(soit en binaire "0001"\) pour un décalage de "1" :

```c
int main(int argc, char **argv) {
  int a = 2;
  int c;

  c = a >> 1;
  return 0;
}
```

En assembleur :

```text
01: 80483db:       55                      push   ebp
02: 80483dc:       89 e5                   mov    ebp,esp
03: 80483de:       83 ec 10                sub    esp,0x10
04: 80483e1:       c7 45 fc 02 00 00 00    mov    DWORD PTR [ebp-0x4],0x2
05: 80483e8:       8b 45 fc                mov    eax,DWORD PTR [ebp-0x4]
06: 80483eb:       d1 f8                   sar    eax,1
07: 80483ed:       89 45 f8                mov    DWORD PTR [ebp-0x8],eax
08: 80483f0:       b8 00 00 00 00          mov    eax,0x0
09: 80483f5:       c9                      leave
10: 80483f6:       c3                      ret
```

L'instruction _sar_ \(pour _shifl arithmetic right_\), ligne 6, permet de décaler de "1" la valeur de la variable stockée à _\[ebp-0x4\]_. Dans le cas de l'exemple la valeur 2 est décalée de "1". Le résultat est stocké dans la variable "c" à la ligne 7. Une subtilité ici est que _sar_ permet de sauvegarder le bit de signe \(il s'agit du bit le plus significatif nommé également _msb_\). Il existe donc aussi l'instruction _shr_ pour _shift right_.

Pour l'exemple, voici le même programme mais cette fois en utilisant des entiers non signés \(donc absence de bit de signe\) :

```c
int main(int argc, char **argv) {
  unsigned int a = 2;
  unsigned int c;

  c = a >> 1;
  return 0;
}
```

En assembleur :

```text
01: 80483db:       55                      push   ebp
02: 80483dc:       89 e5                   mov    ebp,esp
03: 80483de:       83 ec 10                sub    esp,0x10
04: 80483e1:       c7 45 fc 02 00 00 00    mov    DWORD PTR [ebp-0x4],0x2
05: 80483e8:       8b 45 fc                mov    eax,DWORD PTR [ebp-0x4]
06: 80483eb:       d1 e8                   shr    eax,1
07: 80483ed:       89 45 f8                mov    DWORD PTR [ebp-0x8],eax
08: 80483f0:       b8 00 00 00 00          mov    eax,0x0
09: 80483f5:       c9                      leave
10: 80483f6:       c3                      ret
```

A la ligne 6, l'instruction _sar_ précédente a été remplacée par l'instruction _shr_.

