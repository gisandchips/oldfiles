---
ID: 2668
post_title: >
  Generar tiles a partir de un ráster sin
  georeferenciar
author: Jorge Piera Llodrá
post_date: 2013-11-13 21:00:21
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2013/11/13/generar-tiles-a-partir-de-un-raster-sin-georeferenciar/
published: true
---
En estos momentos disponemos una aplicación en la que los clientes tienen que subir un mapa en formato ráster (normalmente jpg o png) y tienen además que añadir 3 puntos para calibrar el mapa estableciendo una correspondencia entre píxels de la imagen y sus respectivas coordenadas geográficas.

Este mapa se puede visualizar fácilmente en una aplicación de escritorio pero hoy he tenido la necesidad de cargarlo en una aplicación Android que utiliza <a href="https://developers.google.com/maps/documentation/android/">Google Maps</a>. Inicialmente he tratado de cargarla como una imagen completa pero he obtenido la archiconocida "MemoryOutofException".

La solución a este problema ha sido crear en el servidor un servicio de tiles a partir de la imagen ráster y de los 3 puntos de georeferenciación que ha introducido el usuario. Esta tarea se puede realizar fácilmente con diversos programas de escritorio pero en mi caso uno de los requisitos es que se tuviera que hacer automáticamente cada vez que el usuario cambie la imagen o modifique un punto de control. Estos son los pasos que he seguido:

<!--more--><strong><span style="font-size: 14px">1. Georeferenciar la imagen</span></strong>

Primero necesitaba una imagen georeferenciada ya que los programas que crean los tiles necesitan saber la ubicación de la imagen. Para ello he utilizado el comando <a href="http://www.gdal.org/gdal_translate.html">gdal_translate</a> de la libraría <a href="http://www.gdal.org/">GDAL</a>.  Suponiendo que los puntos de calibración de mi imagen son X1=lon1, Y1=lat1, X2=lon2, Y2=lat2, X3=lon3 y Y3=lat3 el comando que he tenido que ejecutar es:

[shell]
gdal_translate -gcp X1 Y1 lon1 lat1 -gcp X2 Y2 lon2 lat2 -gcp X3 Y3 lon3 lat3 -a_srs epsg:4326 input.jpg output.tiff
[/shell]

A continuación debemos ejecutar el comando <a href="http://www.gdal.org/gdalwarp.html">gdalwrap</a> que procesará los puntos de calibración y generará una imagen de salida georeferenciada que usaremos para crear los tiles:

[shell]
gdalwarp -s_srs epsg:4326 -t_srs epsg:4326 output.tiff final.out
[/shell]

<em>output.tiff</em> es la imagen generada con el comando anterior y <em>final.out</em> es la imagen de salida georeferenciada.

<strong><span style="font-size: 14px">2. Generación de los tiles</span></strong>
Primero hay que decidir los niveles de zoom que tenemos que crear. En mi caso, he optado por crear niveles del 12 al 17. Para crear los tiles, he usado el comando <a href="http://www.gdal.org/gdal2tiles.html">gdal2tiles</a>:

[bash]
gdal2tiles.py --zoom 12-17 final.out
[/bash]

La ejecución de este comando ha generado una carpeta de salida llamada <em>final</em> que contiene los tiles. El siguiente paso ha sido publicar la carpeta generada en un servidor web para que esté accesible desde Internet mediante la dirección http://www.myserver.com/tiles/
<strong><span style="font-size: 14px">3. Carga de los tiles en Android</span></strong>

Una vez que que los tiles ya están subidos vamos a añadir la capa a la aplicación Android que utiliza Google Maps. Suponiendo que tenemos un objeto de tipo <a href="http://developer.android.com/reference/com/google/android/gms/maps/GoogleMap.html">GoogleMap</a>, sólo hay que añadirle un <a href="http://developer.android.com/reference/com/google/android/gms/maps/model/UrlTileProvider.html">UrlTileProvider</a> del siguiente modo:

[java]

TileProvider tileProvider = new UrlTileProvider(256, 256) {
  @Override
  public URL getTileUrl(int x, int y, int zoom) {
    int newY = (int)Math.pow(2, zoom);
    newY = newY - y - 1;
    String url = &quot;http://www.myserver.com/tiles/&quot; + zoom + &quot;/&quot; + x + &quot;/&quot; + newY + &quot;.png&quot;;
    Log.i(&quot;Tiles&quot;, url);
    try {
      return new URL(url);
    } catch (MalformedURLException e) {
      Log.e(&quot;Tiles&quot;, &quot;Error loading the tile&quot;, e);
      return null;
     }
  }
};
map.addTileOverlay(new TileOverlayOptions().tileProvider(tileProvider));
[/java]



Donde los valores 256,256 pasados en el constructor son el tamaño de los tiles. Los parámetros <em>zoom, x</em> e <em>y</em> pasados en el método <em>getTileUrl</em> tienen una conversión casi directa en el sistema de ficheros del servidor salvo en el caso de la <em>y </em>que hay que hacer una mínima transformación.

<strong>NOTA</strong>: después de pasarme medio día investigando cómo resolver este problema y después de de escribir este post he encontrado <a href="https://developers.google.com/kml/articles/raster">un enlace</a> que explica los mismo que he contado aquí yo pero con más detalle.