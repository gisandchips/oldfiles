---
ID: 1963
post_title: >
  Proyección segura desde UTM ED50 con
  interfaz gráfico
author: jose
post_date: 2011-11-28 21:52:39
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2011/11/28/proyeccion-segura-desde-utm-ed50-con-interfaz-grafico/
published: true
---
Seguramente muchos de vosotros habéis estado en la tesitura de tener que proyectar vuestras capas de datos gis que usan la vetusta y a extinguir proyección UTM con datum ED50. Lo de que dicha proyección "no esta de moda" no es algo causal, puesto que hay un Real Decreto, el 1071, que define como sistema geodésico oficial de referencia para España aquel con el datum ETRS89 y el elipsoide SGR80. Bien es cierto que el decreto permite la ambivalencia de ambos sistemas (ETRS89 y ED50) hasta el año 2015; fecha a partir del cual diremos adiós definitivamente a ED50 (Elipsoide Hayford).
Por otra parte, con el auge de los wikimaps es bastante habitual tener que realizar reproyecciones de las capas gis con ED50 "de toda la vida" al sistema geodésico mundial WGS84, que es lo que admiten casi todas las geowikis (OpenStreetMap, ikiMaps, geoNames, etc.). 
[caption id="attachment_1989" align="aligncenter" width="486" caption="Reproyección mal realizada"]<a href="http://www.gisandchips.org/?attachment_id=1989"><img src="http://www.gisandchips.org/wp-content//ua_etrs89_mal.png" alt="Reproyección mal realizada" width="550" class="aligncenter size-full wp-image-1991" /></a>
<!--more-->
Pero, ¿que es lo que ocurre cuando utilizamos de forma genérica las utilidades de reproyección de los programas de GIS?. Casi todos, generan datos erróneos, que de ninguna manera se corresponden con la realidad. He encontrado translaciones en la X de más de 100 metros, y unos 50 en la Y respecto a la coordenada donde debería estar. 

He probado diferentes métodos (sin transformación, transformación EPSG) y el resultado es muy dispar. Sólo un de los métodos funciona bien, aquel cuya transformación utiliza la rejilla en formato NTv2 del IGN, que fue creado precisamente para la transformación de datos desde ED50.
[caption id="attachment_1991" align="aligncenter" width="486" caption="Reproyección con rejilla del IGN"]<a href="http://www.gisandchips.org/?attachment_id=1991"><img src="http://www.gisandchips.org/wp-content//sigua_ed50.png" alt="Reproyección con rejilla IGN" width="500" class="alignnone size-full wp-image-1991" /></a>
Para este artículo he creído conveniente proponer un método de reprojección que no haga uso de ningún programa GIS, y que podamos lanzarlo cómodamente desde la consola (eso de "cómodamente" a alguno le sonará a coña). Lo que quería era simplemente un programa que le indicase mi dato en UTM ED50 y me lo proyecte a lo que quiera. Finalmente he decidido crear un híbrido que esté a medio camino entre el "terminal" Linux y un interfaz gráfico (en este caso "GTK").
Como no se trata de reinventar la rueda, he utilizado una herramienta con la que me siento muy cómodo y me ha sacado más de una vez de un apuro. Se trata de "ogr2ogr", un comando que forma parte de la suite de GDAL. Este comando está diseñado para cambiar de formato nuestros datos, pero también para reproyectar. Si sintaxis está llena de modificadores que lo hacen a veces un poco críptico. 
Entrando en harina, sí deseamos reproyectar un fichero en ED50 a otra proyección la sintaxis es:
[bash]ogr2ogr -s_srs &quot;EPSG:23030&quot; -t_srs epsg_destino destino origen[/bash]

Este comando tampoco solucionaría los problemas para ED50, por lo que habría que retocar el comando de esta forma para que utilice la rejilla del IGN:
[bash]ogr2ogr -s_srs '+init=epsg:23030 +nadgrids=./peninsula.gsb +wktext' -t_srs epsg_destino origen destino[/bash]   

Coincidiréis conmigo que este comando con todas sus opciones no es muy intuitivo para recordar. Por esta razón he decidido utilizar un interfaz gráfico para el terminal (sí, no habéis oído mal) que permite lanzar los típicos formularios gráficos para rellenar los datos. En definitiva, no se hace nada más que encapsular dicho comando con cuadros de diálogo que se lanzan al X-windows. Hay algunas utilidades para ello: Xdialog (x-windows), dialog (para el terminal), kdialog (para KDE). Yo voy a utilizar uno llamado "Zenity", que está en todas las distribuciones linux. 
Finalmente he creado un script BASH que empotra las peticiones (selección del EPSG, de ficheros, alertas, comprobaciones, etc.) en diálogos hechos con Zenity.
Este es el codigo:
[bash]
#!/bin/sh
zenity --info --text=&quot;OGR2OGR Dialog \n
Autor: José Manuel Mira \n 
GISandCHIPS 2011 \n
Aplicación basada en OGR2OGR para convertir \n 
shapefile proyectado en UTM con datum ED50 en huso 30 (EPSG:23030) a  \n 
Geodésica mundial con datum WGS84 (epsg:4326) o \n
UTM ETRS89 (epsg:25830) utilizando GDAL. \n
NOTA: Esta utilidad necesita la rejilla del IGN \n
descargable en: \n
http://www.gisandchips.org/demos/j3m/ogr/peninsula.gsb&quot;
# Comprobar que está instalado OGR2OGR
if which ogr2ogr &gt; /dev/null; then
echo &quot;GDAL está instalado&quot;
else
echo &quot;No está instalado GDAL&quot; | zenity --error --text=&quot;No está instalado GDAL. Salir \n ¿Ubuntu? sudo apt-get install gdal-bin&quot;
exit
fi
#Comprobamos que tienes la rejilla instalada en el directorio del script
if [ -f ./peninsula.gsb ]
then
  echo &quot;OK, tienes la rejilla&quot; | zenity --info --text=&quot;OK, tienes la rejilla&quot;
