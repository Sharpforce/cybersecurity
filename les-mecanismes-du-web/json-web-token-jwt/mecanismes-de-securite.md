# Mécanismes de sécurité

## Signature \(JWS\)

Le mécanisme de signature permet à l'entité qui délivre le jeton JWT de le signer et à l'entité destinataire de vérifier cette signature. Suivant le type de signature utilisé, il est possible de s'assurer à minima de l'intégrité des données contenues dans le jeton, voir de s'assurer de l'identité de l'entité émettrice.

Le format du jeton évolue donc avec l'ajout d'une troisième partie qu'est la signature du jeton \(également encodée en Base64URL\) :

`Base64URL(HEADER).Base64URL(PAYLOAD).Base64URL(SIGNATURE)`

### Signature à base de clés symétriques

Il est possible de signer le jeton JWT grâce à une clé symétrique. Ce mécanisme permet de s'assurer de l'intégrité des données contenues dans le jeton mais ne permet pas d'authentifier l'émetteur du jeton, car toutes les entités devant traiter le jeton \(délivrer ou vérifier les jetons\) doivent posséder la clé partagée \(donc sont potentiellement en mesure de créer et de signer un jeton\).

![](../../.gitbook/assets/ec1de0fe3572792e7b007c68ceff1a7d.png)

L'ajout de la signature à base de clés symétriques apporte quelques modification au niveau du jeton JWT. Tout d'abord dans l'entête du jeton le champ `alg` va contenir le nom de l'algorithme utilisé. Par exemple : HS256 \(soit HMAC + SHA256\) :

`{  
  "alg": "HS256",  
  "typ": "jwt"  
}`

L'utilisation d'un HMAC pour la signature s'effectue comme ceci :

`HMAC(Base64URL(HEADER).Base64URL(PAYLOAD), secret_key)`

En ajoutant donc la signature, le jeton JWT devient alors :

`Base64URL(HEADER).Base64URL(PAYLOAD).Base64URL(HMAC(Base64URL(HEADER).Base64URL(PAYLOAD), secret_key))`

