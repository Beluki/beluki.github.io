title: Cómo no aprender a programar
date: 2013-09-11
tags: ['programacion']

Si me preguntases, hace un tiempo habría dicho que una buena introducción
práctica a la programación sería tratar de resolver los problemas planteados
en los ejercicios del [Proyecto Euler][]. Escoger un lenguaje, por ejemplo
Scheme o Python, e ir resolviendo.

A día de hoy mi respuesta sería bien distinta.

El primer problema consiste en calcular la suma de todos los números naturales
menores de 1000 que sean múltiplos de 3 o 5. Una posible solución, escrita
en [Racket][], es la siguiente:

    ::racket
    #lang racket/base

    (define (multiple-of-3-or-5? n)
        (or (zero? (modulo n 3))
            (zero? (modulo n 5))))

    (for/fold ([sum 0])
              ([i (in-range 1000)]
               #:when (multiple-of-3-or-5? i))
        (+ sum i))

[Niklaus Wirth][] dijo allá por 1976 que "algoritmos + datos = programas".
Si analizamos el texto anterior vemos que hay algoritmos, expresados en
forma de función: "multiple-of-3-or-5?", y vemos que hay datos, como el
rango de números de 1 a 1000. ¿Podemos decir entonces que es un programa?
Claramente podemos, pero sin embargo no se parece gran cosa a un programa "real".

Cuando estudié informática, me enseñaron a programar de un modo similar. Nos
proponían pequeños ejercicios, [puzles][], cosas como: "encuentra el elemento X
en la lista enlazada Y" o "ordena este vector de N elementos". Creo que es una
manera horrible de aprender.

[puzles]: http://lema.rae.es/drae/?val=puzle

## El otro 90%

La pregunta clave es: ¿qué falta aquí?.

Cada vez estoy más convencido de que la parte difícil de la programación no es
la algoritmia. Diría que el 10% de un programa es pura algoritmia. El otro
90% es control de errores, I/O a archivos o sockets, localización tanto de
lenguaje como de moneda o huso horario, codificación (unicode, shift-jis, etc),
aprovechamiento de recursos (múltiples procesadores, memoria), diseño modular
en componentes y librerías, interactuar con programas y librerías escritas
por otras personas, portabilidad entre arquitecturas, etc...

No hay nada de eso en el programa anterior.

Crear algo en el laboratorio es fácil, es un entorno seguro sin agentes
externos. Todo está bajo un estricto control. En la vida real, ni siquiera
el enunciado del problema está completamente especificado. Los requerimientos
cambian día a día.

La mayoría de problemas importantes en algoritmia ya han sido resueltos. Las
matemáticas son exactas. Sin embargo, se discute día a día sobre patrones y
métodos para estructurar el código que hagan más sencilla su comprensión. Se
discute sobre dinámicas de trabajo en equipo que permitan a las personas
cooperar más eficientemente.

¿Quieres aprender a programar? Lee el código de un programa que uses día a día
y analíza su diseño. Trata de modificarlo. Ignora "qué" e intenta entender el
porqué de cada decisión y sus consecuencias.

[Niklaus Wirth]: http://es.wikipedia.org/wiki/Niklaus_Wirth
[Proyecto Euler]: http://projecteuler.net
[Racket]: http://racket-lang.org

