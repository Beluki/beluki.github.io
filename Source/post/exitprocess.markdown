title: ExitProcess
date: 2013-09-22
tags: ['programacion']

El caso es que estos últimos días he estado dándole vueltas a una idea
interesante para programar un juego, de modo que hoy he estado revisando
[4k][] y [4kGL][], pensando en la posibilidad de usar uno de ellos como
punto de partida.

[4k]: https://github.com/Beluki/4k
[4kGL]: https://github.com/Beluki/4kGL

La idea me gusta lo bastante para haber decidido implementarlo de forma
portable. Probablemente usaré C++ con [SDL][] o [Haxe][], así que el tema
de hacerlo sobre la API de Windows queda descartado.

[Haxe]: http://haxe.org
[SDL]: http://www.libsdl.org

Sin embargo, durante el repaso he visto una cosa que no recordaba
y que me ha llamado la atención. El código relevante es el siguiente:

    ::c++
    void
    Shutdown (UINT uExitCode) {
        ...

        // without WinMainCRTStartup() we must exit the process ourselves:
        ExitProcess(uExitCode);
    }


¿Uh? ¿Por qué se llama manualmente a [ExitProcess][]?

[ExitProcess]: http://msdn.microsoft.com/en-us/library/windows/desktop/ms682658%28v=vs.85%29.aspx

4k y 4kGL están pensados para ser usados sin librería de C. En un programa
normal que sí usa libc, el punto de entrada real no es "WinMain", sino
"WinMainCRTStartup", que se encarga de ejecutar primero WinMain
y luego ExitProcess.

La pregunta se reduce entonces a: ¿Por qué hace falta llamar a ExitProcess?
La respuesta está en la siguiente imagen, tomada de [Process Explorer][]:

[Process Explorer]: http://technet.microsoft.com/en-us/sysinternals/bb896653.aspx

<img src="{{ url_static('7.png') }}" alt="Process Explorer">

Un proceso de Windows no se termina hasta que [todos sus hilos terminan][].
4kGL en realidad no usa uno, sino dos hilos. El segundo procede del driver
de ATI y se lanza de manera automática al utilizar OpenGL.

[todos sus hilos terminan]: http://blogs.msdn.com/b/oldnewthing/archive/2010/08/27/10054832.aspx

