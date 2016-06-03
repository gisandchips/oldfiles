---
ID: 2111
post_title: LiveDVD 5.5 de OSGeo
author: Jorge Piera Llodrá
post_date: 2012-08-21 09:42:13
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2012/08/21/livedvd-5-5-de-osgeo/
published: true
---
OSGeo es una Fundación cuyo objetivo es <em>Apoyar el desarrollo de software geoespacial de código abierto, así como promocionar su uso</em> (<a href="http://wiki.osgeo.org/wiki/Cap%C3%ADtulo_Local_de_la_comunidad_hispanohablante">de su propia web</a>). Periódicamente publican un Live DVD que contiene una gran cantidad de programas GIS  instalados y configurados para poder probarlos sin necesidad de instalar nada en nuestra máquina.

Hace unos días que me <a href="http://live.osgeo.org/en/download.html">descargué la imagen</a> de <a href="https://www.virtualbox.org/">VirtualBox</a> de la versión 5.5 del Live DVD de OSGeo y empecé a "jugar" con ella. En este artículo voy a comentar muy brevemente algunos de los proyectos que me han parecido más interesantes y que no conocía anteriormente. La lista de todos los productos instalados y su descripción se puede encontrar el <a href="http://live.osgeo.org/es/overview/overview.html">su web</a>.
<p style="text-align: center"><img class="aligncenter size-full wp-image-2113" src="http://www.gisandchips.org/wp-content//osgeo.jpg" alt="" width="400" height="300" /></p>

<h3><!--more--></h3>
<h3>Geomajas</h3>
Me enteré de que este proyecto existía en el <a href="http://2010.foss4g.org/">FOSS4G de Barcelona</a> de hace 2 años pero no había oído hablar mucho de él. Geomajas es un cliente GIS web escrito en GWT, la tecnología perfecta para los desarrolladores Java que quieren introducirse en el mundo de las aplicaciones web. Y para los desarrolladores de Javascript, hay un plugin que les permite acceder a Geomajas.

Actualmente soporta WMS, WFS, y distintos tipos de bases de datos espaciales, soportando además edición de fenómenos. Por lo que he podido leer en su web es extensible a base de plugins por lo que se pueden crear clientes personalizados habilitando sólo algunas funcionalidades.

<img class="aligncenter size-full wp-image-2126" src="http://www.gisandchips.org/wp-content//geomajas1.png" alt="" width="400" height="300" />
<h3>Sahana Eden</h3>
Tal y como se  comenta en la web de OSGeo, "El proyecto Sahana se inició por voluntarios de la comunidad de desarrolladores Sri Lanka FOSS para ayudar a sus compatriotas afectados durante el Tsunami de diciembre de 2004 en Asia. El sistema fue utilizado oficialmente por el Gobierno de Sri Lanka y se liberó como Software Libre y de Código Abierto. Posteriormente se reescribió como una herramientas genérica para la administración de desastres y fue incubado con el patrocinio de la Swedish International Development Agency, IBM y la US National Science Foundation. Desde entonces ha sido utilizado por docenas de Gobiernos y Organizaciones No Gubernamentales."

Es una aplicación web en la que se pueden introducir los incidentes que ocurren en situaciones de emergencia y mediante el uso de recursos y de voluntarios se intenta coordinar la atención de estos incidentes.

<img class="aligncenter size-full wp-image-2124" src="http://www.gisandchips.org/wp-content//shane.png" alt="" width="400" height="300" />
<h3>Rasdaman</h3>
Es una tecnología de bases de datos que permite el almacenamiento de coberturas multidimensionales sin límite de tamaño (utilizando la tecnología de BD Postgres). Extiende el lenguaje SQL para poder realizar operaciones ráster.

Se puede acceder a Rasdaman fácilmente mediante GDAL y se incluye un API para poder acceder a los datos mediante el protocolo WCS (en la web incluso se indica que desde Abril del 2012 este API para el OGC WCS 2.0 Core conformance test).
<p style="text-align: center"><img src="http://www.gisandchips.org/wp-content//rasdaman.png" alt="" width="400" height="300" /></p>