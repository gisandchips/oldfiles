---
ID: 1601
post_title: OpenLayers y Panoramio
author: jose
post_date: 2010-05-04 11:59:47
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2010/05/04/openlayers-y-panoramio/
published: true
---
Tras leer el excelente <a href="/2010/03/24/api-de-google-openstreetmap-y-otros-servicios-georreferenciados/">post</a> de <a href="http://www.gisandchips.org/author/pepe/">Pepe</a> relativo a la integración de servicios de geo-localización en un ambiente <a href="/2010/03/24/api-de-google-openstreetmap-y-otros-servicios-georreferenciados/">Google Maps</a> llegué a la conclusión de que resulta muy fácil aplicarlo con dicha API, pero mi inquietud me llevó a investigar en la posibilidad de integrarlo en otra API de tipo mapping de reconocido prestigio, como es el caso de <a href="http://openlayers.org">OpenLayers</a>, que además es 100% Open Source, lo que nos permite añadir múltiples fuentes de datos (wms, wfs, gml, xml, geojson, georss, conjunto de tiles, proveedores de mapa mundiales -OpenStreetMap, GMaps, Virtual Earth, Yahoo Maps, etc.), además de esa libertad de movimientos que le confieren una cierta ventaja sobre su homónima, sin que ello suponga una crítica hacia Google Maps, que entre otras virtudes ha popularizado el uso de los servicios de mapa vía web, acercándolo a un público genérico sin apenas conocimientos.
[caption id="attachment_1618" align="aligncenter" width="486" caption="Openlayers con Panoramio"]<a href="http://www.gisandchips.org/demos/j3m/panoramio/panoramio.html"><img src="http://www.gisandchips.org/wp-content/captura_panoramio2.png" alt="Openlayers con Panoramio" width="486" height="228" class="size-full wp-image-1618" /></a>[/caption]
<!--more-->

A la hora de "copiar" el ejemplo descrito en dicho post para OpenLayer, no he planteado integrar todos los geoservicios, entre otras cosas porque algunos de ellos forman parte de la API casi desde su inicio, y no supone un gran esfuerzo incorporarlo, salvo el de YouTube, que no tuve tiempo de ver, aunque tampoco creo que haya muchos videos geoposicionados. Ejemplos de integración de KML en OL hay muchos en la red, al igual que de GML, GeoJSON o GeoRSS, y también de Flickr, que en el fondo es acceder a un documento XML. Para más información consultas los <a href="http://dev.openlayers.org/releases/OpenLayers-2.9/examples/">ejemplos</a> on-line del portal Openlayers.org

De todos los geoservicios, al que más atención ha prestado ha sido al de <a href="http://www.panoramio.com">Panoramio</a>, entre otras cosas por la ingente cantidad de fotografías georreferenciadas que integra, lo que lo hace muy atractivo para el webmaster. Como muchos sabéis este portal dispone de un <a href="http://www.panoramio.com/api/">servicio REST</a> público que nos permite ejecutar una URL con ciertos parámetros, que finalmente nos devuelve un documento estructurado en formato JSON
<pre>
{
"count": 110,
"photos": [
{
"upload_date": "20 May 2007",
"owner_name": "Carlos Sieiro del Nido",
"photo_id": 2313090,
"longitude": -0.516679,
"height": 75,
"width": 100,
"photo_title": "UNIVERSIDAD DE ALICANTE-Campus de San Vicente del Raspeig.",
"latitude": 38.386611000000002,
"owner_url": "http://www.panoramio.com/user/134716",
"owner_id": 134716,
"photo_file_url": "http://mw2.google.com/mw-panoramio/photos/thumbnail/2313090.jpg",
"photo_url": "http://www.panoramio.com/photo/2313090"
},
{
"upload_date": "16 January 2007",
"owner_name": "\u00a9 www.fotoseb.es - Sebastien Pigneur Jans",
"photo_id": 453228,
"longitude": -0.51438300000000003,
"height": 66,
"width": 100,
"photo_title": "Foto Aerea Universidad Alicante 2 (Foto_Seb)",
"latitude": 38.381700000000002,
"owner_url": "http://www.panoramio.com/user/55833",
"owner_id": 55833,
"photo_file_url": "http://mw2.google.com/mw-panoramio/photos/thumbnail/453228.jpg",
"photo_url": "http://www.panoramio.com/photo/453228"
}]</pre>

}

