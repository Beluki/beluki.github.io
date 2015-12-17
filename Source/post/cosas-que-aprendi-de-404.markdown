title: Cosas que aprendí de... 404
date: 2015-12-17
tags: ['postmortem', 'programacion', 'python']


[404][] es un pequeño programa escrito en Python que permite comprobar el estado
de una serie de enlaces en una página web.

Un ejemplo:


    ::bash
    $ 404.py http://beluki.github.io --threads 10 --internal follow --external check
    404: http://cdimage.debian.org/debian-cd/7.8.0/i386/iso-cd/
    Checked 181 total links in 10.4 seconds.
    55 internal, 126 external.
    0 network/parsing errors, 1 link errors.

Todo empezó probando un lenguaje de programación llamado [Nim][], en el que
implementé un script [parecido][] para testear la librería standard. Frustrado con
múltiples bugs, decidí plantarme y reescribirlo en Python en lugar de Nim y [404][]
fue el resultado.

[404]: https://github.com/Beluki/404
[Nim]: http://nim-lang.org
[parecido]: https://github.com/nim-lang/Nim/issues/2650

## Arquitectura

404 sigue la misma arquitectura exacta que [MultiHash][]. Usa el mismo código para crear
su threadpool y mantener los hilos. Ya [escribí un post][] sobre ello en este blog. Una
vez creado, el resto es cuestión de utilizar [BeautifulSoup][] para leer el HTML y
[Requests][] para ir acumulando el resto de los enlaces.

[escribí un post]: /post/respuesta-inmediata-con-multiples-hilos/
[MultiHash]: https://github.com/Beluki/MultiHash

[BeautifulSoup]: http://www.crummy.com/software/BeautifulSoup
[Requests]: http://docs.python-requests.org/en/latest

Lo cierto es que la mayoría del trabajo lo hacen las dos librerías. BeautifulSoup
ya se encarga de escoger el mejor parser de HTML existente en el sistema y es flexible
con respecto al código en si.

Por eficiencia, 404 solo tiene en cuenta enlaces <a href... e <img src...

    ::python
    link_strainer = SoupStrainer(lambda name, attrs: name == 'a' or name == 'img')

    soup = BeautifulSoup(response.content, 'html.parser', parse_only = link_strainer,
                            from_encoding = response.encoding)

    # a href...
    for tag in soup.find_all('a', href = True):
        ...

    # img src...
    for tag in soup.find_all('img', src = True):
        ...


Del mismo modo, Requests se encarga de todo lo necesario para establecer la
comunicación entre 404 y los servidores. Soporta [SSL][], [redirecciones][],
[Timeouts][] y un montón de cosas más que 404 no necesita. Sin estas dos librerías,
es probable que el código de 404 fuese tranquilamente diez o quince veces más largo y complejo.

[SSL]: http://docs.python-requests.org/en/latest/user/advanced/#ssl-cert-verification
[redirecciones]: http://docs.python-requests.org/en/latest/user/quickstart/#redirection-and-history
[Timeouts]: http://docs.python-requests.org/en/latest/user/quickstart/#timeouts

La única optimización que hace 404 es ignorar la parte [fragment][] de los enlaces
antes de pasar cada link al resto del programa, dado que solo tiene sentido para
el navegador.

[fragment]: https://en.wikipedia.org/wiki/Fragment_identifier

Una pregunta interesante es: ¿qué se considera un error al comprobar un enlace?
¿estado 404? ¿estado 301 (movido permanentemente)?. Para 404 la respuesta es:
hay un error cuando por problemas de conexión o de lectura (imposible leer el HTML)
no se puede obtener un código de estado válido en un tiempo aceptable.

