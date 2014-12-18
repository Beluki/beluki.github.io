title: Cosas que aprendí de... EmbedDIBits
date: 2014-12-18
tags: ['postmortem', 'programacion']

Aunque tras escribir Sokoban decidí crear un framework, 4k no fue el único
proyecto que nació como resultado del juego. Durante el desarrollo de 4k,
empecé dos proyectos más, dos utilidades, ambas escritas en Python.

La primera de las utilidades se llama EmbedXSB. Traduce niveles de Sokoban
en formato [XSB][] a cabeceras de C. Como la cabecera resultado es específica
para Sokoban 4k (diseñada para ocupar lo menos posible), no llegué nunca
a limpiar y mejorar el programa y publicarlo en Github. EmbedXSB me permitió
incluír los 50 niveles originales de Sokoban en Sokoban 4k.

La segunda se llama [EmbedDIBits][]. Lee imágenes en formato PNG, BMP, JPG...
y crea cabeceras de C con un array de bytes que contiene el color de cada
pixel, en formato: 0xAARRGGBB, 32 bits. La idea es poder tomar los sprites
originales de Sokoban y tenerlos directamente en C, sin ningún tipo de archivo
intermedio.

[XSB]: http://sokosolve.sourceforge.net/FileFormatXSB.html

## Python

En su momento escribí EmbedXSB y EmbedDIBits en Python 2. EmbedDIBits usaba
[PIL][] para leer las imágenes. Con el tiempo, porté EmbedDIBits a Python 3
y cambié de PIL a [Pillow][], dado que PIL lleva años sin actualizarse y no
soporta Python 3.

Mi experiencia con el lenguaje y las librerías es excelente. Cosas como parsear
opciones de línea de comandos (en plan "--help" o "--option bla") son muy fáciles
de hacer con argparse. Construcciones como "with open(...)", que cierran
automáticamente un archivo tras utilizarlo, son prácticas. Portar de Python 2
a Python 3 fue trivial, aunque también es cierto que EmbedDIBits es un programa
muy pequeño y simple.

De quejarme de algo sería de la conversión automática de finales de línea
dependiendo de la plataforma actual en la que se ejecuta el código:

    ::python
    # texto\n en Unix
    # texto\r\n en Windows
    # texto\r en Mac OS
    print(texto)

    # el final de línea que quieras, pero ha de ser en bytes:
    eol = b'\n'
    sys.stdout.buffer.write(bytes(texto, "utf-8") + eol)

Me quejo porque tengo la costumbre de permitir al usuario especificar cualquier
final de línea que desee ("--newline formato" en EmbedDIBits). Mi opinión es que
un programa realmente portable debe comportarse igual en todas las plataformas
y la salida que produce ha ser idéntica y reproducible.

[PIL]: http://www.pythonware.com/products/pil
[Pillow]: http://pillow.readthedocs.org

## Utilidades de consola

Quizá la mejor lección que aprendí de EmbedDIBits es a respetar y tener en cuenta
todo aquello que un usuario experimentado espera del comportamiento de un programa
de consola. Algunos ejemplos al respecto:

* Usa el programa stdout/stderr correctamente?
* Provee --help? Son el resto de opciones fáciles de entender por su descripción?
* Si produce archivos... puede imprimir su salida también por stdout?
* Puede redirigirse su stdout/stderr fácilmente?
* Está pensado para combinarse con otras herramientas? (por ejemplo: grep)
* Devuelve 0 o otro valor al terminar dependiendo de si ha habido errores?
* Entiende nombres de archivos con espacios? con carácteres UTF-8?
* Si la tarea que ha de realizar es larga: provee feedback adecuado al usuario?
* Tiene alguna opción para suprimir estos mensajes cuando no son necesarios?

Hay mucho más detrás de lo que parece (para variar) y es fácil olvidarse
de las cosas. Ahora, cada vez que empiezo un programa nuevo, reviso todo esto
y me guío por los que ya he hecho para mantener un standard aceptable.

## Conclusiones

Lo que más me llama la atención de todo esto es que un proyecto relativamente
pequeño, como Sokoban 4k, puede dar lugar al nacimiento de otros 3 proyectos más.

Este patrón se ha vuelto común en mí. Es además una manera cómoda de testear
bien mis librerías, programas, etc..., porque cada proyecto depende de otros.
Al final todos se testean mútuamente.

Repositorios en Github: [EmbedDIBits][]

[EmbedDIBits]: https://github.com/Beluki/EmbedDIBits

