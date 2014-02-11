title: Más detalles que importan
date: 2013-09-07
tags: ['software']

En el primer post de este blog comentaba acerca de un detalle de Thunderbird
con respecto a su tratamiento de archivos adjuntos que me gustó. Este post
es un ejemplo de lo contrario, una lista de pequeños bugs o comportamientos
irritantes en programas que uso a menudo.

Estos bugs son especialmente molestos porque me los he encontrado en programas
que usan miles de personas todos los días y - a priori - no parecen difíciles
de solucionar.

## Windows 7 - redimensionando la barra de estado

<img class="align-left" src="{{ url_static('2.png') }}" alt="Statusbar">

Redimensionar una ventana del explorador que tenga una barra de estado causa
la aparición de una minúscula línea negra vertical en el borde inferior derecho.

Hasta donde he podido ver, ocurre con cualquier tarjeta gráfica e independientemente
de estar usando [Aero][] o el tema básico de escritorio.

Curiosamente, el bug solo ocurre si la ventana se redimensiona verticalmente.
Tras maximizar, minimizar o redimensionar la ventana horizontalmente, la línea
negra desaparece. No me acordé de probar si este problema existe también en
Windows 8.

[Aero]: http://es.wikipedia.org/wiki/Windows_Aero

## SMPlayer - por defecto, el ecualizador no funciona

<img class="align-left" src="{{ url_static('3.png') }}" alt="SMPlayer">

[SMPlayer][] es tranquilamente uno de los mejores reproductores de video para Windows.
Incluye codecs de serie, soporta subtítulos, múltiples pistas de audio o menús de
DVD, etc...

Sin embargo, con la configuración que viene por defecto, no funciona ninguno
de los controles del ecualizador.

Cambiar la salida de video de Direct3D a OpenGL funciona, pero desactiva Aero.
La forma de tener ambos a la vez es marcar esa casilla que reza: "Software equalizer".
No he notado pérdida de rendimiento alguna tras marcarla.

[SMPlayer]: http://smplayer.sourceforge.net

## Racket - no se puede copiar/pegar código en la consola interactiva

<img src="{{ url_static('4.png') }}" alt="Racket">

Tenía que incluír una sobre programación.

Por algún motivo, [Racket][] no lee correctamente código que se haya pegado directamente
en su consola interactiva. Insiste en añadir carácteres como "m" y "R" al código. De
hecho lo peor es que a menudo parece funcionar y luego se rompe en el momento más
inesperado.

Este bug ya ocurría cuando Racket aún se llamaba mzscheme.

[Racket]: http://racket-lang.org

## 7-zip - menú contextual

<img class="align-left" src="{{ url_static('5.png') }}" alt="7-zip">

¿Necesito decir más? Es un ejemplo perfecto de "una imagen vale más que mil palabras".

El menú contextual integrado con el explorador de Windows que ofrece [7-zip][] es útil
pero la configuración por defecto es ridícula.

Un elemento tiene submenú: "Open Archive", mientras que el resto de elementos no
y simplemente aparecen repetidos, con ligeras variaciones según lo que hacen. Ni
siquiera están ordenados según su función.

En mi opinión, un diseño más lógico sería algo tipo: "Open, Add, Compress, Extract,
Test" todos con submenú y dentro de cada uno las opciones particulares específicas
para cada elemento.

[7-zip]: http://www.7-zip.org

## Paint .NET - control de zoom

<img src="{{ url_static('6.png') }}" alt="Paint .NET">

Una cosa que me gusta de la interfaz de [Paint.NET][] es que sus desplegables incluyen
las opciones más comunes, pero al mismo tiempo permiten escribir para poder elegir
valores más concretos.

En el caso del zoom, aparecen valores como 33%, 50%, 66%, 100%, 200%, etc...
Supongamos que quiero 120%, como en la imagen. Mi primer ímpetu es escribir el número
y pulsar enter para aplicar los cambios. Ésto funciona perfectamente y el nuevo zoom
se aplica, excepto que cualquier seleccion actual en la imagen se pierde nada más
pulsar la tecla.

El modo de aplicar un zoom específico pero manteniendo cualquier selección implica
escribir el número y después hacer click en otra parte de la interfaz, sin pulsar enter,
para cambiar el foco del control activo. Ésto no es suficientemente intuitivo.

[Paint.NET]: http://www.getpaint.net

## Dropbox - un instalador poco educado

Aquí no hay foto. El instalador que usa [Dropbox][] decide por su cuenta que es buena
idea matar el proceso "explorer.exe" (el explorador de Windows), para asegurarse de
poder activar su integración.

El problema de hacer ésto es que cualquier operación con el explorador en ese momento
se interrumpe sin más. ¿Estabas moviendo 40 GB? pues ánimo, ahora tienes parte de los
archivos en el origen y parte en el destino, apáñate.

[Dropbox]: http://www.dropbox.com

