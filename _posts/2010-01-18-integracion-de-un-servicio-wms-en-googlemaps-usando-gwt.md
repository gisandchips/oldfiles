---
ID: 1192
post_title: >
  Integración de un servicio WMS en
  GoogleMaps usando GWT
author: Jorge Piera Llodrá
post_date: 2010-01-18 23:26:00
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2010/01/18/integracion-de-un-servicio-wms-en-googlemaps-usando-gwt/
published: true
---
<p style="text-align: justify">Este artículo es una continuación del artículo "<a href="http://www.gisandchips.org/?p=1034">Google Web Toolkit &amp; Google Maps</a>" que se publicó en este mismo blog hace un par de meses. La idea de escribirlo surgió debido a una pregunta que nos hizo una lectora en la que preguntaba cómo se podía integrar un servidor <a href="http://es.wikipedia.org/wiki/WMS">WMS</a> propio en Google Maps utilizando GWT.</p>
<p style="text-align: justify">Este artículo explica paso a paso cómo hacerlo y para ello se ha tomado como servidor de ejemplo un <a href="http://wms.jpl.nasa.gov/wms.cgi">WMS de la NASA,</a> pero se ha escrito el código de forma que haciendo unas pocas modificaciones se puede integrar cualquier otro servidor WMS.</p>
<p style="text-align: center"><a href="http://www.gisandchips.org/wp-content/wms1.png"><img class="size-full wp-image-1196 alignnone" src="http://www.gisandchips.org/wp-content/wms1.png" alt="wms" width="300" height="200" /></a></p>
<p style="text-align: justify">Se parte de un entorno en el que se ha creado un proyecto de GWT llamado GoogleMaps-WMS utilizando para ello el entorno de desarrollo Eclipse. Además se ha creado una página de inicio y se ha añadido un mapa de GoogleMaps en el centro. Cómo crear el proyecto y cómo crear la página de inicio es algo que ya se explicó en el <a href="http://www.gisandchips.org/?p=1034">artículo anterior</a>.</p>
<p style="text-align: justify"><!--more--></p>
<p style="text-align: justify">Antes de empezar a ver código es necesario entender cómo funciona el sistema de tileado de GoogleMaps basado en un <a href="http://es.wikipedia.org/wiki/Quadtree">QuadTree</a>, que no es más que una estructura de datos muy utilizada para crear índices espaciales. El QuadTree  se aplica sobre un área concreta y en el caso de GoogleMaps se aplica sobre toda la superficie terrestre que varía entre la longitud  -180º a 180º y la latitud -90º a 90º (estas cifras no son del todo exactas, ya que la latitud se "acorta" unos grados en los polos)</p>
<p style="text-align: justify">El QuadTree divide  esta superficie en 4 áreas en forma rectangular de tamaño fijo, que en su conjunto pueden ser vistas como una matriz de 2x2 y se les pueden dar nombres según la fila y la columna en la que están. De aquí en adelante  a esas áreas rectangulares las vamos a llamar tiles. La nomenclatura de Google es numérica y crece de Oeste a Este y de Norte a sur de forma que el tile de la parte superior izquierda es el (0,0), el tile de la parte superior derecha es el (1,0), el de la parte inferior izquierda es el (0,1) y el de la parte inferior derecha es el (1,1). Esta matriz de 2x2 es la matriz de tiles para un nivel de zoom 1.</p>
<p style="text-align: justify">Cada una de esos tiles se puede descomponer a su vez en 4 partes iguales de área constante. Si dividimos los 4 tiles que hay inicialmente en 4 partes iguales obtendremos una matriz de 4x4 tiles, que nombraremos matriz de tiles de nivel 2. Podemos utilizar la misma nomenclatura que hemos utilizado para los tiles de nivel de zoom 1, siendo el tile de la esquina superior izquierda el (0,0) y el tile de la esquina inferior derecha el (3,3).</p>
<p style="text-align: justify">Este proceso de descomposición de tiles en 4 tiles de un nivel inferior lo podemos realizar N veces de forma que al final tendremos una estructura de datos de N niveles en los que en cada nivel habrá 4 veces más tiles que en el nivel predecesor. En el caso de GoogleMaps cada uno de estos tiles es una imagen de 256x256 píxels independientemente del nivel en el que nos encontremos, por lo que a mayor nivel de zoom más detalle tendrá la imagen. Para poder entender mejor la nomenclatura de los tiles de Google podemos consultar <a href="http://www.maptiler.org/google-maps-coordinates-tile-bounds-projection/">ésta página web</a>.</p>
<p style="text-align: justify">Una vez entendido el sistema de tileado y la nomenclatura que usa GoogleMaps, vamos a empezar a escribir código.  Existe en el API  que proporciona Google una clase llamada <a href="http://code.google.com/p/gwt-google-apis/source/browse/trunk/google-apis/src/com/google/gwt/maps/client/TileLayer.java?r=41">TileLayer</a> que representa un proveedor de tiles del que se nutrirá el mapa para poder visualizarse. Tenemos por tanto que crear una clase llamada <a href="http://www.gisandchips.org/svn/global/jpiera/GoogleMaps-WMS/src/org/gisandchips/gwt/maps/wms/client/WMSTileLayer.java">WMSTileLayer</a> que herede de TileLayer, lo que implica que hay que implementar algunos métodos:</p>

