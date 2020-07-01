---
author : Alonso Martinez 
title : Notas sueltas - Mastering Regex
...

Introduction 
============

Searching Text Files: Egrep
---------------------------

El utility _egrep_ viene incluído en MacOS y otros!
El comando `egrep` toma el primer argumento "entrecomillado" como un regex, y los argumentos restantes como los archivo(s) en los cuales buscar.

    $ egrep 'regex' <file1> <file2>

Nota importante: _egrep_ remueve los _newlines_ de cada línea al buscar.
Un _newline_ se escribe literalmente como `\n`.

Algunos metacaracteres:
-----------------------
Ejemplos simples: `^` (_caret_) & `$` (_dollar_).
_caret_ representa el inicio de una línea, y _dollar_ representa el fin de la línea.
Actúan de cierta forma como anclas.
Son especiales en el sentido que hacen _match_ (coincide,empareja) con posiciones en la línea, no con texto particular

### Clases de caracteres
El bloque regex `[<chars>]` deja listar caracteres con los que quieres coincidir. Por ejemplo, la expresión `[ea]` hace match con las letras `e` o{.underline} `a`.
Por ejemplo, la expresión `gr[ea]y` hace match con las palabras _gray_ y _grey_, las dos posibles ortografías^[si se dice asi?].
El orden en el que están los caracteres dentro de las clases no interesa.
La implicación es más como $char1 \lor  char2$.
En ese sentido se pueden pensar más como _conjuntos_ y no clases de caracteres, pero en ese caso los elementos del conjunto no están separados por comas.

Dentro de las clases de caracteres podemos usar otro metacaracter: el guión (`-`).
Dentro de una clase de caracteres, el guión indica un rango.
Por ejemplo, `[1-6]` hace match con cualquier número entre 1 y 6.
La expresión `[a-z]` hace match con cualquier letra del alfabeto, pero en minúsculas{.underline}.
Vale la pena decir que el guión solo se considera metacaracter en el contexto de una clase de caracteres, fuera de ella se interpreta como un literal.
Dicho esto, si no indica un rango no se considera metacaracter.
Por ejemplo `[-a]` hace match con los caracteres `-` o `a`.

Otro ejemplo de este fenómeno, son los (usually) metacaracteres `?`, `.`, que dentro del contexto de una clase de caracteres (clase o charclass por brevedad) se interpretan como literales; van a hacer match con texto que contenga "?" o ".".
No obstante, fuera de clases, son metacaracteres que tiene su propio significado.
Por ejemplo, `?` se interpreta como _uno o más_ del patrón anterior.

> Consider character classes theri own mini language.
> The rules regarding wich metacharacters are supported (and what they do) are completely different inside and outside character classes.

Existe un patrón similar a las clases de caracteres, las clases negadas.
En vez de hacer match con los caracteres listados en la clase, hace match con cualquier caracter _salvo_ los que están en la clase. 
Volviendo a la analogía de conjuntos, es el complemento de un conjunto.
Ese patrón se escibe como `[^<chars>]`.^[es buen momento para hacer notar que usamos la notación de corchetes `<>`para denotar que dentro de los corchetes va lo de interés.]

Una vez más, este es un buen ejemplo de cómo el significado o estatus de un caracter depende de dónde se usa.
En este caso el _caret_ no quiere decir inicio de línea, sino que se interpreta como un negador de clase, puesto que se está utilizando en contexto de clases.

### Dot & Pipe
El metacaracter `.` (dot, punto) se interpreta como "cualquier caracter".
Una vez más, un punto dentro de una charclass pierde estatus de metacaracter, y se vuelve un literal.

Ya habíamos discutido que una expresión del tipo `cat` hace match con texto que contiene una `c`seguida de una `a`, y asi.
Por ejemplo, "cat", "vacation", etc...
Es decir, es como pedir encontrar `c` _&_, `a`, _&_ `t`.
El operador lógico es _and_.
Para el operador lógico _or_ tenemos un nuevo metacaracter: el _pipe_ (`|`).
Usualmente nos referemimos a esto como alternancia, donde cada una de las opciones se conoce como subexpresión o alternativa.
El _pipe_ es otro metacaracter que se interpreta como literal dentro de una clase, lo cual tiene sentido ya que los caracteres dentro de las clases tienen un _or_ implícito.

Vale la pena destacar que hay otro metacaracter que podemos usar, y su uso no está tan alejado de su significado en el lenguaje natural.
Para encapsular alternancias, podemos usar paréntesis `(`, `)`.
Volviendo al ejemplo de las diferentes maneras de escribir "gray", la expresión que usamos para ejemplificar charclasses `gr[ae]y` es equivalente a la expresión `gr(a|e)y`.
Lo cual también es un buen ejemplo de a qué nos referíamos con "or implícito" dentro de las clases.

Una distinción importante entre clases y alternancias.
Cuando usamos una alternancia, las subexpresiones pueden ser expresiones regulares completas por sí mismas, mientras que una clase de caracteres solo hace match {con un solo caracter}{.underline}

