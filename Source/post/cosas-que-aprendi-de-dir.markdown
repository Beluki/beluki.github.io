title: Cosas que aprendí de... dir
date: 2015-12-15
tags: ['lua', 'postmortem', 'programacion']

[dir][] es un videojuego multiplataforma escrito en [Lua][] que utiliza el
framework [Löve2D][]. El estilo de juego es parecido a Bejeweled o Tetris:
juntar piezas de un mismo color para que desaparezcan.

Es también mi intento de diseñar un juego que resulte atractivo a audiencias
casuales y hardcore, sin depender de límites de tiempo o reflejos rápidos.

Tiene esta pinta:

<img src="{{ url_static('15.png') }}" alt="dir">

Si quieres descargarlo y echar una partida antes de continuar leyendo
[haz click aquí](https://github.com/Beluki/dir/releases).

[dir]: https://github.com/Beluki/dir
[Lua]: http://www.lua.org
[Löve2D]: https://love2d.org

## Características

[dir][] podría definirse tanto por las cosas que sí tiene:

* Una mecánica original, basada en acumular combos.
* Altamente rejugable. Una partida completa dura entre 5 y 15 minutos.
* Sistema de puntuación y ranking altamente depurado.
* El juego es completamente independiente de la resolución, dado que dibuja
todas las piezas vectorialmente. Todo en [dir][] es o bien un cuadrado o bien un trozo de texto.
* Tres temas de colores.
* Portable. No utiliza el registro de Windows, ni la carpeta Mis Documentos, etc...

... como por las que no:

* Una pantalla de título.
* Múltiples niveles de dificultad.
* Múltiples modos de juego.
* Cargar o Guardar partida.
* Sonido o música.
* Límite de puntuación o final.
* Pantalla de "game over".

## Löve2D

Antes de empezar a programar el juego, evalué múltiples frameworks como posibles
candidatos. Los principales fueron [Haxe][], [Löve2D][], [Monogame][] y [pygame][].

Dos de ellos, Monogame y pygame, los descarté en parte por sus dependencias ([.NET][] y [Python][]
respectivamente) y en parte por [complicaciones](http://stackoverflow.com/questions/23305577/draw-rectangle-in-monogame)
a la hora de hacer tareas que deberían ser realmente simples, como dibujar un cuadrado en pantalla.

En el caso de Haxe, las incompatibilidades entre la versión web y la versión
escritorio fueron la principal razón de descartarlo. Basta con echar un ojo
a la demo de [PiratePig][] en el blog de Joshua Granick para darse cuenta de
las diferencias (compara por ejemplo la versión HTML5 y Flash).

A día de hoy no sé si puedo decir que Löve fuese el framework "perfecto", pero
volvería a utilizarlo para programar [dir][]. La [documentación](https://love2d.org/wiki/Main_Page)
es excelente. El [foro](https://love2d.org/forums/) también.

[Haxe]: http://haxe.org
[Löve2D]: https://love2d.org
[monogame]: http://www.monogame.net
[pygame]: http://pygame.org/hifi.html

[.NET]: http://www.microsoft.com/net
[Python]: https://www.python.org

[PiratePig]: http://www.joshuagranick.com/2012/02/22/nme-game-example-pirate-pig

## Lua

Utilizar Löve2D implica utilizar Lua.

Lua es en general un lenguaje muy bien diseñado con alguna que otra pega
importante, aunque todas las pegas tienen sentido en el contexto donde Lua
suele utilizarse.

Por ejemplo: Lua no tiene una librería standard decente, no tiene excepciones,
no tiene orientación a objetos, no tiene mil cosas que normalmente son necesarias
cuando no estás programando algo embebido. Löve2D suple en parte la falta de esta
funcionalidad a base de módulos adicionales como "love.audio", "love.filesystem",
etc...

Lua tampoco tiene un modo de distinguir "nil" de "variable no inicializada", lo cual
me ha causado muchos más dolores de cabeza a la hora de programar dir que cualquier
módulo aparte que haya tenido que escribir.

## Arquitectura

[dir][] está escrito de modo que [cada parte][] es lo más independiente posible de todas
las demás. Cada componente usa además su propia noción de orientación a objetos (closures
en realidad), con un "self" explícito al estilo Python.

[cada parte]: https://github.com/Beluki/dir/tree/master/Source

Por ejemplo, para inicializar el juego:

    ::lua
    function Game ()
        local self = {}

        -- initialization:
        self.init = function ()
            self.grid_width = 5
            self.grid_height = 5

            self.grid = Grid(self)
            self.hud = Hud(self)
            self.menu = Menu(self)
            self.screen = Screen(self)
            self.state = State(self)
            self.theme = Theme(self)

            self.resize()
            self.restart()
        end

        -- otros "métodos"...

        self.init()
        return self
    end

donde [Grid][], [Hud][], [Menu][], [Screen][], [State][] y [Theme][] son los
módulos que se combinan para dar forma al juego final.

[Grid]: https://github.com/Beluki/dir/blob/master/Source/game/grid.lua
[Hud]: https://github.com/Beluki/dir/blob/master/Source/game/hud.lua
[Menu]: https://github.com/Beluki/dir/blob/master/Source/game/menu.lua
[Screen]: https://github.com/Beluki/dir/blob/master/Source/game/screen.lua
[State]: https://github.com/Beluki/dir/blob/master/Source/game/state.lua
[Theme]: https://github.com/Beluki/dir/blob/master/Source/game/theme.lua

A más bajo nivel, cada módulo es responsable de sus propias animaciones.
Por ejemplo, Grid contiene todas las definiciones necesarias para que las piezas
del grid que están apareciendo, desapareciendo, moviéndose, etc mantengan su
estado entre frame y frame.

Un detalle importante aquí es que en [dir][] solo existe una animación a la vez
en todo el grid. No es posible tener piezas que se muevan, mientras otras desaparecen
al mismo tiempo. Esto simplifica mucho la implementación, porque puede hacerse en
base al concepto de "Tween", donde cada animación se enlaza con la siguiente a través
de una función puente (ver por ejemplo grid.appearing.oncomplete).

El hilo conductor final es [State][], que decide qué hacer en base al ranking y la
puntuación actuales, cada vez que las piezas cambian de estado.

Finalmente, en la carpeta [lib][] hay dos pequeñas librerías. Una para manipular arrays
de dos dimensiones y otra con wrappers sobre la funcionalidad básica de Löve2D. Esta última
es totalmente prescindible, pero hará más fácil portar [dir][] a futuras versiones de Löve
si este cambia.

[lib]: https://github.com/Beluki/dir/tree/master/Source/lib

## Conclusiones

Estoy muy orgulloso del resultado obtenido con [dir][]. Creo que el resultado ha merecido el
esfuerzo. El [record][] actual será además muy difícil de superar.

[record]: /post/grandmaster

## Bonus

Aquí puedes ver un vídeo de un usuario del foro de Löve2D jugando a dir:

<iframe width="560" height="315" src="https://www.youtube.com/embed/B6CJWG67TdE" frameborder="0" allowfullscreen>
</iframe>

Y un port a Android:

<iframe width="560" height="315" src="https://www.youtube.com/embed/sUHYoeOMYGU" frameborder="0" allowfullscreen>
</iframe>

