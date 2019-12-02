# Alignement mémoire

L'alignement mémoire est un mécanisme qui permet d'améliorer les performances des processeurs. Quand il est nécessaire, il est visible avant le prologue de la fonction _main\(\)_ par une suite d'instructions assembleur.

Pour mieux comprendre ce mécanisme, voici l'analyse d'un programme simple effectuant un appel à la méthode _printf\(\)_ :

```c
#include <stdio.h>

int main() {
  printf("Hello World !\n");

  return 0;
}
```

L'assembleur correspondant est le suivant :

```text
01: 804840b:       8d 4c 24 04             lea    ecx,[esp+0x4]
02: 804840f:       83 e4 f0                and    esp,0xfffffff0
03: 8048412:       ff 71 fc                push   DWORD PTR [ecx-0x4]
04: 8048415:       55                      push   ebp
05: 8048416:       89 e5                   mov    ebp,esp
06: 8048418:       51                      push   ecx
07: 8048419:       83 ec 04                sub    esp,0x4
08: 804841c:       83 ec 0c                sub    esp,0xc
09: 804841f:       68 c0 84 04 08          push   0x80484c0
10: 8048424:       e8 b7 fe ff ff          call   80482e0 <puts@plt>
11: 8048429:       83 c4 10                add    esp,0x10
12: 804842c:       b8 00 00 00 00          mov    eax,0x0
13: 8048431:       8b 4d fc                mov    ecx,DWORD PTR [ebp-0x4]
14: 8048434:       c9                      leave
15: 8048435:       8d 61 fc                lea    esp,[ecx-0x4]
16: 8048438:       c3                      ret
```

La ligne 1 copie l'adresse _\[esp+0x4\]_ dans le registre _ECX_. Mais à quoi correspond la valeur du registre _ESP_ \(utilisé à la ligne 2\) et _\[esp+0x4\]_ ? Pour connaître ces valeurs, il est possible d'utiliser un debugger en analyse dynamique. Cela sort légèrement du cadre de l'analyse statique, mais travailler sur des valeurs réelles va permettre une meilleure compréhension du mécanisme.

Pour cela, l'outil **gdb** sera utilisé :

```text
$ gdb nomDuProgramme
```

Il est possible de désassembler la méthode _main\(\)_ comme le ferai **objdump** :

```text
(gdb) set disassembly-flavor intel // Permet d'utiliser la syntaxe Intel
(gdb) disass main
```

```text
Dump of assembler code for function main:
   0x0804840b <+0>:     lea    ecx,[esp+0x4]
   0x0804840f <+4>:     and    esp,0xfffffff0
   0x08048412 <+7>:     push   DWORD PTR [ecx-0x4]
   0x08048415 <+10>:    push   ebp
   0x08048416 <+11>:    mov    ebp,esp
   0x08048418 <+13>:    push   ecx
   0x08048419 <+14>:    sub    esp,0x4
   0x0804841c <+17>:    sub    esp,0xc
   0x0804841f <+20>:    push   0x80484c0
   0x08048424 <+25>:    call   0x80482e0 <puts@plt>
   0x08048429 <+30>:    add    esp,0x10
   0x0804842c <+33>:    mov    eax,0x0
   0x08048431 <+38>:    mov    ecx,DWORD PTR [ebp-0x4]
   0x08048434 <+41>:    leave
   0x08048435 <+42>:    lea    esp,[ecx-0x4]
   0x08048438 <+45>:    ret
End of assembler dump.
```

Pour connaître le contenu du registre _ESP_ il faut dans un premier temps imposer un point d'arrêt \(c'est-à-dire que **gdb** va mettre en pause le programme\), exécuter le programme, puis permettre l'analyse du registre _ESP_. Pour cela :

```text
(gdb) b *0x0804840b
```

```text
Breakpoint 1 at 0x804840b: file prog.c, line 3.
```

Lancer l'exécution du programme qui sera en pause quand le point d'arrêt sera rencontré :

```text
(gdb) run
```

```text
Starting program: /home/debian-user/cracking/prog

Breakpoint 1, main () at prog.c:3
3       int main() {
```

La commande _disass_ permet de connaître la prochaine instruction qui sera exécutée \(la flèche devant l'adresse de l'instruction\). Le point d'arrêt s'applique **avant** l'exécution de l'instruction ciblée et non après :

```text
(gdb) disass main
```

