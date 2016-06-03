---
ID: 290
post_title: >
  Análisis de redes con OpenStreetMap y
  PgRouting en un ambiente web
author: jose
post_date: 2009-10-03 01:50:27
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2009/10/03/analisis-de-redes-con-openstreetmap-y-pgrouting-en-un-ambiente-web/
published: true
syntaxhighlighter_encoded:
  - "1"
---
<strong>OBJETIVO</strong>:

El objetivo de esta serie de artículos es proporcionar una metodología para crear servicios de enrutamiento (routing) en portales web utilizando datos de OpenStreetMap (<a href="http://www.openstreetmap.org/">http://www.openstreetmap.org</a>) que serán añadidos a una geodatabase de PostgreSQL, con la extensión PostGIS y PgRouting.

Para llevar a cabo este servicio son necesarias las siguientes fases:
<ol>
	<li>Instalación de las extensiones PostGIS y PgRouting</li>
	<li>Importación de los datos de OSM</li>
	<li>Conversión a una geodatabase de PostGIS en PostgreSQL</li>
	<li>Explotación de una geodatabase con PgRouting.</li>
	<li>Diseño de la interfaz de usuario. Contenedor HTML y Javascript (Openlayers)</li>
	<li>Creación de lógica de enrutamiento en PHP</li>
</ol>
En la medida en que vaya finalizando los capítulos serán publicados en esta web. En este primer artículo se abordarán las tres primeras fases.

<!--more-->

<strong>OBSERVACIONES</strong>:

Para la realización del presente documento me he basado en las referencias web de los portales de PgRouting (<a href="http://pgrouting.postlbs.org/">http://pgrouting.postlbs.org/</a>) y OpenLayers (<a href="http://openlayers.org/">http://openlayers.org</a>), así como en diferentes páginas encontradas en Internet. En la medida en que se utilice información de otra fuente será convenientemente citada, salvo olvido.

<strong>ÁMBITO DE APLICACIÓN</strong>

Todas las fases se han ensayado satisfactoriamente en los siguientes entornos: Ubuntu 9.04 y Centos 5.3. (clon de Red Hat Enterpise Linux). No obstante, las indicaciones y comandos utilizados en este artículo hacen referencia a este último sistema operativo.

Partimos de la base que tenemos un servidor Linux con el sistema operativo Centos 5.3 de 64 bits, con la base de datos PostgreSQL 8.3 ya instaladas. Por defecto, Centos dispone de PostgreSQL 8.1, si bien hemos preferido añadir los repositorios para la 8.3.7. Para información sobre la instalación ver este enlace: <a href="https://projects.commandprompt.com/public/pgcore">https://projects.commandprompt.com/public/pgcore</a>

<strong><em>NOTA</em></strong><em>: La instalación de PostgreSQL desde este repositorio “oficial” de PostgreSQL no ha sido todo lo gratificante que esperaba. Hay problemas de dependencias con respecto a otras librerías del sistema Centos. Esto en cambio no ocurre con Ubuntu, que por defecto tiene en su paquetería las últimas versión de esta base de datos.</em>

APLICACIONES UTILIZADAS
<ul>
	<li>PostgreSQL 8.3</li>
	<li>PostGIS 1.4</li>
	<li>OpenLayers 2.8</li>
	<li>PgRouting 1.03</li>
	<li>Osm2pgRouting</li>
	<li>Osmosis</li>
</ul>
<strong>INSTRUCCIONES Y RECOMENDACIONES GENERALES:</strong>

Las instrucciones para la instalación o compilación de programas se especifican mediante comandos de la consola Linux, sin tener que recurrir a ninguna herramienta gráfica. Esta forma de actuar puede ser útil para aquellos usuarios que disponen de acceso a un servidor remoto, donde sólo se tiene acceso mediante la consola.

Con el fin de facilitar las tareas conviene crear un directorio donde descargar, descomprimir y compilar las fuentes de los programas. Para ello, crea un directorio de compilación dentro del home del usuario
<code>
$ mkdir /home/&lt;usuario&gt;/compilar</code>

<code> </code>

<code>$ cd /home/&lt;usuario&gt;/compilar
</code>

<strong><em>NOTA</em></strong><em>: El símbolo “$” es sólo el indicativo del prompt del sistema, no es necesario escribirlo.</em>

Otra opción es crear un directorio de fuentes (por ejemplo “src” dentro de la carpeta “usr/local/”, y trabajar siempre como root. Yo particularmente prefiero la primera opción.

<strong>INSTALACIÓN DE POSTGIS</strong>

Aunque es posible utilizar la paquetería de PostGIS que aparece en el repositorio (yum info Postgis) he preferido compilar el paquete con el fin de tener la última versión estable de PostGIS

Sí no estás interesado en compilar PostGIS, puedes instalar el paquete oficial que viene con el repositorio de Postgres para Centos. Utiliza estos comandos
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top"># yum info postgis</td>
</tr>
</tbody></table>
Y obtendrás esta información:
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">Available PackagesName : postgisArch : x86_64Version : 1.3.6Release : 1.rhel5Size : 1.0 MRepo : pgdg83Summary : Geographic Information Systems Extensions to PostgreSQLURL : <a href="http://postgis.refractions.net/">http://postgis.refractions.net/</a>

License : GPL

Description: PostGIS adds support for geographic objects to the PostgreSQL

: object-relational database. In effect, PostGIS “spatially enables”

: the PostgreSQL server, allowing it to be used as a backend spatial

: database for geographic information systems (GIS), much like ESRI’s

: SDE or Oracle’s Spatial extension. PostGIS follows the OpenGIS

: “Simple Features Specification for SQL” and has been certified as

: compliant with the “Types and Functions” profile.</td>
</tr>
</tbody></table>
Luego, como superusuario:
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top"># yum install postgis</td>
</tr>
</tbody></table>
<strong>Compilación de PostGIS 1.4</strong>

Por suerte, PostGIS es una extensión bastante utilizada, y por tanto su compilación está muy bien detallada en el portal de Reflaction Research (<a href="http://postgis.refractions.net/">http://postgis.refractions.net/</a>)

Esta extensión requiera para su compilación de dos librerías de las que depende (dependencias), por una parte PROJ y por otra GEOS. Ambas son bien conocidas y utilizadas en multitud de programas de SIG de código abierto. Ambas son opcionales, y no son necesarias para la compilación, pero sin ellas dejariamos a PostGIS huerfano de utilidades de reproyección y geoprocesamiento.

<strong>Instalación de la librería PROJ.4</strong>

PROJ.4 (<a href="http://trac.osgeo.org/proj/">http://trac.osgeo.org/proj/</a>) es la librería más utilizada para las proyecciones cartográficas y permite, como no podía ser de otra manera realizar conversiones entre diferentes proyecciones cartográficas.

<strong><em>NOTA</em></strong><em>: En el blog de Geomaticblog.net hay un excelente post de como utlizar esta librería desde aplicaciones como GDAL/OGR o GvSIG. Particularmente, me ha resultado útil para transformar datos desde ED50 a WGS84. Enlace: <a href="http://geomaticblog.net/2009/01/23/2009-01-23-ogrs_y_rejillas_ntv2/">http://geomaticblog.net/2009/01/23/2009-01-23-ogrs_y_rejillas_ntv2/</a></em>

Pasos para la compilación:

1. Descargar la fuente

$ wget <a href="http://download.osgeo.org/proj/proj-4.7.0.tar.gz">http://download.osgeo.org/proj/proj-4.7.0.tar.gz</a>

2. Descomprimirla

$ tar xzvf proj-4.7.0.tar.gz

3. Cambio de directorio

$ cd proj-4.7.0

4. Configuración y puesta a punto. El siguiente comando prepara la compilación advirtiéndonos si falta algún que otro paquete

$ ./configure

<em>NOTA: Es posible que en este paso aparezcan errores porque falta algún paquete. Tan sólo fíjate en que es lo que falta e instálalo (con “yum” o “apt” en el caso de Ubuntu). Habitualmente serán necesarias algunas herramienta propias de la compilación, como flex, bison, make, etc. En Ubuntu tienes un paquete específico para ello: “sudo apt-get install build-essential”.</em>

5. Compilación

$ make

6. Colocar las librerias en su sitio como “root”

$ make install

Para “impersonarse” como superusuario sólo tienes que escribir: “<strong>su -</strong>”. En Ubuntu sólo tienes que anteceder la palabra “<strong>sudo</strong>” a cualquier comando que quieras ejecutar como root. En ambos casos te pedirá la clave de acceso del administrador del sistema.

7. Comprobación

$ proj

Obtendremos lo siguiente:
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">Rel. 4.7.1, 23 September 2009usage: proj [ -beEfiIlormsStTvVwW [args] ] [ +opts[=arg] ] [ files ]</td>
</tr>
</tbody></table>
<strong>Instalación de GEOS (Geometry Engine Open Source)</strong>

GEOS (<a href="http://trac.osgeo.org/geos/">http://trac.osgeo.org/geos/</a>) es una librería que proporciona a PostGIS la capacidad de realizar geoprocesamiento. Sin ella no podriamos realizar las típicas operaciones de unión, intersección, diferencia, buffer, etc, todas ellas correspondiente a la especificación SFA (Simple Feature Access) del OGC. GEOS es un porting de la librería JTS (Java Topology Suite) en el lenguaje C++, lo que le convierte en una máquina de geoprocesos bastante eficiente.

Pasos para la compilación

1. Descarga de la fuente

$ wget <a href="http://download.osgeo.org/geos/geos-3.1.1.tar.bz2">http://download.osgeo.org/geos/geos-3.1.1.tar.bz2</a>

2.Descomprimir:

$ tar -xjvf geos-3.1.1.tar.bz2

3.Cambio de directorio

$ cd geos-3.1.1

4.Configuración

$ LDFLAGS=-lstdc++ ./configure

La directiva LDFLAGS tiene como objetivo enlazar expliciatamente PostgreSQL con el estándar C++

5. Compilar

$ make

El tiempo de compilación suele ser bastante largo (en torno a los 7-10 minutos)

6. Colocar librerías en su sitio (Como usuario root):

$ make install

7. Comprobación

$ geos-config –version

Devolverá:

3.1.1

<strong>Instalar PostGIS</strong>

Sí las dos librerías anteriores han sido instaladas correctamente deberás de tener los siguientes archivos en el directorio “/usr/local/lib”. Compruébalo:
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">$ ls /usr/local/lib/libproj*/usr/local/lib/libproj.a /usr/local/lib/libproj.so.0/usr/local/lib/<a href="http://libproj.la/">libproj.la</a> /usr/local/lib/libproj.so.0.5.5/usr/local/lib/libproj.so /usr/local/lib/libproj.so.0.6.6$ ls /usr/local/lib/libgeos*/usr/local/lib/<a href="http://libgeos-3.1.0.so/">libgeos-3.1.0.so</a> /usr/local/lib/<a href="http://libgeos_c.la/">libgeos_c.la</a>/usr/local/lib/<a href="http://libgeos-3.1.1.so/">libgeos-3.1.1.so</a> /usr/local/lib/libgeos_c.so/usr/local/lib/libgeos.a /usr/local/lib/libgeos_c.so.1/usr/local/lib/<a href="http://libgeos.la/">libgeos.la</a> /usr/local/lib/libgeos_c.so.1.5.0

/usr/local/lib/libgeos.so /usr/local/lib/libgeos_c.so.1.6.0</td>
</tr>
</tbody></table>
<strong>Proceso de instalación:</strong>

1. Descarga de las fuentes:

$ wget <a href="http://postgis.refractions.net/download/postgis-1.4.0.tar.gz">http://postgis.refractions.net/download/postgis-1.4.0.tar.gz</a>

2. Descomprimir:

$ tar xzvf postgis-1.4.0

3. Cambiar de directorio

$ cd postgis-1.4.0

4. Configurar

Es el paso que más atención debemos de prestar, puesto que al final del comando emitirá un informe con las dependencias satisfechas

$ ./configure

Sí todo va bien obtendremos un mensaje similar a este:
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">PostGIS is now configured for x86_64-unknown-linux-gnu-------------- Compiler Info -------------C compiler: gcc -g -O2C++ compiler: g++ -g -O2-------------- Dependencies --------------GEOS config: /usr/local/bin/geos-configGEOS version: 3.1PostgreSQL config: /usr/bin/pg_configPostgreSQL version: 8.3

PROJ4 version: 47

PostGIS debug level: 0

-------- Documentation Generation --------

xsltproc: /usr/bin/xsltproc

xsl style sheets:

dblatex:

convert:</td>
</tr>
</tbody></table>
5.Compilación y colocación de las librerías

$ make

$ make install (como root)

<strong>Post-instalación de PostGIS</strong>

El siguiente paso es de vital importancia y nos puede ahorrar muchos problemas en el futuro. Por defecto las librerías se instalan en el directorio ya citado (”/usr/local/lib”), sin embargo, éste directorio no es reconocido por los programas de compilación como un directorio válido para instalar librerías, por lo que es necesario indicárselo de forma expresa. Para ello como root:

1. Edita el fichero “/etc/ld.so.conf” con tu editor preferido (vi, pico, nano, etc):

$ nano /etc/ld.so.conf

Y añade las siguientes líneas al final del documento
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">/usr/local/lib/usr/local/lib64</td>
</tr>
</tbody></table>
2. Finalmente ejecuta este comando

$ /sbin/ldconfig

<strong>INSTALACIÓN DE LA LIBRERÍA PgRouting</strong>

PgRouting (<a href="http://pgrouting.postlbs.org/">http://pgrouting.postlbs.org</a>), al igual que PostGIS, es una extensión de PostgreSQL que permite trabajar con redes

A diferencia de PostGIS, aun no tenemos un paquete que instale PgRouting que satisfaga todas sus dependencias, por ello es necesario optar por la compilación de todas y cada una de las librerías utilizadas.

Librerías de las que depende:
<ul>
	<li>BOOST (<a href="http://www.boost.org/">http://www.boost.org</a> ): Es la librería principal, y la única que es obligatoria. Se trata de un basto conjunto de utilidades, entre las que destacan las referidas al análisis y explotación de grafos.</li>
	<li>GAUL (Genetic Algorithm Utility Library): Esta librería sólo es necesario si deseamos dotar a PostgreSQL de la utilidad de cálculo del llamado “Problema del viajante”, conocida por sus signas en inglés, TSP (Traveling Salesman Problem), es decir, proporcionar la ruta óptima pasando por varios sitios. Enlace: <a href="http://gaul.sourceforge.net/">http://gaul.sourceforge.net/</a></li>
	<li>CGAL (Computacional Geometric Algorithm Library): Librería especializada en el cálculo y procesamiento de estructuras geométricas en 2D y 3D. En el caso de PgRouting proporciona el algoritmo de cálculo conocido como Driving Distance.</li>
</ul>
<em>NOTA: Adicionalmente compilaremos el compilador CMAKE</em>

<strong>BOOST</strong>

Como ya se ha comentado se trata de una librería muy ambiciosa por la gran cantidad de utilidades que engloba, de ahí que se compilación sea muy pesada (en tiempo de compilación). En los repositorios, Centos dispone actualmente de la versión 1.33.1, bastante antigua, si pensamos que la última estable es la 1.4.0. No obstante, para este artículo utilizaré la 1.39.0 que es la que tenemos más ensayada.

Con todo, de esta líbrería sólo necesitamos las utilidades de grafos (graphs)

Para ver la versión de las fuentes boost del repositorio:

$ yum info boost-devel

Sí no queremos complicarnos la vida con la compilación añade el paquete con “yum”:

$ yum install boost-devel

En Ubuntu 9.04, la versión disponible es la 1.34, sería:

$ sudo apt-get install libboost-graph-dev

<strong>Compilación de las fuentes de boost 1.39</strong>

1. Descarga las fuentes:

wget <a href="http://sourceforge.net/projects/boost/files/boost/1.39.0/boost_1_39_0.tar.bz2/download">http://sourceforge.net/projects/boost/files/boost/1.39.0/boost_1_39_0.tar.bz2/download</a>

2. Descomprimir

$ tar –xjvf boost_1_39_0.tar.bz2

3. Cambiamos al usuario administrador

$ su -

4. Instalar dependencias:

$ yum install expat-devel

$ yum install python-devel

$ yum install freetype-devel

5. Cambio de directorio

$ cd /home/&lt;usuario&gt;/compilar/boost_1_39_0

6. Configuración

Para obtener ayuda sobre la configuración

$ ./bootstrap.sh –help

Configuración genérica: Instala todas las librerías en este directorio

$ ./bootstrap.sh –prefix=/usr/local/

Sí sólo necesitamos compilar la librería de cabezera (header library)

$ ./bootstrap.sh –prefix=/usr/local/ --with-libraries=graph

5. Compilar

$ ./bjam install

Es posible que tengas al inicio de la compilación algún mensaje relacionado con la librería EXPAT ques es utilizada por libGraph.  Para evitar este mensaje define estas variables:

$ export EXPAT_INCLUDE=/usr/include

$ export EXPAT_LIBPATH=/usr/lib

Y de nuevo ejecuta

<span style="background-color: #ffffff">$ ./bjam install</span>

<em>NOTA</em><em>: Sí deseas compilar versiones anteriores, como la 1.36 y 1.37, la compilación es más sencilla. Usa los clásicos: ./configure, make, make install</em>

<strong>GAUL</strong>

Instalación de Gaul (librería necesaria para el algoritmo TSP -Traveling Salesman Problem)

$ wget <a href="http://nfsi.dl.sourceforge.net/sourceforge/gaul/gaul-devel-0.1849-0.tar.gz">http://nfsi.dl.sourceforge.net/sourceforge/gaul/gaul-devel-0.1849-0.tar.gz</a>

$ tar xzvf gaul-devel-0.1849-0.tar.gz

$ ./configure --disable-slang

<strong>Instalar CMAKE</strong>

Este compilador sólo se instalará para el usuario, y no estará disponible para todos (no hacer nunca “make install” como root)

$ wget <a href="http://www.cmake.org/files/v2.4/cmake-2.4.8.tar.gz">http://www.cmake.org/files/v2.4/cmake-2.4.8.tar.gz</a>

$ tar -zxvf cmake-2.4.8.tar.gz

$ cd cmake-2.4.8

$ ./configure

Como usuario (¡no como administrador¡)

$ gmake

<strong>CGAL</strong>

Librería necesaria para utilizar el algoritmo Driving Distance). La versión que vamos a compilar es la 3.3.1, aunque existe otra más moderna (3.4). Sin lugar a dudas es la libraría que más problemas ofrece.

$ wget <a href="ftp://ftp.mpi-sb.mpg.de/pub/outgoing/CGAL/CGAL-3.3.1.tar.gz">ftp://ftp.mpi-sb.mpg.de/pub/outgoing/CGAL/CGAL-3.3.1.tar.gz</a>

$ tar xzvf CGAL-3.3.1.tar.gz

$ cd CGAL-3.3.1

Compilar como root:

$ ./install_cgal --prefix=/usr/local --with-boost=n --without-autofind -ni /usr/bin/g++

Las librerías y los include se colocan en estos directorios:

/usr/local/lib/

/usr/local/include/

<strong>INSTALACIÓN DE LA LIBRERÍA PGROUTING</strong>

Una vez satisfechas todas las dependencias de librerías vamos a compilar PgRouting<span> </span>

1. Primero nos descargamos las fuentes desde el repositorio subversión. Recuerda que debes de tener instalada dicha utilidad (“yum install subversión”)

$ svn checkout <a href="http://pgrouting.postlbs.org/svn/pgrouting/trunk">http://pgrouting.postlbs.org/svn/pgrouting/trunk</a> pgrouting

2. Nos cambiamos al directorio

$ cd pgrouting

3. Configuramos

Para ello utilizamos el compilador CMAKE que ya tenemos disponibles.

Compilar (no olvides el punto al final del comando)

La sintaxis de compilación es:

&lt;ruta al ejecutable CMAKE) .

NOTA: No olvides el punto al final. Es para indicarle que compile el directorio actual

Con ello instalaremos la librería, pero apenas nos servirá de nada porque no hemos hecho referencia a que queremos los algoritmos “Driving Distance (DD)” y TSP.

Por ello cambia lo anterior por esto

# ../cmake-2.4.8/bin/cmake -DWITH_TSP=ON -DWITH_DD=ON .

Resultado del configure
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">Output directory for libraries is set to /usr/lib64/pgsqlInstallation directory for libraries is set to /usr/lib64/pgsql and for SQL files is set to /usr/share/postlbsInstallation directory for libraries is set to /usr/lib64/pgsql-- Configuring done-- Generating done-- Build files have been written to: /home/jose/compilar/pgrouting../cmake-2.4.8/bin/cmake -DWITH_TSP=ON -DWITH_DD=ON .

-- Check for working C compiler: /usr/bin/gcc

-- Check for working C compiler: /usr/bin/gcc -- works

-- Check size of void*

-- Check size of void* - done

-- Check for working CXX compiler: /usr/bin/c++

-- Check for working CXX compiler: /usr/bin/c++ -- works

-- Found PostgreSQL: /usr/include/pgsql/server, /usr/lib64/libpq.so

Boost headers were found here: /usr/local/include

Output directory for libraries is set to /usr/lib64/pgsql

-- Found PGROUTING_CORE core: /home/jose/compilar/pgrouting/core/src

Installation directory for libraries is set to /usr/lib64/pgsql and for SQL files is set to /usr/share/postlbs

-- Found GAUL: /usr/local/lib/libgaul.so, /usr/local/lib/libgaul_util.so

Installation directory for libraries is set to /usr/lib64/pgsql

-- Found CGAL: /usr/local/include, /usr/local/lib/libCGAL.so

-- Configuring done

-- Generating done

-- Build files have been written to: /home/jose/compilar/pgrouting</td>
</tr>
</tbody></table>
4. Finalmente generamos la librería

# make

Resultado de la compilación
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">Scanning dependencies of target routing_tsp[ 8%] Building C object extra/tsp/src/CMakeFiles/routing_tsp.dir/tsp.o[ 16%] Building CXX object extra/tsp/src/CMakeFiles/routing_tsp.dir/tsp_solver.oLinking CXX shared library ../../../lib/librouting_tsp.so[ 16%] Built target routing_tspScanning dependencies of target routing_dd[ 25%] Building C object extra/driving_distance/src/CMakeFiles/routing_dd.dir/alpha.o

[ 33%] Building CXX object extra/driving_distance/src/CMakeFiles/routing_dd.dir/alpha_drivedist.o

[ 41%] Building CXX object extra/driving_distance/src/CMakeFiles/routing_dd.dir/boost_drivedist.o

[ 50%] Building C object extra/driving_distance/src/CMakeFiles/routing_dd.dir/drivedist.o

Linking CXX shared library ../../../lib/librouting_dd.so

[ 50%] Built target routing_dd

Scanning dependencies of target routing

[ 58%] Building C object core/src/CMakeFiles/routing.dir/dijkstra.o

[ 66%] Building C object core/src/CMakeFiles/routing.dir/astar.o

[ 75%] Building C object core/src/CMakeFiles/routing.dir/shooting_star.o

[ 83%] Building CXX object core/src/CMakeFiles/routing.dir/boost_wrapper.o

[ 91%] Building CXX object core/src/CMakeFiles/routing.dir/astar_boost_wrapper.o

[100%] Building CXX object core/src/CMakeFiles/routing.dir/shooting_star_boost_wrapper.o

Linking CXX shared library ../../lib/librouting.so

[100%] Built target routing</td>
</tr>
</tbody></table>
5. Por último colocamos los archivos en su sitio (como root):

$ make install

Resultados
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">[ 16%] Built target routing_tsp[ 50%] Built target routing_dd[100%] Built target routingLinking CXX shared library CMakeFiles/CMakeRelink.dir/librouting_tsp.soLinking CXX shared library CMakeFiles/CMakeRelink.dir/librouting_dd.soInstall the project...-- Install configuration: ""

-- Install configuration: ""

-- Install configuration: ""

-- Install configuration: ""

-- Installing /usr/lib64/pgsql/librouting.so

-- Install configuration: ""

-- Installing /usr/share/postlbs/routing_core.sql

-- Installing /usr/share/postlbs/routing_core_wrappers.sql

-- Installing /usr/share/postlbs/routing_topology.sql

-- Installing /usr/share/postlbs/matching.sql SELECT assign_vertex_id('ways', 0.00001, 'the_geom', 'gid');

-- Install configuration: ""

-- Install configuration: ""

-- Installing /usr/lib64/pgsql/librouting_tsp.so

-- Install configuration: ""

-- Installing /usr/share/postlbs/routing_tsp.sql

-- Installing /usr/share/postlbs/routing_tsp_wrappers.sql

-- Install configuration: ""

-- Install configuration: ""

-- Installing /usr/lib64/pgsql/librouting_dd.so

-- Install configuration: ""

-- Installing /usr/share/postlbs/routing_dd.sql

-- Installing /usr/share/postlbs/routing_dd_wrappers.sql</td>
</tr>
</tbody></table>
Ya tenemos PgRouting disponible

<strong>OSM2PGROUTING</strong>

Descripción:

Se trata de una utilidad que permite convertir un fichero de OpenStreetMap, en formato OSM o XML, en una base de datos PostgreSQL que tenga instaladas las extensiones PostGIS y PgRouting.

Esta utilidad está específicamente diseñada para OSM, y realiza un especial tratamiento de los datos teniendo en cuenta los tags OSM, de forma tal que es capaz de generar un grafo adecuado para el ruteo. Bien es cierto que tenemos otras posibilidades, como descargar un shapefile de datos OSM de cualquiera de los servidores existentes (Geofabrik, CloudMade) y convertirlos a PostgreSQL con la utilidad “shp2pgsql” provista por PostGIS, sin embargo el resultado no será el mismo. Veamos un ejemplo

Fíjate en esta zona donde tenemos vías que aparecen a distinto nivel: <a href="http://www.openstreetmap.org/?lat=38.379259&amp;lon=-0.452301&amp;zoom=18&amp;layers=B000FTF">http://www.openstreetmap.org/?lat=38.379259&amp;lon=-0.452301&amp;zoom=18&amp;layers=B000FTF</a>

<img class="alignnone size-full wp-image-288" src="http://www.gisandchips.org/wp-content/osm_cruce.png" alt="osm_cruce" width="470" height="509" />

Sí no utilizamos esta utilidad se generará un nodo por cada cruce de vías, independientemente de que si están o no a diferentes niveles. Osm2pgrouting entiende los tags de OSM y evita colocar nodos donde no le corresponde.

Nodos en OpenStreetMap. Las vías es azul contienen el tag bridge=yes, y por tanto están a diferente nivel

<strong><img class="alignnone size-full wp-image-289" src="http://www.gisandchips.org/wp-content/osm_cruce_josm.png" alt="osm_cruce_josm" width="508" height="466" /></strong>

<strong>Compilar osm2pgrouting</strong>

$ svn checkout <a href="http://pgrouting.postlbs.org/svn/pgrouting/tools/osm2pgrouting/trunk">http://pgrouting.postlbs.org/svn/pgrouting/tools/osm2pgrouting/trunk</a> osm2pgrouting

$ cd osm2pgrouting

$ make

Como administrador:

$ make install

<strong>CREAR UNA PLANTILLA DE GEODATABASE CON POSTGIS Y PGROUTING</strong>

Este es quizás el momento más excitante de todos, puesto que vamos a crear una base de datos con datos ya normalizados para ser explotado con las utilidades de PgRouting.

El proceso siempre es el mismo:
<ol>
	<li>Crear la base de datos</li>
	<li>Añadirle la funcionalidad de PostGIS</li>
	<li>Añadirle la funcionalidad de PgRouting</li>
	<li>Cargar datos desde OSM a la base de datos (osm2pgsql)</li>
	<li>Crear topología</li>
	<li>Explotar la base de datos</li>
</ol>
<strong>Creación de una plantilla de base de datos</strong>

Es una buena práctica tener creada una plantilla de base de datos con todas las extensiones ya incluidas para que podamos utilizarla en cualquier geodatabase que creemos.

Puesto que estamos intentando cargar datos desde OSM es conveniente utilizar una codificación de caracteres que albergue todas las posibilidades multilenguaje que se pueden dar (alfabetización árabe, cirílica, etc)
<ol>
	<li>Sín más preámbulos creamos la base de datos</li>
</ol>
Sintaxis:

$ createdb –h &lt;host&gt; –U &lt;user&gt; -p &lt;puerto&gt; -E &lt;código codificación &lt;nombre base de datos&gt;

En nuestro caso este es el resultado

$ createdb -U postgres -E UTF8 routing
<ol>
	<li>PostGIS exige que la base de datos tenga cargado el lenguaje procedural Pl/PgSql. De hecho la gran mayoría de las funciones de PostGIS y PgRouting están en este lenguaje:</li>
</ol>
$ createlang -U postgres plpgsql &lt;nombre base de datos&gt;

$ createlang -U postgres plpgsql routing
<ol>
	<li>Añadir la funcionalidad PostGIS</li>
</ol>
Ahora necesitamos saber donde ha instalado Postgis los ficheros SQL necesarios para crear la geodatabase. Recomendamos utilizar el comando find para encontrarlos

Para las versiones &lt; 1.4

$ find /usr/ -name lwpostgis.sql

Para versiones &gt;= 1.4

$ find /usr/ -name postgis.sql

/usr/share/lwpostgis.sql

Lo mismo para las referencias espacials

$ find /usr/ -name spatial_ref_sys.sql

/usr/share/spatial_ref_sys.sql

Ahora sólo queda cargar el fichero de funcionalidades en la base de datos:

Para la versión 1.3.x

$ psql -U postgres -f /usr/share/lwpostgis.sql routing

$ psql -U postgres -f /usr/share/spatial_ref_sys.sql routing

Para la version 1.4

$ psql -U postgres -f /usr/share/pgsql/contrib/postgis.sql routing

$ psql -U postgres -f /usr/share/pgsql/contrib/spatial_ref_sys.sql routing
<ol>
	<li>Añadir la funcionalidad PgRouting</li>
</ol>
Más de lo mismo. Primero añadimos la funcionalidad básica:

$ psql -U postgres -f /usr/share/postlbs/routing_core.sql routing

$ psql -U postgres -f /usr/share/postlbs/routing_core_wrappers.sql routing

$ psql -U postgres -f /usr/share/postlbs/routing_topology.sql routing
<ol>
	<li>Luego la funcionalidad para el algoritmo TSP (opcional)</li>
</ol>
$ psql -U postgres -f /usr/share/postlbs/routing_tsp.sql routing

$ psql -U postgres -f /usr/share/postlbs/routing_tsp_wrappers.sql routing
<ol>
	<li>Por ultimo la funcionalidad para el algoritmo “Driving Distance”</li>
</ol>
$ psql -U postgres -f /usr/share/postlbs/routing_dd.sql routing

$ psql -U postgres -f /usr/share/postlbs/routing_dd_wrappers.sql routing
<ol>
	<li>Vamos a comprobar que todo está bien.</li>
</ol>
Accedemos a la base de datos creada:

$ psql -U postgres routing

routing=# <strong>select postgis_full_version();</strong>

Devolverá:
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">postgis_full_version----------------------------------------------------------------------------------------POSTGIS="1.4.0" GEOS="3.1.1-CAPI-1.6.0" PROJ="Rel. 4.7.1, 23 September 2009" USE_STATS(1 row)</td>
</tr>
</tbody></table>
<ol>
	<li>Últimos retoques:</li>
</ol>
Con el objetivo de utilizar en el futuro OpenLayers, y con el fin de arbitrar un sistema de proyección que sea común para el uso de varios proveedores de datos (Google Maps, Yahoo Maps, Microsoft Live Maps, etc), vamos a incluir un nuevo registro en la tabla spatial_ref_sys, para añadir la denominada proyección esférica de Mercator, a veces conocida como “proyección Google”, por ser ésta la que se utiliza en en Google Maps. Lo interesante de esta proyección, a diferencia del sistema geodésico mundial con datum WGS84 es que las unidades están en metros, lo que facilita la compresión de los cálculos de distancias, en contraposición a la medición en grados decimales de arco de circunferencia. Más información aquí:

<a href="http://trac.openlayers.org/wiki/SphericalMercator">http://trac.openlayers.org/wiki/SphericalMercator</a>

En este enlace tienes una imagen de gran formato de la tierra con esta proyección: <a href="http://designintelligences.files.wordpress.com/2009/03/mercator-projection1.jpg">http://designintelligences.files.wordpress.com/2009/03/mercator-projection1.jpg</a>

En definitiva, sólo tenemos que añadir este registro:
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">routing=#  INSERT into spatial_ref_sys (srid, auth_name, auth_srid, srtext, proj4text) values (900913 ,'EPSG',900913,'GEOGCS["WGS 84", DATUM["World Geodetic System1984", SPHEROID["WGS 84", 6378137.0, 298.257223563,AUTHORITY["EPSG","7030"]], AUTHORITY["EPSG","6326"]],PRIMEM["Greenwich", 0.0, AUTHORITY["EPSG","8901"]], NIT["degree",0.017453292519943295], AXIS["Longitude", EAST], AXIS["Latitude", NORTH],AUTHORITY["EPSG","4326"]], PROJECTION["Mercator_1SP"],PARAMETER["semi_minor", 6378137.0],PARAMETER["latitude_of_origin",0.0], PARAMETER["central_meridian", 0.0], PARAMETER["scale_factor",1.0], PARAMETER["false_easting", 0.0], PARAMETER["false_northing", 0.0],UNIT["m", 1.0], AXIS["x", EAST], AXIS["y", NORTH],AUTHORITY["EPSG","900913"]] |','+proj=merc +a=6378137 +b=6378137 +lat_ts=0.0 +lon_0=0.0 +x_0=0.0 +y_0=0 +k=1.0 +units=m +nadgrids=@null +no_defs');</td>
</tr>
</tbody></table>
La proyección esférica de Mercator tiene el código 900913, según el European Petroleum Survey Group (EPSG) mientras que OpenStreetMap utiliza como proyección global  predeterminada el geodésico mundial con datum WGS84, cuyo código es 4326.

Ya nos podemos salir de la consola de Postgresql

routing=# \q
<ol>
	<li>Crear una geodatabase a partir de la plantilla creada</li>
</ol>
Bueno ya está hecho todo el trabajo de la plantilla, ahora el proceso de crear una nueva base de datos con todo ya preparado es sólo una línea;

createdb -U &lt;usuario&gt; -T &lt;plantilla&gt; &lt;nueva bd&gt;

En nuestro ejemplo sería

createdb -U postgres -T routing osmtest

<strong>Truco del día:</strong>

Sí tu servidor no tienes un sistema gráfico funcionando y necesitas llevar un fichero desde tu ordenador al servidor utiliza el comando “scp”. Es muy práctico

Sintaxis:

$ scp &lt;fichero de tu ordenador&gt;@&lt;hostname o IP&gt;:&lt;directorio del servidor donde copiar el fichero&gt;

Ejemplo:

$ scp OSM/alicante_30_09_2009.osm jose@www.gisandchips.org:/home/jose/compilar/osm2pgrouting

<strong>DESCARGAR DATOS DE OPENSTREETMAP</strong>

Esto ya va tomando color. Queda lo más bonito. ¡Paciencia!

Tenemos 2 opciones:

a)     Descargar un fichero OSM ya creado en cualquiera de los portales web que hay a tal efecto (Geofabrik (<a href="http://download.geofabrik.de/osm/">http://download.geofabrik.de/osm/</a>) , CloudMade (<a href="http://downloads.cloudmade.com/">http://downloads.cloudmade.com/</a>) , etc)

Ejemplo:

Descargar OSM de España (69 Mb)<span> </span>

$ wget <a href="http://download.geofabrik.de/osm/europe/spain.osm.bz2">http://download.geofabrik.de/osm/europe/spain.osm.bz2</a>

Descomprimir

$ tar -xjvf spain.osm.bz2

a)     Recortar una zona del planet.osm con Osmosis (<a href="http://wiki.openstreetmap.org/wiki/Osmosis">http://wiki.openstreetmap.org/wiki/Osmosis</a>)

$ wget <a href="http://dev.openstreetmap.org/~bretth/osmosis-build/osmosis-latest-bin.tar.gz">http://dev.openstreetmap.org/~bretth/osmosis-build/osmosis-latest-bin.tar.gz</a>

$ tar xzvf osmosis-latest-bin.tar.gz

$ cd osmosis-0.31/ bin/

La sintaxis básica es la siguiente:

$./osmosis --read-xml &lt;fichero osm matriz&gt; --bb left=&lt;coord oeste&gt; right&lt;coord este&gt; top=&lt;coord norte&gt; bottom=&lt;coord sur&gt; --write-xml &lt;nombre fichero a crear extraido de matriz&gt;

Por supuesto las coordenadas deben de estar en grados decimales del WGS84. Pero, ¿de dónde las saco?

Esta vez viene en nuestra ayuda el portal <a href="http://www.openstreetmap.org/">www.openstreetmap.org</a> que viene con una utilidad (pestaña “Exportar”) para proporcionarnos información sobre la caja que estamos visualizando  en el mapa

<img class="alignnone size-full wp-image-287" src="http://www.gisandchips.org/wp-content/osm_box.png" alt="osm_box" width="483" height="343" />

Ejemplo práctico a un barrio de Alicante

$./osmosis --read-xml /home/jose/OSM/alicante_30_09_2009.osm --bb left=-0.4359 right=-0.42006 top=38.37225 bottom=38.36192 --write-xml mibarrio.osm

<strong>IMPORTAR UN OSM EN LA GEODATABASE</strong>

Después de este tortuoso camino, ya queda lo más fácil importar el OSM, utilizando “osm2pgrouting”. La sintaxis es la siguiente:

$ ./osm2pgrouting -file &lt;fichero osm&gt; -conf &lt;fichero configuración&gt;  -dbname &lt;base de datos&gt; -user &lt;usuario&gt;

Ló único extraño aquí es el fichero de configuración, que es un XML donde se incluyen las tipologías de la vías  que queremos cargar a la base de datos (según OSM: motorway, trunk, primary, secondary, tertiary, residential, etc.). En las fuentes aparece un XML de ejemplo que nos puede servir. Debemos de tener en cuenta que en función de nuestros objetivos utilizaremos unas vías u otras. Por ejemplo, sí sólo queremos las grandes vías nos quedaremos con motorway y trunks. Sí sólo nos interesa el tema caminos rurales y rutas ciclistas nos toca trabajar el XML.

Ejemplo:

./osm2pgrouting -file ./alicante_30_09_2009.osm -conf ./mapconfig.xml -dbname osmtest -user postgres

NOTA: Este proceso suele ser bastante largo, y estará en función del tamaño del OSM. Como ejemplo, el área metropolitana de Alicante tardó unos 5 minutos, en un ordenador bien dotado. Toda España nos llevaría unas cuantas horas.

Tras un periodo de tiempo verás este mensaje
<table border="1" cellspacing="0" cellpadding="0" width="100%"><col span="1" width="256"></col>
<tbody>
<tr>
<td width="100%" valign="top">Ways table created
<span style="background-color: #ffffff">Types table created
Classes table created
http://wiki.openstreetmap.org/wiki/Osmosis</span>create topology

#########################

size of streets: 4742

size of splitted ways : 12780

finished</td>
</tr>
</tbody></table>
Para comprobar que tablas se han creado:

Entramos en la base de datos

psql -U postgres osmtest

Listamos las tablas:

osmtest=# \d
<table border="1" cellspacing="0" cellpadding="0" width="1153">
<tbody>
<tr>
<td width="576" valign="top">List of relationsSchema | Name | Type | Owner

--------+------------------+-------+----------

public | classes | table | postgres

public | geometry_columns | table | postgres

public | nodes | table | postgres

public | spatial_ref_sys | table | postgres

public | types | table | postgres

public | ways | table | postgres

(6 rows)</td>
<td width="576" valign="top"></td>
</tr>
</tbody></table>
La tabla que contiene la geometría es ways;
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">osmtest=# select * from geometry_columns ;f_table_catalog | f_table_schema | f_table_name | f_geometry_column | coord_dimension | srid | type

-----------------+----------------+--------------+-------------------+-----------------+------+-----------------

| public | ways | the_geom | 2 | 4326 | MULTILINESTRING

(1 row)

osmtest=# select srid, f_geometry_column as geometria from geometry_columns;

srid | geometria

------+-----------

4326 | the_geom

(1 row)</td>
</tr>
</tbody></table>
<h2>CREACIÓN DE TOPOLOGÍA DE RED</h2>
Si nos fijamos la tabla ways tiene vacíos los campos <em>source</em> y <em>target</em>, que son los identificadores de nodo. Es necesario crear topología. Para ello recurrimos a una función topológica de pgrouting: <em>“assign_vertex_id”</em>

Lo único que tenemos es un identificador de arco (gid)

Para el caso de datos procedentes de OSM, al estar los datos en grados decimales y la proyección en geodésica (4326) la tolerancia a aplicar es muy pequeña. 0.00001

SELECT assign_vertex_id('ways', 0.00001, 'the_geom', 'gid');

Listado de tablas
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">osmtest=# \dList of relationsSchema | Name | Type | Owner

--------+---------------------+----------+----------

public | classes | table | postgres

public | geometry_columns | table | postgres

public | nodes | table | postgres

public | spatial_ref_sys | table | postgres

public | types | table | postgres

public | vertices_tmp | table | postgres

public | vertices_tmp_id_seq | sequence | postgres

public | ways | table | postgres

(8 rows)</td>
</tr>
</tbody></table>
Ahora la tabla ways está preparada para topología.
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">osmtest=# select gid,source,target from ways;gid | source | target

-------+--------+--------

46 | 46 | 47

187 | 190 | 191

213 | 215 | 216

309 | 308 | 309

327 | 325 | 240

447 | 437 | 438

564 | 554 | 555

649 | 635 | 636

711 | 700 | 701

832 | 814 | 815

882 | 863 | 864

927 | 906 | 907</td>
</tr>
</tbody></table>
Con este extenso post queda por finalizada la primera parte sobre PgRouting con OpenStreetMap. Nos esperan dos nuevos artículos que espero sean de vuestro interés:
<ol>
	<li>Explotación de una geodatabase con OpenStreetMap y PgRouting</li>
	<li>Diseño de una interfaz web con OpenLayers para el análisis de redes.</li>
</ol>