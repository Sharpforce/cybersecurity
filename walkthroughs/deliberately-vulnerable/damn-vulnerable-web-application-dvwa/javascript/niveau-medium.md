# Niveau "Medium"

Le niveau "Medium" contient également un jeton présent dans un champ caché :

![](../../../../.gitbook/assets/971de3f8c6b4b1a4695cd34a12a032b9.png)

Le jeton (à gauche) semble posséder un pattern qui est `"XX" + reverse("ChangeMe") + "XX"` . Je tente directement de faire cela avec la phrase "success". Si cela ne fonctionne pas, je me plongerai dans le code Javascript :

![](../../../../.gitbook/assets/a42126f74ff9997edf02f09ac48f3a81.png)

Bien joué ! J'analyse tout de même le code Javascript par curiosité :

![](../../../../.gitbook/assets/7252968887ec6655de455b1fdcf1524b.png)

Soit :

1. La fonction `do_elsesomething("XX")` est lancé au bout de 300 ms
2. Cette fonction effectue un appel à la méthode `do_something("XX" + "ChangeMe" + "XX")`&#x20;
3. La méthode `do_something("XX" + "ChangeMe" + "XX")` est une fonction de reverse dont le résultat est la chaîne `"XXeMegnahCXX"`
4. Finalement, le résultat est stocké dans le champ `token`
