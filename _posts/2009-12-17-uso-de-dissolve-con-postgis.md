---
ID: 1110
post_title: 'Uso de &#8220;dissolve&#8221; con POSTGIS'
author: Miguel
post_date: 2009-12-17 11:36:51
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2009/12/17/uso-de-dissolve-con-postgis/
published: true
syntaxhighlighter_encoded:
  - "1"
---
Uno de los problemas que tuve al manejar los datos geométricos del INE fue que la capa de municipios constaba de geometría de tipo POLYGON. Lo que necesitaba conseguir era agrupar esa geometría simple y convertirla en MULTIPOLYGON. Eso me dio la idea para el siguiente consejo práctico.

Para ello una simple ojeada a la documentación de <a href="http://postgis.refractions.net/documentation/manual-1.4/">POSTGIS</a> y encontré la función que necesitaba, <strong><a href="http://postgis.refractions.net/documentation/manual-1.4/ST_Union.html">st_union</a></strong>. Esta función nos une la geometría en función de un atributo seleccionado.

Voy a mostrar un ejemplo sencillo, pero muy pedagógico.  En este ejemplo partimos de tres polígonos simples,  su geometría es de tipo POLYGON y quiero unirlos por un atributo común, en su caso el código.

<img class="alignnone size-full wp-image-1127" src="http://www.gisandchips.org/wp-content/polygon.PNG" alt="poligonos" width="417" height="313" />
<!--more-->
Vemos en la imagen anterior los 3 polígonos, dos de los cuales tienen el mismo atributo (campo código).

La SQL quedaría de la siguiente manera:

[sql]select st_union(geometria) from poligonos group by codigo;[/sql]

El problema viene con el resultado de esa consulta, y es la heterogeneidad del tipo de geometría:

<img class="alignnone size-full wp-image-1133" src="http://www.gisandchips.org/wp-content/stunion.PNG" alt="stunion" width="467" height="58" />

Para que eso no ocurra, añadimos la función<strong> <a href="http://postgis.refractions.net/documentation/manual-1.4/ST_Multi.html">st_multi</a></strong>, y conseguimos una homogeneidad del tipo de geometría, convirtiendo todas las geometrías en MULTIPOLYGON:

[sql]select st_multi(st_union(geometria)) from poligonos group by codigo;[/sql]

<img class="alignnone size-full wp-image-1135" src="http://www.gisandchips.org/wp-content/stmultiunion.PNG" alt="stmultiunion" width="478" height="55" />

En caso de que los polígonos sean adyacentes, pasaría a ser un único polígono, es decir,  elimina la frontera entre ambos.

<img class="alignnone size-full wp-image-1142" src="http://www.gisandchips.org/wp-content/polygon21.PNG" alt="polygon2" width="479" height="485" />

Ahora comprobamos que al usar <strong>st_union</strong>, simplemente nos ha creado un polígono a partir de un atributo común, manteniendo la geometría de tipo POLYGON y en este caso ya no hace falta usar st_multi.

<img class="alignnone size-full wp-image-1140" src="http://www.gisandchips.org/wp-content/stunion2.PNG" alt="stunion2" width="470" height="54" /><!--more-->