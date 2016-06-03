---
ID: 2622
post_title: >
  OpenStreetMap en Android con la libreria
  osmdroid
author: Miguel
post_date: 2013-10-16 10:18:24
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2013/10/16/openstreetmap-en-android-osmdroid/
published: true
---
<span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">Después de estar cacharreando un poco con Android y tras mucho tiempo sin aparecer me he animado a escribir una entrada en el blog. Se trata de localizarnos usando el GPS del dispositivo móvil y la cartografía de openstreetmap.</span>

<span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">Se da por hecho que ya tenemos cierta soltura con el IDE Eclipse y Android y que ya tenemos instalado el SDK de Android, por lo tanto no entraré en instalación del SDK de android y su configuración en Eclipse, no obstante os dejo un enlace sobre este tema: <a href="http://developer.android.com/sdk/installing/index.html" target="_blank">http://developer.android.com/sdk/installing/index.html</a>.</span>

<span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">Bien, lo primero es descargarse las librerías que se necesitan para el desarrollo de este ejemplo, para ello descargamos desde la siguiente URL: <a href="https://code.google.com/p/osmdroid/downloads/list" target="_blank">https://code.google.com/p/osmdroid/downloads/list</a> la librería osmdroid-android-3.0.10.jar que es la API de Open Street Map que vamos a usar, y la librería slf4j-android-1.6.1-RC1.jar desde la URL <a href="http://www.slf4j.org/android/" target="_blank">http://www.slf4j.org/android/</a>, la numeración puede cambiar, pero es la que hay en el momento de escribir el post y en principio nos vale la última que haya.</span>

<span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">Creamos un proyecto de Android nuevo:</span>

<span style="font-family: arial, helvetica, sans-serif; font-size: 14px;"><a href="http://www.gisandchips.org/wp-content//newproj.png"><img class="alignnone  wp-image-2554" alt="newproj" src="http://www.gisandchips.org/wp-content//newproj-1024x555.png" width="491" height="266" /></a></span>
<p align="JUSTIFY"><span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">Y presionamos el botón <b>Next</b> hasta que quede habilitado el botón <b>Finish</b>. Una vez creado añadiremos las librerías que nos hemos bajado antes, para ello las seleccionamos desde la carpeta donde estén ubicada y las arrastramos a eclipse a la carpeta <i>/libs</i>, y seleccionamos la opción de "Copy files".</span></p>
<p align="JUSTIFY"><span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">Nos habrá creado automáticamente dos ficheros que, si no hemos cambiado el nombre y hemos dejado el que nos ha puesto eclipse por defecto, serán estos:</span></p>
<p align="JUSTIFY"><span style="font-family: arial, helvetica, sans-serif; font-size: 14px;"><i>/res/layout/<b>activity_main.xml</b></i></span></p>
<p align="JUSTIFY"><span style="font-family: arial, helvetica, sans-serif; font-size: 14px;"><i>/src/org.gisandchips.osmtest/<b>MainActivity.java</b></i></span></p>
<p align="JUSTIFY"><span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">En el primer archivo, el activity_main.xml, insertaremos los componentes necesarios, que para este ejemplo es suficiente con un componente MapView. De manera que podemos eliminar el TextView que nos ha creado y añadir el siguiente código:</span></p>

[code language="xml"]
 &lt;org.osmdroid.views.MapView
        android:id=&quot;@+id/mapView&quot;
        android:layout_width=&quot;match_parent&quot;
        android:layout_height=&quot;match_parent&quot;
        android:layout_alignParentLeft=&quot;true&quot;
        android:layout_alignParentRight=&quot;true&quot;/&gt;
[/code]

<span style="font-family: arial, helvetica, sans-serif; font-size: 14px;"><!--more-->¿Qué es cada línea?, pues el <b>id</b> es el identificador que le daremos al componente y que usaremos para llamarlo desde nuestro código; <b>layout_width</b>, <b>layout_height</b>, <b>layout_alignParentLeft</b>, y <b>layout_alignParentRight</b> hacen referencia a las dimensiones que tendrá este </span><span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">componente MapView y la alineación con respecto al layout.</span>
<p align="JUSTIFY"><span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">El código del layout nos quedará de la siguiente manera:</span></p>
<script type="text/javascript" src="https://gist.github.com/mfernam/6989491.js"></script><span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">Ahora vamos al activity, el archivo con extensión .java, que es donde llamaremos a este layout. Dentro del método onCreate añadimos al final una serie de líneas.</span> <span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">Con esta línea llamamos al control MapView por el id que le hemos puesto al control en el layout y con el cual vamos a interactuar.</span>
[code language="java"]
MapView mapView = (MapView) findViewById(R.id.mapView);
[/code]


<span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">Lo siguiente es asignar el motor de renderizado del mapa que vamos a querer visualizar, podéis probar con otros para ver como queda el resultado.</span>
[code language="java"]
mapView.setTileSource(TileSourceFactory.MAPNIK);
[/code]


<span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">Esta línea es para que podamos usar los gestos con los dedos, es decir, para hacer zoom con el típico pellizco en la pantalla. Hasta aquí ya deberíamos poder ver el mapa, pero si lo ejecutamos en el emulador veremos que no se visualiza nada, esto es porque no le hemos dado los permisos necesarios a la aplicación, como el android.permission.INTERNET, y que debemos especificar en el AndroidManifest.xml. Si lo que queremos es ejecutarlo ya en nuestro dispositivo móvil necesitaremos otro permiso, WRITE_EXTERNAL_STORAGE. No obstante es importante cuando ejecutemos cualquier aplicación observar la ventana de log, ya que nos dará pistas de los errores que podamos tener, como olvidar dar algún permiso o no añadir un activity al AndroidManifest.</span>
[code language="java"]
mapView.setMultiTouchControls(true);
[/code]


