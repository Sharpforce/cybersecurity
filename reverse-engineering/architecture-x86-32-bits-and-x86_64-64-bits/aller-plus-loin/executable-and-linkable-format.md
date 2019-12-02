# Executable and Linkable Format

L'utilitaire _file_ sur un exécutable permet, comme déjà vu précédemment, de connaître l'architecture cible de l'exécutable. Par exemple, le binaire _pwd_ se situant dans le répertoire _/bin/_ sous Linux :

```text
$ file /bin/pwd
/bin/pwd: ELF 32-bit LSB shared object, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=0a35e1f33516bbe83eab8ab3a2af35cd7c5c067e, stripped
```

Et pour sa version 64 bits :

```text
$ file /bin/pwd
/bin/pwd: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=4d961c81fc012293ef7efe2f97c6ca28028bf06d, stripped
```

Le format ELF, pour Executable and Linkable Format, est un format libre de fichier de type binaire. Il s'agit du principal format utilisé sur la plupart des distributions Linux \(et même plus généralement sur les systèmes Unix\). Le format contient de multiples informations décrites dans un ordre précis. L'utilitaire _readelf_, qui sera utilisé ici, permet d'extraire toutes ces informations de l'exécutable ciblé.

Le programme C suivant sera utilisé pour toutes les manipulations à venir :

```c
#include <stdio.h>

int main() {
  printf("Hello World !\n");

  return 0;
}
```

## L'entête du fichier ELF \(ELF file header\)

La première information présente dans un fichier au format ELF est un entête, nommé _file-header_ et contient un certain nombre d'informations intéressantes sur le binaire. L'outil _readelf_ utilisé avec l'option _--file-header_ \(ou tout simplement _-h_\) permet d'extraire cet entête de fichier :

```text
$ readelf --file-header prog
En-tête ELF:
  Magique:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
  Classe:                            ELF32
  Données:                          complément à 2, système à octets de poids faible d'abord (little endian)
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  Version ABI:                       0
  Type:                              EXEC (fichier exécutable)
  Machine:                           Intel 80386
  Version:                           0x1
  Adresse du point d'entrée:         0x8048310
  Début des en-têtes de programme :  52 (octets dans le fichier)
  Début des en-têtes de section :    8076 (octets dans le fichier)
  Fanions:                           0x0
  Taille de cet en-tête:             52 (octets)
  Taille de l'en-tête du programme:  32 (octets)
  Nombre d'en-tête du programme:     9
  Taille des en-têtes de section:    40 (octets)
  Nombre d'en-têtes de section:      36
  Table d'indexes des chaînes d'en-tête de section: 35
```

Est présent l'architecture cible de l'exécutable, soit 32 bits et également l'adresse du point d'entrée du programme. Il s'agit en fait de la première instruction qui sera exécutée lors du lancement du binaire et contrairement à la logique, il ne s'agit pas de la première instruction de la méthode _main\(\)_. En effet, l'adresse du point d'entrée indiqué est _0x08048310_, mais _objdump_ indique que la première instruction de la méthode du programme _main\(\)_ se situe à l'adresse _0x0804840b_ :

```text
00: 0804840b <main>:
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

Toujours grâce à _objdump_, l'adresse _0x08048310_ correspond en fait à la première instruction de la routine nommée _\_start_ :

```text
00: 08048310 <_start>:
01: 8048310:       31 ed                   xor    ebp,ebp
02: 8048312:       5e                      pop    esi
03: 8048313:       89 e1                   mov    ecx,esp
04: 8048315:       83 e4 f0                and    esp,0xfffffff0
05: 8048318:       50                      push   eax
06: 8048319:       54                      push   esp
07: 804831a:       52                      push   edx
08: 804831b:       68 a0 84 04 08          push   0x80484a0
09: 8048320:       68 40 84 04 08          push   0x8048440
10: 8048325:       51                      push   ecx
11: 8048326:       56                      push   esi
12: 8048327:       68 0b 84 04 08          push   0x804840b
13: 804832c:       e8 bf ff ff ff          call   80482f0 <__libc_start_main@plt>
14: 8048331:       f4                      hlt
```

Qui elle-même appelle la routine _\_\_libc_start\_main@plt_. Cette seconde routine prend plusieurs paramètres dont l'adresse de la première instruction de la fonction _main\(\)_ \(il permet également de passer les arguments de la méthode _main\(\)_ : _argc_ et _argv_\).

A noter également en sortie du _readelf_, que le second, le troisième et le quatrième octet de la valeur "magique", soit _0x45_, _0x4c_ et _0x46_, représentent la chaîne "ELF".

## Les entêtes du programme \(Program headers\)

La commande _readelf_ avec l'option _--program-headers_ \(ou _-l_\) permet d'afficher les entêtes du programme :

```text
$ readelf --program-headers prog
Type de fichier ELF est EXEC (fichier exécutable)
Point d'entrée 0x8048310
Il y a 9 en-têtes de programme, débutant à l'adresse de décalage52

