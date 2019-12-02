# Correction - Challenge n°1

Une fois en possession de l'exécutable à analyser, la première étape est de déterminer pour quelle architecture le programme a-t-il été compilé. Pour cela, l'utilitaire _file_ est suffisant :

```text
$ file ./challenge01
./challenge01: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=fd6b5997b1102ed29d726371882b17844efade8f, not stripped
```

Ce binaire a donc été compilé pour une architecture 32 bits. Une première tentative d'exécution du binaire donne :

```text
$ ./challenge01
Usage : ./challenge01 password
```

Le programme attend donc un paramètre nommé _password_. Il est possible de tester un mot de passe quelconque afin d'analyser son comportement :

```text
$ ./challenge01 whatever
Try again ...
```

Pas de chance, le mot de passe n'étant pas correct \(sauf à avoir beaucoup de chance :D\), un message invite à réessayer. Après cette première étape d'exécution, il est possible d'utiliser l'utilitaire _strings_, qui va afficher toutes les chaînes de caractères imprimables du binaire, dont la chaîne "Usage : ./challenge01 password" ainsi que la chaîne "Try again ..." :

```text
$ strings ./challenge01
/lib/ld-linux.so.2
libc.so.6
_IO_stdin_used
exit
puts
strcmp
__libc_start_main
__gmon_start__
GLIBC_2.0
PTRhP
QVhk
UWVS
t$,U
[^_]
Usage : ./challenge01 password
String_is_too_easy!
Congratz !
Try again ...
;*2$"
GCC: (Debian 6.3.0-18+deb9u1) 6.3.0 20170516
crtstuff.c
__JCR_LIST__
deregister_tm_clones
__do_global_dtors_aux
completed.6587
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
challenge01.c
__FRAME_END__
__JCR_END__
__init_array_end
_DYNAMIC
__init_array_start
__GNU_EH_FRAME_HDR
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
strcmp@@GLIBC_2.0
__x86.get_pc_thunk.bx
_edata
__data_start
puts@@GLIBC_2.0
__gmon_start__
exit@@GLIBC_2.0
__dso_handle
_IO_stdin_used
__libc_start_main@@GLIBC_2.0
__libc_csu_init
_fp_hw
__bss_start
main
__TMC_END__
.symtab
.strtab
.shstrtab
.interp
.note.ABI-tag
.note.gnu.build-id
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rel.dyn
.rel.plt
.init
.plt.got
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.jcr
.dynamic
.got.plt
.data
.bss
.comment
```

Ligne 4, 5 et 6 indiquent l'utilisation par le programme des méthodes _exit\(\)_, _puts\(\)_ et _strcmp\(\)_. Les lignes 15 à 18 sont intéressantes, en effet les chaînes "Usage : ./challenge01 password" et "Try again ..." sont bien présentes ainsi que deux autres : "String\_is\_too\_easy!" et "Congratz !". La dernière est sans doute le message affiché quand l'utilisateur renseigne un mot de passe valide. Il ne reste donc plus qu'à tenter l'autre chaîne en tant que mot de passe :

```text
$ ./challenge01 String_is_too_easy!
Congratz !
```

Challenge validé, bien joué !

Il est quand même possible d'aller plus loin dans l'analyse du binaire et d'investiguer au niveau du code assembleur grâce à _objdump_ \(l'analyse de la méthode _main\(\)_ sera suffisant ici\) :

