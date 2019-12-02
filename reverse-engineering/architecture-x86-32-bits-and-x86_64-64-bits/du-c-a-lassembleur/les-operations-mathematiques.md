# Les opérations mathématiques

Dans ce chapitre, les opérations mathématiques regroupent les opérations : addition, soustraction, multiplication, division et modulo. Ils seront effectuées sur des variables de type entiers signés\(int\).

## L'addition

L'addition du chiffre "3" par le chiffre "2" peut se représenter comme ceci :

```c
int main(int argc, char **argv) {
  int a = 3;
  int b = 2;
  int c;

  c = a + b;
  return 0;
}
```

Et en assembleur :

```text
01: 80483db:       55                      push   ebp
02: 80483dc:       89 e5                   mov    ebp,esp
03: 80483de:       83 ec 10                sub    esp,0x10
04: 80483e1:       c7 45 fc 03 00 00 00    mov    DWORD PTR [ebp-0x4],0x3
05: 80483e8:       c7 45 f8 02 00 00 00    mov    DWORD PTR [ebp-0x8],0x2
06: 80483ef:       8b 55 fc                mov    edx,DWORD PTR [ebp-0x4]
07: 80483f2:       8b 45 f8                mov    eax,DWORD PTR [ebp-0x8]
08: 80483f5:       01 d0                   add    eax,edx
09: 80483f7:       89 45 f4                mov    DWORD PTR [ebp-0xc],eax
10: 80483fa:       b8 00 00 00 00          mov    eax,0x0
11: 80483ff:       c9                      leave
12: 8048400:       c3                      ret
```

Ligne 4 et 5, les valeurs "3" et "2" sont stockées respectivement dans _\[ebp-0x4\]_ \(soit "a"\) et dans _\[ebp-0x8\]_ \(soit "b"\). Elles sont ensuite déplacées dans les registres de travail _EDX_ et _EAX_, ligne 6 et 7. Puis, ligne 8, vient l'addition grâce à l'instruction _add_ et aux opérandes _eax_ \("b"\) et _edx_ \("a"\). Après une telle opération le résultat est en général stocké dans le registre _EAX_. L'instruction ligne 9 sauvegarde le résultat contenu dans le registre _EAX_ dans la variable "c" stocké à _\[ebp-0xc\]_.

## La soustraction

La soustraction s'effectue de la même façon que l'addition, il suffit de remplacer l'opération _add_ par l'opération _sub_ :

```c
int main(int argc, char **argv) {
  int a = 3;
  int b = 2;
  int c;

  c = a - b;
  return 0;
}
```

En assembleur :

```text
01: 80483db:       55                      push   ebp
02: 80483dc:       89 e5                   mov    ebp,esp
03: 80483de:       83 ec 10                sub    esp,0x10
04: 80483e1:       c7 45 fc 03 00 00 00    mov    DWORD PTR [ebp-0x4],0x3
05: 80483e8:       c7 45 f8 02 00 00 00    mov    DWORD PTR [ebp-0x8],0x2
06: 80483ef:       8b 45 fc                mov    eax,DWORD PTR [ebp-0x4]
07: 80483f2:       2b 45 f8                sub    eax,DWORD PTR [ebp-0x8]
08: 80483f5:       89 45 f4                mov    DWORD PTR [ebp-0xc],eax
09: 80483f8:       b8 00 00 00 00          mov    eax,0x0
10: 80483fd:       c9                      leave
11: 80483fe:       c3                      ret
```

Seul la ligne 7 est différente par rapport à l'addition. Une autre petite différence est l'absence de l'utilisation d'un registre intermédiaire qui était _EDX_ pour l'addition.

## La multiplcation

La multiplication 3 par 2 peut donner ce qui suit :

```c
int main(int argc, char **argv) {
  int a = 3;
  int b = 2;
  int c;

  c = a * b;
  return 0;
}
```

En assembleur :

