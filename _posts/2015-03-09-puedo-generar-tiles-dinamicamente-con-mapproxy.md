---
ID: 2803
post_title: '¿Puedo generar &#8220;tiles&#8221; dinámicamente con MapProxy?'
author: josetomas
post_date: 2015-03-09 11:29:36
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2015/03/09/puedo-generar-tiles-dinamicamente-con-mapproxy/
published: true
---
<p class="none">Si vuestro escenario es similar al que os presento a continuación la respuesta es sí, y además es bastante sencillo. Éstos son los requisitos:</p>

<ul>
	<li>Una base de datos PostGIS en la que se producen cambios en la geometría frecuentes y normalmente en áreas reducidas.</li>
	<li>Acceso de escritura al fichero de configuración del servidor MapProxy.</li>
	<li>Conexión a un servidor Redis para la difusión de mensajes mediante publicación/suscripción.</li>
</ul>
<p class="none">Bajo estas condiciones, programar la ejecución de MapProxy ante cambios en la geometría de PostGIS es viable, dado que, aunque se lance en tiempo real, el proceso de <em>tiling</em> es selectivo, afecta a un número reducido de <em>tiles</em> y se completa en menos tiempo que si hemos de regenerar toda la <em>cache</em>. La solución consiste básicamente en lanzar una función disparadora que se encargue - cuando una geometría se da de alta, de baja o se modifica - de emitir un mensaje a través de un canal de publicación de Redis, y programar un <em>daemon</em> que se suscriba a dicho canal y lance el proceso de <em>tiling</em>.
<a href="http://www.gisandchips.org/wp-content//redismapproxy.png"><img class="aligncenter size-medium wp-image-2805" src="http://www.gisandchips.org/wp-content//redismapproxy-300x155.png" alt="redismapproxy" width="300" height="155" /></a></p>
<!--more-->
<p class="none">Ésta es la función disparadora <em>notifygeometrychange_trigger</em> en <em>plpgsql</em>. Su cometido principal es formar un mensaje con el formato <strong>[table_name];[INSERT|UPDATE|DELETE];[xmin];[ymin];[xmax];[ymax]</strong> e invocar a la función <em>notify</em> en <em>plpython</em>, que recibe dicho mensaje como argumento de entrada.</p>
<p class="none">En función del tipo de operación (INSERT, UPDATE, DELETE) se obtiene el bounding box de la geometría afectada para incorporar los valores (x y) mínimo y máximo en el mensaje. Por otro lado, también se adjunta el nombre de la tabla en el mensaje, de forma que sea posible discernir la capa afectada en el fichero de configuración de MapProxy.</p>
<script src="https://gist.github.com/quommit/3c8cd42164dea83bbcc6.js"></script>

La función <em>notify</em> utiliza el paquete <a title="redis-py" href="https://github.com/andymccurdy/redis-py">redis-py</a> como interfaz de Redis. Ésta es la función que en última instancia publica el mensaje utilizando el <a title="Redis PUB/SUB" href="http://redis.io/topics/pubsub">mecanismo de publicación/suscripción de Redis</a>.

<script src="https://gist.github.com/quommit/5b7b469648c795534018.js"></script>
<p class="none">Si os interesa conocer cómo podemos automatizar la ejecución de MapProxy, seguid leyendo <a title="¿Puedo generar “tiles” dinámicamente con MapProxy? (2)" href="http://www.gisandchips.org/2015/03/10/puedo-generar-tiles-dinamicamente-con-mapproxy-2/">la segunda entrega de este post</a>.</p>