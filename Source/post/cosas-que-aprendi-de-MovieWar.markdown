title: Cosas que aprendí de... MovieWar
date: 2015-12-18
tags: ['postmortem', 'programacion', 'python']


[MovieWar][] es un trivial multijugador para consola escrito en Python. Es parecido
a "[El precio justo][]", donde los jugadores intentan adivinar la fecha de salida
de una película a partir de su título.

He aquí una foto:

<img src="{{ url_static('16.png') }}" alt="MovieWar">

[MovieWar]: https://github.com/Beluki/MovieWar
[El precio justo]: https://es.wikipedia.org/wiki/The_Price_is_Right

Algunas características interesantes del programa:

* Singleplayer o multiplayer (cualquier número de jugadores, por turnos).
* Incluye 7600 películas populares en la base de datos.
* Dos modos de juego: "película aleatoria" o "un jugador escoge y el resto adivinan fecha".
* Usa [OMDB][] para buscar películas que desconoce y actualiza la DB con las entradas nuevas.
* Soporte de colores opcional, utilizando [Colorama][] (Windows) o colores ansi (Unix, MSYS).

[OMDB]: http://www.omdbapi.com
[Colorama]: https://pypi.python.org/pypi/colorama

## La base de datos

MovieWar es un programa simple, secuencial, basado en turnos. La entrada/salida
de cada paso está claramente definida. El código son unas 700 líneas. La parte
complicada consiste en responder a la siguiente pregunta:

¿Cómo genero una buena base de datos de películas populares para poder jugar?

Mi respuesta fue: utilizando múltiples bases de datos y verificando que todo
coincide antes de añadir ninguna película. Para ello, creé una serie de scripts,
[MovieWarDBGen][] en un repositorio aparte que se encargan de todo el proceso.

[MovieWarDBGen]: https://github.com/Beluki/MovieWarDBGen

## The Movie List

El punto de partida para la base de datos es una lista de 9200 películas que
[Brad Bourland][] recopiló durante años. Esta lista es un "top 9200" y solo
contiene películas anteriores al año 2000. Esto es ideal porque permite verificar
fácilmente el año de salida de cada película con respecto a múltiples bases de datos
y  son películas razonablemente populares (no hay Bollywood ni nada por el estilo).

[Brad Bourland]: http://www.nytimes.com/2010/04/18/movies/18bourland.html?_r=1

El [primer script][] de MovieWarDBGen, usa la API de Freebase para descargar la lista
completa. El [segundo][], convierte los datos a un formato más simple, validando el título
y fecha en el proceso.

El [tercero][], utiliza [OMDB][] para verificar que todas las fechas de salida y títulos
de Freebase coinciden con el esperado en OMDB. De 9169 películas, 7601 concuerdan exactamente
en ambas bases de datos. Este script puede tardar tranquilamente 2 horas en ejecutarse, pero
mejor calidad que cantidad.

Un [cuarto][] script se encarga de colapsar los años de películas que tienen el mismo título
en una sola entrada JSON. Por ejemplo, convierte esto:

    ::json
    {"name": "Jane Eyre", "year": "2011"}
    {"name": "Jane Eyre", "year": "1996"}

En esto otro:

    ::json
    {"name": "Jane Eyre", "years": ["2011", "1996"]}

También comprueba años duplicados.

[primer script]: https://github.com/Beluki/MovieWarDBGen/blob/master/Source/00%20download%20freebase.py
[segundo]: https://github.com/Beluki/MovieWarDBGen/blob/master/Source/01%20convert%20freebase.py
[tercero]: https://github.com/Beluki/MovieWarDBGen/blob/master/Source/02%20match%20omdb.py
[cuarto]: https://github.com/Beluki/MovieWarDBGen/blob/master/Source/03%20collapse%20years.py

## Puntuando

Una vez la base de datos está lista, implementar el juego es fácil.
La única decisión curiosa que toma MovieWar está en cómo puntuar las respuestas
de los jugadores:

MovieWar da 50 puntos por una respuesta exacta y (20 puntos - años de error) por un fallo.
Esto garantiza que la cosa esté un poco reñida al final, debido a penalizaciones
y que cada jugador suele ser experto en diferentes géneros de cine.