En-têtes de programme :
  Type           Décalage Adr. vir.  Adr.phys.  T.Fich. T.Mém.  Fan Alignement
  PHDR           0x000034 0x08048034 0x08048034 0x00120 0x00120 R E 0x4
  INTERP         0x000154 0x08048154 0x08048154 0x00013 0x00013 R   0x1
      [Requesting program interpreter: /lib/ld-linux.so.2]
  LOAD           0x000000 0x08048000 0x08048000 0x005e4 0x005e4 R E 0x1000
  LOAD           0x000f08 0x08049f08 0x08049f08 0x00114 0x00118 RW  0x1000
  DYNAMIC        0x000f14 0x08049f14 0x08049f14 0x000e8 0x000e8 RW  0x4
  NOTE           0x000168 0x08048168 0x08048168 0x00044 0x00044 R   0x4
  GNU_EH_FRAME   0x0004d0 0x080484d0 0x080484d0 0x00034 0x00034 R   0x4
  GNU_STACK      0x000000 0x00000000 0x00000000 0x00000 0x00000 RWE 0x10
  GNU_RELRO      0x000f08 0x08049f08 0x08049f08 0x000f8 0x000f8 R   0x1

 Correspondance section/segment :
  Sections de segment...
   00
   01     .interp
   02     .interp .note.ABI-tag .note.gnu.build-id .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rel.dyn .rel.plt .init .plt .plt.got .text .fini .rodata .eh_frame_hdr .eh_frame
   03     .init_array .fini_array .jcr .dynamic .got .got.plt .data .bss
   04     .dynamic
   05     .note.ABI-tag .note.gnu.build-id
   06     .eh_frame_hdr
   07
   08     .init_array .fini_array .jcr .dynamic .got
```

Il y a donc 9 entêtes de programmes présents au sein de l'exécutable, cette information était également présente dans l'entête du fichier. Certaines propriétés sont présentes pour chacun de ces entêtes : l'adresse ou commence l'entête \(adresse physique et virtuelle\), droits en lecture/écriture/exécution etc.

## Les sections

Il est possible d'afficher l'entête des sections présents dans l'exécutable. Cela permet par exemple de connaître les droits sur chaque section, son adresse de début ou encore sa taille :

```text
$ readelf --section-headers prog
Il y a 36 en-têtes de section, débutant à l'adresse de décalage 0x1f8c:

En-têtes de section :
  [Nr] Nom               Type            Adr      Décala.Taille ES Fan LN Inf Al
  [ 0]                   NULL            00000000 000000 000000 00      0   0  0
  [ 1] .interp           PROGBITS        08048154 000154 000013 00   A  0   0  1
  [ 2] .note.ABI-tag     NOTE            08048168 000168 000020 00   A  0   0  4
  [ 3] .note.gnu.build-i NOTE            08048188 000188 000024 00   A  0   0  4
  [ 4] .gnu.hash         GNU_HASH        080481ac 0001ac 000020 04   A  5   0  4
  [ 5] .dynsym           DYNSYM          080481cc 0001cc 000050 10   A  6   1  4
  [ 6] .dynstr           STRTAB          0804821c 00021c 00004a 00   A  0   0  1
  [ 7] .gnu.version      VERSYM          08048266 000266 00000a 02   A  5   0  2
  [ 8] .gnu.version_r    VERNEED         08048270 000270 000020 00   A  6   1  4
  [ 9] .rel.dyn          REL             08048290 000290 000008 08   A  5   0  4
  [10] .rel.plt          REL             08048298 000298 000010 08  AI  5  24  4
  [11] .init             PROGBITS        080482a8 0002a8 000023 00  AX  0   0  4
  [12] .plt              PROGBITS        080482d0 0002d0 000030 04  AX  0   0 16
  [13] .plt.got          PROGBITS        08048300 000300 000008 00  AX  0   0  8
  [14] .text             PROGBITS        08048310 000310 000192 00  AX  0   0 16
  [15] .fini             PROGBITS        080484a4 0004a4 000014 00  AX  0   0  4
  [16] .rodata           PROGBITS        080484b8 0004b8 000016 00   A  0   0  4
  [17] .eh_frame_hdr     PROGBITS        080484d0 0004d0 000034 00   A  0   0  4
  [18] .eh_frame         PROGBITS        08048504 000504 0000e0 00   A  0   0  4
  [19] .init_array       INIT_ARRAY      08049f08 000f08 000004 04  WA  0   0  4
  [20] .fini_array       FINI_ARRAY      08049f0c 000f0c 000004 04  WA  0   0  4
  [21] .jcr              PROGBITS        08049f10 000f10 000004 00  WA  0   0  4
  [22] .dynamic          DYNAMIC         08049f14 000f14 0000e8 08  WA  6   0  4
  [23] .got              PROGBITS        08049ffc 000ffc 000004 04  WA  0   0  4
  [24] .got.plt          PROGBITS        0804a000 001000 000014 04  WA  0   0  4
  [25] .data             PROGBITS        0804a014 001014 000008 00  WA  0   0  4
  [26] .bss              NOBITS          0804a01c 00101c 000004 00  WA  0   0  1
  [27] .comment          PROGBITS        00000000 00101c 00002d 01  MS  0   0  1
  [28] .debug_aranges    PROGBITS        00000000 001049 000020 00      0   0  1
  [29] .debug_info       PROGBITS        00000000 001069 00032a 00      0   0  1
  [30] .debug_abbrev     PROGBITS        00000000 001393 0000e0 00      0   0  1
  [31] .debug_line       PROGBITS        00000000 001473 0000ca 00      0   0  1
  [32] .debug_str        PROGBITS        00000000 00153d 0002b2 01  MS  0   0  1
  [33] .symtab           SYMTAB          00000000 0017f0 000470 10     34  52  4
  [34] .strtab           STRTAB          00000000 001c60 0001e2 00      0   0  1
  [35] .shstrtab         STRTAB          00000000 001e42 00014a 00      0   0  1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  p (processor specific)