Como podéis ver por cada foto disponemos de un array con los datos que nos interesan, y de forma particular los siguientes:
<ul>
	<li> <strong>owner_name</strong>: autor</li>
	<li> <strong>photo_title</strong>: título</li>
	<li> <strong>latitude</strong> y <strong>longitude</strong>: coordenadas geográficas de geoposicionamiento en datum WGS 84</li>
	<li> <strong>photo_id</strong>: identificador de foto</li>
	<li> <strong>photo_file_url</strong>: URL de la imagen</li>
	<li> <strong>photo_url</strong>: URL del recurso en la web de Panoramio</li>
</ul>
Integrar Panoramio en OpenLayers no resulta tan elegante como en GM, donde sólo hay que escribir 2 líneas, pero con un poco de esfuerzo y el apoyo de la comunidad seguro que alguien es capaz de escribir una <a href="http://docs.openlayers.org/library/formats.html#creating-custom-formats">clase derivada</a> de JSON que facilite las cosas. Por ahora me limitaré a desmenuzar el código de forma secuencial para que se entienda el proceso

Lo primero que debemos de hacer es preparar nuestro proxy para que admita peticiones al host de Panoramio.  En este <a href="http://trac.openlayers.org/wiki/FrequentlyAskedQuestions#ProxyHost">enlace</a> explica como hacerlo

[javascript]
OpenLayers.ProxyHost= &quot;/cgi-bin/proxy.cgi?url=&quot;;
[/javascript]

Posteriormente creamos un par de variables con la URL del servicio REST y los parámetros

[javascript]
url = &quot;http://www.panoramio.com/map/get_panoramas.php&quot;;
parametros = {
   order:'popularity',
   set:'full',
   from:0,
   to:20,
   minx: minx,
   miny: miny,
   maxx: maxx,
   maxy: maxy,
   size:'thumbnail'
}
[/javascript]

Consulta la documentación de la API de Panoramio para más información. Por ahora los parámetros que más nos interesan son:
<ul>
	<li> “<strong>to</strong>”: indica el nº de fotografías que queremos cargar. Máximo 50</li>
	<li> “<strong>minx</strong>”,”<strong>miny</strong>”,”<strong>maxx</strong>”,”<strong>maxy</strong>”: coordenadas de la BBOX donde buscará fotos. Debén de estar en coordenadas geográficas (EPSG: 4326). En mi caso los he obtenido utilizando este código:</li>
</ul>
[javascript]
// Obtener extensiones.
ext = map.getExtent();
var minx = ext.left;
var miny = ext.bottom;
var maxx = ext.right;
var maxy = ext.top; //alert (minx + &quot; &quot; + miny + &quot; &quot; + maxx + &quot; &quot; + maxy);
[/javascript]

Por último sólo nos queda invocar a un HttpRequest de AJAX que ejecute la URL con los parámetros y los envíe a nuestra función: mostrarfotos

[javascript]
OpenLayers.loadURL(url, parametros, this, mostrarfotos);
[/javascript]

<strong>Función mostrarfotos</strong>
Es la encargada de procesar el documento JSON que recibe como argumento

