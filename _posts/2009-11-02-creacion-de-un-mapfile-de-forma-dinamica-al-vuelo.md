---
ID: 383
post_title: 'Creación de un mapfile de forma dinámica (&#8220;al vuelo&#8221;)'
author: pepe
post_date: 2009-11-02 11:47:11
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2009/11/02/creacion-de-un-mapfile-de-forma-dinamica-al-vuelo/
published: true
---
Este artículo va encaminado a la publicación de cartografía en Internet de forma dinámica, utilizando PHPMapScript.

Lo que vamos a intentar conseguir es mostrar en una página web un fichero de mapa sin la necesidad de tener un fichero .map (MapFile) asociado y complejo de mantener, simplemente realizando modificaciones en nuestro código php.

Este es el resultado que queremos obtener:

[caption id="attachment_1085" align="aligncenter" width="300" caption="Mapa de españa"]<img class="size-medium wp-image-1085" title="4b025a29_5fd1_1" src="http://www.gisandchips.org/wp-content/4b025a29_5fd1_1-300x300.gif" alt="Mapa de españa" width="300" height="300" />[/caption]
<p style="text-align: center"></p>
Vamos a ponernos manos a la obra para conseguirlo.

En primer lugar, doy por supuesto que tenemos instalado MapServer y MapScript en nuestro servidor Linux (en mi caso CentOS)  y el Servidor de Bases de Datos PostgreSQL+PostGIS y con la extensión PHP/pgSQL.

Creamos un fichero .php

Lo primero que debemos hacer es cargar la librería PHPMapScript (<em>php_mapscript.so</em>)

<!--more-->

[php]

dl('php_mapscript.so');

//Una vez cargada la librería ya se puede trabajar con las funciones propias de PHPMapScript
//Para empezar hay que crear el fichero mapa y definir las diferentes características que queremos que tenga el mapa (nombre, tamaño,

//Creamos un objeto llamado Map mediante el Constructor de MapObject

$Map=ms_newMapObj(&quot;&quot;);

//Le damos un nombre (p.e. mapa)
$Map-&gt;set(&quot;name&quot;,&quot;Mapa&quot;);

//La dimensión del mapa
$Map-&gt;setSize(600,600);

// La extensión del mapa (la podemos obtener de postgresql mediante la función de postgis st_extent
$Map-&gt;setExtent(626679.9375,4191059,815673.125,4519371);

// Se define el path de la imagen la temporal y la real que se mostrarán por web
$Map-&gt;web-&gt;set(&quot;imagepath&quot;,&quot;/tmp&quot;);
$Map-&gt;web-&gt;set(&quot;imageurl&quot;,&quot;/ms_tmp&quot;);

Una vez hemos definido las características de nuestro fichero de mapa, procedemos a incluir capas.
&lt;pre&gt;//Definimos una capa asociada a nuestro fichero de mapa anteriormente definido
// En este caso la primera capa, y que va a servir de fondo de nuestro mapa va a ser wms
// y del servicio wms que proporciona el &lt;a href=&quot;http://www.idee.es/wms/PNOA/PNOA?Request=GetCapabilities&amp;Service=WMS&quot; target=&quot;_blank&quot;&gt;PNOA&lt;/a&gt;&lt;/pre&gt;
$Layer1=ms_newLayerObj($Map);

$Layer1-&gt;set(&quot;name&quot;,&quot;ortofoto&quot;);

//Tipo Raster

$Layer1-&gt;set(&quot;type&quot;,MS_LAYER_RASTER);

$Layer1-&gt;set(&quot;status&quot;,MS_ON);//El status de la Layer, esto nos permitirá definir layers y mantenerlas ocultas dependiendo de las necesidades que tengamos
$Layer1-&gt;setConnectionType(MS_WMS);
$Layer1-&gt;set(&quot;connection&quot;,&quot;http://www.idee.es/wms/PNOA/PNOA?&quot;);
//Definimos la proyección del wms junto con el nombre de la capa wms, la versión del servidor y el formato de la imagen
&lt;pre&gt;//El que esta capa se muestre o no, no depende de nosotros, depende de una fuente externa que proporciona la capa WMS, en este caso es la del PNOA&lt;/pre&gt;
$Layer1-&gt;setProjection(&quot;init=epsg:23030&quot;);
$Layer1-&gt;setMetadata(&quot;wms_name&quot;,&quot;pnoa&quot;);
$Layer1-&gt;setMetadata(&quot;wms_server_version&quot;,&quot;1.1.1&quot;);
$Layer1-&gt;setMetadata(&quot;wms_format&quot;,&quot;image/png&quot;);

//Como segunda capa asociada a nuestro fichero de mapa anteriormente definido vamos a añadir una capa obtenida de nuestros datos en postgresql
$Layer2=ms_newLayerObj($Map);

//Vamos definiendo las características que va a tener esta nuestra primera capa
$name = $Layer2-&gt;set(&quot;name&quot;,&quot;Municipios&quot;);
$type = $Layer2-&gt;set(&quot;type&quot;,MS_LAYER_POLYGON);
$status = $Layer2-&gt;set(&quot;status&quot;,MS_ON);
//Conexión con nuestra base de datos que va as er de tipo POSTGIS
$Layer2-&gt;setConnectionType(MS_POSTGIS);
//Cadena de conexión a la base de datos donde tenemos nuestros datos
$Layer2-&gt;set(&quot;connection&quot;,&quot;user=postgres dbname=demos host=localhost&quot;);
//Filtro en sql que vamos a introducir para sacar nuestros datos a mostrar
// MUY IMPORTANTE es necesario incluir el unique gid y using srid=23030, si no no funcionará
$Layer2-&gt;set(&quot;data&quot;,&quot;geometria from ine.municipios using unique gid using srid=23030&quot; );
//Una vez definida la estructura d ela capa procedemos a definir el estilo de nuestra capa número 1

$clase = ms_newClassObj($Layer2); //Definimos la clase
$estilo = ms_newStyleObj($clase); //El estilo asociado a dicha clase
$estilo-&gt;color-&gt;setRGB(10,150,190); //Los colores
$estilo-&gt;outlinecolor-&gt;setRGB(0,0,0); //El contorno

//Una vez hemos definido las capas tenemos que indicar que se dibuje el mapa
$Image=$Map-&gt;Draw();

//Muy importante que no se nos olviden estas 2 sentencias que vienen a continuación, si no ponemos que se almacene la imagen en el path que le hemos incluido o no la mostramos por pantalla con un echo, no va a mostarse nada, y caeremos en el desanimo ya que después de definir todo correctamente la página aparece en blanco.

$url_imagen=$Image-&gt;saveWebImage();
echo &quot;&lt;img src=&quot;.$url_imagen.&quot;&gt;&quot;;

[/php]

Y después de tener esto, solo hay que lanzar el navegador y et voilà!, tenemos nuestro primer mapfile utilizando PHPMapScript en nuestro navegador, puede que sea un poco lento a la hora de cargar, pero no depende tanto de nuestra capa en PostGIS sino del Servidor WMS que utilicemos como capa de fondo, que ya no depende de nosotros.

El resultado es este: <a href="http://www.gisandchips.org/demos/mapscript/index.php" target="_blank">http://www.gisandchips.org/demos/mapscript/index.php</a>
<p style="text-align: center;"></p>