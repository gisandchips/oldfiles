---
ID: 387
post_title: Renderizado con Mapnik
author: jose
post_date: 2009-10-22 10:43:27
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2009/10/22/renderizado-de-osmcon-mapnik-para-usar-en-openlayers/
published: true
syntaxhighlighter_encoded:
  - "1"
---
En el planteamiento que presenté en el primer artículo sobre OSM y PgRouting no estaba incluido el tema del renderizado, pero tras indagar el tema de <a href="http://mapnik.org/">Mapnik</a> me he dado cuenta de las magníficas posibilidades que ofrece este renderizador, de ahí que haga un inciso para introducir esta cuestión.

Mapnik es un toolkit en C++ para renderizar varios orígenes de datos, que tiene bindings para Python. La calidad de este paquete, a mi juicio, está por encima de otros renderizadores como Osmarender o Kosmos. Profundizar en él me ha permitido aprender algo de Python, que nunca viene mal y sumergirme en las entrañas de la edición cartográfica. En cualquiera de los casos, los resultados estarán limitados a nuestra destreza para la composición de mapas, pero nunca a las limitaciones del programa.

<!--more-->

Mapnik entiende los siguientes orígenes de datos:
<ul>
	<li>Shapefiles</li>
	<li>PostGIS</li>
	<li>Raster (generalmente TIFF, aunque si se compila Mapnik con GDAL podemos ampliar el espectro de formatos)</li>
</ul>
<span style="font-weight: bold">OBJETIVO</span>:

En muchas de nuestras aplicaciones web incluimos una layer del proveedor OSM, utilizando OpenLayers, Mapfish u otras. El resultado es fácil de implementar, pero tiene el problema de depender del estado de salud del servidor oficial de www.openstreetmap.org. Además, puede que en un momento dado nos planteemos utilizar nuestra propia simbolización, cambiando los colores de las vias, o añadiendo unos iconos diferentes a los oficiales Aquí nos planteamos crear un TMS (Tile Map Service), o lo que es lo mismo, un conjunto de imágenes de mapa (tiles) de reducido tamaño (256 * 256) , personalizadas con nuestros criterios de simbolización gráfica, que están ordenadas en carpetas, cada una de las cuales recibe el nombre del nivel de zoom utilizado, que oscila entre 1 y 18, siguiendo la estructura del "<a href="http://wiki.openstreetmap.org/wiki/Slippy_map_tilenames">mapnik slippy map</a>", que es una forma de ordenar las tiles en función de la proyección utilizada (Mercator).

REFERENCIAS WEB:
<ul>
	<li><a href="http://weait.com/content/build-your-own-openstreetmap-server">http://weait.com/content/build-your-own-openstreetmap-server</a> Interesante portal con artículos sobre OSM y Mapnik. Os lo recomiendo para instalar Mapnik y realizar pruebas</li>
</ul>
<span style="font-weight: bold">DESARROLLO</span>:
<ol>
	<li><span style="background-color: #ffffff"> Compilación de librerías necesarias (Mapnik y osm2pgsql)</span></li>
	<li><span style="background-color: #ffffff">Creación de una geodatabase con datos OSM utilizando Osm2Pgsql</span></li>
	<li><span style="background-color: #ffffff">Configuración de la plantilla de mapa (XML mapfile)</span></li>
	<li><span style="background-color: #ffffff">Pruebas de renderizado</span></li>
	<li><span style="background-color: #ffffff">Creación del TMS</span></li>
	<li><span style="background-color: #ffffff">Implementación del TMS en OpenLayers</span></li>
</ol>
<span style="font-weight: bold">COMPILACIÓN DE LIBRERÍAS</span>:

Entrando en faena vamos a compilar el paquete Mapnik, que como no podía ser de otra manera tiene una dependencia insalvable, BOOST, que ya vimos en el anterior artículo. No obstante indico rápidamente su compilación:

<span style="font-weight: bold">COMPILACIÓN DE BOOST</span>

1. Añadir dependencias. Según la distro necesitaremos una u otra (python-devel, freetype-devel, etc). En este paso vamos a incluir una librería que me ha dado varios quebraderos de cabeza: EXPAT. No sólo es necesaria instalarla, sino que además debemos de indicar algunas variables de configuración. Empezamos por su instalación desde la paquetería

Ubuntu: $  sudo apt-get install libexpat1-dev

