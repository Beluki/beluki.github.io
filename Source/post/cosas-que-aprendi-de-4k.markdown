title: Cosas que aprendí de... 4k
date: 2014-12-10
tags: ['postmortem', 'programacion']

Me he mudado hace unas semanas. El caso es, que con todo el trajín, mis proyectos
han pasado a un segundo plano. Como también tengo este blog un tanto abandonado,
creo que un buen modo de volver a retomarlos es escribir unos cuantos post acerca de
ellos. Hablar de las cosas que salieron bien y las que no tan bien, decisiones de
diseño, etc...

## Sokoban

Hace ya un par de años, escribí un clon de [Sokoban][] en C. La idea era hacerlo
en 4 kilobytes (un .exe de no más de 4096 bytes) al estilo de lo que hacen los
grupos en la [Demoscene][] o la gente de [Java4k][].

Sokoban es un juego con unas reglas extremadamente simples, así que parecía el
candidato ideal. El producto final, aunque sin sonido, tenía gráficos decentes
(con la paleta CGA de Sokoban de MS-DOS) y los 50 niveles originales. Recuerdo
la experiencia como frustrante, intentando sacar bytes de cualquier parte,
pero divertida.

Una anécdota es que el código de dibujar el nivel y el de comprobar si se ha
resuelto estaba mezclado en un solo bucle y ambos se hacían a la vez.

[Demoscene]: http://es.wikipedia.org/wiki/Demoscene
[Java4k]: http://www.java4k.com/index.php?action=home
[Sokoban]: http://es.wikipedia.org/wiki/Sokoban

## Una base común

Tras escribir Sokoban pensé que algo útil sería tener un framework, una especie
de plantilla con una base común reutilizable para demos o juegos de 4 kb. En un
alarde de originalidad, decidí llamar al proyecto... 4k.

Lo primero que aprendí es que es difícil definir una base común.

Los juegos necesitan control de teclado y ratón, las demos no. Las demos suelen
verse a pantalla completa, los juegos en una ventana. Para un juego simple, 2D
via [GDI][] es más que suficiente, las demos a menudo usan [OpenGL][] o [DirectX][].

[DirectX]: http://es.wikipedia.org/wiki/DirectX
[GDI]: http://es.wikipedia.org/wiki/Graphics_Device_Interface
[OpenGL]: http://es.wikipedia.org/wiki/OpenGL

Al final, definí la base común como:

* Crear una ventana con una resolución arbitraria.
* Un bucle (game loop) que funcione a una velocidad constante.
* Poder dibujar en la ventana con doble buffer.

Muchos juegos (especialmente los que caben en 4k), son más cómodos en ventana.
A pantalla completa, hay que preocuparse además de cosas como el aspect ratio
o la tasa de refresco de cada monitor. Aunque ambos pueden autodetectarse
decidí escoger el camino más simple.

El bucle es también el más simple posible. Se asume que todo PC puede ejecutar
la demo a la tasa de frames requerida sin problemas de rendimiento. Ni siquiera
se separa la lógica/renderizado, todo se actualiza a la misma velocidad. Para un
ejemplo de lo que puede complicarse un gameloop, lee [Fix Your Timestep!][]. En
la práctica, para juegos sencillos, el bucle más simple es perfectamente usable.

Decidí no añadir nada para controlar el teclado. En ese punto... ¿por qué no ratón
y joystick?. Muchas demos no necesitan ninguno de ellos. Muchos juegos solo usan
uno de entre todos.

Finalmente, en lugar de hacer una plantilla, hice dos: una GDI y otra OpenGL.

[Fix Your Timestep!]: http://gafferongames.com/game-physics/fix-your-timestep/

## Sin librería standard

En 4k no hay suficiente espacio para usar la librería de C. No hay malloc()
o rand(). Lo cierto es que no tener ninguna de estas funciones no
supuso un problema, ni en las demos de ejemplo que hice para 4k ni en Sokoban.
Normalmente, recrear lo que se necesita es fácil.

De elegir algo problemático, sería lo de siempre: C. Cada vez estoy más
convencido de que usar lenguajes donde un error puede pasar completamente
inadvertido es una mala idea, incluso para programas tan pequeños como 4k.

Tener excepciones también ayudaría a que el código que controla errores
no fuese tan extremadamente repetitivo al tener que comprobar manualmente
cada valor de retorno.

Algo que lo que sin embargo no me quejo es de la API de Windows (al menos de
las partes que he usado para 4k). Está bien documentada, es razonablemente
simple y funciona. [Xlib][] es mucho más coñazo.

[Xlib]: http://es.wikipedia.org/wiki/Xlib

## Demos

<img src="{{ url_static('9.png') }}" alt="demos">

No había mejor modo de testear 4k que usándolo para hacer una demo simple.
En ambos casos (GDI y OpenGL), elegí algo que pudiese crearse combinando
las cosas más primitivas de la API: rectángulos en GDI, cubos en OpenGL.

Otra de las restricciones era que la demo debería funcionar correctamente
a cualquier tamaño de ventana y a cualquier tasa de frames por segundo, lo
que haría más fácil testear.

## Conclusiones

Lo que pasa por mi cabeza al recordar el proyecto es que me parece increíble
que haga falta tanta historia para dibujar unos cuantos rectángulos.

Creo que si hubiese nacido en el 2000, nunca habría aprendido a programar.
Los sistemas operativos ya no incluyen herramientas para ello por defecto.
Falta esa inmediatez de tener siempre un QBasic a mano para trastear, donde
hacer el equivalente a 4k-example son unas 10 líneas.

Es cierto que hoy los sistemas operativos y en general los programas hacen
cosas mucho más complejas que entonces. Hemos avanzado una barbaridad. Quizá
sea la nostalgia, pero tengo la sensación de que en algún punto entre todo este
progreso, se perdió algo importante.