[javascript]
function mostrarfotos(response) {
[/javascript]

El documento JSON, que todavía es una cadena de texto es parseada con OpenLayers.Format.JSON, que la convierte en un objeto que ya podemos tratar:

[javascript]
var json = new OpenLayers.Format.JSON();
var panoramio = json.read(response.responseText);
[/javascript]

Creamos un array de featutes con el total de fotos  (panoramio.photos.length)

[javascript]
var features = new Array(panoramio.photos.length);
[/javascript]

Aplicamos un bucle “for” para recorrer cada fotografía.

[javascript]
for (var i = 0; i &lt; panoramio.photos.length; i++)
{
[/javascript]

Declaramos y almacenamos cada dato de la foto en una variable para recorrerla con comodidad.

[javascript]
var upload_date = panoramio.photos[i].upload_date;
var owner_name = panoramio.photos[i].owner_name;
var photo_id = panoramio.photos[i].photo_id;
var longitude =panoramio.photos[i].longitude;
var latitude = panoramio.photos[i].latitude;
var pheight = panoramio.photos[i].height;
var pwidth = panoramio.photos[i].width;
var photo_title = panoramio.photos[i].photo_title;/
var owner_url = panoramio.photos[i].owner_url;
var owner_id = panoramio.photos[i].owner_id;
var photo_file_url = panoramio.photos[i].photo_file_url;
var photo_url = panoramio.photos[i].photo_url;
[/javascript]

Opcionalmente podemos recoger todas las fotografías en código HTML para ser ubicadas en el BODY con un DIV, en forma de “galería de fotos”

[javascript]
OpenLayers.Util.getElement('galeria').innerHTML +=
     &quot;&lt;a href='&quot;+ photo_url +
     &quot;'&gt;&lt;img src='&quot;+photo_file_url+&quot;' title='&quot;+
     photo_title +&quot;' /&gt;&lt;/a&gt;&amp;nbsp;&quot;;
[/javascript]

Posteriormente en el BODY para representar esta galería sólo hay que escribir:

[html]
&lt;div id=&quot;galeria&quot;&gt;&lt;/div&gt;
[/html]

Creamos un objeto point con las coordenadas de la foto

[javascript]
var fpoint = new OpenLayers.Geometry.Point(longitude,latitude);
[/javascript]

Creo un array de atributos

[javascript]
var atributos = {
   'upload_date' : upload_date,
   'owner_name':owner_name,
   'photo_id':photo_id,
   'longitude':longitude,
   'latitude':latitude,
   'pheight':pheight,
   'pwidth':pwidth,
   'pheight':pheight,
   'photo_title':photo_title,
   'owner_url':owner_url,
   'owner_id':owner_id,
   'photo_file_url':photo_file_url,
   'photo_url':photo_url
}
[/javascript]

Por cada foto creo un elemento (feature) que contiene su posición y sus atributos

[javascript]
features[i] = new OpenLayers.Feature.Vector(fpoint,atributos);
[/javascript]

Terminamos el bucle

[javascript]
}
[/javascript]

Fuera del bucle, todavía en la función mostrar foto nos queda:

Definir el estilo para representar la foto. Tenemos dos opciones
a) Crear un icono único para cada fotografía

[javascript]
// estilo punto
var panoramio_style1 = new OpenLayers.StyleMap(OpenLayers.Util.applyDefaults({
   pointRadius: 15,
   fillColor: &quot;red&quot;,
   fillOpacity: 1,
   strokeColor: &quot;black&quot;,
   externalGraphic: &quot;${photo_file_url}&quot;
}, OpenLayers.Feature.Vector.style[&quot;default&quot;]));
[/javascript]

b) Por cada posición añadimos una miniatura de la foto.

[javascript]
var panoramio_style2 = new OpenLayers.StyleMap(OpenLayers.Util.applyDefaults({
   pointRadius: 7,
   fillColor: &quot;red&quot;,
   fillOpacity: 1,
   strokeColor: &quot;black&quot;,
   externalGraphic: &quot;panoramio-marker.png&quot;
}, OpenLayers.Feature.Vector.style[&quot;default&quot;]));
[/javascript]

Imagen: Ejemplos de estilos

[caption id="attachment_1613" align="aligncenter" width="300" caption="Estilos para Panoramio"]<a href="http://www.gisandchips.org/wp-content/captura_panoramio_styles.png"><img class="size-medium wp-image-1613" src="http://www.gisandchips.org/wp-content/captura_panoramio_styles-300x123.png" alt="Estilos para Panoramio" width="300" height="123" /></a>[/caption]

