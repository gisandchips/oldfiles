---
ID: 942
post_title: >
  Creación de un mapa sensible utilizando
  Pl/PgSQL y PHP_Mapscript
author: Miguel
post_date: 2009-11-06 18:46:56
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2009/11/06/creacion-de-un-mapa-sensible-utilizando-plpgsql-y-php_mapscript/
published: true
syntaxhighlighter_encoded:
  - "1"
---
El objetivo no es otro que convertir nuestro mapa en algo más interactivo para el usuario, dando más información que la meramente visual. El siguiente artículo, muestra el camino para alcanzar nuestra meta,  seguramente no sea la única ni la mejor, pero nos será útil para nuestro fín.

Obviaremos la construcción del mapfile, ya que fue abordado en otro <a href="http://www.gisandchips.org/2009/11/02/creacion-de-un-mapfile-de-forma-dinamica-al-vuelo/">artículo</a>, pero trabajaremos a partir de ese código.

Lo primero que necesitamos hacer es preparar la función en lenguaje PL/pgSQL que nos devolverá las coordenadas de los vértices que componen cada uno de los polígonos a partir de un parámetro, que en nuestro caso será el código municipal, y posteriormente se hará la conversión de esos vértices de metros a pixel con PHP.<!--more-->

[sql]
CREATE OR REPLACE FUNCTION gc_listvertices(cod_municipal character varying)
RETURNS SETOF record AS
$BODY$
DECLARE
countpol bigint;
polygon geometry;
perimeter geometry;
countpoints integer;
i integer;
j integer;
point record;
BEGIN
--Teniendo en cuenta que cada municipio puede estar compuesto de varios polígonos, primero hacemos un conteo de los polígonos de cada municipio
select into countpol count(*) from (select st_dump(geometria) from municipios where codmun= cod_municipal) foo;
for j in 1..countpol loop
--Por cada municipio, nos devolverá los polígonos descompuestos. La función st_dump, lo que nos devuelve es tantas filas como polígonos contiene el municipio en cuestión, es decir nos descompone el MULTIPOLYGON en un array que contiene un único POLYGON
select into polygon geom from (select (st_dump(geometria)).path, (st_dump(geometria)).geom from municipios where codmun = cod_municipal) foo where path[1] = j;
--La función st_exteriorring nos devuelve un LINESTRING, que representa el anillo exterior del polígono dado.
--ATENCIÓN: esta función no trabaja con MULTIPOLYGON, de ahí la descomposición anterior
select into perimeter st_exteriorring(polygon);
--Contamos el número de vértices que componen el POLYGON
select into countpoints st_npoints(polygon);
for i in 1..countpoints loop
--Por último, nos guardamos en una variable, la coordenada X y la coordenada Y de cada vértice.
--ATENCIÓN: puesto que nos devuelve indiscriminadamente las coordenadas de todos los vértices de todos los polígonos que componen el municipio, se ha añadido un campo booleano que nos devuelve true en caso de ser un vértice final.
select into point st_x(st_pointn(perimeter, i)), st_y(st_pointn(perimeter, i)), (i=countpoints) as tend;
return next point;
end loop;
end loop;
END;
$BODY$
LANGUAGE 'plpgsql' VOLATILE
COST 100
ROWS 1000;
ALTER FUNCTION gc_listvertices(character varying) OWNER TO postgres;[/sql]

Ahora sólo queda hacer la conversión con PHP:

[php]
&lt;?php header('Content-Type: text/html; charset=UTF-8'); ?&gt;
&lt;html&gt;
&lt;head&gt;
&lt;title&gt;Ejemplo de mapa sensible&lt;/title&gt;
&lt;style type=&quot;text/css&quot;&gt;
.munInfo{
border: 1px solid #C0C0C0;
background-color: white;
border-bottom-width: 0;
width: 170px;
height: 70px;
position: absolute;
}
&lt;/style&gt;

