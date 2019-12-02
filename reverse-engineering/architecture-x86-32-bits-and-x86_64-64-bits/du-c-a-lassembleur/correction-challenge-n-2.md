# Correction - Challenge n°2

D'abord lançons l'utilitaire _file_ :

```text
$ file ./challenge02
./challenge02: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=f2a23fa2f1ee074c73fd38b6064268826addd0dd, not stripped
```

Ce binaire a donc été compilé pour une architecture 32 bits. Une première tentative d'exécution du binaire donne :

```text
$ ./challenge02
Usage : ./challenge02 password
```

Contrairement au premier challenge, l'utilitaire _strings_ ne pas donne pas d'indice sur le mot de passe :

```text
$ strings ./challenge02
/lib/ld-linux.so.2
libc.so.6
_IO_stdin_used
exit
puts
strlen
__libc_start_main
__gmon_start__
GLIBC_2.0
PTRh0
QVhk
<Guq
<iu_
<vuM
<eu;
<Uu)
UWVS
t$,U
[^_]
Usage : ./challenge02 password
Congratz !
Try harder !
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
challenge02.c
__FRAME_END__
__JCR_END__
__init_array_end
_DYNAMIC
__init_array_start
__GNU_EH_FRAME_HDR
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
__x86.get_pc_thunk.bx
_edata
__data_start
puts@@GLIBC_2.0
__gmon_start__
exit@@GLIBC_2.0
__dso_handle
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

La prochaine étape est la décompilation du binaire grâce à _objdump_ :

```text
$ objdump -d challenge02 -M intel
```

Le code assembleur de la méthode _main\(\)_ commence à être conséquent, seules les parties intéressantes seront affichées :

```text
10: 804847f:       83 3b 02                cmp    DWORD PTR [ebx],0x2
11: 8048482:       74 1a                   je     804849e <main+0x33>
12: 8048484:       83 ec 0c                sub    esp,0xc
13: 8048487:       68 50 86 04 08          push   0x8048650
14: 804848c:       e8 8f fe ff ff          call   8048320 <puts@plt>
15: 8048491:       83 c4 10                add    esp,0x10
16: 8048494:       83 ec 0c                sub    esp,0xc
17: 8048497:       6a 01                   push   0x1
18: 8048499:       e8 92 fe ff ff          call   8048330 <exit@plt>
```

La ligne 10 effectue une comparaison entre la valeur 2 et le registre _EBX_, il s'agit du nombre d'arguments passés à la ligne de commande lors de l'exécution. Si le nombre d'arguments n'est pas égal à 2, alors la ligne 13 permet d'afficher un message grâce à un appel à la méthode _puts\(\)_ puis de sortir du programme en appelant la méthode _exit\(\)_ à la ligne 18. Dans le cas où le nombre d'arguments est correct, le programme continue à la ligne 19 :

```text
19: 804849e:       8b 43 04                mov    eax,DWORD PTR [ebx+0x4]
20: 80484a1:       83 c0 04                add    eax,0x4
21: 80484a4:       8b 00                   mov    eax,DWORD PTR [eax]
22: 80484a6:       83 ec 0c                sub    esp,0xc
23: 80484a9:       50                      push   eax
24: 80484aa:       e8 91 fe ff ff          call   8048340 <strlen@plt>
25: 80484af:       83 c4 10                add    esp,0x10
26: 80484b2:       89 45 f4                mov    DWORD PTR [ebp-0xc],eax
27: 80484b5:       83 7d f4 0b             cmp    DWORD PTR [ebp-0xc],0xb
28: 80484b9:       0f 85 ee 00 00 00       jne    80485ad <main+0x142>
```

Le registre _EBX_ contient le nombre d'arguments de la ligne de commande, donc, _\[ebx+0x4\]_, à la ligne 19, contient le premier argument, qui est le nom du programme. La ligne 20 additionne _0x4_ à _\[ebx+0x4\]_, qui est le second argument, c'est-à-dire le mot de passe renseigné lors du lancement du binaire. Cet argument, stocké dans _EAX_ est ensuite empilé, ligne 23, avant l'appel à la méthode _strlen\(\)_ permettant de récupérer la longueur de la chaîne passée en paramètre. Le retour de la méthode est stocké dans le registre _EAX_, qui est comparé en ligne 27, à la valeur _0xb_ \(_EAX_ est en fait d'abord copié dans _\[ebp-0xc\]_ et la comparaison s'effectue avec _\[ebp-0xc\]_\). La longueur du mot de passe attendu est donc de 11 caractères.

Dans le cas d'une mauvaise longueur de mot de passe, un saut est effectué à l'adresse _0x080485ad_, qui affiche le message "Try harder !" avant de s'arrêter. Par contre, une série de comparaisons est effectuées dans le cas ou la longueur de la chaîne est correcte. Voici la première comparaison :

```text
29: 80484bf:       8b 43 04                mov    eax,DWORD PTR [ebx+0x4]
30: 80484c2:       83 c0 04                add    eax,0x4
31: 80484c5:       8b 00                   mov    eax,DWORD PTR [eax]
32: 80484c7:       0f b6 00                movzx  eax,BYTE PTR [eax]
33: 80484ca:       3c 4e                   cmp    al,0x4e
34: 80484cc:       0f 85 db 00 00 00       jne    80485ad <main+0x142>
```

Le registre _EAX_ contient le mot de passe renseigné par l'utilisateur \(soit _\[ebp+0x8\]_\). La ligne 32 récupère le premier octet du mot de passe, c'est-à-dire le premier caractère et le compare à la valeur _0x4e_. Une table ASCII permet de retrouver le caractère correspondant à cette valeur : "N". Les autres comparaisons sont semblables à la précédente, mais elle s'effectue toujours avec l'octet suivant :

```text
35: 80484d2:       8b 43 04                mov    eax,DWORD PTR [ebx+0x4]
36: 80484d5:       83 c0 04                add    eax,0x4
37: 80484d8:       8b 00                   mov    eax,DWORD PTR [eax]
38: 80484da:       83 c0 01                add    eax,0x1
39: 80484dd:       0f b6 00                movzx  eax,BYTE PTR [eax]
40: 80484e0:       3c 65                   cmp    al,0x65
41: 80484e2:       0f 85 c5 00 00 00       jne    80485ad <main+0x142>
```

Ici c'est la ligne 38 qui est intéressante, car cela permet de comparer, non plus le premier caractère, mais le second \(addition d'un octet, soit _0x1_\). La ligne 40 compare donc le second caractère du mot de passe à la valeur _0x65_ qui représente le caractère "e".

Tout le reste du programme se base sur le même principe, jusqu'à trouver le mot de passe complet : "NeverGiveUp".