$ sudo apt-get install libltdl7-dev libpng12-dev libtiff4-dev libicu-dev libboost-regex-dev libboost-iostreams-dev libboost-filesystem-dev libboost-thread-dev libboost-python1.34.1 libboost-python-dev libfreetype6-dev libcairo2-dev libcairomm-1.0-dev libboost-program-options-dev python-cairo-dev libboost-serialization-dev imagemagick

Centos: $ yum install expat expat-devel.x86_64

NOTA: Fíjate que para expat le indico específicamente la plataformas de 64 bits. Es importante indicarla. Si ponemos "yum install expat-devel" instalará las dos, y nos proporcionará problemas en el futuro (por ejemplo compilando nuestra querida GDAL). Indícale cual es tu plataforma (x86_64 / i386)

2. Define variables para expat:

[shell]&lt;br&gt;export EXPAT_INCLUDE=/usr/include&lt;br&gt;export EXPAT_LIBPATH=/usr/lib&lt;br&gt;[/shell]

3. Descargar la version 1.39 y descomprimir en /usr/local/

[shell]
mkdir /usr/local/include/boost
wget http://downloads.sourceforge.net/project/boost/boost/1.39.0/boost_1_39_0.tar.bz2
tar -xjvf boost_1_39_0.tar.bz2&lt;br&gt;cd boost_1_39_0
[/shell]

4. Compilar:

[shell]./bootstrap.sh --prefix=/usr/local/
./bjam install --includedir=/usr/local/include/boost/
[/shell]

Como root:

[shell]/sbin/ldconfig[/shell]

Este comando es mano de santo para que los compiladores encuentren las librerías.
<span style="font-weight: bold">COMPILAR MAPNIK</span>
Para compilar en Ubuntu os recomiendo seguir las indicaciones de este <a href="http://weait.com/content/build-your-own-openstreetmap-server">enlace</a>. Para Centos estos son los pasos a seguir:

[shell]cd /home/&lt;usuario&gt;/compilar[/shell]

1. Descargar y descomprimir la fuente de Mapnik  (versión 0.6.1)

[shell]wget http://download.berlios.de/mapnik/mapnik-0.6.1.tar.bz2
tar -xjvf mapnik-0.6.1.tar.bz2
cd mapnik-0.6.1&lt;br&gt;
[/shell]

2. Compilar:

[shell]python scons/scons.py configure
python scons/scons.py BOOST_INCLUDES=/usr/local/include/boost/boost-1_39/ BOOST_LIBS=/usr/local/lib BOOST_TOOLKIT=gcc41
[/shell]

Como root:

[shell]
python scons/scons.py BOOST_INCLUDES=/usr/local/include/boost/boost-1_39/ BOOST_LIBS=/usr/local/lib BOOST_TOOLKIT=gcc41 install
[/shell]

3. Actualizar rutas de librerías (como root):

[shell]/sbin/ldconfig[/shell]

4. Comprobación de la instalación:

Las fuentes contienen una demo para comprobar la instalación. Se trata de un script en Python que tras su ejecución llama a algunos shapefiles y genera unas imágenes en PNG. También lo hará en otros formatos (PDF, PS, etc.)  sí has compilado Mapnik con Cairo y PyCairo. En Ubuntu es muy fácil ya que estas librerías se encuentran en el repositorio, pero en Centos he sido incapaz de utilizarlas (si alguno lo consigue su comentario será de agradecer).

Mapnik utiliza la librería <a href="http://www.antigrain.com/">AGG</a> para crear las imágenes, siendo ésta la librería de uso por defecto, aunque también puede utilizar Cairo.
1. Cambia al directorio de pruebas

[shell]
cd &lt;fuente&gt;/demo/python
python rundemo.py
[/shell]

Obtendrás este mensaje:
<pre>Skipping cairo examples as Pycairo not available
3 maps have been rendered in the current directory:
- demo.png
- demo256.png
- demo.jpg
Have a look!</pre>
Bueno, pues ya está. Si las imágenes se han generado Mapnik funciona. Sí te da algun error es que te falta alguna librería. Otra manera de ver que errores hay es utilizando la propia consola de python. Escribe lo que está en negrita:

jose@jose:~/compilar$ <span style="font-weight: bold">python</span>

Python 2.6.2 (release26-maint, Apr 19 2009, 01:56:41)

