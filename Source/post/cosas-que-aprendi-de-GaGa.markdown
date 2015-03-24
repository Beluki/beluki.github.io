title: Cosas que aprendí de... GaGa
date: 2015-02-01
tags: ['postmortem', 'programacion']

GaGa es un pequeño reproductor de radios online para Windows, escrito en C#.
Es parecido a [RadioTray][] para Linux, en el sentido en que se integra con
el [área de notificación][] del sistema.

Tiene esta pinta:

<img src="{{ url_static('11.png') }}" alt="GaGa">

[área de notificación]: http://en.wikipedia.org/wiki/Notification_area
[RadioTray]: http://radiotray.sourceforge.net

## Arquitectura

Con unas 3000 líneas de código en 18 archivos, GaGa es bastante más grande
que la mayoría de proyectos que tengo en Github (MultiHash por ejemplo
tiene unas 400 líneas). ¿Cómo organizarlo?.

Las pautas que seguí fueron las siguientes:

* Cada parte del programa ha de ser lo más independiente posible
  de las demás, de modo que se pueda sustituir si fuese necesario
  o razonar sobre ella por separado.

* Si algo es útil para otros programas, lo escribiré como una librería
  aparte de GaGa. Cuando esté terminado, el programa final será el hilo
  conductor que una todos los componentes entre si.

Y este fue el resultado:

<img src="{{ url_static('12.png') }}" alt="GaGa">

* [Controls][]: ToolStripAeroRenderer permite renderizar un menú contextual
  donde el item seleccionado tiene el mismo color que Aero en Windows,
  mientras que ToolStripLabeledTrackBar es un control que uso para los
  sliders de balance y volumen.

* [Libraries][]: Dos librerías, [mINI][], para leer archivos INI y [LowKey][]
  para poder tomar el control de teclas multimedia a bajo nivel.

* [NotifyIconPlayer][]: Implementa el reproductor, basado en el motor de
  Windows Media Player. Toma control del icono en el tray para mostrar
  su estado actual (por ejemplo, "buffering") o posibles errores.

* [StreamsFile][]: Lee archivos INI y añade sus stream a un menú contextual
  recargándolo automáticamente cuando sea necesario.

[Controls]: https://github.com/Beluki/GaGa/tree/master/Source/GaGa/Controls
[Libraries]: https://github.com/Beluki/GaGa/tree/master/Source/GaGa/Libraries
[NotifyIconPlayer]: https://github.com/Beluki/GaGa/tree/master/Source/GaGa/NotifyIconPlayer
[StreamsFile]: https://github.com/Beluki/GaGa/tree/master/Source/GaGa/StreamsFile

Finalmente, GaGa (GaGa.cs y GaGaSettings.cs) solo implementa el menú y la
lógica de los eventos que se derivan de él, como reproducir una radio al hacer
click en el stream correspondiente. Sirve para unir todo lo anterior.

Curiosidades del diseño:

* Todas las excepciones se controlan a nivel de GaGa.cs en lugar de en cada
  componente. Es el punto más cercano al usuario, donde es más fácil determinar
  qué hacer con cada error (normalmente, mostrar un mensaje).

* Los elementos gráficos son: el menú, icono, tooltip al dejar el ratón sobre
  el icono y los posibles mensajes sobre el icono. Que la clase Player.cs tome
  el control de todos menos el menú simplifica enormemente el código de GaGa.cs

* Los métodos que exponen todas las clases hacia GaGa.cs son los mínimos
  posibles. Por ejemplo, StreamsFile solo expone LoadTo() (cargar el INI en un
  menú) y MustReload() (determina si el INI ha cambiado y necesita recargarse).

Diría que la lección más importante en este punto es: organiza el programa de
modo que necesites tener en tu cabeza el mínimo contenido posible para poder
analizar su funcionamiento.

## mINI y LowKey

[mINI][] es una pequeña librería que sirve para leer archivos INI. Es curioso
que aunque el formato [INI][] sea predominante en Windows, .NET no tenga nada
en la librería standard para usarlo.

Lo más interesante de mINI es que no guarda dato alguno del archivo que está
leyendo. Es una clase abstracta con métodos como OnSection(String section)
que las subclases implementan para decidir qué hacer con los datos. Este
diseño es el que permite recargar el menú de GaGa con la mínima latencia
posible, dado que no hay estructuras de datos intermedias.

Escribí mINI en un fin de semana y la verdad es que estoy contento con el
resultado. Es simple, práctico, funciona.

[LowKey][] es mucho más complicado. Es una librería que responde a pulsaciones
de teclas desde cualquier aplicación utilizando [hooks][] a bajo nivel.
Aunque su código es relativamente corto, hay partes en las que es fácil
cometer un error.

Dos ejemplos:

* Para poder responder a cada pulsación sin bloquear el thread actual,
  uso [BeginInvoke][] en el [Dispatcher][] del thread que creó la instancia
  de LowKey. Esto permite reenviar la pulsación a otras aplicaciones
  inmediatamente, sin esperar a que el evento de LowKey se controle,
  pero también utilizar LowKey en GUIs (si el thread actual es el de la GUI).

* El callback que LowKey usa internamente hacia la API de Windows es estático.
  Esto evita que el [colector de basura lo colecte][].

