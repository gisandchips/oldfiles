---
ID: 1551
post_title: >
  API de Google + OpenStreetMap y otros
  servicios georreferenciados
author: pepe
post_date: 2010-03-24 12:22:00
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2010/03/24/api-de-google-openstreetmap-y-otros-servicios-georreferenciados/
published: true
novalite_template:
  - full
---
Voy a tratar de diseñar un visor de cartografía utilizando el <a href="http://code.google.com/intl/es/apis/maps/" target="_blank"><strong>API </strong>de <strong>Google Maps</strong></a> ( si quereis saber más sobre este tema, podeis leer artículos en esta misma web que hacen referencia:  <a href="http://www.gisandchips.org/2009/11/20/google-web-toolkit-google-maps/" target="_blank">Google Web Toolkit &amp; Google Maps</a><a title="Google Web Toolkit &amp; Google Maps" href="../2009/11/20/google-web-toolkit-google-maps/">,</a><a title="Integración de un servicio WMS en GoogleMaps usando GWT" href="http://www.gisandchips.org/2010/01/18/integracion-de-un-servicio-wms-en-googlemaps-usando-gwt/" target="_self"> Integración de un servicio WMS en GoogleMaps usando GWT </a>)  con la cartografía de <a href="http://www.openstreetmap.org" target="_blank"><strong>OpenStreetMap</strong></a> y además voy a incluir datos de servicios de tipo georeferenciado (<a href="http://www.panoramio.com" target="_blank"><strong>panoramio</strong></a>, <a href="http://www.youtube.com" target="_blank"><strong>youtube</strong></a>, <a href="http://www.flickr.com" target="_blank"><strong>flickr</strong></a>, <strong>kml</strong>, <a href="http://es.wikipedia.org" target="_blank"><strong>wikipedia</strong></a>,...) en dicho mapa.

<a href="http://www.gisandchips.org/wp-content/mapa.png"><img class="alignleft size-medium wp-image-1567" style="margin-left: 5px; margin-right: 5px; border: 0pt none;" src="http://www.gisandchips.org/wp-content/mapa-300x170.png" alt="Muestra un curioso mapa mundial lleno de etiquetas de los servicios georreferenciados" width="300" height="170" /></a>Ya aviso que, una vez que muestro todos los servicios en el mapa,  queda un mapa demasiado lleno de elementos, pero eso  es justo lo que pretendo con este visor, tener todos los elementos en el mismo mapa y que se visualicen tanto fotos, videos, artículos de la wikipedia,kml,... a la vez, permitiéndo incluso añadir muchos más elementos.

En primer lugar, para mostrar el mapa de <strong><a href="http://www.archivosdeprograma.es/google-drive-en-wordpress/" target="_blank">Google</a> Maps</strong>, necesito registrar la clave previamente para el sitio web en cuestion que voy a tratar.

<!--more-->

Y una vez registrada la clave ya se puede realizar el visor,  la programación del visor sería la siguiente:

[php]
//Hojas de estilo de google maps
 &lt;link rel=&quot;stylesheet&quot; type=&quot;text/css&quot; href=&quot;http://ajax.googleapis.com/ajax/libs/yui/2.7.0/build/reset-fonts-grids/reset-fonts-grids.css&quot;&gt;
 &lt;link rel=&quot;stylesheet&quot; type=&quot;text/css&quot; href=&quot;http://ajax.googleapis.com/ajax/libs/yui/2.7.0/build/base/base-min.css&quot;&gt;
 &lt;script type=&quot;text/javascript&quot; src=&quot;http://maps.google.com/maps?file=api&amp;amp;v=2&amp;amp;key=clave_registrada_en_google&quot;&gt;&lt;/script&gt;