```text
$ objdump -d challenge01 -M intel
0804846b <main>:
01: 804846b:       8d 4c 24 04             lea    ecx,[esp+0x4]
02: 804846f:       83 e4 f0                and    esp,0xfffffff0
03: 8048472:       ff 71 fc                push   DWORD PTR [ecx-0x4]
04: 8048475:       55                      push   ebp
05: 8048476:       89 e5                   mov    ebp,esp
06: 8048478:       51                      push   ecx
07: 8048479:       83 ec 14                sub    esp,0x14
08: 804847c:       89 c8                   mov    eax,ecx
09: 804847e:       83 38 02                cmp    DWORD PTR [eax],0x2
10: 8048481:       74 1a                   je     804849d <main+0x32>
11: 8048483:       83 ec 0c                sub    esp,0xc
12: 8048486:       68 70 85 04 08          push   0x8048570
13: 804848b:       e8 a0 fe ff ff          call   8048330 <puts@plt>
14: 8048490:       83 c4 10                add    esp,0x10
15: 8048493:       83 ec 0c                sub    esp,0xc
16: 8048496:       6a 01                   push   0x1
17: 8048498:       e8 a3 fe ff ff          call   8048340 <exit@plt>
18: 804849d:       c7 45 f4 8f 85 04 08    mov    DWORD PTR [ebp-0xc],0x804858f
19: 80484a4:       8b 40 04                mov    eax,DWORD PTR [eax+0x4]
20: 80484a7:       83 c0 04                add    eax,0x4
21: 80484aa:       8b 00                   mov    eax,DWORD PTR [eax]
22: 80484ac:       83 ec 08                sub    esp,0x8
23: 80484af:       ff 75 f4                push   DWORD PTR [ebp-0xc]
24: 80484b2:       50                      push   eax
25: 80484b3:       e8 68 fe ff ff          call   8048320 <strcmp@plt>
26: 80484b8:       83 c4 10                add    esp,0x10
27: 80484bb:       85 c0                   test   eax,eax
28: 80484bd:       75 12                   jne    80484d1 <main+0x66>
29: 80484bf:       83 ec 0c                sub    esp,0xc
30: 80484c2:       68 a3 85 04 08          push   0x80485a3
31: 80484c7:       e8 64 fe ff ff          call   8048330 <puts@plt>
32: 80484cc:       83 c4 10                add    esp,0x10
33: 80484cf:       eb 10                   jmp    80484e1 <main+0x76>
34: 80484d1:       83 ec 0c                sub    esp,0xc
35: 80484d4:       68 ae 85 04 08          push   0x80485ae
36: 80484d9:       e8 52 fe ff ff          call   8048330 <puts@plt>
37: 80484de:       83 c4 10                add    esp,0x10
38: 80484e1:       b8 00 00 00 00          mov    eax,0x0
39: 80484e6:       8b 4d fc                mov    ecx,DWORD PTR [ebp-0x4]
40: 80484e9:       c9                      leave
41: 80484ea:       8d 61 fc                lea    esp,[ecx-0x4]
42: 80484ed:       c3                      ret
```

Afin de simplifier la correction, seules les lignes significatives seront analysées. Pour les néophytes, il est recommandé de revenir sur cette correction après avoir terminé la lecture des autres articles dédiés au reverse engineering.

Les lignes 9 et 10 permettent la vérification du bon nombre d'argument sur la ligne de commande lors du lancement du programme. Le registre _EAX_ est comparé à la valeur 2 ce qui signifie que le programme attend 2 arguments. Le premier argument est le nom du programme, soit "./challenge01" \(plus exactement, la chaîne contient également le chemin absolu du programme\) et le second argument est le mot de passe renseigné \(pour plus de détails voir [Les arguments de la ligne de commande pour les programmes C](https://www.areaprog.com/c/article-229-argc-et-argv-utilisation-des-parametres-de-la-ligne-de-commande)\). Si le nombre d'arguments n'est pas égal à 2, alors une chaîne de caractères, stockée à l'adresse _0x08048570_, ligne 12, est affichée grâce à la fonction _puts\(\)_ à la ligne 13. Le problème est donc maintenant de connaître la chaîne stockée à l'adresse _0x08048570_. L'utilitaire _objdump_ avec les arguments suivants peut faire cela :

```text
$ objdump -s -j .rodata ./challenge01
```