&lt;script&gt;
function info(nombre,area)
{
        document.getElementById('munInfo').innerHTML = &quot;Municipio=&quot;  + nombre +&quot; &lt;br&gt; Area =&quot; + area +&quot; km2&quot;;
}
&lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
&lt;?php
//Este código es el usado en un artículo anterior, pero para hacer más fluido el ejemplo, sólo se han seleccionado los municipios de la Comunidad Valenciana. Lo ideal es automatizar tanto la extensión máxima de la caja de la geometría como su equivalencia en el tamaño en pixeles del mapa. Para hacer el ejemplo más fluido se añaden estos parámetros a mano.
dl('php_mapscript.so');
$Map=ms_newMapObj(&quot;&quot;);
$Map-&gt;set(&quot;name&quot;,&quot;Mapa&quot;);
//Equivalencia en pixeles del tamaño del mapa
$Map-&gt;setSize(432,750);
//Extensión de la caja para los municipios de la Comunidad Valenciana
$Map-&gt;setExtent(626680,4191059,815673.125,4519371);
$Map-&gt;web-&gt;set(&quot;imagepath&quot;,&quot;/tmp&quot;);
$Map-&gt;web-&gt;set(&quot;imageurl&quot;,&quot;/ms_tmp&quot;);
$Layer2=ms_newLayerObj($Map);
$Layer2-&gt;set(&quot;name&quot;,&quot;Municipios&quot;);
$Layer2-&gt;set(&quot;type&quot;,MS_LAYER_POLYGON);
$status = $Layer2-&gt;set(&quot;status&quot;,MS_ON);
$Layer2-&gt;setConnectionType(MS_POSTGIS);
$Layer2-&gt;set(&quot;connection&quot;,&quot;user=****** dbname=demos host=localhost&quot;);
//La consulta ha sido modificada con respecto a la original, para mostrar sólo los municipios de la Comunidad Valenciana
$Layer2-&gt;set(&quot;data&quot;,&quot;geometria from (select geometria,gid from  ine.municipios where com = '10') as foo  using unique gid using srid=23030&quot;);
$clase = ms_newClassObj($Layer2);
$estilo = ms_newStyleObj($clase);
$estilo-&gt;color-&gt;setRGB(217,233,107);
$estilo-&gt;outlinecolor-&gt;setRGB(0,0,0);
$Image=$Map-&gt;Draw();
$url_imagen=$Image-&gt;saveWebImage();
//Añadimos que la imagen será usada como un mapa de html
echo &quot;&lt;img src=&quot;.$url_imagen.&quot; usemap='#sensible'&gt;&quot;;
echo &quot;&lt;map name='sensible'&gt;&quot;;

//Lo siguiente, es conectarse a la base de datos para consultar los municipios que vamos a mostrar en el mapa
$conn = pg_connect(&quot;host=localhost port=5432 dbname=demos user=****** password=******&quot;);
$sqlmun= &quot;select nombre04 as nombre,codmun as codmunicipal, area(geometria) as area from ine.municipios where com = '10'&quot;;
$poligono = pg_query($conn,$sqlmun);
$rowspol=pg_num_rows($poligono);
for ($h=0; $h&lt;$rowspol; $h++)
{
        $arraypol=pg_fetch_array($poligono,$h);
        $nombre=$arraypol['nombre'];
        $codmun=$arraypol['codmunicipal'];
        $area=round(($arraypol['area']/1000000),2);

        $pxmin = 626680;
        $pymin = 4191059;
        $pxmax = 815673.125;
        $pymax = 4519371;

        //distancia en metros de los ejes
        $eqx=$pxmax-$pxmin;
        $eqy=$pymax-$pymin;
//Llegados a este punto ejecutamos la función anterior que creamos en la base de datos, para que nos devuelva las coordenadas de los vértices de cada uno de los polígonos
        $sqlpolcoords=&quot;select * from ine.gc_listvertices('$codmun') as foo(x float, y float,tend boolean)&quot;;
        $polcoords= pg_query($conn,$sqlpolcoords);
        $rowspolcoords=pg_num_rows($polcoords);
        for ($j=0; $j&lt;$rowspolcoords; $j++)
        {
//Obtenemos los tres parámetros por separado que nos devuelve la función y hacemos la conversión de metros a pixel. Como he comentado anteriormente lo ideal es que esto se realice de manera automatizada, sin que introduzcamos nosotros la anchura y la altura del mapa
                $arraypolcoords=pg_fetch_array($polcoords,$j);
                $x=$arraypolcoords['x'];
                $y=$arraypolcoords['y'];
                $tend=$arraypolcoords['tend'];
                $xcal=$x-$pxmin;
                $ycal=$pymax-$y;

                $resx=round(((432*$xcal)/$eqx),0);
                $resy=round(((750*$ycal)/$eqy),0);
                $coordenadas.=$resx.&quot;,&quot;.$resy.&quot;,&quot;;
//Lo que va a hacer este parámetro es cortar cada polígono cada vez que devuelva true, de manera que en el caso de Alicante que son cuatro polígonos, los separa cada uno en un área distinta
                if($tend=='t')
                {
                        echo&quot;&lt;area shape=poly coords='$coordenadas' href=\&quot;javascript:void(0);\&quot; onClick=\&quot;munInfo('$nombre','$area');\&quot; alt='$nombre'&gt;\n&quot;;
//Para el ejemplo he creado una simple función en javascript que nos muestre el nombre y el área en un div cuando pasemos el ratón por encima de cualquier municipio.
                        $coordenadas=&quot;&quot;;
                }
        }
}
echo&quot;&lt;/map&gt;&quot;;
?&gt;
&lt;div id=&quot;munInfo&quot; class=&quot;munInfo&quot;&gt;&lt;/div&gt;
&lt;/body&gt;
&lt;/html&gt;
[/php]

El resultado del ejemplo será este <a href="http://www.gisandchips.org/demos/mapscript/index_sense.php">mapa sensible</a>.