else
  echo &quot;Descargando rejilla ...&quot;
  wget http://www.gisandchips.org/demos/j3m/ogr/peninsula.gsb 2&gt;=1 | sed -u 's/.*\ \([0-9]\+%\)\ \+\([0-9.]\+\ [KMB\/s]\+\)$/\1\n# Downloading \2/' | zenity --progress --title=&quot;Descargando rejilla IGN ...&quot;
fi

origen=$(zenity --file-selection --title=&quot;Selecciona un shapefile en proyección ED50&quot;)
case $? in
  0)  
    echo &quot;\&quot;$origen\&quot; selected.&quot;;;
  1)
    exit;;
   -1)
    exit;;
esac

# obtener extensión del fichero
ext=${origen#*.}
echo $ext
if [ $ext != 'shp' ]
then
   echo &quot;No es un shapefile. Salir&quot; | zenity --error --text=&quot;NO es un shapefile. Salir&quot;
   exit
fi

destino=$(zenity --entry --text &quot;Nombre del Shapefile de salida (sin .shp)?&quot; --entry-text &quot;destino&quot;); echo $destino
if [ $? = 1 ]
then
 exit
else
 echo $?
fi

epsg=$(zenity  --list  --text &quot;Selecciona la proyección de salida&quot; --radiolist  --column &quot;Sel&quot; --column &quot;EPSG&quot; --column &quot;Nombre&quot; TRUE 4326 &quot;Geodésica WGS84&quot; FALSE 25830 &quot;UTM ETS89 HUSO 30&quot;  FALSE 25831 &quot;UTM ETS89 HUSO 31&quot;  FALSE 25829 &quot;UTM ETS89 HUSO 29&quot;); 

echo &quot;ogr2ogr -s_srs '+init=epsg:23030 +nadgrids=./peninsula.gsb +wktext' -t_srs EPSG:&quot;${epsg} ${destino}-${epsg}.shp $origen
ogr2ogr -s_srs '+init=epsg:23030 +nadgrids=./peninsula.gsb +wktext' -t_srs EPSG:${epsg} ${destino}-${epsg}.shp $origen
zenity --info --text &quot;Shapefile ${destino}-${epsg}.shp creado satisfactoriamente&quot;
[/bash]
Guardamos el script como conversor.sh, le damos permisos de ejecución y lo lanzamos con 
[bash]sh ./conversor.sh[/bash]
 Estos son algunos de los diálogos que aparecen:
<img src="http://www.gisandchips.org/wp-content//dialogo_shapefile.png" alt="Indicar shapefile" />
<img src="http://www.gisandchips.org/wp-content//dialogo_rejilla.png" alt="Descarga rejilla IGN" />
<img src="http://www.gisandchips.org/wp-content//dialogo_ok.png" alt="Fin diálogo" />
<img src="http://www.gisandchips.org/wp-content//dialogo_epsg.png" alt="Lista SRS" />
Sí, ya se lo que estáis pensando (tanto rollo para una sola línea de comando). Es cierto, pero resulta útil y sencillo para aquellos que no se quieren complicar la vida. Además, con ligeras modificaciones lo puedo hacer extensible a las islas, otros husos, escribir un log con comandos para reutilizarlos en el futuro en modo BATCH, etc)
He realizado algunas pruebas para comprobar la validez de los datos reproyectados utilizando coordenadas de vértices geodésicos y visualizadas sobre un fondo con el WMS del PNOA. El resultado salta a la vista: "lo ha cuadrado"
[caption id="attachment_1990" align="aligncenter" width="486" caption="Comparación original (EPSG:23030) y proyectada a WGS84 (EPSG:4326)"]<a href="http://www.gisandchips.org/?attachment_id=1990"><img src="http://www.gisandchips.org/wp-content//comparacion_23030-4326.png" alt="comparación 23030-4326" width="570" class="alignnone size-full wp-image-1990" /></a>

Sí deseas probar el script <a href="/demos/j3m/ogr/vgeo.zip" title="Descarga vértices geodésicos">aquí</a> tienes unas shapes de prueba (vértices geodésicos en 23030) de la Comunidad Valenciana)