```text
01: 80483db:       55                      push   ebp
02: 80483dc:       89 e5                   mov    ebp,esp
03: 80483de:       83 ec 10                sub    esp,0x10
04: 80483e1:       c7 45 fc 03 00 00 00    mov    DWORD PTR [ebp-0x4],0x3
05: 80483e8:       c7 45 f8 02 00 00 00    mov    DWORD PTR [ebp-0x8],0x2
06: 80483ef:       8b 45 fc                mov    eax,DWORD PTR [ebp-0x4]
07: 80483f2:       0f af 45 f8             imul   eax,DWORD PTR [ebp-0x8]
08: 80483f6:       89 45 f4                mov    DWORD PTR [ebp-0xc],eax
09: 80483f9:       b8 00 00 00 00          mov    eax,0x0
10: 80483fe:       c9                      leave
11: 80483ff:       c3                      ret
```

Pas de nouveauté, excepté à la ligne 7, l'instruction _imul_, utilisée pour la multiplication d'entiers signés \(si les entiers étaient non signés, alors l'instruction _mul_ aurait été utilisée\).

## La division

La soustraction de 4 par 2 donne ceci :

```c
int main(int argc, char **argv) {
  int a = 4;
  int b = 2;
  int c;

  c = a / b;
  return 0;
}
```

En assembleur :

```text
01: 80483db:       55                      push   ebp
02: 80483dc:       89 e5                   mov    ebp,esp
03: 80483de:       83 ec 10                sub    esp,0x10
04: 80483e1:       c7 45 fc 04 00 00 00    mov    DWORD PTR [ebp-0x4],0x4
05: 80483e8:       c7 45 f8 02 00 00 00    mov    DWORD PTR [ebp-0x8],0x2
06: 80483ef:       8b 45 fc                mov    eax,DWORD PTR [ebp-0x4]
07: 80483f2:       99                      cdq
08: 80483f3:       f7 7d f8                idiv   DWORD PTR [ebp-0x8]
09: 80483f6:       89 45 f4                mov    DWORD PTR [ebp-0xc],eax
10: 80483f9:       b8 00 00 00 00          mov    eax,0x0
11: 80483fe:       c9                      leave
12: 80483ff:       c3                      ret
```

La ligne 6 permet de copier le contenu de _\[ebp-0x4\]_ \(soir la valeur 4, c'est à dire "a"\) dans le registre _EAX_, il s'agit en fait ici du dividende de l'opération. L'instruction _cdq_ \(littéralement _Convert Double to Quad_\), ligne 7, permet de stocker le dividende non sur 32 bits mais sur 64 bits. Pour cela, le processeur étend le bit de signe de _EAX_ sur _EDX_, ce qui donne alors _EDX:EAX_. La division est effectuée à la ligne 8. L'instruction _idiv_ \(si les entiers étaient non signés, alors l'instruction _div_ aurait été utilisée\) n'admet qu'un seul opérande : le diviseur \(le dividende est déjà stocké dans _EDX:EAX_\). Après l'instruction, le quotient sera stocké dans le registre _EAX_ tandis que _EDX_ contiendra le reste de la division.

## Le modulo

Pour rappel, le modulo est le reste de la vision :

```c
int main(int argc, char **argv) {
  int a = 4;
  int b = 3;
  int c;

  c = a % b;
  return 0;
}
```

En assembleur :

```text
01: 80483db:       55                      push   ebp
02: 80483dc:       89 e5                   mov    ebp,esp
03: 80483de:       83 ec 10                sub    esp,0x10
04: 80483e1:       c7 45 fc 04 00 00 00    mov    DWORD PTR [ebp-0x4],0x4
05: 80483e8:       c7 45 f8 03 00 00 00    mov    DWORD PTR [ebp-0x8],0x3
06: 80483ef:       8b 45 fc                mov    eax,DWORD PTR [ebp-0x4]
07: 80483f2:       99                      cdq
08: 80483f3:       f7 7d f8                idiv   DWORD PTR [ebp-0x8]
09: 80483f6:       89 55 f4                mov    DWORD PTR [ebp-0xc],edx
10: 80483f9:       b8 00 00 00 00          mov    eax,0x0
11: 80483fe:       c9                      leave
12: 80483ff:       c3                      ret
```

Le code assembleur est le même que pour la division \(voir l'exemple précédent\). Il faut toutefois se rappeler que, lors d'une division, le reste est stocké dans le registre _EDX_. C'est pour cette raison que, ligne 9, le contenu du registre _EDX_ est copié dans _\[ebp-0xc\]_ \(soit la variable "c"\). Pour rappel, dans la division précédente ,la variable "c" avait pour valeur le contenu du registre _EAX_ \(le quotient\).

