# Correction - Challenge n°3

Comme d'habitude, l'utilisation de l'outil _file_ permet de connaître un peu mieux notre cible :

```text
$ file ./challenge03
./challenge03: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=666e14b0c6b4bbd0ce1b341cc7ab5c05e2d9ad50, not stripped
```

Ce binaire a donc été compilé pour une architecture 32 bits. Une première tentative d'exécution du binaire donne :

```text
$ ./challenge03
Usage : ./challenge03 password
```

Cela ressemble fort aux précédents challenges. L'utilisation de l'outil _strings_ permet d'analyser les chaînes de caractères stockées dans le binaire :

```text
$ strings ./challenge03
/lib/ld-linux.so.2
libc.so.6
_IO_stdin_used
exit
puts
strlen
__libc_start_main
__gmon_start__
GLIBC_2.0
PTRh
QVhk
UWVS
t$,U
[^_]
Usage : ./challenge03 password
1&5&10&*0/*%&
Try harder !
Well done !
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
challenge03.c
__FRAME_END__
__JCR_END__
__init_array_end
_DYNAMIC
__init_array_start
__GNU_EH_FRAME_HDR
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
checkPassword
__x86.get_pc_thunk.bx
_edata
__data_start
puts@@GLIBC_2.0
__gmon_start__
exit@@GLIBC_2.0
__dso_handle
initPassword
_IO_stdin_used
strlen@@GLIBC_2.0
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

A la ligne 17, une chaîne de caractère attire l'attention. Il est possible de la tester directement en tant que mot de passe, avec un peu de chance cela peut fonctionner \(attention à bien entourer le mot de passe de guillemets pour ne pas que les caractères soient interprétés par la ligne de commande\) ... :

```text
$ ./challenge03 "1&5&10&*0/*%&"
Try harder !
```

Mais sans succès. Prochaine étape, la décompilation du binaire grâce à _objdump_ :

```text
$ objdump -d challenge03 -M intel
```

Cette fois, en plus de la méthode _main\(\)_, deux autres méthodes sont présentes : la méthode _initPassword\(\)_ et _checkPassword\(\)_. Ces deux méthodes doivent sans doute être appelées par la méthode _main\(\)_, l'analyse peut alors commencer par celle-ci :

```text
00: 0804846b <main>:
01: 804846b:       8d 4c 24 04             lea    ecx,[esp+0x4]
02: 804846f:       83 e4 f0                and    esp,0xfffffff0
03: 8048472:       ff 71 fc                push   DWORD PTR [ecx-0x4]
04: 8048475:       55                      push   ebp
05: 8048476:       89 e5                   mov    ebp,esp
06: 8048478:       53                      push   ebx
07: 8048479:       51                      push   ecx
08: 804847a:       83 ec 10                sub    esp,0x10
09: 804847d:       89 cb                   mov    ebx,ecx
10: 804847f:       83 3b 02                cmp    DWORD PTR [ebx],0x2
11: 8048482:       74 1a                   je     804849e <main+0x33>
12: 8048484:       83 ec 0c                sub    esp,0xc
13: 8048487:       68 10 86 04 08          push   0x8048610
14: 804848c:       e8 8f fe ff ff          call   8048320 <puts@plt>
15: 8048491:       83 c4 10                add    esp,0x10
16: 8048494:       83 ec 0c                sub    esp,0xc
17: 8048497:       6a 01                   push   0x1
18: 8048499:       e8 92 fe ff ff          call   8048330 <exit@plt>
19: 804849e:       83 ec 0c                sub    esp,0xc
20: 80484a1:       8d 45 ec                lea    eax,[ebp-0x14]
21: 80484a4:       50                      push   eax
22: 80484a5:       e8 2a 00 00 00          call   80484d4 <initPassword>
23: 80484aa:       83 c4 10                add    esp,0x10
24: 80484ad:       8b 43 04                mov    eax,DWORD PTR [ebx+0x4]
25: 80484b0:       83 c0 04                add    eax,0x4
26: 80484b3:       8b 00                   mov    eax,DWORD PTR [eax]
27: 80484b5:       83 ec 08                sub    esp,0x8
28: 80484b8:       50                      push   eax
29: 80484b9:       8d 45 ec                lea    eax,[ebp-0x14]
30: 80484bc:       50                      push   eax
31: 80484bd:       e8 32 00 00 00          call   80484f4 <checkPassword>
32: 80484c2:       83 c4 10                add    esp,0x10
33: 80484c5:       b8 00 00 00 00          mov    eax,0x0
34: 80484ca:       8d 65 f8                lea    esp,[ebp-0x8]
35: 80484cd:       59                      pop    ecx
36: 80484ce:       5b                      pop    ebx
37: 80484cf:       5d                      pop    ebp
38: 80484d0:       8d 61 fc                lea    esp,[ecx-0x4]
39: 80484d3:       c3                      ret
```

Les lignes 1 à 5 vont permettre de récupérer les arguments de la ligne de commande, _ECX_ va contenir le nombre d'arguments et _EBX_ la liste des arguments. Ces deux registres sont sauvegardés sur la pile aux lignes 6 et 7. Ligne 9, _ECX_ est copié dans _EBX_ puis, ligne 10, _EBX_ est comparé à la valeur 2, c'est-à-dire au nombre attendu d'arguments. Si l'utilisateur ne renseigne pas le nombre correct d'arguments, alors la sortie du programme s'effectue en affichant un message via la méthode _puts\(\)_ \(ligne 14\), puis le programme s'arrête par l'appel à la méthode _exit\(\)_ \(ligne 18\).

Dans le cas contraire, l'exécution se poursuit ligne 19, qui réserve de l'espace mémoire sur la pile. L'instruction de la ligne 20 charge l'adresse correspondant à _\[epb-0x14\]_ dans _EAX_, puis est empilée. Il s'agit donc ici du seul paramètre de la fonction _initPassword\(\)_ appelée à la ligne 22. Voici le code assembleur de cette fonction :

```text
00: 080484d4 <initPassword>:
01: 80484d4:       55                      push   ebp
02: 80484d5:       89 e5                   mov    ebp,esp
03: 80484d7:       8b 45 08                mov    eax,DWORD PTR [ebp+0x8]
04: 80484da:       c6 40 04 43             mov    BYTE PTR [eax+0x4],0x43
05: 80484de:       8b 45 08                mov    eax,DWORD PTR [ebp+0x8]
06: 80484e1:       c7 40 08 2f 86 04 08    mov    DWORD PTR [eax+0x8],0x804862f
07: 80484e8:       8b 45 08                mov    eax,DWORD PTR [ebp+0x8]
08: 80484eb:       c7 00 0d 00 00 00       mov    DWORD PTR [eax],0xd
09: 80484f1:       90                      nop
10: 80484f2:       5d                      pop    ebp
11: 80484f3:       c3                      ret
```

Ici, l'analyse doit se faire ici en prenant en compte les lignes 3 à 8. Il s'agit de l'utilisation d'une structure \(le pointeur de cette structure a été passé en paramètre lors de l'appel à _initPassword\(\)_\). Le registre _EAX_ est donc utilisé comme base de la structure, la ligne 3 et 4 affectent la valeur _0x43_ à _\[eax+0x4\]_. Les lignes 5 et 6 affectent l'adresse _0x0804862f_ à _\[eax+0x8\]_ et les lignes 7 et 8 affectent la valeur _0xd_ à _\[eax\]_. La fonction _initPassword\(\)_ ne comporte aucune valeur de retour, la variable passée en paramètre étant un pointeur, son contenu est directement modifié sans avoir à utiliser le retour de la fonction. Il serait intéressant de connaître le contenu de l'adresse _0x0804862f_ stocké dans la structure. Pour cela :

```text
objdump -s -j .rodata ./challenge03

