---
ID: 492
post_title: >
  Aumentar velocidad de carga de un
  mapfile utilizando simplificación de
  geometrías
author: pepe
post_date: 2009-11-05 12:48:58
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2009/11/05/aumentar-velocidad-de-carga-de-un-mapfile-utilizando-simplificacion-de-geometrias/
published: true
---
En el anterior <a href="http://www.gisandchips.org/2009/11/02/creacion-de-un-mapfile-de-forma-dinamica-al-vuelo/" target="_blank">post</a> creamos un mapfile dinámicamente con PHPMapScript, ahora lo que vamos a ver es como conseguir que la velocidad de carga de este mapfile se reduzca de forma considerable utilizando simplificación de geometrías mediante una función en PostGIS, esperando obtener un mapa con los municipios de toda España.
<p style="text-align: center;">Resultado esperado:<img class="aligncenter size-full wp-image-1082" title="4b024cb4_7df_1" src="http://www.gisandchips.org/wp-content/4b024cb4_7df_1.gif" alt="Mapa simplificado" width="300" height="300" /></p>
<p style="text-align: center;"></p>
El código de generación del mapfile será el mismo que en el  <a href="http://www.gisandchips.org/2009/11/02/creacion-de-un-mapfile-de-forma-dinamica-al-vuelo/" target="_blank">post</a> de creación del mapfile "al vuelo", tan solo habría que cambiar la forma de obtener los datos, para que en lugar de obtenerlos con una simple sentencia sql, llamar a una función que obtuviera las geometrías simplificadas a costa de perder nivel de detalle (inapreciable a la vista).

<!--more-->

Cuando se trata de simplificar geometrías hay que tener en cuenta la dimensión del mapa que queremos obtener para en función de esta dimensión, poder obtener de alguna forma un coeficiente de simplificación que se adecue a la simplificación que pretendemos obtener (es decir, ni que no nos haga casi simplificación, ni que simplifique demasiado).

Para obtener este factor de simplificación, podemos hacer varias operaciones, en este caso realizaremos una operación como la siguiente, dividiremos el resultado de restar a la xmax la xmin de la entensión de la geometria entre el ancho del mapa que queremos obtener.

Es decir, obtenemos la extensión de la geometría que estamos analizando , sacamos al xmax y la xmin, las restamos y el resultado lo dividimos entre el ancho del mapa que queremos obtener:

(xmax(extent(geometria) )- xmin(extent(geometria)))/Ancho del mapa

Creamos una función en sql para obtener el factor de simplificación que deseamos:

[php]
create or replace function fs(float8) returns float8 AS $$
 select (xmax(extent(geometria))-xmin(extent(geometria)))/$1 from ine.municipios;
$$ LANGUAGE SQL;
[/php]

Una vez que tenemos el factor de simplificación obtenido con la función anterior, ya podemos aplicar la simplificación con el factor de simplificación obtenido anteriormente.

Podemos hacer una comparativa del aumento de velocidad entre realizar una consulta sin simplificación y otra con la simplificación, por ejemplo para un mapa bastante grande de 4000px.

1ª consulta: "<strong><em>select geometria  from municipios2</em></strong>"

2ª consulta:"<strong><em>select st_simplify(geometria,factor_simplificacion) from municipios2</em></strong>"

En PostgreSQL, al ejecutar las dos sentencias podemos comparar y obtener:

1ª consulta
<ul>
	<li>Tiempo de ejecución de la consulta: <strong>50,84 segundos</strong></li>
	<li>Numero de vertices: <strong>393666</strong></li>
</ul>
2ª consulta
<ul>
	<li>Tiempo de ejecución de la consulta: <strong>19,51 segundos</strong></li>
	<li>Numero de vertices: <strong>141610</strong></li>
</ul>
De esta manera comprobamos que se produce un aumento en la velocidad a cambio de reducir el nivel de detalle, circunstancia que nos viene bien de cara a la publicación web, ya que no necesitamos un nivel de detalle elevado.

Para constaruir ahora la layer en MapScript, la sentencia SQL para obtener los datos sería como esta:

[php]
$name = $Layer2-&gt;set(&quot;name&quot;,&quot;Municipios&quot;);
$type = $Layer2-&gt;set(&quot;type&quot;,MS_LAYER_POLYGON);
$status = $Layer2-&gt;set(&quot;status&quot;,MS_ON);
//Conexión con nuestra base de datos que va as er de tipo POSTGIS
$Layer2-&gt;setConnectionType(MS_POSTGIS);
//Cadena de conexión a la base de datos donde tenemos nuestros datos

//En primer lugar, debemos obtener el factor de simplificación que obtenemos mediante consulta a postgresql desde php:
$conexion = pg_pconnect(&quot;host=localhost port=5432 dbname=demos user=postgres&quot;);

$sql=&quot;select fs(&quot;.$Map-&gt;width.&quot;)&quot;;
$resultado=pg_exec($conexion,$sql);
$factor=pg_result($resultado,0,0);

$Layer2-&gt;set(&quot;connection&quot;,&quot;user=postgres dbname=demos host=localhost&quot;);
//Filtro en sql que vamos a introducir para sacar nuestros datos a mostrar, aplicando el factor obtenido anteriormente
$data=&quot;geometria from (select st_simplify(geometria,&quot;.$factor.&quot;) as geometria, nombre,provincia,gid from municipios) as foo using SRID=23030, using unique gid&quot;;
$Layer2-&gt;set(&quot;data&quot;,$data);
//Una vez definida la estructura d ela capa procedemos a definir el estilo de nuestra capa número 1
[/php]

Para la comparación de velocidad de creación de un mapfile con una única capa PostGIS utilizando una sentencia SQL normal y utilizando simplificación de geometrías, utilizando para ello varios test de medición de velocidad de carga, asumimos que vamos a crear un mapa bastante grande de una dimensión de 2000x2000 pixeles.

<a href="http://www.gisandchips.org/demos/mapscript/index_norm.php" target="_blank">Sin simplificación de geometrías </a>(tal y como se hizo en el artículo anterior, pero con una sola capa y de tamaño 2000x2000 píxeles)

<a href="http://www.gisandchips.org/demos/mapscript/index_simp.php" target="_blank">Con simplificación de geometrías</a> (utilizando la función que hemos programado anteriormente)

Para realizar la comparativa de cargas he utilizado la herramienta <a href="http://www.websitegoodies.com/tools/speed-test.php" target="_blank">Speed-Test </a>

El resultado que he obtenido es el siguiente:
(no se aprecia mucha diferencia, pero es debido a que se ha guardado en la caché y no es significativa la diferencia)
- Con la sentencia normal (sin simplificación)
<table style="border: 1px solid #cccccc;" border="0">
<tbody>
<tr>
<td style="padding: 0pt 5px;"><strong>URL:</strong></td>
<td style="padding: 0pt 5px;">http://www.gisandchips.org/demos/mapscript/index_norm.php</td>
</tr>
<tr>
<td style="padding: 0pt 5px;"><strong>Load Time:</strong></td>
<td style="padding: 0pt 5px;">1.048 seconds</td>
</tr>
<tr>
<td style="padding: 0pt 5px;"><strong>Page Size:</strong></td>
<td style="padding: 0pt 5px;">0.19 kb</td>
</tr>
</tbody></table>
- Utilizando el factor de simplificación y realizando una simplificación de las geometrías:
<table style="border: 1px solid #cccccc;" border="0">
<tbody>
<tr>
<td style="padding: 0pt 5px;"><strong>URL:</strong></td>
<td style="padding: 0pt 5px;">http://www.gisandchips.org/demos/mapscript/index_simp.php</td>
</tr>
<tr>
<td style="padding: 0pt 5px;"><strong>Load Time:</strong></td>
<td style="padding: 0pt 5px;">1.013 seconds</td>
</tr>
<tr>
<td style="padding: 0pt 5px;"><strong>Page Size:</strong></td>
<td style="padding: 0pt 5px;">0.18 kb</td>
</tr>
</tbody></table>