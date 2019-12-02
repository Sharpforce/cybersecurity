# Les binaires "stripped"

Il est possible grâce à une option du compilateur _gcc_ de supprimer la table des symboles de l'exécutable. Il s'agit de l'option "-s" dont voici sa description :

```text
Remove all symbol table and relocation information from the executable.
```

Si un binaire est compilé avec cette option alors le binaire est dit "stripped". Cele permet de générer un exécutable moins lourd \(il prendra moins de place sur le disque\) et de gagner légèrement en performance. L'inconvénient pour le reverser, c'est qu'il est beaucoup plus difficile d'analyser ce type de fichier. Par exemple le programme suivant compilé avec puis sans cette option de _gcc_ :

```c
#include <stdio.h>

int main() {
  printf("Hello World !\n");

  return 0;
}
```

```text
$ gcc -g prog.c -o prog -fno-stack-protector -z execstack -fno-pic -no-pie
$ file ./prog
./prog: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=c18f8e28d2fc4ba568d24b47f43973e54afd8713, not stripped
```

La sortie de l'utilitaire _file_ indique que le fichier analysé est _not stripped_. La taille de ce fichier est alors :

```text
$ ls -sh ./prog
12K ./prog
```

Voici le même programme mais recompilé avec l'option "-s" :

```text
$ gcc -s prog.c -o prog -fno-stack-protector -z execstack -fno-pic -no-pie
$ file ./prog
./prog: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=75e565630ca18366ed4b8567ebd2e701c826e935, stripped
```

Le fichier est indiqué ici comme étant "stripped", c'est-à-dire amputé de sa table des symboles \(et quelques autres informations\). Sa taille est également plus petite ici :

```text
$ ls -sh ./prog
8,0K ./prog
```

Mais pourquoi cela rend il plus difficile la rétro-analyse du binaire ? L'utilisation de _objdump_ donne la réponse. Voici la sortie de l'utilitaire pour la version dite "stripped" :