Soit avec une clé secrète ayant pour valeur "secret\_key" :

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6Imp3dCJ9.eyJ1c2VybmFtZSI6IkFub255bWUiLCJyb2xlIjoiVXNlciJ9._qpO3Uagw4kt8SbDWBy2AC7sF8H_dqF6r8I9eqK7Dtc
```

### Signature à base de cryptographie asymétrique

La signature à base de cryptographie asymétrique \(utilisant donc une biclé privée/publique\) permet, en de plus de s'assurer de l'intégrité des données du jeton, d'authentifier le fournisseur de celui-ci. Le fournisseur du jeton possède la clé privée et les services devant vérifier le jeton possède \(ou du moins ont accès à\) la clé publique.

![](../../.gitbook/assets/af3eaeef01665f0742276cb047b9d215.png)

Seul la valeur de l'algorithme au niveau du header du jeton JWT change. Par exemple en se basant sur un algorithme de type RSA accompagné de l'algorithme de hachage SHA256 :

`{  
  "alg": "RS256",  
  "typ": "jwt"  
}`

Le jeton JWT supporte également en général les algorithmes à base de courbes elliptiques ECDSA :

`{  
  "alg": "ES256",  
  "typ": "jwt"  
}`

Mis à part cela, le format du jeton ne change pas.

### Liste des algorithmes supportés

Voici la liste des algorithmes supportés selon le [RFC 7518](https://tools.ietf.org/html/rfc7518#section-3) :

<table>
  <thead>
    <tr>
      <th style="text-align:left">
        <p>Valeur du champ</p>
        <p>d&apos;ent&#xEA;te &quot;alg&quot;</p>
      </th>
      <th style="text-align:left">Algorithme</th>
      <th style="text-align:center">Exigences de mise en oeuvre</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">HS256</td>
      <td style="text-align:left">HMAC using SHA-256</td>
      <td style="text-align:center">Obligatoire</td>
    </tr>
    <tr>
      <td style="text-align:left">HS384</td>
      <td style="text-align:left">HMAC using SHA-384</td>
      <td style="text-align:center">Facultatif</td>
    </tr>
    <tr>
      <td style="text-align:left">HS512</td>
      <td style="text-align:left">HMAC using SHA-512</td>
      <td style="text-align:center">Facultatif</td>
    </tr>
    <tr>
      <td style="text-align:left">RS256</td>
      <td style="text-align:left">RSASSA-PKCS1-v1_5 using SHA-256</td>
      <td style="text-align:center">Recommand&#xE9;</td>
    </tr>
    <tr>
      <td style="text-align:left">RS384</td>
      <td style="text-align:left">RSASSA-PKCS1-v1_5 using SHA-384</td>
      <td style="text-align:center">Facultatif</td>
    </tr>
    <tr>
      <td style="text-align:left">RS512</td>
      <td style="text-align:left">RSASSA-PKCS1-v1_5 using SHA-512</td>
      <td style="text-align:center">Facultatif</td>
    </tr>
    <tr>
      <td style="text-align:left">ES256</td>
      <td style="text-align:left">ECDSA using P-256 and SHA-256</td>
      <td style="text-align:center">Recommand&#xE9;</td>
    </tr>
    <tr>
      <td style="text-align:left">ES384</td>
      <td style="text-align:left">ECDSA using P-256 and SHA-384</td>
      <td style="text-align:center">Facultatif</td>
    </tr>
    <tr>
      <td style="text-align:left">ES512</td>
      <td style="text-align:left">ECDSA using P-256 and SHA-512</td>
      <td style="text-align:center">Facultatif</td>
    </tr>
    <tr>
      <td style="text-align:left">PS256</td>
      <td style="text-align:left">RSASSA-PSS using SHA-256 and MGF1 with SHA-256</td>
      <td style="text-align:center">Facultatif</td>
    </tr>
    <tr>
      <td style="text-align:left">PS384</td>
      <td style="text-align:left">RSASSA-PSS using SHA-384 and MGF1 with SHA-384</td>
      <td style="text-align:center">Facultatif</td>
    </tr>
    <tr>
      <td style="text-align:left">PS512</td>
      <td style="text-align:left">RSASSA-PSS using SHA-512 and MGF1 with SHA-512</td>
      <td style="text-align:center">Facultatif</td>
    </tr>
    <tr>
      <td style="text-align:left">None</td>
      <td style="text-align:left">Pas de siganture</td>
      <td style="text-align:center">Facultatif</td>
    </tr>
  </tbody>
</table>## Chiffrement \(JWE\)

Le chiffrement du jeton JWT apporte le principe de sécurité supplémentaire qu'est la confidentialité \(rendre illisible le jeton par les personnes n'ayant pas l'autorisation de le lire\). Un jeton JWT chiffré \(JWE\) comporte plusieurs parties supplémentaires, ce qui amène le jeton à six parties au total :

`HEADER.JWE_ENCRYPTED_KEY.JWE_IV.ADDITIONAL_AUTHENTICATION_DATA.CIPHERED_PAYLOAD.AUTHENTICATION_TAG`

L'entête est le même que celui d'un jeton JWS à l’exception de quelques champs supplémentaires. 

La seconde partie est la clé symétrique utilisée pour chiffrer le contenu du jeton. En effet, la cryptographie asymétrique a le désavantage d'être beaucoup moins rapide que la cryptographie symétrique. Pour contrer cela, la clé publique est alors utilisée pour chiffrer une clé symétrique qui elle-même sert à chiffrer le contenu à protéger. L'algorithme symétrique utilisé est en fait indiqué dans l'entête du jeton \(ils 'agit d'un des champs supplémentaires\) sous le nom de `enc`.

Certains algorithmes de chiffrement nécessite l'utilisation d'un IV \(Initialization Vector / Vecteur d'initialisation\). Cet IV est également nécessaire au déchiffrement, il est donc présent en tant que troisième composant du JWE.

L'AAD \(Additional Authentication Data\) n'est pas chiffré mais il ne peut être modifié car il est protégé en intégrité. Cette partie est ajoutée lorsque l'algorithme spécifié dans le l'entête du jeton est de type Authenticated Encryption with Associated Data \(AEAD\).

{% hint style="info" %}
l'AEAD est un mode opératoire pour le chiffrement par bloc permettant d'assurer simultanément la confidentialité ainsi que l'authenticité des données.
{% endhint %}

La cinquième partie est tout simplement les données à protéger et donc chiffrées avec la clé disponible dans la seconde partie \(qui est donc elle-même chiffré avec la clé publique de l'entité émettrice\).

La dernière partie est une donnée additionnelle qui permet de s'assurer de l'intégrité des données de la quatrième partie.

Toutes ces parties sont bien sur encodées en Base64URL comme pour les autres types de jetons :

```text
Base64URL(HEADER).Base64URL(JWE_ENCRYPTED_KEY).Base64URL(JWE_IV).Base64URL(ADD).Base64URL(CIPHERED_PAYLOAD).Base64URL(AUTHENTICATION_TAG)
```

Le jeton JWE devient donc : 

```text
eyJhbGciOiJSU0ExXzUiLCJlbmMiOiJBMTI4Q0JDLUhTMjU2In0.UGhIOguC7IuEvfNPVaXsGMoLOmwvc1GyqlIKOK1nN94nHPoltGRhWhw7Zx0-kFm1NJn8LE9XShH59i8J0PH5ZZyNfGy2xGdULU7sHNF6Gp2vPLgNZ__deLKxGHZ7PcHALUzoOegEI-8E66jX2E4zyJKx-YxzZIItRzC5hlRirb6Y5Cl_p-ko3YvkkysZIFNPccxRU7qve1WYPxqbb2Yw8kZqa2rMWI5ng8OtvzlV7elprCbuPhcCdZ6XDP0_F8rkXds2vE4X-ncOIM8hAYHHi29NX0mcKiRaD0-D-ljQTPcFPgwCp6X-nZZd9OHBv-B3oWh2TbqmScqXMR4gp_A.AxY8DCtDaGlsbGljb3RoZQ.ADD.KDlTtXchhZTGufMYmOYGS4HffxPSUrfmqCHXaI9wOGY.9hH0vgRfYgPnAHOd8stkvw
```

A l'instar de la signature, le chiffrement d'un jeton JWT peut se faire également avec une clé partagée \(key-wrapper, cryptographie symétrique\) sans passer par une biclé.

### Liste des algorithmes supportés

Voici la liste des algorithmes supportés selon le [RFC 7518](https://tools.ietf.org/html/rfc7518#section-4) :

<table>
  <thead>
    <tr>
      <th style="text-align:left">
        <p>Valeur du champ</p>
        <p>d&apos;ent&#xEA;te &quot;alg&quot;</p>
      </th>
      <th style="text-align:left">Algorithme</th>
      <th style="text-align:left">Champ d&apos;ent&#xEA;te suppl&#xE9;mentaire</th>
      <th style="text-align:left">Exigences de mise en oeuvre</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">RSA1_5</td>
      <td style="text-align:left">RSAES-PKCS1-v1_5</td>
      <td style="text-align:left">N/A</td>
      <td style="text-align:left">Recommand&#xE9;</td>
    </tr>
    <tr>
      <td style="text-align:left">RSA-OAEP</td>
      <td style="text-align:left">RSAES OAEP (param&#xE8;tres par d&#xE9;faut)</td>
      <td style="text-align:left">N/A</td>
      <td style="text-align:left">Recommand&#xE9;</td>
    </tr>
    <tr>
      <td style="text-align:left">RSA-OAEP-256</td>
      <td style="text-align:left">RSAES OAEP using SHA-256 and MGF1</td>
      <td style="text-align:left">N/A</td>
      <td style="text-align:left">Facultatif</td>
    </tr>
    <tr>
      <td style="text-align:left">A128KW</td>
      <td style="text-align:left">AES Key Wrap with default initial value using 128-bit key</td>
      <td style="text-align:left">N/A</td>
      <td style="text-align:left">Recommand&#xE9;</td>
    </tr>
    <tr>
      <td style="text-align:left">A192KW</td>
      <td style="text-align:left">AES Key Wrap with default initial value using 192-bit key</td>
      <td style="text-align:left">N/A</td>
      <td style="text-align:left">Facultatif</td>
    </tr>
    <tr>
      <td style="text-align:left">A256KW</td>
      <td style="text-align:left">AES Key Wrap with default initial value using 256-bit key</td>
      <td style="text-align:left">N/A</td>
      <td style="text-align:left">Recommand&#xE9;</td>
    </tr>
    <tr>
      <td style="text-align:left">dir</td>
      <td style="text-align:left">Direct use of a shared symmetric key as the CEK</td>
      <td style="text-align:left">N/A</td>
      <td style="text-align:left">Recommand&#xE9;</td>
    </tr>
    <tr>
      <td style="text-align:left">ECDH-ES</td>
      <td style="text-align:left">Elliptic Curve Diffie-Hellman Ephemeral Static key agreement using Concat
        KDF</td>
      <td style="text-align:left">&quot;epk&quot;, &quot;apu&quot;, &quot;apv&quot;</td>
      <td style="text-align:left">Recommand&#xE9;</td>
    </tr>
    <tr>
      <td style="text-align:left">ECDH-ES+A128KW</td>
      <td style="text-align:left">ECDH-ES using oncat KDF and CEK wrapped with &quot;A128KW&quot;</td>
      <td
      style="text-align:left">&quot;epk&quot;, &quot;apu&quot;, &quot;apv&quot;</td>
        <td style="text-align:left">Recommand&#xE9;</td>
    </tr>
    <tr>
      <td style="text-align:left">ECDH-ES+A192KW</td>
      <td style="text-align:left">ECDH-ES using oncat KDF and CEK wrapped with &quot;A192KW&quot;</td>
      <td
      style="text-align:left">&quot;epk&quot;, &quot;apu&quot;, &quot;apv&quot;</td>
        <td style="text-align:left">Facultatif</td>
    </tr>
    <tr>
      <td style="text-align:left">ECDH-ES+A256KW</td>
      <td style="text-align:left">ECDH-ES using oncat KDF and CEK wrapped with &quot;A256KW&quot;</td>
      <td
      style="text-align:left">&quot;epk&quot;, &quot;apu&quot;, &quot;apv&quot;</td>
        <td style="text-align:left">Recommand&#xE9;</td>
    </tr>
    <tr>
      <td style="text-align:left">A128GCMKW</td>
      <td style="text-align:left">Key wrapping with AES GCM using 128-bit key</td>
      <td style="text-align:left">&quot;iv&quot;, &quot;tag&quot;</td>
      <td style="text-align:left">Facultatif</td>
    </tr>
    <tr>
      <td style="text-align:left">A192GCMKW</td>
      <td style="text-align:left">Key wrapping with AES GCM using 192-bit key</td>
      <td style="text-align:left">&quot;iv&quot;, &quot;tag&quot;</td>
      <td style="text-align:left">Facultatif</td>
    </tr>
    <tr>
      <td style="text-align:left">A256GCMKW</td>
      <td style="text-align:left">Key wrapping with AES GCM using 256-bit key</td>
      <td style="text-align:left">&quot;iv&quot;, &quot;tag&quot;</td>
      <td style="text-align:left">Facultatif</td>
    </tr>
    <tr>
      <td style="text-align:left">PBES2-HS256+A128KW</td>
      <td style="text-align:left">PBES2 with HMAC SHA-256 and &quot;A128KW&quot; wrapping</td>
      <td style="text-align:left">&quot;p2s&quot;, &quot;p2c&quot;</td>
      <td style="text-align:left">Facultatif</td>
    </tr>
    <tr>
      <td style="text-align:left">PBES2-HS384+A192KW</td>
      <td style="text-align:left">PBES2 with HMAC SHA-384 and &quot;A192KW&quot; wrapping</td>
      <td style="text-align:left">&quot;p2s&quot;, &quot;p2c&quot;</td>
      <td style="text-align:left">Facultatif</td>
    </tr>
    <tr>
      <td style="text-align:left">PBES2-HS5124+A256KW</td>
      <td style="text-align:left">PBES2 with HMAC SHA-512 and &quot;A256KW&quot; wrapping</td>
      <td style="text-align:left">&quot;p2s&quot;, &quot;p2c&quot;</td>
      <td style="text-align:left">Facultatif</td>
    </tr>
  </tbody>
</table>## 

