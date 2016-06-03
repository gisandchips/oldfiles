---
ID: 1034
post_title: 'Google Web Toolkit &amp; Google Maps'
author: Jorge Piera Llodrá
post_date: 2009-11-20 15:40:09
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2009/11/20/google-web-toolkit-google-maps/
published: true
syntaxhighlighter_encoded:
  - "1"
dcssb_short_url:
  - http://tinyurl.com/6fzbgzr
---
<a href="http://code.google.com/intl/es-ES/webtoolkit/">Google Web Toolkit</a> (GWT) es un framework creado por Google que permite al programador hacer aplicaciones web utilizando el lenguaje de programación Java. Existe un plugin de Eclipse que se utiliza para facilitar la generación de aplicaciones GWT donde se puede depurar código utilizando el depurador de código del entorno. Una vez que la aplicación esta depurada, se tiene que pasar por el compilador GWT que genera código HTML y Javascript. La idea es poder generar páginas que utilicen la tecnología Ajax (HTML + Javascript asíncrono), sin ser necesario escribir ni una sola línea de Javascript.

<a href="http://maps.google.es">Google Maps</a> es un servicio de mapas que ofrece Google que tiene un <a href="http://code.google.com/intl/es-ES/apis/maps/documentation/v3/">API</a> que permite integrarlo en cualquier página web utilizando Javascript. Pero si estamos utilizando GWT, existe además un jar que contiene un API para Java lo que permite a un programador de GWT poder integrar la tecnología de Google Maps en sus páginas web. Este artículo explica ese API y mediante ejemplos.<!--more-->

En este tutorial vamos a crear un proyecto de GWT y le vamos a añadir un mapa de Google Maps. Partiremos de un entorno en el que se ha instalado un Eclipse y se ha instalado el plugin de GWT. A continuación hay que crear un proyecto del tipo "Google -&gt; Web Application Project" que tiene que tener un nombre (GoogleMapsDemo) y un paquete raíz(org.gisandchips.gwt.maps). No es necesario utilizar los mismos nombres que se usan en el ejemplo, pero si se hace así se podrá seguir este tutorial sin la necesidad modificar nada.
<p style="text-align: center"></p>
<a href="http://www.gisandchips.org/wp-content/Pantallazo-New-Web-Application-Project-.png"><img class="size-full wp-image-1035 alignnone" src="http://www.gisandchips.org/wp-content/Pantallazo-New-Web-Application-Project-.png" alt="Pantallazo-New Web Application Project" width="300" height="260" /></a>

El wizard de creación de un proyecto GWT crea un proyecto que se puede ejecutar: para ello seleccionamos la opción "Run -&gt; Debug Configurations -&gt; Web Application" y seleccionaremos GoogleMapsDemo que es el lanzador que ha creado el wizard por nosotros. Pulsamos en "Debug" y si todo va bien se abrirá un navegador que muestra un formulario con un único campo de texto y un botón para enviar.

Podemos rellenar el campo de texto con cualquier valor y al pulsar el botón se abrirá una ventana dónde el servidor nos da la bienvenida. ¿Y qué tiene esto de especial? Lo que realmente ha ocurrido es que el cliente ha enviado una petición asíncrona (¡el famoso Ajax!) y ha esperado a que el servidor le devuelva una respuesta para mostrarla.

<a href="http://www.gisandchips.org/wp-content/Screenshot-Web-Application-Starter-Project-.png"><img class="size-full wp-image-1074 alignnone" src="http://www.gisandchips.org/wp-content/Screenshot-Web-Application-Starter-Project-.png" alt="Screenshot-Web Application Starter Project" width="400" height="300" /></a>

Ahora que ya sabemos crear y ejecutar una aplicación en GWT vamos a crear una aplicación utilizando Google Maps. Primero hay que <a href="http://code.google.com/p/gwt-google-apis/downloads/list">descargarse</a> la librería de Google Maps para GWT desde la página web de Google (gwt-maps-XXX), hay que descomprimirla, y hay que copiar el jar gwt-maps a la carpeta "war -&gt; WEB-INF -&gt; lib" de nuestro proyecto. A continuación hay que modificar el path del proyecto ("Project -&gt; Properties -&gt; Java Build Path -&gt; Libraries -&gt; Add Jars") y añadir la librería que acabamos de bajar.

Con esto ya tenemos acceso desde el Eclipse a las clases de Google Maps, pero para tener acceso en tiempo de ejecución tenemos que editar el fichero "org.gisandchips.gwt.maps.GoogleMapsDemo.gwt.xml" y añadir una línea para incluir la dependencia en el proyecto. Al final el fichero quedará de esta forma:

[xml]

