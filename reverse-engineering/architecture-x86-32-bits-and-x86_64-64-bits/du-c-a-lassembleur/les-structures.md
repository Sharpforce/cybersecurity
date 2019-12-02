# Les structures

Les structures en C permettent de créer des variables personnalisées. Un chapitre leur est dédié ici, car en assembleur, le moyen d'accéder à ces différentes variables, peut être difficile à appréhender dans un premier temps.

## Un exemple simple de structure

Une structure contenant ici trois variables de type entier. La troisième variable est vouée à contenir le résultat de l'addition de la première et de la seconde variable :

```c
struct MyStruct {
  int op1;
  int op2;
  int total;
};

int main(int argc, char **argv) {
  struct MyStruct myStruct;
  myStruct.op1 = 5;
  myStruct.op2 = 10;

  myStruct.total = myStruct.op1 + myStruct.op2;
}
```

Son équivalent en assembleur :

```text
01: 80483db:       55                      push   ebp
02: 80483dc:       89 e5                   mov    ebp,esp
03: 80483de:       83 ec 10                sub    esp,0x10
04: 80483e1:       c7 45 f4 05 00 00 00    mov    DWORD PTR [ebp-0xc],0x5
05: 80483e8:       c7 45 f8 0a 00 00 00    mov    DWORD PTR [ebp-0x8],0xa
06: 80483ef:       8b 55 f4                mov    edx,DWORD PTR [ebp-0xc]
07: 80483f2:       8b 45 f8                mov    eax,DWORD PTR [ebp-0x8]
08: 80483f5:       01 d0                   add    eax,edx
09: 80483f7:       89 45 fc                mov    DWORD PTR [ebp-0x4],eax
10: 80483fa:       b8 00 00 00 00          mov    eax,0x0
11: 80483ff:       c9                      leave
12: 8048400:       c3                      ret
```

Le prologue est aux lignes 1, 2 et 3. Ligne 4, la valeur 5 est copiée à l'adresse _\[ebp-0xc\]_ \(il s'agit donc ici de la variable 'op1'\), puis, ligne 5, la valeur 10 est copiée à l'adresse _\[ebp-0x8\]_ \(il s'agit ici de la variable 'op2'\). Finalement, ligne 9, le résultat de l'addition est copié à l'adresse _\[ebp-0x4\]_. Ici, bien que l'utilisation d'une structure soit faite, le code assembleur est le même que pour l'utilisation de variables locales :

```c
int main(int argc, char **argv) {
  int op1;
  int op2;
  int total;

  op1 = 5;
  op2 = 10;

  total = op1 + op2;
}
```

Et l'assembleur généré qui est le même qu'avec l'utilisation d'une structure :

```text
01: 80483db:       55                      push   ebp
02: 80483dc:       89 e5                   mov    ebp,esp
03: 80483de:       83 ec 10                sub    esp,0x10
04: 80483e1:       c7 45 fc 05 00 00 00    mov    DWORD PTR [ebp-0x4],0x5
05: 80483e8:       c7 45 f8 0a 00 00 00    mov    DWORD PTR [ebp-0x8],0xa
06: 80483ef:       8b 55 fc                mov    edx,DWORD PTR [ebp-0x4]
07: 80483f2:       8b 45 f8                mov    eax,DWORD PTR [ebp-0x8]
08: 80483f5:       01 d0                   add    eax,edx
09: 80483f7:       89 45 f4                mov    DWORD PTR [ebp-0xc],eax
10: 80483fa:       b8 00 00 00 00          mov    eax,0x0
11: 80483ff:       c9                      leave
12: 8048400:       c3                      ret
```

## Pointeur de structure

