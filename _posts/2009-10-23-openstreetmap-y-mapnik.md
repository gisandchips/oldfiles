---
ID: 420
post_title: OpenStreetMap y Mapnik
author: jose
post_date: 2009-10-23 13:42:23
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2009/10/23/openstreetmap-y-mapnik/
published: true
syntaxhighlighter_encoded:
  - "1"
---
<strong>Objetivo</strong>:
Siguiendo con la serie de artículos sobre OSM vamos a introducirnos en las posibilidades que nos ofrece <a href="http://mapnik.org/">Mapnik </a>para renderizar datos de OSM contenidos en una geodatabase que hemos creado con "osm2pgsql" (ver artículo <a href="http://www.gisandchips.org/2009/10/22/renderizado-de-osmcon-mapnik-para-usar-en-openlayers/#more-387">previo</a>).
El objetivo final es crear un fichero de mapa (mapfile) en XML para utilizarlo con Python y los bindings de Mapnik. Una vez realizado este proceso podremos generar tanto imágenes como conjuntos de teselas (tiles)

<img src="http://www.gisandchips.org/wp-content/ali_zoom14-300x300.png" alt="" width="300" height="300" />

<!--more-->

<strong>Descarga de utilidades OSM para Mapnik</strong>
En el repositorio subversion de OSM tenemos todo lo necesario para montar nuestro propio motor de renderizado de imágenes y conjuntos de tiles.
Para descargar estas utilidades nos situamos en la raíz de nuestro "home" y procedemos a la descarga:

[bash]
svn co http://svn.openstreetmap.org/applications/rendering/mapnik/ mapnik
cd mapnik
[/bash]

Este es el contenido de la carpeta "mapnik":
<ul>
	<li> <em>symbols</em>: carpeta con los iconos utilizados "oficialmente" en el "slippy map" de www.openstreetmap.org</li>
	<li><em>osm_template.xml</em>:  plantilla por defecto para generar un mapfile de OSM</li>
	<li><em>set_mapnik.env</em>:  script de configuración de directorios, datasources, etc.</li>
	<li><em>generate_image.py</em>:  script para generar una imagen</li>
	<li><em>generate_tiles.py</em>:  script para generar tiles</li>
	<li>otros archivos</li>
</ul>
<strong>Descarga de shapefiles</strong>:

OSM necesita una serie de shapefiles para poder renderizar aspectos como la costa, paises, masas de agua, etc.
Lo ideal es que dentro del directorio "mapnik" creemos una carpeta donde almacenarlos

[bash]
mkdir world_boundaries
[/bash]

Ahora descargamos los shapefiles y los descomprimimos

[bash]
echo &quot;Tamaño 50 Mb&quot;
wget http://tile.openstreetmap.org/world_boundaries-spherical.tgz
echo &quot;processed_p: Tamaño 227 Mb&quot;
wget http://tile.openstreetmap.org/processed_p.tar.bz2
echo &quot;shoreline_300: Tamaño 46 Mb&quot;
$ wget http://tile.openstreetmap.org/shoreline_300.tar.bz2
cd world_boundaries
echo &quot;Descomprimir&quot;
tar -xvzf world_boundaries-spherical.tgz
tar jxvf processed_p.tar.bz2
tar jxvf shoreline_300.tar.bz2
[/bash]

De todas las shapes de estos archivos nos interesan fundamentalmente tres:
<ul>
	<li><em>shoreline_300</em> : polígonos con los contienentes y en cuadrículas, utilizado para el renderizado de la costa. El detalle es muy tosco, por lo que se utiliza para escalas superiores a 1/600.000</li>
	<li><em>processed_p</em>:  polígonos con los contienentes y en cuadrículas, utilizado para el renderizado de la costa. Tienen gran detalle, por lo que se utiliza para escalas inferiores a 1/600.000</li>
	<li><em>builtup_area</em>:  polígonos con masas de agua continentales (lagos)</li>
</ul>
<strong>Configuración de Mapnik para OSM</strong>

El siguiente paso es configurar el fichero "set_mapnik.env" donde especificaremos las rutas a las carpetas de símbolos, shapefiles y conexiones a PostGIS.
Este fichero será utilizado por la plantilla de OSM ("osm_template.xml") para sustituir las variables contenidas entre "%", como por ejemplo "%HOST%", por nuestros valores personalizados.

Contenido del fichero "set_mapnik.env"

[python]
#!/bin/sh
# Nombre del mapfile
export MAPNIK_MAP_FILE=/home/&lt;usuario&gt;/mapnik/osm.xml

# carpeta de símbolos
export MAPNIK_SYMBOLS_DIR=/home/&lt;usuario&gt;/mapnik/symbols

