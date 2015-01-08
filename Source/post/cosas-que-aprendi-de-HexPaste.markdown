title: Cosas que aprendí de... HexPaste
date: 2015-01-08
tags: ['postmortem', 'programacion']

Siempre me ha gustado la literatura, especialmente la poesía. Por eso, a veces
comparto aquí las cosillas que voy escribiendo. También por eso, el primer
canal de IRC al que entré hace años era de poesía: #poesia en IRC-Hispano.

Aunque ahora es muy diferente, por aquel entonces el canal era un pequeño y
acogedor grupo de amigos compartiendo poemas. Lorca, Miguel Hernández, Rafael
De León... un poco de todo. De todo esto hace unos 12 años.

Lo primero que aprendí es que copiar y pegar un poema a mano no es práctico.
La mayoría de usuarios (yo incluído) usábamos un comando de [mIRC][] que se
llama /play para recitar.

Tiene esta pinta:

<img src="{{ url_static('10.png') }}" alt="play">

Lo más útil es que permite especificar un retardo entre líneas de modo que el
poema vaya apareciendo poco a poco y sea cómodo de leer. Así fue como recité
durante años, hasta que llegó un momento en el que quise probar linux y otros
sistemas operativos.

## XChat

Lo más parecido a mIRC para linux es [XChat][]. XChat está bien pero no tiene
nada que se parezca al comando /play, así que sin un plugin para poder recitar
volví a tener que recitar los poemas a mano.

Aunque por aquel entonces tampoco tenía mucha idea de programación, decidí intentar
hacer mi propio plugin para XChat que me permitiese recitar poemas en el canal.
Este fue el origen de HexPaste, que es quizá el programa más antiguo (al menos
sus raíces) de todos los que tengo actualmente en Github.

En su momento, escribí ese plugin en [Perl][] y era extremadamente limitado.
Por no poder no podías ni cambiar de pestaña durante un recitado porque siempre
pegaba el texto a la pestaña actualmente activa.

[mIRC]: http://www.mirc.com
[XChat]: http://xchat.org
[Perl]: https://www.perl.org

## HexChat y HexPaste

Tras unos cuantos años un poco alejado del canal y sus quehaceres, al final volví.
Supongo que el gusanillo de la poesía está siempre ahí. XChat es ahora un proyecto
fantasma (su última actualización es de hace 5 años) y lo que se lleva es [HexChat][].

Con más conocimientos, decidí reescribir aquel plugin que había hecho en Perl pero
en Python, mucho más robusto y práctico que antaño. Esta vez, añadiría cosas como
poder recitar en múltiples canales a la vez o que sustituyese líneas en blanco en
un archivo por espacios.

[HexPaste][] es en realidad un programa muy simple. Son unas 350 líneas de código
escritas en Python 3. No hubo grandes problemas durante su desarrollo en cuestión
de arquitectura porque tenía bastante claro lo que buscaba.

El mayor problema fue la [API para Python de HexChat][]. Esta API es poco más que
la API de C pero expuesta a Python, sin ningún tipo de ayuda o intención por hacerla
más idiomática a lo que un programador de Python esperaría:

* Muchos campos son bitfields.

* Un "contexto" (canal, query) del tipo que [hexchat.get_context()][] devuelve
  no es "hashable" y no se puede usar como clave en un diccionario de Python.

* Muchas veces HexChat se come las excepciones, sin mostrar mensaje de error alguno.

* Como se usan timers en lugar de threads es extremadamente fácil hacer que HexChat
  pete completamente debido a un bug y tengas que reabrirlo.

Tengo la teoría de que es esta API la culpable de que la mayoría de plugins para
HexChat (y antes XChat) sean juguetes comparados con los que existen para mIRC
y otros clientes de IRC.

Por lo demás, tacos mediante, HexPaste fue más o menos fácil de crear. Y fácil
de portar de Python 2 a Python 3. Es uno de los proyectos que más he usado de
los que he hecho.

[HexChat]: http://hexchat.github.io

[API para Python de HexChat]: http://hexchat.readthedocs.org/en/latest/script_python.html
[hexchat.get_context()]: http://hexchat.readthedocs.org/en/latest/script_python.html#hexchat.get_context

## Conclusiones

Creo que la lección más importante que he aprendido de HexPaste es que la necesidad
es una buena compañera de proyectos.

Muchos programas parten de ideas interesantes a explorar, donde hay que hacer una
mayor labor de diseño que de programación. Este parte de necesitar algo que no existe
para aliviar la incomodidad de copiar/pegar a mano. Hay pocas satisfacciones mejores
que usar diaramente algo que has creado tú.

Repositorios en Github: [HexPaste][]

[HexPaste]: https://github.com/Beluki/HexPaste

