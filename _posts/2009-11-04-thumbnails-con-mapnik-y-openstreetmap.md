---
ID: 543
post_title: Thumbnails con Mapnik y OpenStreetMap
author: jose
post_date: 2009-11-04 21:12:13
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2009/11/04/thumbnails-con-mapnik-y-openstreetmap/
published: true
syntaxhighlighter_encoded:
  - "1"
---
¿Alguna vez te has visto en la tesitura de conocer el estado en OpenStreetMap de una determinada provincia, municipio, comunidad autónoma, región etc. en una fecha concreta? Pues bien, en este artículo intentaremos afrontar esta cuestión desde una perspectiva más pedagógica que profesional

<strong>Objetivo</strong>:
Crear thumbnails (imágenes reducidas) de cada provincia que serán generadas cada semana para ver la evolución histórica de las aportaciones de usuarios a OpenStreetMap. Cada imagen tendrá como nombre la fecha de su creación, y se almacenarán cada una en su carpeta de provincia correspondiente. Estas imágenes serán visibles en un servicio web para su consulta pública.

Fases:

Para llevar a cabo este objetivo precisamos de:
<ol>
	<li>Actualizar cada semana la base de datos con el último "planet osm"</li>
	<li>Generar las imágenes de cada provincia (mediante un script en Python)</li>
	<li>Colocarlas en el lugar adecuado de nuestro servidor web</li>
	<li>Programar la tarea para que se ejecute de forma automática cada semana, sin intervención del usuario.</li>
</ol>
Figura: <em>Ejemplo del renderizado de una imagen de la provincia de Alicante</em>
<p style="text-align: center"></p>


[caption id="attachment_585" align="alignnone" width="400" caption="Tile de Alicante el 3 de noviembre de 2009"]<img class="size-full wp-image-585" src="http://www.gisandchips.org/wp-content/2009-11-03.png" alt="Tile de Alicante" width="400" height="400" />[/caption]

<!--more-->

<strong>Descripción:</strong>
A lo largo de este pequeño artículo mostraremos las posibilidades conjuntas de OSM, PostGIS y Mapnik para que, con muy pocas líneas de código, seamos capaces de generar un repositorio de imágenes de cada provincia (en nuestro caso, aunque se puede adaptar a comunidades autónomas, municipios, etc.) que serán generadas por un script en python programado para ejecutarse una vez a la semana. Dicho script utilizará los módulos de Mapnik para generar imágenes a partir de un mapfile, y PsycoPg para llamar a sentencias SQL de PostgreSQL.

<!--more-->

<strong>Punto de partida:</strong>
Tenemos instalado un servidor con las siguientes aplicaciones instaladas y configuradas:
<ul>
	<li>Python</li>
	<li>PostgreSQL y PostGIS</li>
	<li>Mapnik y utilidad Mapnik de OSM</li>
	<li>Una tabla de PostGIS con las provincias</li>
	<li>Un servidor web para mostrar los resultados</li>
</ul>
Actualizar la base de datos de OSM cada semana

Para realizar esta fase necesitamos un script que automatice todas las funciones:
<ul>
	<li>Borrar la bbdd existente</li>
	<li>Descarga del planet</li>
	<li>Creación de la bbdd con PostGIS</li>
	<li>Carga de datos con osm2pgsql</li>
</ul>
<strong>Script Bash para recargar la base de datos de OSM</strong>

[bash]
#!/bin/sh
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
echo &quot;Añadir datos d espain.osm a la bbdd osm&quot;
echo &quot;Tiempo aproximado: 11 minutos&quot;
/home/&lt;user&gt;/compilar/osm2pgsql/osm2pgsql --slim -H localhost -U postgres -d osm ./spain.osm
echo &quot;fin&quot;
exit 0
[/bash]

<strong>Programar tarea con CRONTAB</strong>
Como ya sabes, los "crones" son tareas (<em>deaemon</em>) que se programan para ejecutarse en el momento (tiempo) que decidamos. Sí deseas más información aquí tienes un buen enlace: <a href="http://dns.bdat.net/documentos/cron/x50.html">http://dns.bdat.net/documentos/cron/x50.html</a>

<strong>Generar el árbol de directorios</strong>.
Creamos un script bash para generar las carpetas de cada provincia. Vamos a utilizar el código de cada provincia por cuestión de comodidad

[bash]
#!/bin/bash
# Crear directorios para provincias.

for provincias in 01 02 03 04 05 06 07 08 09 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52
do
  echo $provincias
  mkdir -p $provincias
done

exit 0
[/bash]

Este sencillo script, que seguro podrás mejorar, debemos de ejecutarlo en el directorio de nuestro website donde se alojarán el servicio para ver la evolución de OSM