[BeginInvoke]: https://msdn.microsoft.com/en-us/library/cc190824%28v=vs.110%29.aspx
[Dispatcher]: https://msdn.microsoft.com/en-us/library/System.Windows.Threading.Dispatcher%28v=vs.110%29.aspx
[colector de basura lo colecte]: http://stackoverflow.com/questions/9102814/how-to-hook-an-application

[INI]: http://en.wikipedia.org/wiki/INI_file
[hooks]: http://en.wikipedia.org/wiki/Hooking

## .NET Framework

C# es, en general, un lenguaje excelente. Un buen ejemplo es el modo en el
que GaGa hace la animación en su icono de "buffering", que tiene esta
pinta:

<img src="{{ url_static('13.gif') }}" alt="Buffering">

Implementado así (fragmento):

    ::csharp
    // iconos que vamos a usar:
    private readonly Icon[] bufferingIcons;

    // temporizador para cambiar de icono:
    private readonly DispatcherTimer bufferingIconTimer;

    // índice del icono actual en el array bufferingIcons:
    private Int32 currentBufferingIcon;

    public Player(NotifyIcon icon)
    {
        ...

        // 300 milisegundos entre icono e icono:
        bufferingIconTimer = new DispatcherTimer(DispatcherPriority.Background);
        bufferingIconTimer.Interval = TimeSpan.FromMilliseconds(300);
        bufferingIconTimer.Tick += OnBufferingIconTimerTick;
        currentBufferingIcon = 0;
    }

    private void OnBufferingIconTimerTick(Object sender, EventArgs e)
    {
        // cambiar icono:
        notifyIcon.Icon = bufferingIcons[currentBufferingIcon];

        // ir al siguiente icono o volver el primero si es necesario:
        currentBufferingIcon++;
        if (currentBufferingIcon == bufferingIcons.Length)
        {
            currentBufferingIcon = 0;
        }
    }

Un simple [DispatcherTimer][] que se ejecuta en el thread actual con prioridad
lo más baja posible (dado que no es algo que requiera precisión). Ojalá todos
los lenguajes tuviesen algo similar.

Por desgracia, no puedo decir lo mismo de [Windows Forms][] o de otras partes
de la librería standard. Aunque sobre el papel es genial, en la práctica hay
comportamientos extraños y bugs que hacen su uso incómodo y que llevan
fácilmente a errores por parte del programador.

[DispatcherTimer]: https://msdn.microsoft.com/en-us/library/system.windows.threading.dispatchertimer%28v=vs.110%29.aspx
[Windows Forms]: http://en.wikipedia.org/wiki/Windows_Forms

Algunos ejemplos (no exhaustivo):

* Cuando en un menú (de ContextMenuStrip) hay un item antes de un submenú,
  a veces el submenú no se abre automáticamente al pasar el ratón por encima.
  Esto no es un problema en casi ninguna aplicación porque normalmente los
  submenús están ordenados primero. GaGa respeta el orden que el usuario
  establece en el archivo INI.

* El reproductor (MediaPlayer, de System.Windows.Media) continúa bajando
  datos de streams online tras llamar al método Stop(), es necesario ejecutar
  Close() también. Ejecutar Close() resetea el volumen y balance a su valor
  por defecto.

* Pulsar la tecla alt cierra automáticamente un menú contextual.

* ContextMenu (no ContextMenuStrip) guarda su anchura en algún tipo de caché.
  Es necesario borrar y recrear el menú entero si se desea que su contenido
  sea dinámico o la anchura del menú es incorrecta.

* Ejecutar Clear() para [limpiar un ContextMenuStrip][] es insuficiente.
  Hace falta llamar a Dispose() para borrar todos los ToolStripMenuItem
  o controles que este contenga.

* [File.GetLastWriteTimeUtc][] devuelve 12:00, 1 de Enero de 1601 cuando un
  archivo no existe. A pesar de ello, puede lanzar 5 tipos de excepciones
  distintas cuando el archivo existe pero no hay permisos para acceder
  a él o casos similares.

* No hay modo standard de abrir un ContextMenuStrip en el punto exacto donde
  está el ratón al hacer click, tal como se muestra automáticamente al hacer
  click derecho en el icono. Es necesario acceder a un método privado:
  ShowInTaskBar (usando Reflection) para poder hacerlo.

[File.GetLastWriteTimeUtc]: https://msdn.microsoft.com/en-us/library/system.io.file.getlastwritetimeutc%28v=vs.110%29.aspx
[limpiar un ContextMenuStrip]: http://stackoverflow.com/questions/1969024/does-calling-clear-disposes-the-items-also

## Conclusiones

Estoy muy satisfecho con GaGa.

Casi todos mis programas son utilidades de consola diseñadas para usuarios
avanzados (o para programadores), pero GaGa es útil para cualquier persona con
un ordenador. Ha sido un reto interesante.

Además, una vez conseguí escuchar Radio GaGa de Queen en el propio GaGa
(en la radio KissFM), lo que no tiene precio.

Repositorios en Github: [GaGa][] - [LowKey][] - [mINI][]

[GaGa]: https://github.com/Beluki/GaGa
[LowKey]: https://github.com/Beluki/LowKey
[mINI]: https://github.com/Beluki/mINI

