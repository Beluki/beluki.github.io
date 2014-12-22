title: Cosas que aprendí de... Frozen-Blog
date: 2014-12-22
tags: ['postmortem', 'programacion']

Cuando empecé con esto de crear proyectos y compartirlos online, pensaba
que una vez tuviese suficientes, crearía un blog donde ir contando mis
progresos. Las cosas que salieron bien, las que no tan bien, decisiones
de diseño, los pequeños detalles interesantes...

Una vez llegó el momento, no tenía muy claro qué hacer. Sabía que quería algo
simple, que no necesite una base de datos aparte (como [Wordpress][]) e
independiente de servicios online concretos (como [Blogger][]).

[Blogger]: https://www.blogger.com
[Wordpress]: https://wordpress.org

## Github y Jekyll

Afortunadamente, pronto encontré una alternativa. Entre las muchas cosas
que ofrece Github, está [Jekyll][]: un generador de blogs estáticos.
El resultado (el HTML que Jekyll genera) puede alojarse en el propio Github.

Como no es oro todo lo que reluce... tampoco tardé en encontrar mil problemas:

* Pésimo soporte para Windows. Ha habido ocasiones en que Jekyll ha estado
  meses sin funcionar por culpa de algún bug en las librerías que usa.
  A día de hoy Windows ni siquiera está soportado oficialmente.

* El sistema de plantillas: [Liquid][] es extremadamente limitado.
  Hasta Jekyll 2.2.0 era imposible ordenar las categorías alfabéticamente.

* En modo interactivo, Jekyll regenera la web entera con cada cambio.
  En cuanto tienes un número de posts considerable, digamos 200, esto
  es muy lento e ineficiente.

* Aunque Jekyll está escrito en Ruby, depende de Python y Pygments para tener
  sintaxis de colores decente. Con el tiempo, se añadió soporte para [Rouge][],
  que soporta menos lenguajes, pero no necesita Python.

Tras descartar Jekyll, me puse a buscar otros generadores parecidos. Recuerdo
que probé unos diez. Muchos tenían serias limitaciones o eran demasiado
complicados. Aunque había alguno prometedor (por ejemplo [Pelican][]) decidí
crear uno propio.

[Jekyll]: https://help.github.com/articles/using-jekyll-with-pages
[Liquid]: http://liquidmarkup.org
[Pelican]: https://github.com/getpelican/pelican
[Rouge]: https://github.com/jneen/rouge

## Conclusiones

Repositorios en Github: [Frozen-Blog][] - [MetaFiles][]

[Frozen-Blog]: https://github.com/Beluki/Frozen-Blog
[MetaFiles]: https://github.com/Beluki/MetaFiles

