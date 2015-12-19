title: Cosas que aprendí de... Yava
date: 2015-12-19
tags: ['postmortem', 'programacion']

[Yava][] es un lanzador de aplicaciones escrito en C#, pensado para ser usado
principalmente con emuladores como [Dolphin][], [Mame][], [Snes9x][], etc...
Es un pequeño menú desde el que ejecutar los juegos de una manera cómoda y rápida.

[Dolphin]: https://dolphin-emu.org
[Mame]: http://mamedev.org
[Snes9x]: http://snes9x.ipherswipsite.com

Tiene esta pinta:

<img src="{{ url_static('17.png') }}" alt="Yava">

[Yava]: https://github.com/Beluki/Yava

Aunque ya existen muchos programas parecidos (como [LaunchBox][] o [mGalaxy][]) no pude
encontrar ninguno que cumpliese los siguientes requisitos:

* Fuese portable (funcione desde un pincho USB).
* No utilice una base de datos interna, solo carpetas y archivos.
* Sea fácil de configurar, preferiblemente en un solo archivo.
* Solo ejecute los juegos, delegando el resto del trabajo a cada emulador.

[LaunchBox]: https://www.launchbox-app.com
[mGalaxy]: http://www.mgalaxy.com

## Arquitectura

Yava es prácticamente idéntico a [GaGa][] en diseño.

<img src="{{ url_static('18.png') }}" alt="Yava">

[GaGa]: http://beluki.github.io/post/cosas-que-aprendi-de-GaGa
[LowKey]: https://github.com/Beluki/LowKey
[mINI]: https://github.com/Beluki/mINI

Usa las mismas librerías: [mINI][] y [LowKey][]. Los componentes son también
los mismos, excepto que no existe el concepto de Player (no hay reproductor)
y StreamsFile es ahora FoldersFile, que es el fichero donde reside la configuración
de todos los juegos/emuladores.

Las únicas partes complicadas del código de Yava son probablemente el código que se
encarga de recordar qué rom está seleccionada para cada carpeta (líneas 276 en
adelante en [Yava.cs][]) y lo relativo a ejecutar el emulador final pasándole la ruta
correcta de la carpeta donde está.

[Yava.cs]: https://github.com/Beluki/Yava/blob/master/Source/Yava/Yava.cs#L276

## Paths

¿No es tan complicado, no? Solo es ejecutar el emulador + la rom. El problema es
que no hay una ruta correcta, sino múltiples rutas que pueden ser absolutas o relativas.

Por ejemplo, si el archivo Folders.ini contiene...

    ::ini
    [Wii]
    path = Romset\Wii
    executable = Emulators\Dolphin 4.0.3\Play\Dolphin.exe
    parameters = --batch --exec "%FILEPATH%"
    extensions = iso, wbfs

Entonces:

* "path" es una ruta relativa o absoluta a la carpeta donde están las roms de Wii.

* "executable" es una ruta relativa o absoluta a la carpeta donde reside el emulador, Dolphin.

* "parameters" contiene los argumentos a pasarle a Dolphin, donde el %FILEPATH% es la ROM a usar.

## Dos curiosidades

¿Cuál es el rendimiento que cabe esperar de un programa como Yava? Según mis pruebas
tarda menos de 1 segundo en cargar las más de 45.000 roms de Mame directamente del
sistema de archivos, sin base de datos ni nada. Buscar es instantáneo.

¿Qué ocurre si configuramos el archivo Folders.ini así?

    ::ini
    [Snes]
    path = Romset\Snes
    executable = %FILEPATH%
    extensions = smc

Yava intentará abrir las roms con extensión .smc con la aplicación asociada a ellas
por defecto (sea el emulador que sea). Es posible hacer esto para cualquier extensión.