&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot;?&gt;
&lt;!DOCTYPE module PUBLIC &quot;-//Google Inc.//DTD Google Web Toolkit 1.7.1//EN&quot; &quot;http://google-web-toolkit.googlecode.com/svn/tags/1.7.1/distro-source/core/src/gwt-module.dtd&quot;&gt;
&lt;module rename-to='googlemapsdemo'&gt;
 &lt;!-- Inherit the core Web Toolkit stuff.                        --&gt;
 &lt;inherits name='com.google.gwt.user.User'/&gt;

 &lt;!-- Inherit the default GWT style sheet.  You can change       --&gt;
 &lt;!-- the theme of your GWT application by uncommenting          --&gt;
 &lt;!-- any one of the following lines.                            --&gt;
 &lt;inherits name='com.google.gwt.user.theme.standard.Standard'/&gt;
 &lt;!-- &lt;inherits name='com.google.gwt.user.theme.chrome.Chrome'/&gt; --&gt;
 &lt;!-- &lt;inherits name='com.google.gwt.user.theme.dark.Dark'/&gt;     --&gt;

 &lt;!-- Other module inherits                                      --&gt;

 &lt;!-- Specify the app entry point class.                         --&gt;
 &lt;entry-point class='org.gisandchips.gwt.maps.client.GoogleMapsDemo'/&gt;

 &lt;!-- Load the Google Maps GWT bindings from the gwt-google-apis project --&gt;
 &lt;!-- Added by projectCreator if you use the -addModule argument --&gt;
 &lt;inherits name=&quot;com.google.gwt.maps.GoogleMaps&quot; /&gt;
 &lt;script src=&quot;http://maps.google.com/maps?gwt=1&amp;amp;file=api&amp;amp;v=2&amp;amp;sensor=false&quot; /&gt;
&lt;/module&gt;

[/xml]

Tal y como se indica en el fichero XML, la clase "org.gisandchips.gwt.maps.client.GoogleMapsDemo" es la que contiene el punto de entrada a la aplicación, por lo tanto es aquí donde hay que introducir el código Java que generará la página web. Por otra parte, el fichero "GoogleMapsDemo.html" que hay en la carpeta war es el que contiene el HTML de entrada y que habrá que editar para personalizar una nuestra página de inicio. En nuestro caso vamos a modificar el título y a añadir una etiqueta donde se situará el mapa. El "body" del documento quedaría de la siguiente forma:

[html]

&lt;body&gt;

 &lt;!-- OPTIONAL: include this if you want history support --&gt;
 &lt;iframe src=&quot;javascript:''&quot; id=&quot;__gwt_historyFrame&quot; tabIndex='-1' style=&quot;position:absolute;width:0;height:0;border:0&quot;&gt;&lt;/iframe&gt;

 &lt;h1&gt;Mi primer ejemplo con GWT y Google Maps&lt;/h1&gt;

 &lt;table align=&quot;center&quot;&gt;
 &lt;tr&gt;
 &lt;td id=&quot;mapContainer&quot;&gt;&lt;/td&gt;
 &lt;/tr&gt;
 &lt;/table&gt;
 &lt;/body&gt;

[/html]

A continuación hay que editar la clase "org.gisandchips.gwt.maps.client.GoogleMapsDemo". Eliminamos todo el código del método "onModuleLoad" y añadimos el siguiente código java:

[java]

public void onModuleLoad() {
 MapWidget mapWidget = new MapWidget();
 mapWidget.setSize(&quot;1000&quot;, &quot;500&quot;);

 RootPanel.get(&quot;mapContainer&quot;).add(mapWidget);

}

[/java]

Ahora ya podemos lanzar el proyecto una vez más para ver los resultados. Se abrirá el navegador y tendremos que ver una página web con un mapa en el centro donde podremos hacer panning y desplazarnos.

<a href="http://www.gisandchips.org/wp-content/Screenshot-Web-Application-Starter-Project-1.png"><img class="size-full wp-image-1091 alignnone" src="http://www.gisandchips.org/wp-content/Screenshot-Web-Application-Starter-Project-1.png" alt="Screenshot-Web Application Starter Project" width="500" height="350" /></a>

Ahora que sabemos cómo introducir un mapa de todo el planeta en nuestra aplicación, vamos a modificar algunas de las propiedades del objeto "MapWidget" que es el que representa al mapa. Primero vamos a añadir algunos componentes al mapa mediante el método "addControl".

Podemos introducir una barra de navegación que permitirá hacer zoom añadiendo un objeto de tipo "LargeMapControl". También podemos añadir un combo que permitirá seleccionar los distintos formatos de GoogleMaps utilizando el objeto "MenuMapTypeControl".  El método "onModuleLoad" quedaría así:

[java]
public void onModuleLoad() {
MapWidget mapWidget = new MapWidget();
mapWidget.setSize(&quot;1000&quot;, &quot;500&quot;);

//Añadimos la barra de navegación
mapWidget.addControl(new LargeMapControl());

//Añadimos el combo con los formatos
mapWidget.addControl(new MenuMapTypeControl());

RootPanel.get(&quot;mapContainer&quot;).add(mapWidget);
}
[/java]

Ahora volvemos ejecutar la aplicación y podremos ver los nuevos componentes introducidos en el mapa.

Llagados a esta punto ya hemos comentado los pasos básicos que hay que hacer para integrar un mapa de Google Maps en una web realizada mediante GWT. En los próximos artículos veremos cómo se pueden hacer búsquedas en el mapa y cómo se puede personalizar.