<span style="background-color: #ffffff">[GCC 4.3.3] on linux2</span><span style="background-color: #ffffff">Type "help", "copyright", "credits" or "license" for more information.</span>
<span style="background-color: #ffffff">&gt;&gt;&gt;<span style="font-weight: bold"> import mapnik</span></span>
&gt;&gt;&gt; <span style="font-weight: bold">exit()</span>

<span style="background-color: #ffffff">Sí tras escribir "import mapnik" obtienes algunas líneas de error es que algo te falta.  De lo contrario el intérprete de Python entiende que la llamada a la librería funciona y no devuelve nada.</span>
<span style="font-weight: bold">CREACIÓN DE UNA GEODATABASE CON EL PLANET DE OSM</span>

Nuestro objetivo es ahora crear una geodatabase de PostGIS con datos procedente de un planet.osm. Con ello,  conseguiremos tener un repositorio de datos cuyo único fin es ser utilizado por Mapnik para renderizar tiles, y no para otras funciones como la de routing.  Por esta razón utilizamos "osm2pgsql". Esta herramienta suele formar parte de los repositorios de las distros, por lo que sólo necesitas:
<pre>Ubuntu: sudo apt-get install osm2pgsql
Centos: yum install osm2pgsql</pre>
<span style="font-weight: bold">Compilación de osm2pgsql</span>:

[bash]&lt;br&gt;svn co http://svn.openstreetmap.org/applications/utils/export/osm2pgsql/&lt;br&gt;cd osm2pgsql&lt;br&gt;make&lt;br&gt;[/bash]

<span style="background-color: #ffffff">Crear una geodatabase con datos de OSM es una labor sencilla, pero ten en cuenta que OSM cambia constantemente, por lo que la validez de tus datos tienen una caducidad. Por esta razón es importante que articulemos algún sistema para que cada cierto tiempo la base de datos de OSM se actualice sólo. Por ello te recomiendo que personalices el siguiente script en bash que te facilitará la labor:</span><span style="background-color: #ffffff"> </span>

[bash]
#!/bin/sh
echo &quot;Script en Bash para actualizar un planet.osm en una base de datos OSM con Postgis v. 1.4.&quot;
echo &quot;Borrando spain.osm.bz2&quot;
rm spain.osm.bz2
echo &quot;Descargando fichero osm de Spain desde Cloudmade.com&quot;
wget http://downloads.cloudmade.com/europe/spain/spain.osm.bz2
echo &quot;Descomprimiendo&quot;
bzip2 -d spain.osm.bz2
echo &quot;Borrando bbdd osm&quot;
dropdb -h localhost -U postgres osm
echo &quot;Creando bbdd osm&quot;
createdb -U postgres -E UTF8 osm
echo &quot;Instalar lenguaje pl/pgsql y Postgis en bbdd osm&quot;
createlang -U postgres plpgsql osm
psql -U postgres -f /usr/share/pgsql/contrib/postgis.sql osm
psql -U postgres -f /usr/share/pgsql/contrib/spatial_ref_sys.sql osm
echo &quot;Añadir datos de spain.osm a la bbdd osm&quot;
echo &quot;Tiempo aproximado: 11 minutos&quot;
osm2pgsql --slim -H localhost -U postgres -d osm ./spain.osm
echo &quot;fin&quot;
[/bash]

NOTA: Es muy importante utilizar el parámetro "slim" en osm2pgsql, puesto que permite introducir datos directamente a la base de datos, lo que resulta muy útil con planet.osm de mucho peso. Si no se usa, cargará en memoria todo el osm hasta que ocupe toda la memoria de tu ordenador.  Contenido de la geodatabase
<pre>psql -h localhost -U postgres osm
Welcome to psql 8.3.8 (server 8.3.7), the PostgreSQL interactive terminal.

Type:  \copyright for distribution terms
       \h for help with SQL commands
       \? for help with psql commands
       \g or terminate with semicolon to execute query
       \q to quit

osm=# \d
                     List of relations
 Schema |            Name            |   Type   |  Owner
--------+----------------------------+----------+----------
 public | geometry_columns           | table    | postgres
 public | planet_osm_line            | table    | postgres
 public | planet_osm_line_pid_seq    | sequence | postgres
 public | planet_osm_nodes           | table    | postgres
 public | planet_osm_point           | table    | postgres
 public | planet_osm_point_pid_seq   | sequence | postgres
 public | planet_osm_polygon         | table    | postgres
 public | planet_osm_polygon_pid_seq | sequence | postgres
 public | planet_osm_rels            | table    | postgres
 public | planet_osm_roads           | table    | postgres
 public | planet_osm_roads_pid_seq   | sequence | postgres
 public | planet_osm_ways            | table    | postgres
 public | spatial_ref_sys            | table    | postgres