&lt;script
type=&quot;text/javascript&quot; charset=&quot;utf-8&quot;&gt;
 google.load(&quot;maps&quot;, &quot;2.x&quot;);

 function load()
 {
 if (!GBrowserIsCompatible())
 return;

 //definicion de variable de copyright del mapa
 var copyOSM = new GCopyrightCollection(&quot;GIS&amp;Chips 2010&quot;);
 copyOSM.addCopyright(new GCopyright(1, new GLatLngBounds(new GLatLng(-90,-180), new GLatLng(90,180)), 0, &quot; &quot;));

 //definicion de las tiles de OpenStreetMap que iran por bajo del mapa
 var tilesMapnik     = new GTileLayer(copyOSM, 1, 17, {tileUrlTemplate: 'http://tile.openstreetmap.org/{Z}/{X}/{Y}.png'});
 var tilesOsmarender = new GTileLayer(copyOSM, 1, 17, {tileUrlTemplate: 'http://tah.openstreetmap.org/Tiles/tile/{Z}/{X}/{Y}.png'});
 var tilesCycle = new GTileLayer(copyOSM, 1, 17, {tileUrlTemplate: 'http://a.andy.sandbox.cloudmade.com/tiles/cycle/{Z}/{X}/{Y}.png'});

 var mapMapnik     = new GMapType([tilesMapnik],     G_NORMAL_MAP.getProjection(), &quot;Mapnik&quot;);
 var mapOsmarender = new GMapType([tilesOsmarender], G_NORMAL_MAP.getProjection(), &quot;Osmarend&quot;);
 var mapCycleMap = new GMapType([tilesCycle], G_NORMAL_MAP.getProjection(), &quot;CycleMap&quot;);

 //definicion de mapa incluyendo los tres tiles de OpenStreetMap

 var map           = new GMap2(document.getElementById(&quot;map&quot;), { mapTypes: [mapMapnik, mapOsmarender,mapCycleMap] });

 //se añaden los controles de mapa del API de Google Maps
 map.addControl(new GLargeMapControl()); //control de desplazamiento y acercamiento de gran tamaño empleado en  Google Maps.
 map.addControl(new GMapTypeControl());  //Permite alternar entre los diferentes tipos de mapas (en este caso las tres tiles)
 map.addControl(new GOverviewMapControl()); //Este control nos permite mostrar una ventana de vista rápida contraible

//Situado el centro, las coordenadas de la Universidad de Alicante
 map.setCenter( new GLatLng(38.38575, -0.51486), 16);

 map.enableScrollWheelZoom();
 map.enableContinuousZoom();

 //Se definen como GLayers los servicios georeferenciados que quiero incluir

 var wikipedia = new GLayer(&quot;org.wikipedia.es&quot;);
 var panoramio = new GLayer(&quot;com.panoramio.all&quot;);
 var youtube = new GLayer(&quot;com.youtube.all&quot;);
 var webcam = new GLayer(&quot;com.google.webcams&quot;);

 map.addOverlay(wikipedia);
 map.addOverlay(panoramio);
 map.addOverlay(youtube);
 map.addOverlay(webcam);

 //En este caso como los servicios son un xml se añaden como GGeoXML(tanto el api de flickr como cualquier KML que quiera incluir lo debo definir as�)
 var geoXmlFlickr;
 geoXmlFlickr = new GGeoXml(&quot;http://api.flickr.com/services/feeds/geo/?tags=universidad+alicante&quot;);
 map.addOverlay(geoXmlFlickr);

//Ejemplo de introducción de un KML
 var tram = new GGeoXml(&quot;http://www.sigua.ua.es/web/utils/acceso/kml/TRAM4.kml&quot;);
 map.addOverlay(tram);
 }
[/php]

<p style="text-align: center;">Aqui se puede ver como se muestra en el mapa el KML importado
<a href="http://www.gisandchips.org/wp-content/kml.png"><img class="size-medium wp-image-1560 aligncenter" src="http://www.gisandchips.org/wp-content/kml-300x246.png" alt="Se muestra como queda la integracion de un archivo kml con información sobre una linea de tranvia en Alicante en el mapa anteriormente desarrollado" width="300" height="246" /></a></p>


[html]

 &lt;/script&gt;

 &lt;/head&gt;

 &lt;body onload=&quot;load()&quot; onunload=&quot;GUnload()&quot;&gt;
 &lt;div&gt;
 &lt;div role=&quot;main&quot;&gt;
 &lt;div&gt;
 &lt;div id=&quot;map&quot; style=&quot;width: 100%; height: 600px;&quot;&gt;&lt;/div&gt;

 &lt;/div&gt;
 &lt;/div&gt;
 &lt;/div&gt;

[/html]

El visor completo con todos los datos estaría accesible desde la siguiente URL:
<a class="aligncenter" title="Mapa " href="http://www.gisandchips.org/demos/pepe/osm/test.html" target="_blank">http://www.gisandchips.org/demos/pepe/osm/test.html</a>

Como digo, este mapa incluye tantos elementos que a veces es un poco extraño, pero es justo lo que pretendía, incluir muchos servicios en un mismo mapa.