```

Certaines sections sont déjà maintenant bien connues : _.rodata_ qui contient des données constantes et en lecture seule, _.text_ qui contient le code exécutable, _.data_ pour les variables globales et statiques initialisées ou encore _.bss_ pour les variables globales et statiques non initialisées.

## La table des symboles

Les symboles sont des données \(des indications\) qui permettent au développeur de deboguer son programme plus facilement. Par exemple, grâce à ces symboles, il sera possible d'indiquer à _gdb_ de poser un point d'arrêt \(breakpoint\), au symbole nommé _main_ pour indiquer la méthode _main\(\)_. Si ces informations ne sont pas présentes, il serait plus difficile \(mais pas impossible\) d'effectuer une rétro-ingenierie sur un binaire.

```text
$ readelf --symbols  ./prog
Table de symboles « .dynsym » contient 5 entrées :
   Num:    Valeur Tail Type    Lien   Vis      Ndx Nom
     0: 00000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 00000000     0 FUNC    GLOBAL DEFAULT  UND puts@GLIBC_2.0 (2)
     2: 00000000     0 NOTYPE  WEAK   DEFAULT  UND __gmon_start__
     3: 00000000     0 FUNC    GLOBAL DEFAULT  UND __libc_start_main@GLIBC_2.0 (2)
     4: 080484bc     4 OBJECT  GLOBAL DEFAULT   16 _IO_stdin_used

