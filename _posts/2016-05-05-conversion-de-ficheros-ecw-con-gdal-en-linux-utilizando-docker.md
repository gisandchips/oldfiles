---
ID: 2928
post_title: >
  Conversión de ficheros ECW con GDAL en
  Linux utilizando docker
author: jose
post_date: 2016-05-05 13:41:02
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2016/05/05/conversion-de-ficheros-ecw-con-gdal-en-linux-utilizando-docker/
published: true
novalite_template:
  - full
---
Seguramente alguna vez os habéis visto en la tesitura de estar trabajando en Linux y deseas utilizar ortofotos, o mapas procedente de instituciones como el IGN, o de institutos cartográficos autonómicos (en mi caso el ICV).

Generalmente el formato raster más utilizado para ortos y mapas suele ser el ECW, el formato de compresión creado entonces por ERDAS y que utiliza el algoritmo de compresión Wavelet (el mismo que el JP2).
<a href="http://www.gisandchips.org/wp-content//ign6.jpg" rel="attachment wp-att-2954"><img class="aligncenter wp-image-2954 size-large" src="http://www.gisandchips.org/wp-content//ign6-449x304.jpg" alt="ign6" width="449" height="304" /></a>
<!--more-->

Este formato puede utilizarse en casi todos los GIS desktop que quieras, libres o privativos, sobre todo si trabajas con Windows. Sin embargo el problema se acentúa sí utilizas Linux. Aquí el panorama cambia, y los programas deben de ser compilados de forma implícita con esta librería, pero que no es lo habitual. Para compilarlo se utiliza una librería desfasada (2013) y difícil de encontrar (aquí → <a href="http://downloada.erdas.com/software/2013/ECWJP2SDK/erdas-ecwjp2sdk-v5.0-linux.zip">http://downloada.erdas.com/software/2013/ECWJP2SDK/erdas-ecwjp2sdk-v5.0-linux.zip</a> ). Hay algunos programas, como es el caso de GvSIG que lo suelen incluir, pero otros como Gdal o Qgis, mi favorito, ni lo incluyen. Es necesario compilarlo y la verdad es que suele dar bastante dolores de cabeza. Si quieres intentar compilarlo para Qgis <a href="http://makina-corpus.com/blog/metier/2013/howto-install-qgis-with-ecw-support-on-linux">aquí</a> tienes una ayuda

OBJETIVO:

La idea original era descargar varias ortos y mapas del IGN en el formato ECW de una zona concreta utilizando el “Centro de descargas” del CNIG, para luego convertirlos a otro formato más estándar, aun a costa de sacrificar parte de mi disco duro. Me planteé convertirlos a GeoTIFF, y luego a JPEG pero con fichero de georreferenciación (WLD). Finalmente creé un PDF con las imágenes para su impresión

<strong>FASE 1: DESCARGA DE DATOS:</strong>

En mi caso necesitaba obtener las ortos del municipio de Alicante, que es bastante extenso y cubre varias hojas del MTN50.

<a href="http://www.gisandchips.org/wp-content//ign1.png" rel="attachment wp-att-2931"><img class="aligncenter wp-image-2931 size-large" src="http://www.gisandchips.org/wp-content//ign1-449x304.png" alt="ign1" width="449" height="304" /></a>

Pero el IGN proporciona la posibilidad de descargarlo en un proceso por lotes utilizando Java Web Start. En el caso de Ubuntu necesitas instalar este paquete: <b>icedtea-netx</b>

<code>sudo apt-get install icedtea-netx</code>

Una vez que iniciamos la descarga, se abre un fichero en formato JNLP (Java Network Launching Protocol)

<a href="http://www.gisandchips.org/wp-content//ign2.png" rel="attachment wp-att-2932"><img class="aligncenter wp-image-2932 size-large" src="http://www.gisandchips.org/wp-content//ign2-449x299.png" alt="ign2" width="449" height="299" /></a>

Luego verás como se ejecuta el applet:

<a href="http://www.gisandchips.org/wp-content//ign3.png" rel="attachment wp-att-2933"><img class="aligncenter wp-image-2933 size-large" src="http://www.gisandchips.org/wp-content//ign3-449x304.png" alt="ign3" width="449" height="304" /></a>

Este sistema nos ahorra estar delante del ordenador varias horas esperando la descarga de forma manual.

De esta forma obtendrás un conjunto de archivos en formato ZIP que deberás descomprimir. Esto se lleva su tiempo puesto que los archivos ECW suelen ser muy pesados (la mayoría por encima del giga). Creará una carpeta por cada ZIP

<strong>FASE 2: CONVERSIÓN DE DATOS</strong>

Una vez descargados y descomprimidos los ECW me dispuse a convertirlos a GeoTIFF., utilizando GDAL y automatizando el proceso mediante un script bash. Es aquí donde en vez de compilar GDAL hice uso de un docker realizado por Klokantech.com optimizado para trabajar con este formato. Sí estás familiarizado con los dockers la instalación te resultará muy facil, y si no busca información en la red.

Aquí tienes unos enlaces de como utilizar este docker
<ul>
	<li><a href="http://blog.klokantech.com/2015/02/gdal-in-docker-install-run-with-single.html">http://blog.klokantech.com/2015/02/gdal-in-docker-install-run-with-single.html</a></li>
	<li><a href="https://hub.docker.com/r/klokantech/gdal/">https://hub.docker.com/r/klokantech/gdal/</a></li>
</ul>
Para ver cuales son los formatos soportados por este GDAL del docker utiliza este comando:

<code>docker run -ti --rm -v $(pwd):/data klokantech/gdal gdalinfo --formats</code>

Sí quieres filtrar para ver solo los referidos a ECW utiliza también el comando “grep”
<code>sudo docker run -ti --rm -v $(pwd):/data klokantech/gdal gdalinfo --formats | grep ECW </code>

Obtendrás:
<code>ECW (rw): ERDAS Compressed Wavelets (SDK 3.x)
JP2ECW (rw+v): ERDAS JPEG2000 (SDK 3.x)
</code>

Puedes entrar en el bash del docker de esta manera

<code>docker run -ti --rm -v $(pwd):/data klokantech/gdal /bin/bash</code>

y luego ejecutar cualquiera de los comandos de GDAL

O bien lo puedes invocar directamente. Ejemplo con información de un fichero con “gdalinfo”

<code>docker run -ti --rm -v $(pwd):/data klokantech/gdal gdalinfo fichero.ecw</code>

Obtendrás:
<code>Driver: ECW/ERDAS Compressed Wavelets (SDK 3.x)
Files: Documentos/IGN-MTN/MTN25/MTN25-0893c1-2015-Elche_Elx/MTN25-0893c1-2015-Elche_Elx.ecw
Size is 19533, 10435
Coordinate System is:
GEOGCS["ETRS89",
DATUM["European_Terrestrial_Reference_System_1989",
SPHEROID["GRS 1980",6378137,298.257222101,
AUTHORITY["EPSG","7019"]],
AUTHORITY["EPSG","6258"]],
PRIMEM["Greenwich",0,
AUTHORITY["EPSG","8901"]],
UNIT["degree",0.01745329251994328,
AUTHORITY["EPSG","9122"]],
AUTHORITY["EPSG","4258"]]
Origin = (-0.865912443070925,38.344086734392704)
Pixel Size = (0.000011276676452,-0.000011276676452)
Metadata:
COLORSPACE=RGB
COMPRESSION_RATE_TARGET=10
VERSION=2
Corner Coordinates:
Upper Left ( -0.8659124, 38.3440867) ( 0d51'57.28"W, 38d20'38.71"N)
Lower Left ( -0.8659124, 38.2264146) ( 0d51'57.28"W, 38d13'35.09"N)
Upper Right ( -0.6456451, 38.3440867) ( 0d38'44.32"W, 38d20'38.71"N)
Lower Right ( -0.6456451, 38.2264146) ( 0d38'44.32"W, 38d13'35.09"N)
Center ( -0.7557788, 38.2852507) ( 0d45'20.80"W, 38d17' 6.90"N)
Band 1 Block=256x256 Type=Byte, ColorInterp=Red
Description = Red
Overviews: 9766x5217, 4883x2608, 2441x1304, 1220x652, 610x326, 305x163
Band 2 Block=256x256 Type=Byte, ColorInterp=Green
Description = Green
Overviews: 9766x5217, 4883x2608, 2441x1304, 1220x652, 610x326, 305x163
Band 3 Block=256x256 Type=Byte, ColorInterp=Blue
Description = Blue
Overviews: 9766x5217, 4883x2608, 2441x1304, 1220x652, 610x326, 305x163
</code>

Para convertir un ECW a GeoTIFF se puede utilizar el comando “gdal_translate”. Ver ejemplo:
<code>gdal_translate -a_srs EPSG:4230 -of GTiff &lt;fichero_origen&gt; &lt;fichero destino&gt; </code>

Ejemplo:

<code>docker run -ti --rm -v $(pwd):/data klokantech/gdal gdal_translate -a_srs EPSG:4230 -of GTiff orto.ecw orto.tif</code>

En el caso de las ortos este proceso nos puede llevar horas. No obstante mi objetivo era automatizar tareas así que cree un script en bash que se metiese en cada directorio y convirtiese el archivo

Este es el script:
<code>#!/bin/bash
# conversor de ECW a TIF
for f in `find /data/Documentos/IGN-MTN/MTN50 -name *.ecw`
do
echo "Convirtiendo $f"
OUTFILE=$(basename "$f" .ecw)
gdal_translate -a_srs EPSG:4230 -of GTiff $f "$OUTFILE".tif
done
</code>

OJO con el uso del comando “find”. Utiliza las comillas de apertura y cierre.

Al cabo de un buen rato ya tenemos los fichero en formato GeoTIFF para verlos en cualquier GIS. Ten en cuenta que los TIFF son archivos sin compresión, por lo que su tamaño suele ser desmesurado. Compara el tamaño de uno de los ejemplos
<ul>
	<li>original ECW: 1,3 Gb</li>
	<li>original convertido a TIFF: <strong>27 Gb</strong></li>
</ul>
También podemos convertir los TIF creados a JPEG con una ligera modificación

<code>#!/bin/bash
# conversor de TIF a JPEG
for f in `find /data/MTN25-TIF -name *.tif`
do
echo "Convirtiendo $f"
OUTFILE=$(basename "$f" .tif)
gdal_translate -a_srs EPSG:4230 -of JPEG -scale -co worldfile=yes $f "$OUTFILE".jpg
done</code>

Captura en Qgis
<a href="http://www.gisandchips.org/wp-content//ign4.png" rel="attachment wp-att-2934"><img class="aligncenter wp-image-2934 size-large" src="http://www.gisandchips.org/wp-content//ign4-449x304.png" alt="ign4" width="449" height="304" /></a>

<strong>FASE 3: CAMBIO DE DENSIDAD</strong>

Sí te descargas mapas (en el PNOA no ocurre lo mismo), en cada carpeta hay un fichero LEEME.TXT con este contenido:
<code>El archivo zip contiene 3 tipos de archivos:
*.jpg: archivo sin georreferenciar prodedente del escaneado de los fondos de la Cartoteca del Instituto Geográfico Nacional. Resolución 250 ppp.
*.ecw archivo georreferenciado a partir del jpg anterior, en el sistema geodésico original del mapa y en coordenadas geográficas (longitud y latitud) sin proyección y a 400 ppp.
*.prj archivo auxiliar con los datos de georreferenciación que algunos programas necesitan para la correcta interpretación del ecw.
</code>
Como ves podríamos haber utilizado el fichero JPEG, junto con el PRJ, pero la resolución es distinta, 250 ppp frente a los 400 del ECW)

Desconozco la razón, pero los TIF creados con GDAL a partir del ECW tienen una densidad menor, por lo que es necesario cambiarla . Para ello utilizo (fuera del docker) la utilidad “<strong>convert</strong>” que forma parte del conjunto de utilidades <em>Imagemagick</em>

La sintaxis es la siguiente:

<code>convert -verbose -density 400 &lt;fichero_origen&gt; &lt;fichero destino&gt;</code>

Desde la terminal también podemos lanzar esta sentencia para aplicarlo a todo un directorio:

<code>for file in *.jpg; do convert -verbose -density 400 $file $file; done</code>

Des esta forma obtendrás ficheros listos para imprimir con una calidad más que aceptable

Captura de pantalla con fichero en GIMP

<a href="http://www.gisandchips.org/wp-content//ign5.png" rel="attachment wp-att-2935"><img class="aligncenter wp-image-2935 size-large" src="http://www.gisandchips.org/wp-content//ign5-449x304.png" alt="ign5" width="449" height="304" /></a>

<strong>FASE 4: PREPARANDO FICHERO PARA IMPRESIÓN</strong>

Sí tienes que crear un archivo de impresión en formato PDF con todos tus imágenes de mapas para llevarlo que lo impriman puedes utilizar de nuevo la utilidad de “convert”

<code>convert *.jpg archivo.pdf</code>

Eso es todo amigos