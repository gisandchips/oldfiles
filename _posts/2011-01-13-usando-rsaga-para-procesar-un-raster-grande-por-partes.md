---
ID: 1692
post_title: 'Usando RSAGA para procesar un raster &#8220;grande&#8221; por partes'
author: benizar
post_date: 2011-01-13 19:20:21
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2011/01/13/usando-rsaga-para-procesar-un-raster-grande-por-partes/
published: true
---
<span style="font-weight: normal;font-size: 13px">Hola a todos, hoy propongo una de las posibles soluciones a un problema que suele aparecer trabajando con GIS: ¿qué hacer cuando queremos procesar un raster “relativamente grande” y los GIS de escritorio más populares tienen problemas de memoria o no acaban el proceso?</span>

En algunas ocasiones la solución a estos problemas sería “trocear” el raster y procesarlo por partes. Esto lo podríamos hacer con varios software y en todos ellos sería interesante poder automatizar la tarea al máximo.

[caption id="attachment_1695" align="aligncenter" width="240" caption="1. Modelo digital raster"]<a href="http://www.gisandchips.org/wp-content/mdt1.jpg"><img class="size-medium wp-image-1695 " src="http://www.gisandchips.org/wp-content/mdt1-300x300.jpg" alt="" width="240" height="240" /></a>[/caption]

En este post propongo realizar una prueba con RSAGA, que es un modulo de R que permite acceder a las funciones disponibles en la consola de SAGA GIS. Para este post he trabajado en Windows, con R 2.10.1 y SAGA 2.0.4, aunque supongo que no habrá problemas con usar otras versiones más recientes. Además, deberéis instalar en R el paquete RSAGA.<!--more-->

[caption id="attachment_1696" align="aligncenter" width="238" caption="2. Shapefile de polígonos con que trocearemos el raster"]<a href="http://www.gisandchips.org/wp-content/municipios1.jpg"><img class="size-medium wp-image-1696 " src="http://www.gisandchips.org/wp-content/municipios1-298x300.jpg" alt="" width="238" height="240" /></a>[/caption]

Para esta práctica podéis <a title="descargar ficheros de prueba" href="http://dl.dropbox.com/u/17558342/rsaga_tests.rar" target="_blank">descargar dos ficheros</a> (Imagenes 1 y 2), uno raster y otro vectorial (polígonos). A efectos didácticos se utilizan ficheros pequeños, pero esto resultaría más útil en caso de necesitar fragmentar más el raster. Por ejemplo, en artículos anteriores he presentado código para trabajar imágenes a nivel de parcelas agrícolas. En aquellos artículos trabajábamos las imágenes de una en una y ya estaban recortadas, pero las parcelas podrían ser miles y no queremos hacer eso a mano.
<h3>Conociendo SAGA GIS</h3>
Lo principal, como en tantos otros casos, sería conocer como se trocea un fichero raster con SAGA GIS. Para ello abrimos la GUI de SAGA y vamos a la pestaña “modules” que tiene forma de TOC, generalmente se encuentra a la izquierda de la ventana principal. En esta pestaña podemos encontrar las distintas librerías que agrupan módulos que se relacionan por algún motivo. Explorar estas librerías y practicar con ellas es el mejor modo de estar seguro de lo que se quiere hacer.

[caption id="attachment_1697" align="aligncenter" width="138" caption="3. TOC de SAGA GIS donde podemos explorar las librerías y los módulos disponibles. "]<a href="http://www.gisandchips.org/wp-content/toc.jpg"><img class="size-medium wp-image-1697" src="http://www.gisandchips.org/wp-content/toc-172x300.jpg" alt="" width="138" height="240" /></a>[/caption]

Finalmente, he decidido que lo mejor es empezar por separar el shapefile original por polígonos (shapes-tools/separate shapes) y después realizar un recorte por cada uno de los nuevos shapefiles (shapes-grid/clip grid with polygon). Practicar con el interfaz gráfico es un bueno modo de estar seguros de los parámetros que nos pide cada módulo.

