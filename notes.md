---
author : Alonso Martinez 
title : Notas sueltas - Mastering Regex
...

Introduction 
============

Searching Text Files: Egrep
---------------------------

El utility _egrep_ viene inclído en MacOS y otros!
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