**Note :** La section _.rodata_ \(read-only data\) est la section stockant les variables de types constantes \(lecture seule\).

```text
./challenge01:     format de fichier elf32-i386

Contenu de la section .rodata :
 8048568 03000000 01000200 55736167 65203a20  ........Usage :
 8048578 2e2f6368 616c6c65 6e676530 31207061  ./challenge01 pa
 8048588 7373776f 72640053 7472696e 675f6973  ssword.String_is
 8048598 5f746f6f 5f656173 79210043 6f6e6772  _too_easy!.Congr
 80485a8 61747a20 21005472 79206167 61696e20  atz !.Try again
 80485b8 2e2e2e00                             ....
```

L'adresse indiquée à la première ligne se rapproche beaucoup de celle qui nous intéresse mais ce n'est pas encore cela. Il faut donc pousser un peu plus loin le raisonnement et comprendre que l'adresse _0x08048568_ contient la valeur héxa "03", l'adresse _0x08048569_ contient "00", _0x0804856a_ contient également "00", pour enfin arriver à l'adresse _0x08048570_ qui contient "55_"_, soit le caractère "U". Mais il faut bien se souvenir que dans un programme, la fin d'une chaîne est représentée par le caractère fin de chaîne ayant pour valeur "\0" \(0x00\). Le prochain caractère de ce type est disponible à l'adresse _0x0804858e_. La chaîne qui sera donc affichée grâce à la méthode _puts\(\)_ est en définitif : "Usage : ./challenge01 password". Le traitement du cas où le nombre d'arguments n'est pas correct est maintenant terminé.

Dans le cas ou il y a bien 2 arguments, un saut dans le programme est effectué à l'adresse _0x0804849d_. La ligne 18 copie le contenu l'adresse _0x0804858f_ dans _\[ebp-0xc\]_. Qu'y a-t-il donc à cette adresse ? L'exécution des étapes précédentes permettent de savoir que cette adresse est en fait le début de la chaîne "String\_is\_too\_easy!" :

```text
 8048588 7373776f 72640053 7472696e 675f6973  ssword.String_is
 8048598 5f746f6f 5f656173 79210043 6f6e6772  _too_easy!.Congr
```

Ligne 25 la méthode _strcmp\(\)_, prend en arguments deux chaînes de caractères à comparer \(les deux _push_ à la ligne 23 et 24\). Le premier _push_ \(bien se rappeler qu'il s'agit donc du dernier paramètre de la fonction\) est la chaîne stockée à _\[ebp-0xc\]_ soit "String_is\_too\_easy!"_. Le premier argument est le registre _EAX_, qui contient en fait ici l'adresse du second argument passé en ligne de commande, c'est-à-dire le mot de passe renseigné par l'utilisateur.

La ligne 27 effectue un test intéressant :

```text
test   eax,eax
```

Il faut savoir que si les deux chaînes sont équivalentes, alors la fonction _strcmp\(\)_ retourne la valeur 0. La ligne 27 permet donc de tester que le retour de la fonction \(contenu dans _EAX_\) est égale à 0. Cette instruction est comparable à l'instruction suivante \(mais en plus rapide, il s'agit d'une optimisation\) :

```text
cmp eax, 0
```

Si le résultat du test est différent de 0, la ligne 28 effectue un saut vers l'adresse _0x080484d1_. Le programme va alors afficher la chaîne stockée à l'adresse _0x080485ae_, soit "Try again ..." :

```text
 80485a8 61747a20 21005472 79206167 61696e20  atz !.Try again
 80485b8 2e2e2e00                             ....
```

En cas d'égalité des chaînes, le programme affiche la chaîne stockée à l'adresse _0x080485a3_, soit "Congratz !" :

```text
 8048598 5f746f6f 5f656173 79210043 6f6e6772  _too_easy!.Congr
 80485a8 61747a20 21005472 79206167 61696e20  atz !.Try again
```