<strong>Generar imágenes de OSM por provincia</strong>
Este script en Python hace uso de los módulos de Mapnik para generar la imagen, y de Psycopg para ejecutar sentencias SQL. Fíjate que la clave consiste en calcular por cada provincia las columnas con los valores de la extensión (xmin, ymin, xmax, ymax). Opcionalmente puedes añadirle una tolerancia para que el límite no se circunscriba a la caja de la provincia, sino que sea como una especie de buffer.
En nuestro caso, la tabla "provincias" está en la proyección UTM-ED50 del uso 30 (epsg: 23030), por lo que para utilizarlo con los datos de OSM necesitamos proyectar los valores de la extensión a Mercator (EPSG: 900913)

NOTA: Si no tienes instalago psycopg (o psycopg2 según la distro) lo puedes instalar con

[bash]yum install python-psycopg2.x86_64[/bash]

Script de Python

[python]
#!/usr/bin/env python
## DNS de la bbdd que contiene las provincias
DSN = 'host=localhost user=postgres dbname=demos'

import sys, psycopg, mapnik
## Nota: Si obtienes algun error aqui, cambia psycopg por psycopg2
from datetime import date

## mapnik
mapfile = 'osm.xml'
## tamaño de la imagen
m = mapnik.Map(400, 400)

if len(sys.argv) &gt; 1:
    DSN = sys.argv[1]

print 'Abrir conexion usando dns', DSN
conn = psycopg.connect(DSN)
## Nota: Si obtienes algun error aqui, cambia psycopg por psycopg2
curs = conn.cursor()

curs.execute('select codigo, st_xmin(st_extent(st_transform(geometria,900913))) as xmin, st_ymin(st_extent(st_transform(geometria,900913))) as ymin, st_xmax(st_extent(st_transform(geometria,900913))) as xmax, st_ymax(st_extent(st_transform(geometria,900913))) as ymax from ine.provincias group by codigo')

row = curs.fetchone()

rows = curs.fetchall()
print 'Creando imagenes...';
for row in rows:
    now = date.today()
    ## A cada imagen le asignamos como nombre la fecha de creación de la imagen
    map_output = 'miniaturas/'+row[0]+'/'+now.strftime(&quot;%Y-%m-%d&amp;&quot;)+'.png'
    mapnik.load_map(m, mapfile)
    bbox = mapnik.Envelope(mapnik.Coord(row[1],row[2]), mapnik.Coord(row[3],row[4]))
    m.zoom_to_box(bbox)
    mapnik.render_to_file(m, map_output)

conn.commit()
[/python]

Ejecutamos el script

[bash]
python miniatura_provincias.py
[/bash]

Por último sólo nos queda hacer visible a nuestro servidor web las tiles. Para ello tenemos dos opciones:

a) Copiamos el directorio miniaturas a nuestra carpeta de publicación web

[bash]cp /home/mapnik/miniaturas /var/www/&lt;tu_ruta&gt;/[/bash]

b) Crear un enlace rígido en nuestra web que apunte al directorio "miniaturas"

[bash]ln -d /home/mapnik/miniaturas /var/www/&amp;lt;tu_ruta&amp;gt;/miniaturas[/bash]

Este segundo método es más recomendable, puesto que nos permite aislar la lógica de nuestra aplicación del servicio web.

<strong>Test</strong>
Probamos el ejemplo
<a href="http://www.gisandchips.org/demos/j3m/miniaturas/">http://www.gisandchips.org/demos/j3m/miniaturas/</a>

<strong>Automatización:</strong>

Sí queremos automatizar estas tareas, de forma que cada semana obtengamos una nueva imagen, cuyo nombre de imagen es la fecha en que se lanza el <a href="http://www.telefonica.net/web2/dinsalpa/crontab/crontab.htm">cron</a>, debemos de lanzar un script que, como root, realice estos pasos. A modo de ejemplo, este sería el Bash:

[bash]
#!/bin/sh
cd &lt;ruta_directorio_volcado_bbdd_osm&gt;
python ./&lt;fichero python para generar miniaturas&gt;
cp /home/mapnik/miniaturas /var/www/&lt;tu_ruta&gt;
[/bash]

Este artículo es el tercero de una serie formada por:
<ul>
	<li><a href="http://www.gisandchips.org/2009/10/22/renderizado-de-osmcon-mapnik-para-usar-en-openlayers/#more-387">Renderizado con Mapnik</a></li>
	<li><a href="http://www.gisandchips.org/2009/10/23/openstreetmap-y-mapnik/">Mapnik y OpenStreetMap</a></li>
	<li>Thumbnails con Mapnik y OpenStreetMap  (este post)</li>
	<li>Uso de TMS de OpenStreetMap con OpenLayers</li>
	<li>TMS de OpenStreetMap con Mod_tile</li>
</ul>