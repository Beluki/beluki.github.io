title: Cosas que aprendí de... MQLite
date: 2015-12-14
tags: ['postmortem', 'programacion', 'python']

[MQLite][] es un pequeño dialecto basado en [MQL][], el lenguaje que se utiliza en
[Freebase][] para realizar búsquedas. Está escrito en Python 3 y es un compilador
sencillo a un [AST][] recursivo de nodos que implementan un método "match".

Aunque el núcleo de MQLite es un subset de MQL, en realidad implementa bastantes más
cosas que el original, debido a la necesidad de realizar búsquedas sobre [JSON][]
arbitrario en lugar de una base de datos concreta.

Como lo mejor es un ejemplo... El siguiente comando devuelve la lista de repositorios
que tengo en Github con al menos 1 fork activo.

    ::bash
    $ curl https://api.github.com/users/Beluki/repos
    | MQlite.py '[{"name": null, "forks": null, "forks >": 0}]'
    [
        {
            "name": "GaGa",
            "forks": 2
        },
        {
            "name": "MQLite",
            "forks": 1
        }
    ]


[AST]: https://en.wikipedia.org/wiki/Abstract_syntax_tree
[JSON]: https://en.wikipedia.org/wiki/JSON
[Freebase]: https://www.freebase.com
[MQLite]: https://github.com/Beluki/MQLite
[MQL]: http://wiki.freebase.com/wiki/MQL

## Arquitectura

MQLite es un proyecto relativamente simple porque no necesita un parser. Utiliza la misma
sintaxis que JSON. Una vez tenemos los datos a buscar y un archivo .json donde buscarlos,
solo queda preprocesar la búsqueda para poder hacerla del modo más eficientemente posible.

Uno de los límites que me puse al implementarlo, es que MQLite nunca debería modificar
los datos JSON sobre los que busca, ni ordenándolos ni optimizándolos. Al fin y al cabo, si
realmente necesitas eso, te conviene más una base de datos real.

Si no podemos modificar los datos, solo queda representar la búsqueda. En el caso de MQLite,
está hecha a partir de una serie de nodos (objetos) con un método: "match" que devuelve o bien
una "respuesta" o "vacío" dependiendo de si los datos de la búsqueda coinciden con alguno
existente en la base de datos.

Como "None" es un resultado válido, "vacío" se representa con una clase propia:

    ::python
    class _NoMatch(object):
        """
        A custom class to represent no matches.
        Needed because None is a legitimate match.
        """
        pass

    NoMatch = _NoMatch()


Y "concuerda con cualquier cosa" se representa así:

    ::python
    class MatchAny(object):
        """
        Match any input data.
        """
        def match(self, data):
            return data


La mayoría de nodos son sencillos. Cadenas, números o booleanos solo concuerdan consigo mismos.
Otros datos, como las listas o los diccionarios, permiten operaciones más complejas.

Un ejemplo de búsqueda una vez transformada a esta representación, podría ser:

    ::json
    [{ "name": null, "age >": 25, "__sort__": "age" }]

Que queda así:

    ::python
    MatchList(
        MatchDict(
            [{ "name": MatchAny() }],
            [{ "age": ConstraintBiggerThan(25) }],
            [{ "__sort__": DirectiveSort("age") }]
        )
    )


Y significa:

Dame el nombre de todos los items en la base de datos que cumplan la condición de tener una edad
mayor de 25 e imprímelos en orden ascencente.

## Dificultades

La parte más complicada de escribir MQLite fue con bastante diferencia encontrar una sintaxis
adecuada para expresar las búsquedas. En el [README][] hay numerosos ejemplos de búsquedas complejas
con directivas, constraints y todo tipo de opciones.

Dos limitaciones de MQLite son:

Es imposible escribir una cláusula "A o B" para dos claves distintas.

    ::json
    [{ "a:age >": 20, "b:name in" ["a", "b", "c"] }]


Algunas directivas, en particular "\_\_limit\_\_", son sensibles al orden.

    ::json
    [{ "name": null, "__sort__": "age", "__limit__": 3 }]

Esta última búsqueda ordenará PRIMERO y limitará después en lugar de hacerlo al contrario.
En su API, MQLite usa [OrderedDict][] para asegurarse de mantener el orden correcto en cada
búsqueda (dado que los diccionarios de Python no garantizan un orden concreto).

[OrderedDict]: https://docs.python.org/3/library/collections.html#collections.OrderedDict
[README]: https://github.com/Beluki/MQLite

## Compatibilidad con Python

Aunque MQLite está pensado para JSON, puede buscar en prácticamente cualquier cosa iterable
de Python. Para hacerlo, basta con usar el objeto "Pattern" de la API en lugar de "JSONPattern".

    ::python
    data = [
        {
            "name": "James",
            "age": 23,
            "student": False,
            "hobbies": ["chess", "football", "basketball"]
        },
        {
            "name": "John",
            "age": 35,
            "student": True,
            "grades": { "chemistry": "C", "english": "A" },
            "hobbies": ["reading", "swimming", "painting"]
        }
    ]

    pattern = Pattern([{ "name": None, "age in": set([20, 21, 22, 23]) }])
    print(pattern.match(data))

Como añadido, MQLite incluye una [pequeña shell][] para poder experimentar con distintas
búsquedas de manera rápida, explorar datasets, etc... También hay una suite de [tests][] disponible.

[pequeña shell]: https://raw.githubusercontent.com/Beluki/MQLite/master/Source/MQLiteSH.py
[tests]: https://github.com/Beluki/MQLite/blob/master/Test/MQTest.py

