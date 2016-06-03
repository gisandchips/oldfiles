---
ID: 2833
post_title: >
  ¿Puedo generar “tiles”
  dinámicamente con MapProxy? (2)
author: pepe
post_date: 2015-03-10 09:41:23
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2015/03/10/puedo-generar-tiles-dinamicamente-con-mapproxy-2/
published: true
wip_template:
  - full
---
<h1>¿Puedo generar “tiles” dinámicamente con MapProxy? (2)</h1>
Continuando con el <a title="¿Puedo generar “tiles” dinámicamente con MapProxy?" href="http://www.gisandchips.org/2015/03/09/puedo-generar-tiles-dinamicamente-con-mapproxy/">post anterior</a> , el proceso que vamos a seguir es el siguiente, tras emitir en el canal de <strong>redis</strong>, lo que vamos a realizar es una suscripción al canal de redis desde Python, procesar el mensaje recibido tras una actualización de geometría, actualizar un fichero de tileado (seed) para MapProxy y lanzar el proceso de tileado (seeding) para la nueva bbox de la geometría actualizada.

Partimos de que tenemos el <a title="Fichero de configuración de mapproxy" href="http://mapproxy.org/docs/1.6.0/configuration.html" target="_blank">fichero de configuración de mapproxy</a> (yaml) que incluye las caches definidas, las capas, bbox,... , también tenemos un fichero de "seed" o tileado (<a href="http://es.wikipedia.org/wiki/YAML" target="_blank">yaml</a>) como este:

[code]
# #####################################################################
#               MapProxy example seed configuration
# #####################################################################
#
# This is _not_ a runnable configuration, but it contains most
# available options in meaningful combinations.
#
# Use this file in addition to the documentation to see where and how
# things can be configured.

seeds:
  myseed1:
    caches: [cache1,cache2,cache3]
    srs: ['EPSG:3857']
    coverages: [mycoverage]
    levels:
      from: 18
      to: 22
    refresh_before:
      # re-generate tiles older than this date
      time: 2015-01-15T12:35:00
coverages:
  mycoverage:
     #bbox de ejemplo
     bbox: [716107.129003602, 4250947.24206775,716292.878733323 ,4251328.38112309]
     bbox_srs: &quot;EPSG: 25830&quot;
[/code]

A continuación en <strong>python</strong> procedemos a crear un script que realice las operaciones que anteriormente hemos descrito. (NOTA: Al intentarlo con Php daba problemas de edición de yaml)

<!--more-->

<script src="https://gist.github.com/torrespri/e445ddbae07ec76b50ff.js"></script>En primer lugar, se lee el fichero tipo "seed" de configuración de <a href="http://mapproxy.org" target="_blank">Mapproxy</a> anteriormente creado. Conectamos con el canal de <a title="Redis " href="http://redis.io" target="_blank"><strong>redis</strong></a> mediante <strong>pubsub</strong> y nos mantenemos a la escucha esperando a que llegue un mensaje, cuando llega, se lee, se trocea la cadena de mensaje y se va editando el archivo <strong>yaml</strong>. Una vez finalizado se almacena en la carpeta donde estamos trabajando y se lanza el script de tileado (seed) de mapproxy.


[code]
subprocess.call([&quot;mapproxy-seed&quot;,&quot;-f&quot;,&quot;/var/datos/mymapproxy_sigua/mapproxy_base_completa.yaml&quot;,&quot;-c&quot;,&quot;4&quot;,&quot;yamlpython.yaml&quot;])
[/code]


De esta forma en python se lanza la ejecución del proceso normal de tileado (seed) de mapproxy en lugar de lanzarlo a mano de la siguiente forma:


[code]

mapproxy-seed -f &lt;fichero_de_configuracion_mapproxy.yaml&gt; -c &lt;nº de procesos concurrentes&gt; &lt;fichero_de_tileado_seed_mapproxy.yaml&gt;

[/code]


Tras realizar esta operación lanzando el script con la instrucción python &lt;nombre_fichero.py&gt; se queda en ejecución en escucha de modificaciones en la base de datos de postgresql que se vean reflejadas en redis y que redis envíe el mensaje tal y como se describió en el<a title="¿Puedo generar “tiles” dinámicamente con MapProxy?" href="http://www.gisandchips.org/2015/03/09/puedo-generar-tiles-dinamicamente-con-mapproxy/"> anterior artículo</a>. Podemos chequear el funcionamiento añadiendo una nueva geometría en <a title="Quantum GIS" href="http://www.qgis.org" target="_blank">QGIS</a> , almacenamos la geometría en al base de datos y a continuación en el terminal vemos la secuencia de ejecución realizada. Pero no nos quedamos ahí, nos gustaría que este script se estuviera ejecutando continuamente como un <a title="Daemon Linux" href="http://es.wikipedia.org/wiki/Demonio_%28inform%C3%A1tica%29" target="_blank"><strong>daemon</strong></a> más de linux. Para realizar esta operación lo que realizamos es lo siguiente: En la carpeta /etc/init.d/ creamos un archivo con el nombre del daemon que queremos que se ejecute constantemente, le damos permisos de ejecución con chmod +x .

Código del daemon que se va a ejecutar :<script src="https://gist.github.com/torrespri/ec97adb8f53330f82a48.js"></script>

A continuación para programar la ejecución:

[code]
cd /etc/rc3.d/
ln -s ../init.d/&lt;nombre_daemon_creado&gt; S95&lt;nombre_daemon_creado&gt;
/etc/init.d/redismap start
[/code]

Y ya debería funcionar correctamente el tileado, podemos chequearlo monitorizándolo con htop

[caption id="attachment_2854" align="aligncenter" width="300"]<a href="http://www.gisandchips.org/wp-content//htop.png"><img class="wp-image-2854 size-medium" src="http://www.gisandchips.org/wp-content//htop-300x10.png" alt="Comprobación de la ejecución" width="300" height="10" /></a> Monitorización del fichero redismap en htop[/caption]