(13 rows)</pre>
<span style="font-weight: bold">INTRODUCCIÓN A MAPNIK CON PYTHON</span>
Como ya he dicho, no soy un especialista en este lenguaje, aunque su sintaxis me parece muy clara para entenderla sin problemas.
Lo primero que debemos de tener en cuenta es que el diagrama de flujo de Mapnik exige para su uso de un script en Python (o desde la misma consola de Python) donde se aprecien las siguientes fases:

1. Cargar los paquetes (bindings) de Mapnik para Python
2. Indicar la fuente de datos (datasource) de cada layer a representar
3. Definir la simbología de cada layer
4. Indicar los parámetros de la imagen a crear: tamaño, nombre del fichero

Veamos el ejemplo más sencillo: Crear una imagen a partir de un shapefile

[python]
#!/usr/bin/env python
# Fichero: hola_mundo.py
# Cargamos los bindings de mapnik
import mapnik
# Definimos un objeto mapa, con un tamaño (en píxeles) y una proyección según PROJ4
m = mapnik.Map(600,300,'+proj=latlong +datum=WGS84')
# Indicamos el color de fondo
m.background = mapnik.Color('steelblue')
# Creamos un estilo. Un estilo es un tipo de objeto que puede almacenar una o varias reglas (rules)
# para la representación cartográfica
s = mapnik.Style()
# Creamos un objeto rule
r=mapnik.Rule()
# Aplicamos parámetros a esa regla. En este caso:
# 1. relleno del polígono
# 2. simbología de la línea (outline)
r.symbols.append(mapnik.PolygonSymbolizer(mapnik.Color('#f2eff9')))
r.symbols.append(mapnik.LineSymbolizer(mapnik.Color('rgb(50%,50%,50%)'),0.1))
# asociamos la &quot;rule&quot; (r) al estilo (s)
s.rules.append(r)
# Asignamos un nombre al estilo
m.append_style('My Style',s)
# Creamos un objeto layer de mapnik: un nombre y una projección. En este caso la geodésica mundial EPSG:4326
lyr = mapnik.Layer('world','+proj=latlong +datum=WGS84')
# Definimos el origen de datos
lyr.datasource = mapnik.Shapefile(file='/Users/path/to/your/data/world_borders')
# Asociamos a la layer el estilo que hemos creado
lyr.styles.append('My Style')
# Asociamos la layer al objeto mapa (m)
m.layers.append(lyr)
# Indicamos que se renderizará el total de la extensión del shapefile
m.zoom_to_box(lyr.envelope())
# Renderizamos la imagen: indicando el objeto mapa (m) y el nombre del fichero de imagen a crear
mapnik.render_to_file(m,'world.png', 'png')
# Si queremos que los parámetros indicados en el objeto mapa sean serializados a un fichero
# mapfile (XML) sólo tenemos que añadir esta linea
mapnik.save_map(m,'mi_mapfile.xml')
[/python]

