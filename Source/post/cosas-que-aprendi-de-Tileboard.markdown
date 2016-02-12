title: Cosas que aprendí de... Tileboard
date: 2016-02-12
tags: ['postmortem', 'programacion', 'python']

[Tileboard][] es un programa escrito en Python 3 que genera diagramas de
juegos de mesa, como por ejemplo ajedrez. Lo escribí porque hace un tiempo
inventé una variante de ajedrez que utiliza un tablero distinto al habitual
y no encontré ningún otro programa que pudiese dibujarlo.

Sobre el juego en si, de momento no diré nada, pero he aquí una foto el tablero
en cuestión, que tiene 5x7 casillas en lugar de las habituales:

<img src="{{ url_static('22.png') }}" alt="Duchess">

[Tileboard]: https://github.com/Beluki/Tileboard

Básicamente, lo que buscaba era un programa que:

* Pudiese generar cualquier tablero "parecido" al de ajedrez.
* Con o sin coordenadas.
* Con cualquier tipo de piezas (othello, reversi, etc...).
* En cualquier tamaño, tanto de imagen como de casillas.
* Personalizable en cuestión de colores, fuentes, etc...
* Usase poca memoria para ello (para poder imprimir el tablero real, en grande).

¿No parece difícil no?

## Arquitectura

Como librería para manipular imágenes, decidí escoger [Pillow][]. Ya la
usé en su momento para otros proyectos con buenos resultados. Es rápida
y práctica.

[Pillow]: https://pypi.python.org/pypi/Pillow/3.1.0

En cuestión de código, Tileboard tiene unas 900 líneas, divididas más
o menos en las siguientes tareas:

* Parsear posiciones FEN.
* Representar el tablero a dibujar.
* Conversiones de base para los bordes (base26).
* Parsear notación algebraica (A1, H2...)
* Cargar imágenes y fuentes.
* Cálculos de tamaño de imagen, borde, etc...
* Funciones utilidad de ayuda para dibujar.
* Funciones que realmente dibujan el tablero.
* El parser de línea de comandos.
* Dibujado del tablero en si.

La mayoría del código es simple (y podría serlo más), excepto quizá
los cálculos de cómo dibujar las letras/números de los bordes y del
tamaño de la imagen final, que depende del tamaño de las piezas del
tablero y de los elementos que la componen (borde, líneas adyacentes...)

Lo que también es, es repetitivo. Una buena muestra de ello podría ser:

    ::python
    # outer outline:
    if not options.outer_outline_disable:
        draw_rectangle_outline(image,
                                  x1 = 0,
                                  y1 = 0,
                                  x2 = image.width - 1,
                                  y2 = image.height - 1,
                               width = outer_outline_size - 1,
                               color = options.outer_outline_color)

    # border:
    if not options.border_disable:
        draw_rectangle_outline(image,
                                  x1 = outer_outline_size,
                                  y1 = outer_outline_size,
                                  x2 = image.width - outer_outline_size - 1,
                                  y2 = image.height - outer_outline_size - 1,
                               width = border_size - 1,
                               color = options.border_color)

Y así para cada elemento del tablero, incluyendo línea exterior, borde, línea
interior, casillas, piezas y cruces y puntos para señalar posiciones. Es tedioso
pero diría sin duda que volvería a escribirlo igual si tuviese que hacerlo.

## Algunas lecciones aprendidas

Me quedo con:

* Dibujar siempre todo por separado.
* Partir de (0, 0) al dibujar. Es mucho más fácil razonar con coordenadas absolutas.
* Separar funciones con conocimiento específico y utilidades, especialmente de dibujo.
* Tener modo de desactivar cada elemento para poder depurar cómodamente.

Y la más importante: **leer la documentación** de las librerías que uso.

De la de [ImageDraw][]:

    ::python
    PIL.ImageDraw.Draw.rectangle(xy, fill=None, outline=None)

    Draws a rectangle.
    Parameters:

        xy: Four points to define the bounding box.
            Sequence of either [(x0, y0), (x1, y1)] or [x0, y0, x1, y1].
            The second point is just outside the drawn rectangle.

            ...

[ImageDraw]: https://pillow.readthedocs.org/en/3.1.x/reference/ImageDraw.html

"The second point is just outside the drawn rectangle." De ahí el -1 en casi todas
las funciones de dibujado de Tileboard y varias horas de trabajo de depuración.

Es curioso que al final lo más fácil de todo el asunto fuese dibujar las piezas.

## Conclusiones

La verdad es que estoy bastante satisfecho con el resultado. Quizá me pasé un poco
en la cantidad de argumentos que tiene, pero es flexible y práctico. En el
README hay ejemplos de otros juegos como las Damas o incluso de un Bejeweled
con sus gemas.

Dejo una imagen de bonus, mostrando el proceso de dibujado de un tablero
normal de ajedrez. Faltan algunos frames que estarían en otras imágenes,
dado que este tablero no tiene huecos, cruces o puntos, pero la idea es la misma:
ir construyendo todo de manera independiente.

(la imagen comienza siendo completamente transparente):

<img src="{{ url_static('23.gif') }}" alt="Tileboard GIF">

Como con todos los proyectos que he estado revisando, he [hecho release][].

[hecho release]: https://github.com/Beluki/Tileboard/releases