<span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">Abrimos el AndroidManifest.xml seleccionamos la pestaña <i>Permissions</i> y añadimos los permisos necesarios para el usuario. </span> <span style="font-family: arial, helvetica, sans-serif; font-size: 14px;"><a href="http://www.gisandchips.org/wp-content//permisos1.png"><img class="alignnone  wp-image-2561" alt="permisos1" src="http://www.gisandchips.org/wp-content//permisos1-1024x555.png" width="491" height="266" /></a></span> <span style=";font-family: arial, helvetica, sans-serif; font-size: 14px;">Si lo ejecutamos en el emulador veremos un mapa mundi repetido, es normal, sólo hemos pintado un mapa, sin especificar zoom, ni una posición.</span> <span style="font-family: arial, helvetica, sans-serif; font-size: 14px;"><a href="http://www.gisandchips.org/wp-content//mapa1.png"><img class="alignnone  wp-image-2562" alt="mapa1" src="http://www.gisandchips.org/wp-content//mapa1-1024x555.png" width="491" height="266" /></a></span> <span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">Ahora nos queda “jugar” con el mapa, para que por ejemplo nos muestre nuestra localización y con un zoom determinado. Dentro del método onCreate añadiremos un MapController para hacer uso del zoom o centrar el mapa, un GeoPoint que determinará donde estamos, y un SimpleLocationOverlay, que es la capa donde se dibujará nuestra localización.</span> <span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">De manera que siguiendo el código anterior añadimos al final el siguiente bloque de código:</span>
[code language="java"]
MapController mapController = mapView.getController();
mapController.setZoom(12);
GeoPoint myLocation = new GeoPoint(GetMyLocation());
SimpleLocationOverlay myLocationOverlay = new SimpleLocationOverlay(this);
mapView.getOverlays().add(myLocationOverlay);
mapController.setCenter(myLocation);
myLocationOverlay.setLocation(myLocation);
[/code]
[code language="java"]
Location getMyLocation(){
LocationManager locationManager = (LocationManager) this.getSystemService(Context.LOCATION_SERVICE);
Return locationManager.getLastKnownLocation(LocationManager.GPS_PROVIDER);
}
[/code]


<span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">¿Qué hace este código? El <b>mapController</b> se ocupa de los métodos que trabajan con el mapa que hemos cargado en el mapView, <b>myLocation</b> es la variable que almacenará el punto con la localización que conseguimos del GPS del móvil y <b>myLocationOverlay</b> es una capa que dibujará nuestra localización con un icono por defecto, que es la silueta de un hombrecito. </span> <span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">Para la localización desde nuestro dispositivo móvil creamos un método privado, recordad que estará fuera del método onCreate (para hacer el código un poco más claro), que consta de dos líneas la primera que llama al servicio y la segunda que devuelve la <i>Location</i>. Lo normal es crearnos una clase a parte que controle todo lo relativo al GPS, pero esto nos vale para hacernos una idea de su funcionamiento.</span> <span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">Si lo ejecutáis desde el emulador nos dará primero un error de permisos, eso es porque hay que dar permisos de acceso al GPS al AndroidManifest.xml, que se llama ACCESS_FINE_LOCATION.</span> <span style="font-family: arial, helvetica, sans-serif; font-size: 14px;"><a href="http://www.gisandchips.org/wp-content//permisos2.png"><img class="alignnone  wp-image-2564" alt="permisos2" src="http://www.gisandchips.org/wp-content//permisos2-1024x555.png" width="491" height="266" /></a></span> <span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">Aun así nos seguirá dando un error, esto es porque el emulador no tiene GPS, por lo que llegados a este punto podemos o bien ejecutar la aplicación en nuestro dispositivo móvil o cambiar la línea donde creamos la variable myLocation, que en mi caso quedaría:</span>
[code language="java"]
GeoPoint myLocation = new GeoPoint(38.350643,-0.486578);
[/code]


<span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">El resultado de esto se verá en el emulador de esta manera:</span> <span style="font-family: arial, helvetica, sans-serif; font-size: 14px;"><a href="http://www.gisandchips.org/wp-content//mapa2.png"><img class="alignnone  wp-image-2565" alt="mapa2" src="http://www.gisandchips.org/wp-content//mapa2.png" width="342" height="408" /></a></span> 
<span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">Y aquí una captura de como se ve en el dispositivo móvil:</span> <a href="http://www.gisandchips.org/wp-content//Screenshot_2013-10-15-16-53-35.png"><img class="alignnone  wp-image-2641" alt="mapa3" src="http://www.gisandchips.org/wp-content//Screenshot_2013-10-15-16-53-35.png" width="288" height="480" /></a> 
<span style="font-family: arial, helvetica, sans-serif; font-size: 14px;">A continuación dejo el código completo de la clase MainActivity.java:</span><script type="text/javascript" src="https://gist.github.com/mfernam/6990415.js"></script>

Enlaces de interes:
<a href="http://android.martinpearman.co.uk/b4a/osmdroid/documentation/native_android_library/index.html?org/osmdroid/api/package-use.html">http://android.martinpearman.co.uk/b4a/osmdroid/documentation/native_android_library/index.html?org/osmdroid/api/package-use.html</a>
<a href="http://developer.android.com/sdk/index.html">http://developer.android.com/sdk/index.html</a>
<a href="http://www.openstreetmap.org/">http://www.openstreetmap.org/</a>