En utilisant un pointeur de structure \(souvent utilisé pour passer la structure en paramètre d'une fonction\), le code assembleur va utiliser une valeur de référence qui se trouve être la base de la structure. Par exemple :

```c
typedef struct MyStruct MyStruct;
struct MyStruct {
  int length;
  char *str;
};

void initStruct(MyStruct *myStruct);

int main(int argc, char **argv) {
  MyStruct myStruct;
  initStruct(&myStruct);

  return 0;
}

void initStruct(MyStruct *myStruct) {
  myStruct->str = "just a string";
  myStruct->length = 14;
}
```

**Note :** Ici, la longueur de la chaîne est hardcodée pour éviter le code superflu dû à l'appel de la méthode _strlen\(\)_.

Son équivalent en assembleur :

```text
080483db <main>:
01: 80483db:       8d 4c 24 04             lea    ecx,[esp+0x4]
02: 80483df:       83 e4 f0                and    esp,0xfffffff0
03: 80483e2:       ff 71 fc                push   DWORD PTR [ecx-0x4]
04: 80483e5:       55                      push   ebp
05: 80483e6:       89 e5                   mov    ebp,esp
06: 80483e8:       51                      push   ecx
07: 80483e9:       83 ec 14                sub    esp,0x14
08: 80483ec:       83 ec 0c                sub    esp,0xc
09: 80483ef:       8d 45 f0                lea    eax,[ebp-0x10]
10: 80483f2:       50                      push   eax
11: 80483f3:       e8 10 00 00 00          call   8048408 <initStruct>
12: 80483f8:       83 c4 10                add    esp,0x10
13: 80483fb:       b8 00 00 00 00          mov    eax,0x0
14: 8048400:       8b 4d fc                mov    ecx,DWORD PTR [ebp-0x4]
15: 8048403:       c9                      leave
16: 8048404:       8d 61 fc                lea    esp,[ecx-0x4]
17: 8048407:       c3                      ret

08048408 <initStruct>:
01: 8048408:       55                      push   ebp
02: 8048409:       89 e5                   mov    ebp,esp
03: 804840b:       8b 45 08                mov    eax,DWORD PTR [ebp+0x8]
04: 804840e:       c7 40 04 b0 84 04 08    mov    DWORD PTR [eax+0x4],0x80484b0
05: 8048415:       8b 45 08                mov    eax,DWORD PTR [ebp+0x8]
06: 8048418:       c7 00 0e 00 00 00       mov    DWORD PTR [eax],0xe
07: 804841e:       90                      nop
08: 804841f:       5d                      pop    ebp
09: 8048420:       c3                      ret
```

Dans cet exemple, seul l'étude de la fonction appelée sera étudiée. Tout d'abord, la création du nouveau contexte grâce à la ligne 1 et 2. Puis, à la ligne 3, la copie de la valeur située à _\[ebp+0x8\]_ dans le registre _EAX_. Ligne 4, l'adresse _0x080484b0_ est copiée dans _\[eax+0x4\]_. La ligne 5 correspond à nouveau au rechargement de _\[ebp+0x8\]_ dans le registre _EAX_. Ligne 6, la copie de la valeur 14 \(soit "0xe"\) dans _\[eax\]_. Il faut en fait comprendre le code assembleur comme ceci : _\[eax\]_ contient l'adresse de base de la structure passée en paramètre de la fonction \(initialement dans _\[ebp+0x8\]_ et récupérée à la ligne 3 et 5\). A l'adresse _\[eax\]_ se trouve donc le premier paramètre, qui est la longueur de la chaîne, car la ligne 6 copie la valeur 14 dans \[eax\]. Etant donné qu'il s'agit d'un entier, il possède donc une taille de 4 octets. Le second paramètre est un pointeur, à l'adresse _\[eax+0x4\]_, soit 4 octets de plus que l'adresse de base de la structure et qui pointe vers la chaîne de caractère "str" \(affectation à la ligne 4\). Si, par exemple, un troisième paramètre était présent il aurait été accessible à l'adresse _\[eax+0x8\]_ \(car un pointeur possède également une taille de 4 octets\), et ainsi de suite.

