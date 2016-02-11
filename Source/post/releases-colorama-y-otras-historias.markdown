title: Releases, Colorama y otras historias
date: 2016-02-11
tags: ['programacion', 'python']

Esta semana he estado revisando todos mis proyectos en Github y testeándolos
un poco con las versiones nuevas de los compiladores y librerías que usan.
No hay cambios en los Changelog más allá de que compilan con VS 2015 o el
último MinGW, Python 3.5... este tipo de cosas.

He hecho release de todos, así que si quieres tener los últimos ejecutables
de [dir][], [GaGa][], [Yava][], etc... están calentitos salidos del horno.

[dir]: https://github.com/Beluki/dir
[GaGa]: https://github.com/Beluki/GaGa
[Yava]: https://github.com/Beluki/Yava

[22 proyectos][]. Y aún me parece que empecé ayer.
Solo hay un proyecto con cambios reales y necesarios, del que ya hice un
postmortem en su momento: [MovieWar][].

[22 proyectos]: https://github.com/Beluki?tab=repositories
[MovieWar]: https://github.com/Beluki/MovieWar

## Colorama y MovieWar

[Colorama][] es la librería que uso en MovieWar para poder escribir con colores
en la terminal. Es portable y funciona correctamente en Windows, Linux, etc...
O eso se supone.

Probando en [MSYS2][] en lugar de [MSYS][], me di cuenta de que MovieWar no mostraba
color alguno. ¿Uh?. La razón es que [mintty][] no se comporta como una consola
normal sino "no interactiva" y que además, ya es capaz por si misma de reconocer
los códigos de colores ANSI sin necesidad de usar librería alguna.

[Colorama]: https://pypi.python.org/pypi/colorama
[MSYS2]: https://msys2.github.io
[MSYS]: http://www.mingw.org/wiki/msys
[mintty]: https://mintty.github.io

Como es imposible saber bajo que consola se está ejecutando MovieWar o las cosas
soporta, mi solución fue la siguiente:

* Asumir por defecto que la terminal no soporta color alguno.
* Dos parámetros: "--ansi-colors" y "--colorama" para el usuario pueda elegir.
* Usar siempre los [códigos de escape ANSI][] y cerrarlos a mano, sin autoreset en
colorama.

[códigos de escape ANSI]: https://en.wikipedia.org/wiki/ANSI_escape_code

El código tiene esta pinta (nota: no se pueden poner ambos parámetros a la vez):

    ::python
    # initialize colors as empty strings until we know we want them:
    player_colors = ['', '', '', '', '']
    game_color = ''

    enable_colors = False

    if options.colorama and HAVE_COLORAMA:
        colorama.init()
        enable_colors = True

    if options.ansi_colors:
        enable_colors = True

    if enable_colors:
        # red, green, yellow, magenta, cyan:
        player_colors = ['31m', '32m', '33m', '35m', '36m']

        # white:
        game_color = '37m'

Y la función para imprimir en color:

    ::python
    def print_color(color, message = ''):
        """
        Print a message to stdout, possibly in color.
        When the message is empty, print a newline.

        When color is an ansi code: use it for coloring.
        When color is an empty string: behave like the regular print().
        Auto-reset color back to default after printing.
        """
        if color == '':
            print(message, flush = True)
        else:
            print('\033[1;' + color + message + '\033[0m', flush = True)

Y ahora todo funciona como debería en cualquier terminal:

Debian:

<img src="{{ url_static('19.png') }}" alt="MovieWar Debian">

Windows, cmd.exe:

<img src="{{ url_static('20.png') }}" alt="MovieWar cmd.exe">

Windows, MSYS2:

<img src="{{ url_static('21.png') }}" alt="MovieWar MSYS2">

## Otras historias

En medio de todo el embrollo, también he terminado un proyecto "nuevo"
que ya está disponible en Github: [Tileboard][]. Quizá haga un postmortem
sobre él en el futuro. Es uno de esos programas que parecen triviales a
simple vista, pero no lo son. De momento el README tiene bastante información.

[Tileboard]: https://github.com/Beluki/Tileboard

Dejo también una curiosidad de la que [mi pareja][] me ha hablado y que
me ha gustado mucho: [Astropix][] o "Astronomy Picture of the Day", donde
hay una foto nueva sobre el cosmos cada día con explicación y todo.

Si les visitas, no dejes de echar un ojo al archivo, porque llevan haciendo
esto desde el [16 de Junio del 95][]. Hay muchas preciosas. [Me encanta ésta][].

[mi pareja]: http://verleeryescuchar.blogspot.com.es
[Astropix]: http://apod.nasa.gov/apod/astropix.html
[16 de Junio del 95]: http://apod.nasa.gov/apod/archivepix.html
[Me encanta ésta]: http://apod.nasa.gov/apod/ap000130.html