En el caso de _egrep_, hay un flag opcional, `-i` que hace las expresiones _case-insensitive_

    $ egrep -i '<expression>' <file1> <file2>

Word Boundaries
---------------
Las expresiones del tipo `cat` hacen match no solo con la palabra, sino con strings que tienen el patrón incrustado, como "vacation".
Para resolver este problema _egrep_ tiene (o por lo menos, mi versión) herramientas llamadas _metasecuencias_ que actúan como fronteras de palabras.
Estas metasecuencias son `\<` y `\>`, que actúan como `^` y `$` para marcar inicio y fin de la línea, pero para palabras^[esta es la primera implementación de regex que veo que hace esto. Quizás sea específico a _egrep_. Por ejemplo RegExr no tiene esto].
Y por palabras, léase secuencias alfanuméricas.

Por si solos `<` y `>` no son metacaracteres, son literales que dan match con "<" y ">" respectivamente.
Les llamamos metasecuencias porque solo combinadas con otro caracter, el _backslash_ `\`, se convierten en no-literales.

| Metacharacter | Name | Matches            |
|---------------|------|--------------------|
| `.`           | dot  | Any one character  |
| `[ ]`         | character class | Any character listed |
| `[^ ]`        | negated character class | Any character not listed |
|---------------|------|---------------------|
| `^`   | caret | The position at start of the line | 
| `$` | dollar | The position at end of line |
| `\<` |  | Start of word |
| `\>` |  | End of word |
| `|`  | pipe | Matches eiter expression it separates |
| `( )` | parentheses | limiter |

Table: Resumen de metacaracteres vistos hasta ahora.

Quantifiers
------------
Los cuantificadores sirven para marcar cantidades de caracteres.
Por ejemplo, uno o ninguno, varios, todos los que puedas.

El metacaracter `?` quiere decir opcional.
Actúa haciendo al caracter que lo precede opcional en los strings que hacen match con la expresión.
Por ejemplo `colou?r` hace match con "color y con "colour".

El metacaracter `+`quiere decir "uno o más del caracter al que precede", y `*`quiere decir "cualquier número del caracter precedente, incluido cero".

En conjunto, `+`, `?`, `*` se llaman cuantificadores.

|   | Minimum Required | Maximum to try |
|---|------------------|----------------|
|`?`| none | One allowed; none required |
|`*`| none | Unlimited allowed; none required|
|`+`| one  | Unlimited allowed; one required |

Table: Summary of quantifiers. 

Asi como hay cuantificadores para al menos uno, ilimitados, etc... hay una forma de especificar entre $n$ y $m$ coincidencias (_matches_).
Eso se logra con la metasecuencia `{<n>,<m>}`.^[nótese que no hay espacio entre `<n>` y `<m>`]

Parentheses and Backreferences
------------------------------
Más allá de su papel de "agrupadores", los paréntesis tienen más usos.
En algunas implementaciones de regex los paréntesis "tienen memoria" de el texto con el cual coincide la expresión contenida en ellos.
A estos a veces se les dice _capturing groups_.

Para el cookbook:

    \<([A-Za-z]+) +\1\>

Busca palabras repetidas^[pero _egrep_ solo las ve si están en la misma línea.]

A la metasecuencia `\1` se llama _backreference_ y como su nombre sugiere, hace referencia a el texto capturado.
Si se tienen más paréntesis de captura las metasecuencias como `\2`, `\3`, etc... están disponibles.
Los grupos de paréntesis se van numerando de izquierda a derecha.

En algunas implementaciones, como RegExr, las backreferences se hacen con `$1, $2`, etc...
^[El libro menciona que hay ciertas versiones de _egrep_ que tienen un bug con la opción `-i` y backreferences. Por eso no puede hacer ambos, y solo hace backreferencing. Es decir, puede encontrar "cat cat", pero no "The the". La versión instalada en mi computadora tiene ese problema.]

Para incluír metacaracteres como literales en un patrón, se "escapan".
Es decir, se preceden con un _backslash_ (`\`).

Otro ejemplo para el cookbook

    "[^"]*"

Encuentra cosas contenidas entre comillas

Más ejemplos. Adios _egrep_
===========================
Otras implementaciones (como la de Perl, que es medio estándar) otras construcciones y tienen un conjunto más ámplio de metacaracteres. 

| Metachar | Matches |
|-------|-------------------|
| `\t`  | Tab character     |
| `\n`  | Newline           |
| `\r`  | Carriage-return   |
| `\s`  | Any whitespace    |
| `\S`  | Anything not in `\s`|
| `\w`  | Short for `[a-aA-Z0-9_]` _i.e_ word |
| `\W`  | Negated `\w`      |
| `\d`  | Short for `[0-9]` _i.e_ a digit |
| `\D`  | Anythong not `\d` |

Table: Nuevos metacaracteres

Modifying text with regex
-------------------------
En Perl, el comando

    $var =~ m/regex/

Busca coincidencias en el texto.
Hay una manera de hacer más que eso, y además de buscar, reemplazar texto con con un patrón como este

    $var =~ s/regex/replacement/

Nótese como cambió el comando que se le da a Perl de `m`, que podemos leer como _match_, a `s`, que podemos leer como substitute.

Como siempre, las diagonales `/` delimitan la expresión, y también el patrón de reemplazo.

Nota: Mientras _egrep_ tiene _word boundaries_, los metacaracteres `\<` y `\>`, Perl y otras implementaciones inspiradas en Perl tienen el _catch-all_ `\b` que sirve como comodín entre `\<` y `\>` y coincide con tanto inicio como fin de palabra.

Lookaround
----------
Los _lookarounds_ son como los conceptos de _line start_, _word-boundary_, etc... en el sentido de que no coinciden con texto sino con posiciones.
De hecho, los _line start_ y constructos similares son casos particulares de _lookarounds_.
Vemos dos tipos particulares: _lookahead_ y _lookbehind_.
Ambos tiene versiones positivas y negativas

El _lookahead_ se fija en el texto de adelante (hacia la derecha) para verificar si coincide con la subexpresión que contiene, y tiene éxito si dicho texto coincide con la subexpresión.
Un _lookahead_ positivo se hace con la expresión `(?=)`.
Decimos positivo para referirnos a que tendrá éxito si la subexpresión contenida entre `(?=` y el paréntesis de cierre `)` coincide con el texto al que se está mirando; en este caso a la derecha.

El _lookbehind_ se fija en el texto "de atrás" es decir, el de la izquierda.
El patrón para un _lookbehind_ positivo es el patrón `(?<=)` donde una vez más, `(?<=` actúa como paréntesis de apertura, y `)` como paréntesis de cierre.

Algo muy importante de los _lookarounds_ es que no consumen texto.
Es decir, si coinciden con algún string o una parte de el, el texto de coincidencia no forma parte de lo que se extrae al final.
Justo como _caret_ o _dollar_, no coinciden con texto sino con posiciones.
Por ejemplo, teniendo el string "200 pounds" la expresión `\d+` coincidiría con uno o más dígitos, en este caso coincidiría con todo el número "200".
Ahora bien, con el mismo texto, la expresión `(?=\d+)` coincidiría con el _inicio_ del número "200".
Ahora el _lookbehind_ de la expresión `(?<=\d+)` coincidiría con el _final_ del número "200".
Trataremos de ilustrarlo mejor 

     200 pounds
    ^   ^
    |   L Coincide con (?<=\d+)
    L Coincide con (?=\d+)

Claro que la magia del _lookbehing_ es cuando se combina con expresiones que si consumen texto.
El hecho de que coinciden con posiciones dentro del texto y no consumen texto por si solos, las hace una herramienta como _caret_ o _dollar_, muy útiles y una buena adición al arsenal.

Un ejemplo poderoso: 

Supongamos por ejemplo que estamos haciendo cálculos y queremos obtener el número del resultado en un formato bonito; es decir, con comas para que sea más legible.
La convención es usar comas cada tres dígitos contando de derecha a izquierda para marcar miles, cientos de miles, millones, etc...
Para resolver el problema, encontramos la ubicaciones que _preceden_ a tres dígitos.
Esto ya es facil de poner en términos de regex.
El "preceder" a algo se resuelve fácil con un _lookahead_, en este caso, algo como `(?=\d)` que coincide con las ubicaciones que preceden a un solo dígito.
Es como pedir tener al menos un dígito a la derecha.
Podemos encontrar las que preceden a exactamente tres dígitos con `(?=\d\d\d)`.
Si envolvemos a las tres `\d` dentro de un grupo podemos permitir que haya uno o más "juegos" de este grupo más adelante en la expresión, y finalizamos añadiendo _dollar_ al final para asegurarnos que no hay cosas adicionales.
Al final tenemos la expresión `(\d\d\d)+?` por si sola coincide con los tres últimos dígitos de la línea, pero la expresión dentro de un _lookahead_, es decir, `(?=(\d\d\d)+$)` coincide con todas las **posiciones** en el texto que preceden a un grupo de tres dígitos.

La expresión está casi lista, pero si se la aplicamos al número "123456" nos devolvería ",123,456".
Eso ya está mucho más cerca de cómo escribimos números, pero tiene una coma extra al principio del número que no debería ir ahí.
Para eso, pedimos que además de preceder a tres números, tenga por lo menos uno a la iquierda.
Eso lo logramos añadiendo `(?<=\d)` que coincide con las posiciones que tienen un dígito a la izquierda.

La expresión final en forma de comando de Perl (para el cookbook) queda como:

    s/(?<=\d)(?=(\d\d\d)+$)/,/g

O en una versión "más limpia": 

    (?<=\d)(?=(?:\d\d\d)+$)

En la que solo se usan los grupos de captura que son estrictamente necesarios.
Vale la pena mencionar que la expesión anterior solo funciona con números que están en su propia línea (o por lo menos al final de una), pero no es difícil extender la expresión para poder manejar números incrustados en líneas.