Table de symboles « .symtab » contient 71 entrées :
   Num:    Valeur Tail Type    Lien   Vis      Ndx Nom
     0: 00000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 08048154     0 SECTION LOCAL  DEFAULT    1
     2: 08048168     0 SECTION LOCAL  DEFAULT    2
     3: 08048188     0 SECTION LOCAL  DEFAULT    3
     4: 080481ac     0 SECTION LOCAL  DEFAULT    4
     5: 080481cc     0 SECTION LOCAL  DEFAULT    5
     6: 0804821c     0 SECTION LOCAL  DEFAULT    6
     7: 08048266     0 SECTION LOCAL  DEFAULT    7
     8: 08048270     0 SECTION LOCAL  DEFAULT    8
     9: 08048290     0 SECTION LOCAL  DEFAULT    9
    10: 08048298     0 SECTION LOCAL  DEFAULT   10
    11: 080482a8     0 SECTION LOCAL  DEFAULT   11
    12: 080482d0     0 SECTION LOCAL  DEFAULT   12
    13: 08048300     0 SECTION LOCAL  DEFAULT   13
    14: 08048310     0 SECTION LOCAL  DEFAULT   14
    15: 080484a4     0 SECTION LOCAL  DEFAULT   15
    16: 080484b8     0 SECTION LOCAL  DEFAULT   16
    17: 080484d0     0 SECTION LOCAL  DEFAULT   17
    18: 08048504     0 SECTION LOCAL  DEFAULT   18
    19: 08049f08     0 SECTION LOCAL  DEFAULT   19
    20: 08049f0c     0 SECTION LOCAL  DEFAULT   20
    21: 08049f10     0 SECTION LOCAL  DEFAULT   21
    22: 08049f14     0 SECTION LOCAL  DEFAULT   22
    23: 08049ffc     0 SECTION LOCAL  DEFAULT   23
    24: 0804a000     0 SECTION LOCAL  DEFAULT   24
    25: 0804a014     0 SECTION LOCAL  DEFAULT   25
    26: 0804a01c     0 SECTION LOCAL  DEFAULT   26
    27: 00000000     0 SECTION LOCAL  DEFAULT   27
    28: 00000000     0 SECTION LOCAL  DEFAULT   28
    29: 00000000     0 SECTION LOCAL  DEFAULT   29
    30: 00000000     0 SECTION LOCAL  DEFAULT   30
    31: 00000000     0 SECTION LOCAL  DEFAULT   31
    32: 00000000     0 SECTION LOCAL  DEFAULT   32
    33: 00000000     0 FILE    LOCAL  DEFAULT  ABS crtstuff.c
    34: 08049f10     0 OBJECT  LOCAL  DEFAULT   21 __JCR_LIST__
    35: 08048350     0 FUNC    LOCAL  DEFAULT   14 deregister_tm_clones
    36: 08048380     0 FUNC    LOCAL  DEFAULT   14 register_tm_clones
    37: 080483c0     0 FUNC    LOCAL  DEFAULT   14 __do_global_dtors_aux
    38: 0804a01c     1 OBJECT  LOCAL  DEFAULT   26 completed.6587
    39: 08049f0c     0 OBJECT  LOCAL  DEFAULT   20 __do_global_dtors_aux_fin
    40: 080483e0     0 FUNC    LOCAL  DEFAULT   14 frame_dummy
    41: 08049f08     0 OBJECT  LOCAL  DEFAULT   19 __frame_dummy_init_array_
    42: 00000000     0 FILE    LOCAL  DEFAULT  ABS prog.c
    43: 00000000     0 FILE    LOCAL  DEFAULT  ABS crtstuff.c
    44: 080485e0     0 OBJECT  LOCAL  DEFAULT   18 __FRAME_END__
    45: 08049f10     0 OBJECT  LOCAL  DEFAULT   21 __JCR_END__
    46: 00000000     0 FILE    LOCAL  DEFAULT  ABS
    47: 08049f0c     0 NOTYPE  LOCAL  DEFAULT   19 __init_array_end
    48: 08049f14     0 OBJECT  LOCAL  DEFAULT   22 _DYNAMIC
    49: 08049f08     0 NOTYPE  LOCAL  DEFAULT   19 __init_array_start
    50: 080484d0     0 NOTYPE  LOCAL  DEFAULT   17 __GNU_EH_FRAME_HDR
    51: 0804a000     0 OBJECT  LOCAL  DEFAULT   24 _GLOBAL_OFFSET_TABLE_
    52: 080484a0     2 FUNC    GLOBAL DEFAULT   14 __libc_csu_fini
    53: 08048340     4 FUNC    GLOBAL HIDDEN    14 __x86.get_pc_thunk.bx
    54: 0804a014     0 NOTYPE  WEAK   DEFAULT   25 data_start
    55: 0804a01c     0 NOTYPE  GLOBAL DEFAULT   25 _edata
    56: 080484a4     0 FUNC    GLOBAL DEFAULT   15 _fini
    57: 0804a014     0 NOTYPE  GLOBAL DEFAULT   25 __data_start
    58: 00000000     0 FUNC    GLOBAL DEFAULT  UND puts@@GLIBC_2.0
    59: 00000000     0 NOTYPE  WEAK   DEFAULT  UND __gmon_start__
    60: 0804a018     0 OBJECT  GLOBAL HIDDEN    25 __dso_handle
    61: 080484bc     4 OBJECT  GLOBAL DEFAULT   16 _IO_stdin_used
    62: 00000000     0 FUNC    GLOBAL DEFAULT  UND __libc_start_main@@GLIBC_
    63: 08048440    93 FUNC    GLOBAL DEFAULT   14 __libc_csu_init
    64: 0804a020     0 NOTYPE  GLOBAL DEFAULT   26 _end
    65: 08048310     0 FUNC    GLOBAL DEFAULT   14 _start
    66: 080484b8     4 OBJECT  GLOBAL DEFAULT   16 _fp_hw
    67: 0804a01c     0 NOTYPE  GLOBAL DEFAULT   26 __bss_start
    68: 0804840b    46 FUNC    GLOBAL DEFAULT   14 main
    69: 0804a01c     0 OBJECT  GLOBAL HIDDEN    25 __TMC_END__
    70: 080482a8     0 FUNC    GLOBAL DEFAULT   11 _init
```

Par exemple, l'indication _main_ indique ici qu'il s'agit d'une fonction débutant à l'adresse _0x0804840b_.