# carpeta de shapefiles
export MAPNIK_WORLD_BOUNDARIES_DIR=/home/&lt;usuario&gt;/mapnik/world_boundaries

# Carpeta donde se generarán las tiles
export MAPNIK_TILE_DIR=/home/&lt;usuario&gt;/mapnik/tiles/

# Host de PostgreSQL
export MAPNIK_DBHOST='localhost';

# Puerto de PostgreSQL
export MAPNIK_DBPORT=''

# Nombre de la base de datos del planet OSM
export MAPNIK_DBNAME='osm'

# Nombre del usuario de Postgres con derechos sobre la base de datos OSM
export MAPNIK_DBUSER='postgres'

# Password del usuario de PostgreSQL
export MAPNIK_DBPASS='';

# Prefijo utilizado en las tablas. Este prefijo es generado si se utiliza
# el parámetro &quot;-p &lt;nombre&gt;&quot; en la utilidad &quot;osm2pgsql&quot;. Si no lo utilizamos
# se entiende que será &quot;planet_&quot;;
export MAPNIK_PREFIX=''
$*
#-----------------------------------------------------------------------------
[/python]

NOTA: En el script sustituye "" por tu nombre de usuario.

<strong>Configuración de la plantilla de OSM</strong>

Como ya hemos indicado, entre los ficheros proporcionados en la carpeta "mapnik", tenemos uno muy especial, "osm_template.xml". Como habeis deducido se trata de la plantilla que utiliza Mapnik para generar el mapfile ("osm.xml"). Sí no tocamos nada de este archivo obtendremos un renderizado de imágenes o tiles igual que el utilizado por el "slippy map" de <a href="http://www.openstreetmap.org">www.openstreetmap.org</a>. Es aquí, por tanto, donde debemos de agudizar nuestro ingenio para establecer una simbología acorde con nuestros intereses: cambiar colores, retocar las SQL de las layers, añadir o cambiar iconos, etc.

En lo referente a este artículo, sólo vamos a realizar pequeños cambios.

1. Vamos a resaltar las paradas del tranvía para que aparezca el logotipo "oficial" por el que se conoce en Alicante a este medio de transporte (<a href="http://www.fgvalicante.com">TRAM</a>)

En el documento cambiamos este texto:

[xml]
     &lt;Rule&gt;
      &lt;MaxScaleDenominator&gt;100000&lt;/MaxScaleDenominator&gt;
      &lt;MinScaleDenominator&gt;25000&lt;/MinScaleDenominator&gt;
      &lt;Filter&gt;[railway]='station' and not [disused]='yes'&lt;/Filter&gt;
      &lt;PointSymbolizer file =  &quot;%SYMBOLS_DIR%/station_small.png&quot; type=&quot;png&quot; width=&quot;6&quot; height=&quot;6&quot; /&gt;
    &lt;/Rule&gt;
[/xml]

Por este otro

[xml]
    &lt;Rule&gt;
      &lt;MaxScaleDenominator&gt;100000&lt;/MaxScaleDenominator&gt;
      &lt;MinScaleDenominator&gt;25000&lt;/MinScaleDenominator&gt;
      &lt;Filter&gt;[railway]='station' and not [disused]='yes'&lt;/Filter&gt;
      &lt;PointSymbolizer file =  &quot;%SYMBOLS_DIR%/tram2.png&quot; type=&quot;png&quot; width=&quot;20&quot; height=&quot;20&quot; /&gt;
    &lt;/Rule&gt;
[/xml]

2. Por otra parte vamos a cambiar el color de fondo para las áreas residenciales
Color: <span style="color: #F6CEE3">♦♦♦♦♦</span>

[xml]
    &lt;Rule&gt;
      &lt;MaxScaleDenominator&gt;1000000&lt;/MaxScaleDenominator&gt;
      &lt;MinScaleDenominator&gt;1000&lt;/MinScaleDenominator&gt;
      &lt;Filter&gt;[landuse] = 'residential'&lt;/Filter&gt;
      &lt;PolygonSymbolizer&gt;
        &lt;CssParameter name=&quot;fill&quot;&gt;#F6CEE3&lt;/CssParameter&gt;
      &lt;/PolygonSymbolizer&gt;
    &lt;/Rule&gt;
[/xml]

<strong>Crear imágenes</strong>

Una vez configurado nuestro mapfile ("osm.xml") pasamos a generar imágenes utilizando el script "generate_image.py". Te recomendamos que hagas una copia de ese archivo para no trabajar con el original. Con un editor de textos tienes que modificar los siguientes parámetros:

[python]
#!/bin/sh
# ... resto de código
    mapfile = &quot;osm.xml&quot;
    map_uri = &quot;image.png&quot;

    #---------------------------------------------------
    #  Change this to the bounding box you want
    #
    # Área metropolitana de Alicante
    ll = (-0.5534, 38.3245, -0.4267, 38.4071)
    #---------------------------------------------------
    z = 18
    imgx = 50 * z
    imgy = 50 * z
# ... resto de código
[/python]

Cómo ves sólo es necesario modificar tres aspectos:
<ol>
	<li>Extensión de la imagen en grados decimales: xmin,ymin,xmax,ymax</li>
	<li>Nivel de zoom: del 1 (mundo) al 18 (máximo de detalle)</li>
	<li>Dimensiones de la imagen en píxeles</li>
</ol>
NOTA:

Para evitar resultados desagradables intenta ajustar el BBOX y las dimensiones de la imagen.

Por último debemos de realizar los siguientes pasos para generar la imagen

1. Leer las variables que hemos definido en "set_mapnik.env"

[bash]source set-mapnik-env[/bash]

2.  Utilizar la plantilla "osm-template.xml" para generar el fichero "oxm.xml"

[bash]
./customize-mapnik-map &gt;$MAPNIK_MAP_FILE
[/bash]

3. Lanzar el script:

[bash]python mi_genera_imagen.py[/bash]

Este es el resultado:

Zoom 14
<img src="http://www.gisandchips.org/wp-content/ali_zoom14-300x300.png" alt="" width="300" height="300" />

Zoom 18 (fíjate en el renderizado de las estaciones del TRAM)
<img src="http://www.gisandchips.org/wp-content/ali_zoom18.png" alt="" width="300" height="300" />

<strong>Crear las tiles</strong>

Esta es la parte de Mapnik que tiene más interés para nosotros, puesto que nos permitirá generar nuestro propio TMS para ser utilizado con independencia del "Slippy map" de OpenStreetMap.

1. Creamos el directorio donde se guardarán las tiles creadas:

[bash]mkdir -p $MAPNIK_TILE_DIR[/bash]

2. Editamos el fichero "generate_tiles.py" para definir las zonas a renderizar

[python]
#!/usr/bin/python
# resto de código
# directorio donde se almacenan las imágenes
        tile_dir = home + &quot;/osm/tiles/&quot;

    #-------------------------------------------------------------------------
    #
    # Change the following for different bounding boxes and zoom levels
    #
    # Start with an overview
    # World
    bbox = (-180.0,-90.0, 180.0,90.0)

    render_tiles(bbox, mapfile, tile_dir, 0, 5, &quot;World&quot;)

    # Europa
    minZoom = 6
    maxZoom = 8
    bbox = (1.0,10.0, 20.6,50.0)
    render_tiles(bbox, mapfile, tile_dir, minZoom, maxZoom)

    # Provincia de Alicante
    bbox = (-1.498,37.622,0.529,38.946)
    render_tiles(bbox, mapfile, tile_dir, 9, 11 , &quot;Europe+&quot;)

    # Área metropolitana de la ciudad de Alicante
    bbox = (-0.6171,38.295,-0.3637,38.4602)
    render_tiles(bbox, mapfile, tile_dir, 12, 18 , &quot;Alicante&quot;)
[/python]

Cómo habrás podido deducir vamos a generar tiles en cuatro intervalos de escala:
<ul>
	<li>Escalas pequeñas: Zoom del 1 al 5</li>
	<li>Escalas medias: 6 al 8</li>
	<li>Escalas grandes (más detalles): 9 al 11</li>
	<li>Escalas de máximo detalle): 12 al 18</li>
</ul>
3. Ejecutamos el script

[bash]python ./mi_genera_tiles.py[/bash]

4. Comprobación:
Fíjate que en la carpeta "tiles" tendremos una carpeta por cada nivel de zoom. En los niveles superiores es donde habrán más imágenes (las de mayor detalle)

[bash]ls tiles[/bash]

Por fin, ya hemos creado nuestro directorio de tiles listo para ser utilizado por OpenLayers como un TMS.

Esto es todo. Espero que os haya gustado. Comentarios y críticas serán bien recibidas

Este artículo es el segundo de una serie formada por:
<ul>
	<li><a href="http://www.gisandchips.org/2009/10/22/renderizado-de-osmcon-mapnik-para-usar-en-openlayers/#more-387">Renderizado con Mapnik</a></li>
	<li>Mapnik y OpenStreetMap (este post)</li>
	<li>Thumbnails con Mapnik y OpenStreetMap</li>
	<li>Uso de TMS de OpenStreetMap con OpenLayers</li>
	<li>TMS de OpenStreetMap con Mod_tile</li>
</ul>