```text
Dump of assembler code for function main:
=> 0x0804840b <+0>:     lea    ecx,[esp+0x4]
   0x0804840f <+4>:     and    esp,0xfffffff0
   0x08048412 <+7>:     push   DWORD PTR [ecx-0x4]
   0x08048415 <+10>:    push   ebp
   0x08048416 <+11>:    mov    ebp,esp
   0x08048418 <+13>:    push   ecx
   0x08048419 <+14>:    sub    esp,0x4
   0x0804841c <+17>:    sub    esp,0xc
   0x0804841f <+20>:    push   0x80484c0
   0x08048424 <+25>:    call   0x80482e0 <puts@plt>
   0x08048429 <+30>:    add    esp,0x10
   0x0804842c <+33>:    mov    eax,0x0
   0x08048431 <+38>:    mov    ecx,DWORD PTR [ebp-0x4]
   0x08048434 <+41>:    leave
   0x08048435 <+42>:    lea    esp,[ecx-0x4]
   0x08048438 <+45>:    ret
End of assembler dump.
```

Il faut maintenant afficher le contenu du registre _ESP_ :

```text
(gdb) x/x $esp
```

```text
0xbffff66c:     0xb7e2f286
```

Puis de _\[esp+0x4\]_ :

```text
(gdb) x/x $esp+0x4
```

```text
0xbffff670:     0x00000001
```

En résultat, _ESP_ contient l'adresse _0xbffff66c_ et _\[esp+0x4\]_ contient _0xbffff670_. A noter ici que la zone mémoire à l'adresse _0xbffff670_ \(soit _\[esp+0x4\]_\) contient la valeur 1, c'est-à-dire le nombre d'arguments passé à la ligne de commande lors du lancement du programme. En résumé, la première ligne sauvegarde donc l'adresse mémoire, qui contient le nombre d'arguments de la ligne de commande, dans le registre _ECX_. La seconde ligne effectue un **ET** logique entre le registre _ESP_ et la valeur _0xfffffff0_. Le résultat aura donc les 28 bits les plus à gauche égaux à ceux de _ESP_, mais les 4 bits les plus à droite seront mis à 0. _ESP_ aura alors pour valeur final _0xbffff660_. Le pointeur de la pile pointe donc maintenant vers une adresse plus basse, cela augmente la taille de la pile de 12 octets \(il y a 12 octets entre _0xbffff66c_ et _0xbffff660_\). La ligne 3 empile la valeur _\[ecx-0x4\]_ sur la pile. Cette valeur est reconnaissable, puisque _ECX_ contient en fait _\[esp+0x4\]_ alors _\[ecx-0x4\]_ contient l'ancienne valeur de _ESP_, qui se retrouve donc maintenant sur la pile. Il faut savoir que la valeur de _\[ecx-0x4\]_ \(ligne 3\), qui est également l'ancienne valeur de _ESP_ est en fait l'adresse de retour de la fonction _main\(\)_. En effet, bien que la fonction _main\(\)_ soit la première fonction visible pour le développeur, ce n'est pas la première exécutée lors du lancement du programme. La fonction appelant la méthode _main\(\)_ et à laquelle la méthode _main\(\)_ retourne à la fin de son exécution se nomme _lic\_start\_main\(\)_. Pour plus d'information sur cette fonction [http://refspecs.linuxbase.org/LSB\_2.0.1/LSB-PDA/LSB-PDA/baselib---libc-start-main-.html](http://refspecs.linuxbase.org/LSB_2.0.1/LSB-PDA/LSB-PDA/baselib---libc-start-main-.html)

Mais alors pourquoi tout cela ? La ligne 2 modifie le registre _ESP_ pour que l'adresse soit alignée à une valeur de 16 octets. Cela permet de s'assurer que toutes les informations stockées sur la pile pendant le reste du programme seront également alignées de la même façon. Ce mécanisme est tout simplement une optimisation permettant au processeur de meilleures performances.

L'explication de ce mécanisme permet également de mieux comprendre une autre notion déjà vue : la réservation de l'espace sur la pile quand des variables locales sont utilisées. Un exemple simple est celui-ci :

```c
int main(int argc, char **argv) {
  int a = 5;
  int b = 8;

  return 0;
}
```

```text
01: 80483db:       55                      push   ebp
02: 80483dc:       89 e5                   mov    ebp,esp
03: 80483de:       83 ec 10                sub    esp,0x10
04: 80483e1:       c7 45 fc 05 00 00 00    mov    DWORD PTR [ebp-0x4],0x5
05: 80483e8:       c7 45 f8 08 00 00 00    mov    DWORD PTR [ebp-0x8],0x8
06: 80483ef:       b8 00 00 00 00          mov    eax,0x0
07: 80483f4:       c9                      leave
08: 80483f5:       c3                      ret
```

Soit une fonction _main\(\)_ allouant deux variables locales "a" et "b". La ligne 3 du code assembleur permet de réserver _0x10_ d'espace mémoire sur la pile, soit 16 octets. Pourtant, la variable "a" qui est un entier, occupe seulement 4 octets, il en est de même pour la variable "b", soit au total 8 octets. Pourquoi avoir réservé 8 octets supplémentaire sur la pile ? Tout simplement pour garder l'alignement de 16 octets, toujours afin d'améliorer les performances du processeur.