Ahora creamos la vector layer asignándole el estilo escogido

[javascript]
var vectorPano = new OpenLayers.Layer.Vector(&quot;Panoramio fotos&quot;, {
   styleMap: panoramio_style2
});
[/javascript]

Añadimos los elementos:

[javascript]
vectorPano.addFeatures(features);
[/javascript]

Añadimos la layer al mapa

[javascript]
map.addLayer(vectorPano);
[/javascript]

Definimos para terminar la función el comportamiento del evento que se desencadenará cuando seleccionemos ( click con el ratón) cada fotografía

[javascript]
selectControl = new OpenLayers.Control.SelectFeature(vectorPano,
   {onSelect: onFeatureSelect, onUnselect: onFeatureUnselect});
map.addControl(selectControl);
selectControl.activate();
}
[/javascript]

Como ves, se ejecutará la función “onFeatureSelect” cada vez que se selecciona un feature, y “onFeatureUnselect” cuando no hay nada seleccionado.

Con este código tenemos toda la lógica para obtener un vector layer con su posición y atributos. Ahora sólo nos queda definir el comportamiento de los eventos. En este caso vamos a crear un “popup” por cada elemento.

[javascript]
// popups
function onPopupClose(evt) {
   selectControl.unselect(selectedFeature);
}
function onFeatureSelect(feature) {
   selectedFeature = feature;

   // HTML del PopUp
      mensaje = &quot;&lt;a href='http://www.panoramio.com'&gt;&lt;img src='./panoramio_header-logo.png' alt='Panoramio' width='146' height='27' /&gt;&lt;/a&gt;&lt;br&gt;&quot;+
	      &quot;&lt;h2&gt;&quot;+ feature.attributes.photo_title + &quot;&lt;/h2&gt;&lt;p&gt;&quot; +
	      &quot;&lt;a href='&quot;+ feature.attributes.photo_url+
		  &quot;'&gt;&lt;img src='http://mw2.google.com/mw-panoramio/photos/small/&quot; +
		  feature.attributes.photo_id + &quot;.jpg' border='0' alt=''&gt;&lt;/a&gt;&lt;br&gt;&quot; +
	      &quot;autor: &lt;a href='&quot;+ feature.attributes.owner_url+&quot;'&gt;&quot;+feature.attributes.owner_name +&quot;&lt;/a&gt;&quot;;

   popup = new OpenLayers.Popup.FramedCloud(&quot;chicken&quot;,
      feature.geometry.getBounds().getCenterLonLat(),
      null,
      mensaje,
      null, true, onPopupClose);
   feature.popup = popup;
   map.addPopup(popup);
}
function onFeatureUnselect(feature) {
   map.removePopup(feature.popup);
   feature.popup.destroy();
   feature.popup = null;
}
[/javascript]

Ya lo tenemos todo listo. Sólo nos queda verlo en un navegador. ¿A que esperas?

<a href="http://www.gisandchips.org/demos/j3m/panoramio/panoramio.html">Ver demo en proyección Mercator con fondo de OpenStreetMap</a>
<a href="http://www.gisandchips.org/demos/j3m/panoramio/panoramio_4326.html">Ver demo en WGS84</a>

<strong>Nota de interés:</strong>
<blockquote>
Por último nos gustaría hacer una consideración de especial relevancia. Este código tiene sentido cuando todas las layers (WMS y vector layer de Panoramio) se encuentran en la misma proyección, en este caso se utiliza el datum WGS84 en coordenadas geográficas (EPSG: 4326). En el caso de que utilices en el mapa otra proyección debes de recordar que has de pasarle a la URL de Panoramio los parámetros de caja en WGS84, y lo mismo ocurre cuando el servicio te devuelve los puntos de cada foto, que ahora debes de hacer lo contrario, es decir, convertirla de EPSG:4326 a la proyección que estés utilizando.
</blockquote>