```text
$ objdump -d ./prog -M intel
./prog:     format de fichier elf32-i386


Déassemblage de la section .init :

080482a8 <.init>:
 80482a8:       53                      push   ebx
 80482a9:       83 ec 08                sub    esp,0x8
 80482ac:       e8 8f 00 00 00          call   8048340 <__libc_start_main@plt+0x50>
 80482b1:       81 c3 4f 1d 00 00       add    ebx,0x1d4f
 80482b7:       8b 83 fc ff ff ff       mov    eax,DWORD PTR [ebx-0x4]
 80482bd:       85 c0                   test   eax,eax
 80482bf:       74 05                   je     80482c6 <puts@plt-0x1a>
 80482c1:       e8 3a 00 00 00          call   8048300 <__libc_start_main@plt+0x10>
 80482c6:       83 c4 08                add    esp,0x8
 80482c9:       5b                      pop    ebx
 80482ca:       c3                      ret

Déassemblage de la section .plt :

080482d0 <puts@plt-0x10>:
 80482d0:       ff 35 04 a0 04 08       push   DWORD PTR ds:0x804a004
 80482d6:       ff 25 08 a0 04 08       jmp    DWORD PTR ds:0x804a008
 80482dc:       00 00                   add    BYTE PTR [eax],al
        ...

080482e0 <puts@plt>:
 80482e0:       ff 25 0c a0 04 08       jmp    DWORD PTR ds:0x804a00c
 80482e6:       68 00 00 00 00          push   0x0
 80482eb:       e9 e0 ff ff ff          jmp    80482d0 <puts@plt-0x10>

080482f0 <__libc_start_main@plt>:
 80482f0:       ff 25 10 a0 04 08       jmp    DWORD PTR ds:0x804a010
 80482f6:       68 08 00 00 00          push   0x8
 80482fb:       e9 d0 ff ff ff          jmp    80482d0 <puts@plt-0x10>

Déassemblage de la section .plt.got :

08048300 <.plt.got>:
 8048300:       ff 25 fc 9f 04 08       jmp    DWORD PTR ds:0x8049ffc
 8048306:       66 90                   xchg   ax,ax

Déassemblage de la section .text :

08048310 <.text>:
 8048310:       31 ed                   xor    ebp,ebp
 8048312:       5e                      pop    esi
 8048313:       89 e1                   mov    ecx,esp
 8048315:       83 e4 f0                and    esp,0xfffffff0
 8048318:       50                      push   eax
 8048319:       54                      push   esp
 804831a:       52                      push   edx
 804831b:       68 a0 84 04 08          push   0x80484a0
 8048320:       68 40 84 04 08          push   0x8048440
 8048325:       51                      push   ecx
 8048326:       56                      push   esi
 8048327:       68 0b 84 04 08          push   0x804840b
 804832c:       e8 bf ff ff ff          call   80482f0 <__libc_start_main@plt>
 8048331:       f4                      hlt
 8048332:       66 90                   xchg   ax,ax
 8048334:       66 90                   xchg   ax,ax
 8048336:       66 90                   xchg   ax,ax
 8048338:       66 90                   xchg   ax,ax
 804833a:       66 90                   xchg   ax,ax
 804833c:       66 90                   xchg   ax,ax
 804833e:       66 90                   xchg   ax,ax
 8048340:       8b 1c 24                mov    ebx,DWORD PTR [esp]
 8048343:       c3                      ret
 8048344:       66 90                   xchg   ax,ax
 8048346:       66 90                   xchg   ax,ax
 8048348:       66 90                   xchg   ax,ax
 804834a:       66 90                   xchg   ax,ax
 804834c:       66 90                   xchg   ax,ax
 804834e:       66 90                   xchg   ax,ax
 8048350:       b8 1f a0 04 08          mov    eax,0x804a01f
 8048355:       2d 1c a0 04 08          sub    eax,0x804a01c
 804835a:       83 f8 06                cmp    eax,0x6
 804835d:       76 1a                   jbe    8048379 <__libc_start_main@plt+0x89>
 804835f:       b8 00 00 00 00          mov    eax,0x0
 8048364:       85 c0                   test   eax,eax
 8048366:       74 11                   je     8048379 <__libc_start_main@plt+0x89>
 8048368:       55                      push   ebp
 8048369:       89 e5                   mov    ebp,esp
 804836b:       83 ec 14                sub    esp,0x14
 804836e:       68 1c a0 04 08          push   0x804a01c
 8048373:       ff d0                   call   eax
 8048375:       83 c4 10                add    esp,0x10
 8048378:       c9                      leave
 8048379:       f3 c3                   repz ret
 804837b:       90                      nop
 804837c:       8d 74 26 00             lea    esi,[esi+eiz*1+0x0]
 8048380:       b8 1c a0 04 08          mov    eax,0x804a01c
 8048385:       2d 1c a0 04 08          sub    eax,0x804a01c
 804838a:       c1 f8 02                sar    eax,0x2
 804838d:       89 c2                   mov    edx,eax
 804838f:       c1 ea 1f                shr    edx,0x1f
 8048392:       01 d0                   add    eax,edx
 8048394:       d1 f8                   sar    eax,1
 8048396:       74 1b                   je     80483b3 <__libc_start_main@plt+0xc3>
 8048398:       ba 00 00 00 00          mov    edx,0x0
 804839d:       85 d2                   test   edx,edx
 804839f:       74 12                   je     80483b3 <__libc_start_main@plt+0xc3>
 80483a1:       55                      push   ebp
 80483a2:       89 e5                   mov    ebp,esp
 80483a4:       83 ec 10                sub    esp,0x10
 80483a7:       50                      push   eax
 80483a8:       68 1c a0 04 08          push   0x804a01c
 80483ad:       ff d2                   call   edx
 80483af:       83 c4 10                add    esp,0x10
 80483b2:       c9                      leave
 80483b3:       f3 c3                   repz ret
 80483b5:       8d 74 26 00             lea    esi,[esi+eiz*1+0x0]
 80483b9:       8d bc 27 00 00 00 00    lea    edi,[edi+eiz*1+0x0]
 80483c0:       80 3d 1c a0 04 08 00    cmp    BYTE PTR ds:0x804a01c,0x0
 80483c7:       75 13                   jne    80483dc <__libc_start_main@plt+0xec>
 80483c9:       55                      push   ebp
 80483ca:       89 e5                   mov    ebp,esp
 80483cc:       83 ec 08                sub    esp,0x8
 80483cf:       e8 7c ff ff ff          call   8048350 <__libc_start_main@plt+0x60>
 80483d4:       c6 05 1c a0 04 08 01    mov    BYTE PTR ds:0x804a01c,0x1
 80483db:       c9                      leave
 80483dc:       f3 c3                   repz ret
 80483de:       66 90                   xchg   ax,ax
 80483e0:       b8 10 9f 04 08          mov    eax,0x8049f10
 80483e5:       8b 10                   mov    edx,DWORD PTR [eax]
 80483e7:       85 d2                   test   edx,edx
 80483e9:       75 05                   jne    80483f0 <__libc_start_main@plt+0x100>
 80483eb:       eb 93                   jmp    8048380 <__libc_start_main@plt+0x90>
 80483ed:       8d 76 00                lea    esi,[esi+0x0]
 80483f0:       ba 00 00 00 00          mov    edx,0x0
 80483f5:       85 d2                   test   edx,edx
 80483f7:       74 f2                   je     80483eb <__libc_start_main@plt+0xfb>
 80483f9:       55                      push   ebp
 80483fa:       89 e5                   mov    ebp,esp
 80483fc:       83 ec 14                sub    esp,0x14
 80483ff:       50                      push   eax
 8048400:       ff d2                   call   edx
 8048402:       83 c4 10                add    esp,0x10
 8048405:       c9                      leave
 8048406:       e9 75 ff ff ff          jmp    8048380 <__libc_start_main@plt+0x90>
 804840b:       8d 4c 24 04             lea    ecx,[esp+0x4]
 804840f:       83 e4 f0                and    esp,0xfffffff0
 8048412:       ff 71 fc                push   DWORD PTR [ecx-0x4]
 8048415:       55                      push   ebp
 8048416:       89 e5                   mov    ebp,esp
 8048418:       51                      push   ecx
 8048419:       83 ec 04                sub    esp,0x4
 804841c:       83 ec 0c                sub    esp,0xc
 804841f:       68 c0 84 04 08          push   0x80484c0
 8048424:       e8 b7 fe ff ff          call   80482e0 <puts@plt>
 8048429:       83 c4 10                add    esp,0x10
 804842c:       b8 00 00 00 00          mov    eax,0x0
 8048431:       8b 4d fc                mov    ecx,DWORD PTR [ebp-0x4]
 8048434:       c9                      leave
 8048435:       8d 61 fc                lea    esp,[ecx-0x4]
 8048438:       c3                      ret
 8048439:       66 90                   xchg   ax,ax
 804843b:       66 90                   xchg   ax,ax
 804843d:       66 90                   xchg   ax,ax
 804843f:       90                      nop
 8048440:       55                      push   ebp
 8048441:       57                      push   edi
 8048442:       56                      push   esi
 8048443:       53                      push   ebx
 8048444:       e8 f7 fe ff ff          call   8048340 <__libc_start_main@plt+0x50>
 8048449:       81 c3 b7 1b 00 00       add    ebx,0x1bb7
 804844f:       83 ec 0c                sub    esp,0xc
 8048452:       8b 6c 24 20             mov    ebp,DWORD PTR [esp+0x20]
 8048456:       8d b3 0c ff ff ff       lea    esi,[ebx-0xf4]
 804845c:       e8 47 fe ff ff          call   80482a8 <puts@plt-0x38>
 8048461:       8d 83 08 ff ff ff       lea    eax,[ebx-0xf8]
 8048467:       29 c6                   sub    esi,eax
 8048469:       c1 fe 02                sar    esi,0x2
 804846c:       85 f6                   test   esi,esi
 804846e:       74 25                   je     8048495 <__libc_start_main@plt+0x1a5>
 8048470:       31 ff                   xor    edi,edi
 8048472:       8d b6 00 00 00 00       lea    esi,[esi+0x0]
 8048478:       83 ec 04                sub    esp,0x4
 804847b:       ff 74 24 2c             push   DWORD PTR [esp+0x2c]
 804847f:       ff 74 24 2c             push   DWORD PTR [esp+0x2c]
 8048483:       55                      push   ebp
 8048484:       ff 94 bb 08 ff ff ff    call   DWORD PTR [ebx+edi*4-0xf8]
 804848b:       83 c7 01                add    edi,0x1
 804848e:       83 c4 10                add    esp,0x10
 8048491:       39 fe                   cmp    esi,edi
 8048493:       75 e3                   jne    8048478 <__libc_start_main@plt+0x188>
 8048495:       83 c4 0c                add    esp,0xc
 8048498:       5b                      pop    ebx
 8048499:       5e                      pop    esi
 804849a:       5f                      pop    edi
 804849b:       5d                      pop    ebp
 804849c:       c3                      ret
 804849d:       8d 76 00                lea    esi,[esi+0x0]
 80484a0:       f3 c3                   repz ret

Déassemblage de la section .fini :

080484a4 <.fini>:
 80484a4:       53                      push   ebx
 80484a5:       83 ec 08                sub    esp,0x8
 80484a8:       e8 93 fe ff ff          call   8048340 <__libc_start_main@plt+0x50>
 80484ad:       81 c3 53 1b 00 00       add    ebx,0x1b53
 80484b3:       83 c4 08                add    esp,0x8
 80484b6:       5b                      pop    ebx
 80484b7:       c3                      ret
```

Première question, où est passé la méthode _main\(\)_ ? En effet, plus aucune information de ce type n'est présent. En fait, la méthode _main\(\)_ est présente au sein de la section _.text_, qui contient les instructions à exécuter. Il est quand même possible de retrouver la méthode _main\(\)_ grâce à ces quelques lignes situées au début de la section :

```text
8048327:       68 0b 84 04 08          push   0x804840b
804832c:       e8 bf ff ff ff          call   80482f0 <__libc_start_main@plt>
```

L'adresse _0x0804840b_ qui est empilée à la première ligne, correspond à l'adresse de début de la méthode principale. Tout n'est donc pas perdu, mais la suppression de la table des symboles et des informations d'adresses complique la tâche de l'analyse, car bien sur, cela va plus loin que l'absence du début de la méthode _main\(\)_.