[java]
public class WMSTileLayer extends TileLayer{
public WMSTileLayer(String baseServerUrl, String layers, String styles, Projection projection){
super();
}
@Override
public String getTileURL(Point tile, int zoom) {
return null;
}
@Override
public double getOpacity() {
return 1.0;
}
@Override
public boolean isPng() {
return true;
}
[/java]
<p style="text-align: justify">En el método <span style="font-style: italic">getOpacity</span> hay que devolver la transparencia de la imagen que varía entre 0 y 1. Si queremos que la imagen se visualice en el mapa tendremos que poner el valor 1.0.</p>
<p style="text-align: justify">En el método <span style="font-style: italic">isPng</span> hay que indicar si la imagen que se devuelve es png. En nuestro caso hemos devuelto <span style="font-style: italic">true</span> ya que las imágenes devueltas las pedimos en png.</p>
<p style="text-align: justify">Pero en el tercer método de la clase es dónde está la complejidad de esta clase. El método <span style="font-style: italic">getTileURL</span> acepta como parámetros un objeto de tipo Point y un entero. El entero hace referencia el nivel de zoom mientras que el Point es una pareja de enteros que hacen referencia al nombre del tile que hay que devolver.  Si hemos entendido la estructura de QuadrTree y la nomenclatura que usa Google en los tiles podremos entender el significado de estos parámetros. A medida que el usuario se desplaza por el mapa, el componente encargado de recuperar los tiles invoca al método <span style="font-style: italic">getTileURL</span> para recuperar la URL de los tiles del área en la que se encuentra. Este método se invocará normalmente unas cuántas veces por cada vez que el usuario modifique el área del mapa que está visualizando.</p>
<p style="text-align: justify">En el ejemplo de código hemos visto cómo en la implementación del método <span style="font-style: italic">getTileURL</span> se devolvía el valor <span style="font-style: italic">null</span> y ahora lo vamos a modificar para que devuelva la URL del tile que se está pidiendo en los argumentos del método y que  tendrá que ser descargado desde un servidor WMS. Podríamos haber creado un componente específico para un servidor WMS concreto, pero en lugar de eso hemos implementado un componente algo más genérico para que se pueda utilizar con otros servidores de mapas.</p>
<p style="text-align: justify">Para ello hemos añadido a la clase <a href="http://www.gisandchips.org/svn/global/jpiera/GoogleMaps-WMS/src/org/gisandchips/gwt/maps/wms/client/WMSTileLayer.java">WMSTileLayer</a> algunos parámetros que se establecen en el constructor. Estos parámetros son la dirección del servidor WMS, la lista de capas a cargar y los estilos con los que se desea visualizar la capa. Cómo conocer los valores de la capa(s) que soporta un servidor o los valores del estilo(s) soportados está fuera del alcance de este artículo, pero bastaría con hacer la operación <span style="font-style: italic">GetCapabilities</span> sobre el servidor.</p>
<p style="text-align: justify">El código resultante quedaría del siguiente modo:</p>
<p style="text-align: justify"></p>

[java]
public class WMSTileLayer extends TileLayer{
private String baseServerUrl;
private String layers;
private String styles;
private Projection projection;
private String getMapUrl = null;
public WMSTileLayer(String baseServerUrl, String layers, String styles, Projection projection){
super(null, 0, 20);
this.baseServerUrl = baseServerUrl;
this.layers = layers;
this.styles = styles;
//construct the base URL
String urlBase = baseServerUrl + &quot;?&quot; +
&quot;FORMAT=image/png&amp;amp;SERVICE=WMS&amp;amp;REQUEST=GetMap&amp;amp;WIDTH&quot; +
&quot;=256&amp;amp;HEIGHT=256&amp;amp;VERSION=1.1.0&amp;amp;&quot; +
&quot;STYLES=&quot; + styles + &quot;&amp;amp;&quot; +
&quot;LAYERS=&quot;+ layers + &quot;&amp;amp;&quot;;
getMapUrl = urlBase;
}
@Override
public String getTileURL(Point tile, int zoom) {
return null;
}
@Override
public double getOpacity() {
return 1.0;
}
@Override
public boolean isPng() {
return true;
}
[/java]
<p style="text-align: justify"></p>
<p style="text-align: justify">Además de los parámetros comentados anteriormente, podemos ver que se ha incluido un cuarto parámetro llamado projection, que se utiliza para identificar la proyección y deberá ser el mismo que el utilizado para crear el componente contenedor de mapas de GoogleMaps.</p>
<p style="text-align: justify">En el constructor lo que se está haciendo es básicamente crear la URL que servirá de base para lanzar la petición <span style="font-style: italic">GetMap</span> a un servidor WMS.Todavía faltan algunos parámetros para que la petición sea completa, pero estos parámetros los vamos a añadir en el método <span style="font-style: italic">getTileURL</span> que es el último método que nos queda por ver para terminar con la explicación.</p>
<p style="text-align: justify">Para simplificar el problema, vamos a suponer que el servidor WMS que estamos utilizando soporta el sistema de coordenadas en latitud/longitud (EPSG:4326). El problema será entonces pasar un tile de GoogleMaps definido mediante un nombre y un nivel de zoom a un par de coordenadas que definan el mismo tile en latitud/longitud. Para ello utilizamos el API de GoogleMaps y una vez tengamos ese tile, creamos la Url que lo contiene, de modo que el método getTileURL quedará de la siguiente forma:</p>

[java]
@Override
public String getTileURL(Point tile, int zoom) {
//Transforma las coordenadas a coordenadas Lat/Lon
Point tileIndexLLPoint = Point.newInstance(tile.getX() * 256, (tile.getY() + 1) * 256);
Point tileIndexURPoint = Point.newInstance(((tile.getX() + 1) * 256), (tile.getY()) * 256);
LatLng llPoint = projection.fromPixelToLatLng(tileIndexLLPoint, zoom, true);
LatLng urPoint = projection.fromPixelToLatLng(tileIndexURPoint, zoom, true);
String url =  getMapUrl + &quot;SRS=EPSG:4326&amp;amp;BBOX=&quot; +
	llPoint.getLongitude() + &quot;,&quot; +
	llPoint.getLatitude() + &quot;,&quot; +	
	urPoint.getLongitude() + &quot;,&quot; +
	urPoint.getLatitude();
return url;
}
[/java]

En las variables <span style="font-style: italic">llPoint</span> y <span style="font-style: italic">urPoint</span> se calculan las coordenadas de la esquina inferior izquierda y de la esquina superior derecha del tile que en latitud/longitud equivale al tile de GoogleMaps. Añadiendo estas coordenadas a la URL mediante el parámetro <span style="font-style: italic">BBOX</span> y añadiendo el parámetro <span style="font-style: italic">SRS </span>se forma la URL que se devuelve en el método y que será usada por el componente gráfico para recuperar las imágenes y mostrarlas en pantalla. Ahora ya sólo falta crear un objeto <a href="http://www.gisandchips.org/svn/global/jpiera/GoogleMaps-WMS/src/org/gisandchips/gwt/maps/wms/client/WMSTileLayer.java">WMSTileLayer</a> y asociarlo al componente gráfico de GoogleMaps. La clase que hace esto es <a href="http://www.gisandchips.org/svn/global/jpiera/GoogleMaps-WMS/src/org/gisandchips/gwt/maps/wms/client/GoogleMaps_WMS.java">GoogleMaps_WMS</a>, y el código es el sigueinte:

[java]
public class GoogleMaps_WMS implements EntryPoint {
 private MapWidget map;
 private String wmsURL = &quot;http://wms.jpl.nasa.gov/wms.cgi&quot;;
 private String wmsLayers = &quot;global_mosaic_base&quot;;
 private String wmsStyles = &quot;pseudo&quot;;
 private String mapName = &quot;Nasa&quot;;
 public void onModuleLoad() {
//Coordenadas centrales para posicionar el mapa
LatLng latLonCenter = LatLng.newInstance(0,0);
//Creamos el mapa con un tamaÃ±o fijo
map = new MapWidget(latLonCenter, 2);
map.setSize(&quot;520px&quot;, &quot;520px&quot;);
Projection projection = new MercatorProjection(20);
//Creamos el servicio WMS que nutrirá a GoogleMaps
MapType myWMSMap = new MapType(new TileLayer[] {
new WMSTileLayer(wmsURL, wmsLayers, wmsStyles, projection)},
projection,
mapName);
//Añadimos el servicio WMS al mapa
map.addMapType(myWMSMap);
//Añadimos algunas opciones al mapa
MapUIOptions options = MapUIOptions.newInstance(Size.newInstance(400,400));
options.setMapTypeControl(true);
map.setUI(options);
// Add the map to the HTML host page
RootPanel.get(&quot;mapContainer&quot;).add(map);
}
}
[/java]

En el ejemplo, la clase tiene algunos atributos:
<ul>
	<li>wmsURL: para definir la URL del servicio WMS.</li>
	<li>wmsLayers: lista separada por comas de las capas a mostrar.</li>
	<li>wmsStyle: estilo con el que se desea mostrar la capa</li>
	<li>mapName: nombre del mapa que aparecerá en el interfaz de usuario de GoogleMaps</li>
</ul>
Cambiando estos atributos se puede acceder a cualquier otro servidor WMS.