./challenge03:     format de fichier elf32-i386

Contenu de la section .rodata :
 8048608 03000000 01000200 55736167 65203a20  ........Usage :
 8048618 2e2f6368 616c6c65 6e676530 33207061  ./challenge03 pa
 8048628 7373776f 72640031 26352631 30262a30  ssword.1&5&10&*0
 8048638 2f2a2526 00547279 20686172 64657220  /*%&.Try harder
 8048648 21005765 6c6c2064 6f6e6520 2100      !.Well done !.
```

L'adresse _0x0804862f_ est l'adresse d'une chaîne de caractères qui est donc, en héxadécimal : 0x31/0x26/0x35/0x26/0x31/0x30/0x26/0x2a/0x30/0x2f/0x2a/0x25/0x26/0x00, soit "1&5&10&_0/_%&" \(le dernier caractère en héxadécimal est le caractère NULL indiquant la fin de la chaîne\). Ce mot de passe ne fonctionne pas directement, il a déjà été testé au début de l'analyse. Un traitement doit sans doute donc être effectué par le programme afin de retrouver le mot de passe originel.

De retour dans la méthode _main\(\)_, l'analyse se poursuit jusqu'à l'appel de la prochaine méthode _checkPassword\(\)_ :

```text
00: 0804846b <main>:
23: 80484aa:       83 c4 10                add    esp,0x10
24: 80484ad:       8b 43 04                mov    eax,DWORD PTR [ebx+0x4]
25: 80484b0:       83 c0 04                add    eax,0x4
26: 80484b3:       8b 00                   mov    eax,DWORD PTR [eax]
27: 80484b5:       83 ec 08                sub    esp,0x8
28: 80484b8:       50                      push   eax
29: 80484b9:       8d 45 ec                lea    eax,[ebp-0x14]
30: 80484bc:       50                      push   eax
31: 80484bd:       e8 32 00 00 00          call   80484f4 <checkPassword>
```

Les lignes 24, 25 et 26 ont pour objectif de charger l'adresse de la chaîne renseignée par l'utilisateur. Pour rappel, _\[ebx\]_ contient le nombre d'arguments sur la ligne de commande, _\[ebx+0x4\]_ contient l'adresse du premier argument, soit le nom du programme et _\[ebx+0x8\]_ contient l'adresse de l'argument saisi par l'utilisateur. Cet argument est mis sur la pile à la ligne 28. Une autre donnée est empilée à la ligne 30, il s'agit du pointeur de structure rencontrée plus tôt. Il est maintenant clair que la méthode _checkPassword\(\)_ accepte deux paramètres : un premier paramètre qui est le pointeur de la structure contenant la chaîne "1&5&10&_0/_%&", la valeur _0xd_ ainsi que la valeur _0x43_ et un second paramètre qui est le mot de passe saisi par l'utilisateur. Voici le code assembleur de la méthode _checkPassword\(\)_ :

```text
00: 080484f4 <checkPassword>:
01: 80484f4:       55                      push   ebp
02: 80484f5:       89 e5                   mov    ebp,esp
03: 80484f7:       83 ec 18                sub    esp,0x18
04: 80484fa:       83 ec 0c                sub    esp,0xc
05: 80484fd:       ff 75 0c                push   DWORD PTR [ebp+0xc]
06: 8048500:       e8 3b fe ff ff          call   8048340 <strlen@plt>
07: 8048505:       83 c4 10                add    esp,0x10
08: 8048508:       89 c2                   mov    edx,eax
09: 804850a:       8b 45 08                mov    eax,DWORD PTR [ebp+0x8]
10: 804850d:       8b 00                   mov    eax,DWORD PTR [eax]
11: 804850f:       39 c2                   cmp    edx,eax
12: 8048511:       74 12                   je     8048525 <checkPassword+0x31>
13: 8048513:       83 ec 0c                sub    esp,0xc
14: 8048516:       68 3d 86 04 08          push   0x804863d
15: 804851b:       e8 00 fe ff ff          call   8048320 <puts@plt>
16: 8048520:       83 c4 10                add    esp,0x10
17: 8048523:       eb 5f                   jmp    8048584 <checkPassword+0x90>
18: 8048525:       c7 45 f4 00 00 00 00    mov    DWORD PTR [ebp-0xc],0x0
19: 804852c:       eb 3c                   jmp    804856a <checkPassword+0x76>
20: 804852e:       8b 55 f4                mov    edx,DWORD PTR [ebp-0xc]
21: 8048531:       8b 45 0c                mov    eax,DWORD PTR [ebp+0xc]
22: 8048534:       01 d0                   add    eax,edx
23: 8048536:       0f b6 10                movzx  edx,BYTE PTR [eax]
24: 8048539:       8b 45 08                mov    eax,DWORD PTR [ebp+0x8]
25: 804853c:       8b 48 08                mov    ecx,DWORD PTR [eax+0x8]
26: 804853f:       8b 45 f4                mov    eax,DWORD PTR [ebp-0xc]
27: 8048542:       01 c8                   add    eax,ecx
28: 8048544:       0f b6 08                movzx  ecx,BYTE PTR [eax]
29: 8048547:       8b 45 08                mov    eax,DWORD PTR [ebp+0x8]
30: 804854a:       0f b6 40 04             movzx  eax,BYTE PTR [eax+0x4]
31: 804854e:       31 c8                   xor    eax,ecx
32: 8048550:       38 c2                   cmp    dl,al
33: 8048552:       74 12                   je     8048566 <checkPassword+0x72>
34: 8048554:       83 ec 0c                sub    esp,0xc
35: 8048557:       68 3d 86 04 08          push   0x804863d
36: 804855c:       e8 bf fd ff ff          call   8048320 <puts@plt>
37: 8048561:       83 c4 10                add    esp,0x10
38: 8048564:       eb 1e                   jmp    8048584 <checkPassword+0x90>
39: 8048566:       83 45 f4 01             add    DWORD PTR [ebp-0xc],0x1
40: 804856a:       8b 45 08                mov    eax,DWORD PTR [ebp+0x8]
41: 804856d:       8b 00                   mov    eax,DWORD PTR [eax]
42: 804856f:       3b 45 f4                cmp    eax,DWORD PTR [ebp-0xc]
43: 8048572:       7f ba                   jg     804852e <checkPassword+0x3a>
44: 8048574:       83 ec 0c                sub    esp,0xc
45: 8048577:       68 4a 86 04 08          push   0x804864a
46: 804857c:       e8 9f fd ff ff          call   8048320 <puts@plt>
47: 8048581:       83 c4 10                add    esp,0x10
48: 8048584:       c9                      leave
49: 8048585:       c3                      ret
```

Il est facile d'identifier que la valeur _\[ebp+0xc\]_ correspond au mot de passe renseigné par l'utilisateur, mot de passe qui est ensuite passé en paramètre de la fonction _strlen\(\)_ afin d'en calculer sa longueur. Cette longueur est comparée au premier champ de la structure \(_\[eax\]_ ou _\[ebp+0x8\]_, ligne 9\) qui a pour valeur _0xd_, soit 13. En effet, _\[ebp+0x8\]_ correspond au second empilement lors de l'appel à la méthode _checkPassword\(\)_. Le mot de passe doit donc posséder une longueur de 13 caractères. La ligne 18 affecte la valeur 0 à _\[ebp-0xc\]_, puis un saut inconditionnel est effectué jusqu’à l'instruction de la ligne 40. Les lignes 40 à 42 comparent la longueur de la chaîne \(stockée dans la première variable de la structure\) à la valeur 0 précédemment mise sur la pile, à _\[ebp-0xc\]_ exactement. Cela ressemble fort à une boucle, comparant le compteur _\[ebp-0xc\]_, initialisé à 0, à la longueur de la chaîne. La ligne 43 confirme l'hypothèse, un saut "si inférieur" est effectué, reprenant l'exécution à la ligne 20. Le saut "si inférieur" correspond bien aux boucles du type :

```c
for(int compteur = 0 ; compteur < longueurChaîne ; compteur++) {
 // Traitement
}
```

Les lignes 21 à 23 chargent les caractères du mot de passe de la ligne de commande. Pour chaque tour de boucle, la chaîne _avance_ de la valeur du compteur \(ligne 22\) puis le caractère est stocké dans le registre _EDX_ \(ligne 23\). La ligne 25 charge dans le registre _ECX_ l'adresse de la chaîne présente dans la structure étudiée plus haut. Une addition entre le compteur et le pointeur est effectuée à la ligne 27, permettant ainsi de faire avancer le _curseur_ sur la chaîne. Cela pourrait être représenté comme ceci :

```c
for(int compteur = 0 ; compteur < longueurChaîne ; compteur++) {
  char edx = motDePasseUtilisateur[compteur]; // le caractère évolue à chaque boucle en incrémentant l'adresse par le compteur puis stocké dans _EDX_ 
  char ecx = structure->motDePasseDansStructure[compteur]; // le caractère évolue à chaque boucle en incrémentant l'adresse par le compteur puis stocké dans _EDX_ 
}
```

Les lignes 29 et 30 récupèrent dans _EAX_ la valeur _0x43_ présent dans la structure. Ce qui revient à :

```c
for(int compteur = 0 ; compteur < longueurChaîne ; compteur++) {
  char edx = motDePasseUtilisateur[compteur]; // le caractère évolue à chaque boucle en incrémentant l'adresse par le compteur puis stocké dans _EDX_ 
  char ecx = structure->motDePasseDansStructure[compteur]; // le caractère évolue à chaque boucle en incrémentant l'adresse par le compteur puis stocké dans _EDX_ 
  val eax = structure->0x43;
}
```

Ensuite, les lignes 30 et 31 effectuent respectivement un ou-exclusif puis une comparaison entre le résultat \(du ou-exclusif\) et la valeur qui représente le caractère traité au niveau du mot de passe renseigné par l'utilisateur :

```c
for(int compteur = 0 ; compteur < longueurChaîne ; compteur++) {
  char edx = motDePasseUtilisateur[compteur]; // le caractère évolue à chaque boucle en incrémentant l'adresse par le compteur puis stocké dans _EDX_ 
  char ecx = structure->motDePasseDansStructure[compteur]; // le caractère évolue à chaque boucle en incrémentant l'adresse par le compteur puis stocké dans _EDX_ 
  val eax = structure->0x43;

  if((ecx ^ eax) != edx) {
    exit();
  }
}
```

Si le caractère _xoré_ du mot de passe contenu dans la structure n'est pas le même que celui du mot de passe renseigné alors le programme se termine par l'intermédiaire d'un saut inconditionnel \(ligne 38\). Cette boucle se répète pour chaque caractère du mot de passe. Il ne reste donc plus qu'à effectuer un ou-exclusif entre la valeur _0x43_ et chaque caractère du mot de passe de la structure, soit "1&5&10&_0/_%&" afin de retrouver le mot de passe originel et valider le challenge :

```text
   1    &    5    &    1    0    &    *    0    /    *    %    &     (ASCII)
^
   0x43 0x43 0x43 0x43 0x43 0x43 0x43 0x43 0x43 0x43 0x43 0x43 0x43  (hexadécimal)
=
   0x72 0x65 0x76 0x65 0x72 0x73 0x65 0x69 0x73 0x6c 0x69 0x66 0x65  (hexadécimal)
```

Une petite recherche grâce à une table ASCII donne comme mot de passe : "reverseislife".

```text
$ ./challenge03 reverseislife
Well done !
```