Fíjate en la línea que hace referencia a la proyección a utilizar. ¿De donde saco estos parámetros? En esta página puedes encontrar los parámetros necesarios: <a href="http://spatialreference.org/ref/epsg/">http://spatialreference.org/ref/epsg/</a>. Veamos algunos ejemplos:
<ul>
	<li><span style="background-color: #ffffff">EPSG 4326: +proj=latlong +datum=WGS84 (<a href="http://spatialreference.org/ref/epsg/4326/proj4/">http://spatialreference.org/ref/epsg/4326/proj4/</a>)</span></li>
	<li><span style="background-color: #ffffff">EPSG 900913: <span>+proj=merc +lon_0=0 +k=1 +x_0=0 +y_0=0 +ellps=WGS84 +datum=WGS84 +units=m +no_defs (<a href="http://spatialreference.org/ref/sr-org/6627/proj4/">http://spatialreference.org/ref/sr-org/6627/proj4/</a>)</span></span></li>
	<li><span style="background-color: #ffffff">EPSG 23030: <span>+proj=utm +zone=30 +ellps=intl +units=m +no_defs (<a href="http://spatialreference.org/ref/epsg/23030/proj4/">http://spatialreference.org/ref/epsg/23030/proj4/</a>)</span></span></li>
</ul>
Para ejecutar este script sólo tienes que indicarle que es un ejecutable:

[shell]&lt;br&gt;chmod +x hola_mundo.py&lt;br&gt;[/shell]

Y ahora lo ejecutamos:

[shell]&lt;br&gt;python hola_mundo.py&lt;br&gt;[/shell]

Resultado:
<p style="text-align: center"></p>

<div class="mceTemp mceIEcenter"><dl> <dt><img class="size-medium wp-image-704" src="http://www.gisandchips.org/wp-content/hola_mundo-300x150.png" alt="hola munco con Mapnik" width="300" height="150" /></dt> <dd>hola mundo con Mapnik</dd> </dl></div>
<span style="font-weight: bold">Mapfile de Mapnik</span>
Como ves, con muy pocas líneas tenemos una imagen creada con Mapnik, pero, ¿porque no separamos la lógica de representación y los orígenes de datos en un fichero de mapa (mapfile)? Posiblemente este nos recuerda a más de uno los famosos "mapfile" de MapServer, y realmente bastante se parecen a ellos, pero esta vez la lógica está en un fichero XML.  Manos a la obra. Primero generamos el script de Python sin referencia alguna a la simbología y los datasources:

[python]
#!/usr/bin/env python
import mapnik
mapfile = 'hola_mundo.xml'
map_output = 'hola_mundo.png'
m = mapnik.Map(600, 300)
mapnik.load_map(m, mapfile)
bbox = mapnik.Envelope(mapnik.Coord(-180.0, -90.0), mapnik.Coord(180.0, 90.0))
m.zoom_to_box(bbox)
mapnik.render_to_file(m, map_output)
[/python]

Fíjate que en la tercera línea llamamos a un fichero de mapa. Se trata de un fichero XML con una estructura definida, donde se indican los datasources (layers) y su representación (styles y rules dentro de ella)

[xml]
&lt;?xml version=&quot;1.0&quot; encoding=&quot;utf-8&quot;?&gt;
&lt;!DOCTYPE Map&gt;
&lt;Map bgcolor=&quot;steelblue&quot; srs=&quot;+proj=latlong +datum=WGS84&quot;&gt;
  &lt;Style name=&quot;My Style&quot;&gt;
    &lt;Rule&gt;
      &lt;PolygonSymbolizer&gt;
        &lt;CssParameter name=&quot;fill&quot;&gt;#f2eff9&lt;/CssParameter&gt;
      &lt;/PolygonSymbolizer&gt;
      &lt;LineSymbolizer&gt;
        &lt;CssParameter name=&quot;stroke&quot;&gt;rgb(50%,50%,50%)&lt;/CssParameter&gt;
        &lt;CssParameter name=&quot;stroke-width&quot;&gt;0.1&lt;/CssParameter&gt;
      &lt;/LineSymbolizer&gt;
    &lt;/Rule&gt;
  &lt;/Style&gt;&lt;
  &lt;Layer name=&quot;world&quot; srs=&quot;+proj=latlong +datum=WGS84&quot;&gt;
    &lt;StyleName&gt;My Style&lt;/StyleName&gt;
    &lt;Datasource&gt;
      &lt;Parameter name=&quot;type&quot;&gt;shape&lt;/Parameter&gt;
      &lt;Parameter name=&quot;file&quot;&gt;&lt;ruta al shapefile, pero sin extensión .shp&gt;&lt;/Parameter&gt;
    &lt;/Datasource&gt;
  &lt;/Layer&gt;
&lt;/Map&gt;
[/xml]

En resumen tenemos:
- Fondo de mapa
- Style --&gt; Rule --&gt; Simbología
- Layer --&gt; Proyección --&gt; DataSource --&gt; Style

Estos dos ejemplos utilizan shapefile. Para indicar un datasource PostGIS hay que especificar estas líneas

a) En el caso del fichero Python sin XML

[python]
# proyección UTM ED-50 (España) --&gt; EPSG 23030
lyr = mapnik.Layer('spain','+proj=utm +zone=30 +ellps=intl +units=m +no_defs')
lyr = mapnik.Layer('Layer de PostGIS')
lyr.datasource = mapnik.PostGIS(host='localhost',user='postgres',password='',dbname='hispania',table='provincias')
lyr.styles.append('Mi estilo')
[/python]

b) En el caso de un XML este sería el contenido de la parte referente a la layer

[xml]
&lt;Layer name=&quot;provincias&quot; status=&quot;on&quot; srs=&quot;+proj=utm +zone=30 +ellps=intl +units=m +no_defs&quot;&gt;
    &lt;StyleName&gt;mi estilo&lt;/StyleName&gt;
    &lt;Datasource&gt;
      &lt;Parameter name=&quot;type&quot;&gt;postgis&lt;/Parameter&gt;
      &lt;Parameter name=&quot;host&quot;&gt;localhost&lt;/Parameter&gt;
      &lt;Parameter name=&quot;user&quot;&gt;postgres&lt;/Parameter&gt;
      &lt;Parameter name=&quot;password&quot;&gt;&lt;/Parameter&gt;
      &lt;Parameter name=&quot;dbname&quot;&gt;hispania&lt;/Parameter&gt;
      &lt;Parameter name=&quot;table&quot;&gt;provincias&lt;/Parameter&gt;
      &lt;Parameter name=&quot;estimate_extent&quot;&gt;false&lt;/Parameter&gt;
      &lt;Parameter name=&quot;extent&quot;&gt;-510000.0,3800000.0,1200000.0,4900000.0&lt;/Parameter&gt;
    &lt;/Datasource&gt;
&lt;/Layer&gt; 
[/xml]

El parámetro table (provincias) podemos sustituir el nombre de una tabla geométrica por una consulta SQL encerrada entre paréntesis:
(select * from provincias)

Resultado:
<img src="http://www.gisandchips.org/wp-content/hola_spain_xml-300x300.png" alt="hola spain" width="300" height="300" />

Enlazando con lo último, en el momento en que definimos en una "layer" un "datasource", sea un shapefile o PostGIS, con su correspondiente "style", tenemos a disposición de las "rules" todas las columnas o campos del datasource, que pueden ser utilizados como filtros, lo que resulta muy util para la edición de cartografía temática.
Fichero: mapa_spain_xml2.py

[python]
#!/usr/bin/env python
import mapnik
mapfile = 'provincias2.xml'
map_output = 'hola_spain_xml2.png'
m = mapnik.Map(600, 600)
mapnik.load_map(m, mapfile)
bbox = mapnik.Envelope(mapnik.Coord(-510000.0, 3800000.0), mapnik.Coord(1200000.0, 4900000.0))
m.zoom_to_box(bbox)
mapnik.render_to_file(m, map_output)
[/python]

Ahora el fichero
Fichero provincias2.xml

[xml]
&lt;?xml version=&quot;1.0&quot; encoding=&quot;utf-8&quot;?&gt;
&lt;!DOCTYPE Map&gt;
&lt;Map bgcolor=&quot;steelblue&quot; srs=&quot;+proj=utm +zone=30 +ellps=intl +units=m +no_defs&quot;&gt;
  &lt;Style name=&quot;mi estilo provincias&quot;&gt;
    &lt;Rule&gt;
      &lt;PolygonSymbolizer&gt;
        &lt;CssParameter name=&quot;fill&quot;&gt;#f2eff9&lt;/CssParameter&gt;
      &lt;/PolygonSymbolizer&gt;
      &lt;LineSymbolizer&gt;
        &lt;CssParameter name=&quot;stroke&quot;&gt;rgb(50%,50%,50%)&lt;/CssParameter&gt;
        &lt;CssParameter name=&quot;stroke-width&quot;&gt;0.1&lt;/CssParameter&gt;
      &lt;/LineSymbolizer&gt;
    &lt;/Rule&gt;
  &lt;/Style&gt;
  &lt;Style name=&quot;mi estilo vias&quot;&gt;
&lt;!-- autopistas --&gt;
    &lt;Rule&gt;
      &lt;Filter&gt;[level] = 41&lt;/Filter&gt;
      &lt;LineSymbolizer&gt;
        &lt;CssParameter name=&quot;stroke&quot;&gt;#506077&lt;/CssParameter&gt;
        &lt;CssParameter name=&quot;stroke-width&quot;&gt;4&lt;/CssParameter&gt;
      &lt;/LineSymbolizer&gt;
      &lt;TextSymbolizer name=&quot;codigo&quot; face_name=&quot;DejaVu Sans Book&quot; size=&quot;11&quot; fill=&quot;#000&quot; halo_radius=&quot;1&quot; spacing=&quot;400&quot; placement=&quot;line&quot; /&gt;
    &lt;/Rule&gt;
&lt;!-- nacionales --&gt;
    &lt;Rule&gt;
      &lt;Filter&gt;[level] = 40&lt;/Filter&gt;
      &lt;LineSymbolizer&gt;
        &lt;CssParameter name=&quot;stroke&quot;&gt;#a37b48&lt;/CssParameter&gt;
        &lt;CssParameter name=&quot;stroke-width&quot;&gt;2&lt;/CssParameter&gt;
       &lt;/LineSymbolizer&gt;
    &lt;/Rule&gt;
  &lt;/Style&gt;
&lt;Layer name=&quot;provincias&quot; status=&quot;on&quot; srs=&quot;+proj=utm +zone=30 +ellps=intl +units=m +no_defs&quot;&gt;
    &lt;StyleName&gt;mi estilo provincias&lt;/StyleName&gt;
    &lt;Datasource&gt;
      &lt;Parameter name=&quot;type&quot;&gt;postgis&lt;/Parameter&gt;
      &lt;Parameter name=&quot;host&quot;&gt;localhost&lt;/Parameter&gt;
      &lt;Parameter name=&quot;user&quot;&gt;postgres&lt;/Parameter&gt;
      &lt;Parameter name=&quot;password&quot;&gt;&lt;/Parameter&gt;
      &lt;Parameter name=&quot;dbname&quot;&gt;hispania&lt;/Parameter&gt;
      &lt;Parameter name=&quot;table&quot;&gt;provincias&lt;/Parameter&gt;
      &lt;Parameter name=&quot;estimate_extent&quot;&gt;false&lt;/Parameter&gt;
      &lt;Parameter name=&quot;extent&quot;&gt;-510000.0,3800000.0,1200000.0,4900000.0&lt;/Parameter&gt;
    &lt;/Datasource&gt;&lt;br&gt;&lt;/Layer&gt;
&lt;Layer name=&quot;vias&quot; status=&quot;on&quot; srs=&quot;+proj=utm +zone=30 +ellps=intl +units=m +no_defs&quot;&gt;
    &lt;StyleName&gt;mi estilo vias&lt;/StyleName&gt;
    &lt;Datasource&gt;
      &lt;Parameter name=&quot;type&quot;&gt;postgis&lt;/Parameter&gt;
      &lt;Parameter name=&quot;host&quot;&gt;localhost&lt;/Parameter&gt;
      &lt;Parameter name=&quot;user&quot;&gt;postgres&lt;/Parameter&gt;
      &lt;Parameter name=&quot;password&quot;&gt;&lt;/Parameter&gt;
      &lt;Parameter name=&quot;dbname&quot;&gt;hispania&lt;/Parameter&gt;
      &lt;Parameter name=&quot;table&quot;&gt;vias&lt;/Parameter&gt;
      &lt;Parameter name=&quot;estimate_extent&quot;&gt;false&lt;/Parameter&gt;
      &lt;Parameter name=&quot;extent&quot;&gt;-510000.0,3800000.0,1200000.0,4900000.0&lt;/Parameter&gt;
    &lt;/Datasource&gt;
&lt;/Layer&gt;
&lt;/Map&gt;
[/xml]

Este mapfile consta de:
- 2 datasources
- provincias --&gt; asociado al style "mi estilo provincias"
- vias --&gt; asociado al style "mi estilo vias"
- 2 styles
- "mi estilo provincias"  --&gt; con una sola "rule"
- "mi estilo vias" --&gt; con 2 "rules"
--&gt; Autopistas: Filtro: "[level] = 40", Texto: ver TextSymbolizer
--&gt; Nacionales: [level] = 40

Resultado:

<img src="http://www.gisandchips.org/wp-content/hola_spain_xml2-300x300.png" alt="hola spain 2" width="300" height="300" />

Este artículo es el primero de una serie formada por:
<ul>
	<li>Renderizado con Mapnik (este post)</li>
	<li>Mapnik y OpenStreetMap</li>
	<li>Thumbnails con Mapnik y OpenStreetMap</li>
	<li>Uso de TMS de OpenStreetMap con OpenLayers</li>
	<li>TMS de OpenStreetMap con Mod_tile</li>
</ul>