title: Cosas que aprendí de... MultiHash
date: 2015-01-21
tags: ['postmortem', 'programacion']

MultiHash es un pequeño programa escrito en Python que calcula algoritmos
como md5, sha1, sha256 etc... al estilo de [md5sum][] o [sha1sum][]
de [coreutils][].

Su principal baza es que puede calcular múltiples algoritmos (de ahí el nombre)
leyendo cada archivo una sola vez. Además, puede dividir el trabajo en hilos,
donde cada hilo lee y calcula los algoritmos para un archivo. Es posible
ejecutar tantos hilos como se desee de manera concurrente, aunque lo usual
es utilizar el mismo número que procesadores hay en el sistema.

Este post será corto porque la parte más interesante, de la que más aprendí,
es sin duda el threadpool y ya hice en su día [un post][] sobre ello,
detallando todo el proceso. Aún así, alguna que otra cosa sí puedo contar...

[md5sum]: http://www.gnu.org/software/coreutils/manual/html_node/md5sum-invocation.html#md5sum-invocation
[sha1sum]: http://www.gnu.org/software/coreutils/manual/html_node/sha1sum-invocation.html#sha1sum-invocation
[coreutils]: http://www.gnu.org/software/coreutils

[un post]: http://beluki.github.io/post/respuesta-inmediata-con-multiples-hilos/

## Python y múltiples hilos

A cualquiera que conozca Python un poco, le extrañará que lo haya escogido para
crear MultiHash. Un estigma comunmente asociado a Python es que su soporte para
threads es horrible debido al [GIL][].

Yo no estoy de acuerdo.

El GIL o "Global Interpreter Lock" es un mecanismo que solo permite ejecutar
código Python (bytecode) en un thread a la vez. Es imposible aprovechar
múltiples procesadores con código Python puro.

La palabra importante es "puro". Las extensiones escritas en C (o en [Cython][])
pueden saltarse esta limitación, permitiendo usar tantos threads como sea
necesario. Si uno mira el código de [hashlib.h][], la cabecera principal de
la librería que MultiHash usa para calcular los algoritmos, encuentra:

    ::c
    /*
     * Helper code to synchronize access to the hash object when the GIL is
     * released around a CPU consuming hashlib operation. All code paths that
     * access a mutable part of obj must be enclosed in a ENTER_HASHLIB /
     * LEAVE_HASHLIB block or explicitly acquire and release the lock inside
     * a PY_BEGIN / END_ALLOW_THREADS block if they wish to release the GIL for
     * an operation.
     */

    #ifdef WITH_THREAD
    #include "pythread.h"
        #define ENTER_HASHLIB(obj) \
            if ((obj)->lock) { \
                if (!PyThread_acquire_lock((obj)->lock, 0)) { \
                    Py_BEGIN_ALLOW_THREADS \
                    PyThread_acquire_lock((obj)->lock, 1); \
                    Py_END_ALLOW_THREADS \
                } \
            }
        #define LEAVE_HASHLIB(obj) \
            if ((obj)->lock) { \
                PyThread_release_lock((obj)->lock); \
            }
    #else
        #define ENTER_HASHLIB(obj)
        #define LEAVE_HASHLIB(obj)
    #endif

[Cython]: http://cython.org
[GIL]: https://en.wikipedia.org/wiki/Global_Interpreter_Lock
[hashlib.h]: https://hg.python.org/cpython/file/tip/Modules/hashlib.h

Todos los algoritmos usan ENTER_HASHLIB() para saltarse el GIL antes de
empezar sus cálculos, así que es perfectamente viable usar threads. El
rendimiento de MultiHash escala perfectamente a tantos procesadores como
haya en el sistema.

Además, según mi experiencia, hay dos tipos de programas:

* Aquellos en los que el rendimiento "no importa", porque están limitados
  por otros factores, como la velocidad de transmisión de datos en un socket.
  Un ejemplo de esto es un cliente de IRC o el comando "ping". En estos casos
  el rendimiento de Python es más que suficiente.

* Aquellos en los que hasta el último segundo importa, como un raytracer o
  procesamiento de imágenes. En estos casos, con o sin GIL, Python nunca te
  daría una velocidad aceptable, así que no queda otra que escribir la parte
  importante en un lenguaje como C.

Aún así, sí existen modos de saltarse el GIL con código Python puro, como
usar [multiprocessing][], que usa subprocesos en lugar de threads.

[multiprocessing]: https://docs.python.org/3/library/multiprocessing.html

## Texto o binario

Cuando se abre un archivo, se puede abrir en modo texto o en modo binario.
Increíblemente, md5sum, sha1sum, etc... abren sus archivos en modo binario
por defecto.

Linux (y en general Unix), no distingue entre archivos de texto y binario,
pero Windows sí y el resultado de un checksum es diferente. Esto ha llevado
a [problemas de compatibilidad][] en sistemas como [Cygwin][] (o en ports
de md5sum a Windows).

MultiHash siempre lee los archivos en modo binario, que es lo único que tiene
sentido para calcular hashes. Por compatibilidad con las herramientas de
coreutils, su salida añade un asterisco '*' antes del nombre de cada fichero.

[Cygwin]: http://cygwin.com
[problemas de compatibilidad]: http://lists.gnu.org/archive/html/bug-coreutils/2009-03/msg00376.html

## Conclusiones

En el último postmortem escribía que HexPaste fue creado por necesidad, porque
no había una herramienta que me permitiese pegar poemas poco a poco en IRC.

MultiHash sin embargo, nació de una idea muy simple: "¿Y si calculamos todos los
algoritmos a la vez?". La intención es aplicarlo a cosas como las [ISOs de Debian][]
u otras distribuciones, que siempre incluyen todos los checksum.

Ha sido también mi primera experiencia creando un programa multihilo y la verdad
es que no me puedo quejar, ha sido divertido. Es un poco trampa, porque dado que
todos los hilos son completamente independientes entre sí, no me he tenido que
pegar con mutex y las cosas realmente complicadas.

Repositorios en Github: [MultiHash][]

[ISOs de Debian]: http://cdimage.debian.org/debian-cd/7.8.0/i386/iso-cd/
[MultiHash]: https://github.com/Beluki/MultiHash

