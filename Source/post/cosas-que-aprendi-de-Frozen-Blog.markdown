title: Cosas que aprendí de... Frozen-Blog
date: 2014-12-22
tags: ['postmortem', 'programacion']

Cuando empecé con esto de crear proyectos y compartirlos online, pensaba
que una vez tuviese suficientes, crearía un blog donde ir contando mis
progresos. Las cosas que salieron bien, las que no tan bien, decisiones
de diseño, pequeños detalles interesantes...

Una vez llegó el momento, no tenía muy claro qué hacer. Sabía que quería algo
simple, que no necesite una base de datos aparte (como [Wordpress][]) e
independiente de servicios online concretos (como [Blogger][]).

[Blogger]: https://www.blogger.com
[Wordpress]: https://wordpress.org

## Github y Jekyll

Afortunadamente, pronto encontré una alternativa. Entre las muchas cosas
que ofrece Github, está [Jekyll][]: un generador de blogs estáticos.
El resultado (el HTML que Jekyll genera) puede alojarse en el propio Github.

Como no es oro todo lo que reluce... tampoco tardé en encontrar problemas:

* Pésimo soporte para Windows. Ha habido ocasiones en que Jekyll ha estado
  meses sin funcionar por culpa de algún bug en las librerías que usa.
  A día de hoy Windows ni siquiera está soportado oficialmente.

* El sistema de plantillas: [Liquid][] es extremadamente limitado.
  Hasta Jekyll 2.2.0 era imposible ordenar las categorías alfabéticamente.

* En modo interactivo, Jekyll regenera la web entera con cada cambio.
  En cuanto tienes un número de posts considerable (digamos 200) esto
  es muy lento e ineficiente.

* Aunque Jekyll está escrito en Ruby, depende de Python y Pygments para tener
  sintaxis de colores decente. Con el tiempo, se añadió soporte para [Rouge][],
  que soporta menos lenguajes, pero no necesita Python.

Tras descartar Jekyll, me puse a buscar otros generadores parecidos. Recuerdo
que probé unos diez. Muchos tenían serias limitaciones o eran demasiado
complicados. Aunque había alguno prometedor (por ejemplo [Pelican][]) decidí
crear uno propio. Lo que no sabía es que terminaría escribiendo no uno, sino
cuatro generadores distintos, cada uno con sus ventajas y desventajas.

[Jekyll]: https://help.github.com/articles/using-jekyll-with-pages
[Liquid]: http://liquidmarkup.org
[Pelican]: https://github.com/getpelican/pelican
[Rouge]: https://github.com/jneen/rouge

## Frozen-Flask

Lo que me llevó a crear Frozen-Blog, fue un post de [Nicolas Perriault][] en el
que utiliza [Flask][] y [Frozen-Flask][] para generar un blog simple.
El código completo son unas 35 líneas de Python.

Pensé: ¿Y si generalizamos esta idea? ¿Y si le añado todo lo que se espera de
un generador estático? No tardé en añadir categorías (múltiples por post),
paginación, soporte para sintaxis de colores, configuración externa y todo lo
que fui necesitando.

Pronto me di cuenta de que [Flask-FlatPages][] era un tanto limitado.
En particular, está ligado a Flask. Es imposible crear múltiples instancias
en una sola aplicación. Por eso creé también [MetaFiles][], un fork de
Flask-FlatPages que es independiente de Flask, Jinja2 o cualquier librería
externa y que usé en Frozen-Blog tanto para posts como para páginas.

Una curiosidad sobre Flask es que sus archivos de configuración no son JSON
o INI, sino código Python. En Frozen-Blog aprovecho esto para poder hacer
cosas como la siguiente:

    ::python

    # cambia el formato de fecha/hora a castellano en todo el blog:
    import locale
    locale.setlocale(locale.LC_TIME, 'spanish')

[Flask]: http://flask.pocoo.org
[Flask-FlatPages]: https://github.com/SimonSapin/Flask-FlatPages
[Frozen-Flask]: https://github.com/SimonSapin/Frozen-Flask
[Nicolas Perriault]: https://nicolas.perriault.net/code/2012/dead-easy-yet-powerful-static-website-generator-with-flask/

## HTML y CSS

Con el generador listo, solo faltaba un tema simple y elegante para presentar
el blog. Creo que puedo decir sin miedo a equivocarme que esta fue la parte
más difícil de todo el proyecto.

HTML5 y CSS3 son relativamente simples, pero cada navegador tiene la manía
de interpretarlos de modo diferente. El tema que escribí para Frozen-Blog
funciona en cualquier navegador, hasta en IE6 o en navegadores de texto
para consola (como lynx). También permite hacer zoom correctamente sin
deformar la maquetación del contenido.

La lección más importante en este sentido fue que es imposible testear en
suficientes entornos sin máquinas virtuales o sin hacer uso de herramientas
online como [BrowserStack][] o los [validadores de HTML y CSS][] de la W3C.

[BrowserStack]: http://www.browserstack.com
[validadores de HTML y CSS]: http://validator.w3.org

## Conclusiones

Estoy satisfecho con el resultado y sobre todo con las librerías que he usado
para llegar a él. Flask y Jinja 2 son excelentes. Todo lo que hace
[Armin Ronacher][] suele serlo.

Por poner un "pero": este proyecto me hace darme cuenta de lo complicadas
que son la mayoría de páginas web o blogs que hay por ahí. La gente se
sorprende cuando abre un blog y tarda menos de 1 segundo en cargar. Yo opino
que esto debería ser la norma y no la excepción. ¿Cuánto de todo lo que ha
cargado es contenido? ¿Cuánto es realmente necesario?

Repositorios en Github: [Frozen-Blog][] - [MetaFiles][]

[Armin Ronacher]: http://lucumr.pocoo.org

[Frozen-Blog]: https://github.com/Beluki/Frozen-Blog
[MetaFiles]: https://github.com/Beluki/MetaFiles

