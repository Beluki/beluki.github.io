title: Stack Overflow
date: 2014-03-06
tags: ['programacion']

Leo una [entrada en Stack Overflow][] conmemorando su lanzamiento en la que
el autor invita a los usuarios a crear el programa más pequeño posible
que pueda causar un error de [stack overflow][].

He aquí una posible solución. No es pequeña, pero es interesante. Es algo
que puede ocurrir en el uso diario y que provoca un stack overflow en
casi todas las implementaciones de lenguajes que conozco.

La idea es tan simple como crear dos estructuras de datos extremadamente
profundas y comprobar su igualdad. El siguiente ejemplo usa listas en Python.

    ::python
    >>> def deep_list(n):
            result = []
            for i in range(n):
                result = [result]
            return result

    >>> deep_list(10000) == deep_list(10000)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    RuntimeError: maximum recursion depth exceeded in comparison

La mayoría de implementaciones tienen en cuenta la longitud de las estructuras
de datos a la hora de escribir el código que las compara, pero no su
profundidad. El mismo código produce un stack overflow en las implementaciones
actuales de Ruby, Lua, C#, Java y muchos otros lenguajes.

Lo mismo, escrito en Scheme y ejecutado en [SISC][].

    ::scheme
    > (define (deep-list n)
          (let loop ((ls '()) (n n))
              (if (zero? n)
                  ls
                  (loop (cons ls '()) (- n 1)))))

    > (equal? (deep-list 10000) (deep-list 10000))
    Exception in thread "main" java.lang.StackOverflowError
      at sisc.data.Pair.car(Unknown Source)
      at sisc.data.Pair.valueEqual(Unknown Source)
      at sisc.data.Pair.valueEqual(Unknown Source)
      ...

Dos excepciones notables: [Scheme48][] y [Racket][] soportan listas
de profundidad arbitraria, únicamente limitadas por la cantidad
de memoria disponible en el sistema.

[entrada en Stack Overflow]: http://stackoverflow.com/questions/62188/stack-overflow-code-golf
[stack overflow]: http://en.wikipedia.org/wiki/Stack_overflow

[Racket]: http://racket-lang.org
[Scheme48]: http://s48.org
[SISC]: http://sisc-scheme.org