Antes de empezar a usar RSAGA he tenido que averiguar los nombres exactos de los módulos y los parámetros que aceptan. Esto lo he hecho con los métodos rsaga.get.modules() y rsaga.get.usage(). Aunque generalmente se puede averiguar también viendo los nombres en el SAGA GUI o mirando en la carpeta donde se contienen las *.dll.
<h3>Trabajando con RSAGA</h3>
[shell]
## Abrimos la consola de R y cargamos la librería
library(RSAGA)

## Seleccionamos el fichero raster que queremos trocear y una capa vectorial que queramos usar como límites. También especificamos el directorio donde van los outputs. Para trabajar con Windows recomiendo rutas sin espacios.
raster&lt;- file.choose()
poligonos&lt;- file.choose()
directorio &lt;-choose.dir()

## Consultamos hasta encontrar la herramienta que nos separa un shapefile en varios, obteniendo un shapefile por cada polígono, o lo que quisiéramos. Por ejemplo:
## rsaga.get.modules(&quot;shapes_grid&quot;)
## rsaga.get.usage(&quot;shapes_tools&quot;, 7)
## Se ejecuta el método con los parámetros necesarios.
rsaga.geoprocessor(lib=&quot;shapes_tools&quot;, module=7, param=list(SHAPES=poligonos, PATH=directorio, NAMING=0, FIELD=6))
[/shell]

Una vez ejecutado este proceso ya disponemos de un shapefile por cada polígono, esto significa que aunque el shapefile original tuviera miles de registros solamente cargaremos en memoria uno a la vez. Esto ya supone un ahorro importante de recursos.

[shell]
# Listamos los nuevos shapefiles
shapefiles &lt;- list.files(directorio, full.names=T, pattern=&quot;\\.shp&quot;)
## Por último utilizamos RSAGA para realizar tantos recortes del raster como polígonos habíamos extraído
for(i in 1:length(shapefiles)){
select &lt;- shapefiles[i]&lt;/code&gt;
rsaga.geoprocessor(lib=&quot;shapes_grid&quot;, module=7, param=list(OUTPUT=select, INPUT=raster, POLYGONS=select))
}#fin bucle
[/shell]
<h3>Conclusiones</h3>
Tras ver este ejemplo nos queda más claro el uso de RSAGA y se visualiza bien como se podrían automatizar gran número de tareas habituales en los GIS. Intentar realizar estas tareas herramientas básicas y generales podría ser algo trabajoso.

En nuestro fichero de prácticas había 18 polígonos por lo que nuestro resultado es una carpeta donde encontramos guardados 18 shapefiles y 18 ficheros raster. Los recortes quedan como se puede ver en la siguiente imagen:

[caption id="attachment_1702" align="aligncenter" width="300" caption="4. Uno de los 18 raster resultantes de los recortes y al fondo el shapefile original"]<a href="http://www.gisandchips.org/wp-content/recorte.jpg"><img class="size-medium wp-image-1702" src="http://www.gisandchips.org/wp-content/recorte-300x289.jpg" alt="" width="300" height="289" /></a>[/caption]

En caso de ser necesario, se podría realizar más tareas sobre todos estos ficheros, con este software o con otros programas. Incluso se plantea la posibilidad de usar multithreading en R para ejecutar tareas con RSAGA. Proximamente haremos alguna demostración de esto.

Antes de acabar, solo quiero recordar quando trabajamos con RSAGA en Windows es posible que nos afecten algunas particularidades sobre el modo en que se forman las rutas (espacios, barras de directorios…) y los mensajes de error que aparecen no son de gran ayuda.

——————————————————————

Si queréis contactar podéis enviarme un email (asunto: gisandchips):

Benito M. Zaragozí

benito.zaragozi@ua.es