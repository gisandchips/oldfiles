---
ID: 1380
post_title: 'Cartografía temática II: Cálculo de intervalos de clase con PL/R'
author: jose
post_date: 2010-02-04 15:53:22
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2010/02/04/cartografia-tematica-ii-calculo-de-intervalos-de-clase-con-plr/
published: true
---
[caption id="attachment_1416" align="aligncenter" width="300" caption="Población por municipios de Madrid (2008) "]<a href="http://ec2-54-228-172-21.eu-west-1.compute.amazonaws.com/demos/j3m/plr/test5.php"><img class="size-medium wp-image-1416" src="http://www.gisandchips.org/wp-content/madrid-300x228.png" alt="Población por municipios de Madrid (2008)" width="300" height="228" /></a>[/caption]

Como ya vimos en la <a href="http://www.gisandchips.org/2010/02/04/cartografia-tematica-i-phpcolorbrewer/">primera parte</a> de esta mini-serie dedicada a la cartografía temática, hasta ahora sólo tenemos resuelta una parte, la asignación de paletas de colores para nuestros mapas o semiología cartográfica, pero queda otra parte por resolver, la lógica estadística, es decir, calcular cuales serán los valores de cada uno de los intervalos de clase que intervienen en una distribución numérica.
<!--more-->
Como ya es habitual en<a href="http://www.gisandchips.org"> </a><a href="http://www.gisandchips.org">www.gisandchips.org</a> tenemos una especial predilección por almacenar nuestra fuente de datos en tablas espaciales contenidas en una base de datos PostgreSQL-PostGIS, y esta vez no va a ser menos.

Generalmente, el cálculo de los puntos de ruptura de una distribución ha sido tratado habitualmente desde el punto de vista del cliente (<em>gvSIG, Quantum GIS, ArcGIS</em>, etc), descargándose a la aplicación  los datos oportunos (shapefiles, PostGIS, etc) para escoger posteriormente algún método de agrupamiento o clustering. Esta solución resulta apropiada cuando los datos a representar proceden de diferentes fuentes, cambios de geometría constantes realizamos muchos temáticos distintos, o cualquier otra variable a considerar. Pero sin embargo, cuando necesitamos representar la misma realidad espacial (municipios, provincias, comunidades autónomas, continentes, etc) desde diferentes aspectos (demografía, economía, agriculturas, etc) no resulta tán cómodo delegar esta función a la aplicación cliente.

Pongamos un ejemplo para clarificar el tema: Imaginemos que disponemos de datos de población agregados a nivel de municipios, donde existen tantas columnas como censos de población se hayan hecho, y deseamos generar en un portal de Internet mapas temáticos a requerimiento del usuario (año del censo, nº de clases, método de agrupamiento o <em>clustering </em>y paleta de colores). Crear un mapa coroplético de la población lleva implícito un conocimiento de la distribución de los datos (valores mínimos y máximos de la serie, media), y sobre todo las medidas de dispersión o de variabilidad: desviación típica, varianza, co-varianza, coeficiente de correlación de Pearson. Con estos parámetros estamos en condiciones de definir cual será el método de cálculo de intervalo de clase más apropiado para nuestra distribución, para posteriormente asignar la paleta de colores acorde al fenómeno a representar (este último aspecto ya lo tenemos solucionado con nuestra clase PHPcolorBrewer). Con una aplicación de GIS de escritorio sólo podemos realizar un mapa para visualizarlo en pantalla o imprimirlo, pero en un ambiente web(GIS) este forma de actuar carece de lógica, pues estamos limitando al usuario la posibilidad de elegir estos parámetros (método de clustering, número de clases y paleta de colores), ya que es muy complicado obtener datos de un GIS y realizar los cálculos de los intervalos de clases en la própia página web (<em>JavaScript, PHP</em>).

[caption id="attachment_1432" align="aligncenter" width="300" caption="Flujo cartografía temática"]<a href="http://www.gisandchips.org/wp-content/plr.png"><img class="size-medium wp-image-1432" src="http://www.gisandchips.org/wp-content/plr-300x175.png" alt="Flujo cartografía temática" width="300" height="175" /></a>[/caption]

En este artículo vamos a solucionar esta cuestión proponiendo un método que nos permita desde el servidor, es decir, desde el gestor de base de datos, calcular los intervalos de clase. Para ello no nos queda más remedio que recurrir a la creación de funciones o procedimientos almacenados que contemplen dicha lógica. Realizarlo con <em>PLPg/SQL</em> es una solución, pero programar estas funciones requiere de mucho esfuerzo al no formar parte de las funciones nativas del servidor (por ejemplo, no existe una función que calcule la desviación típica). ¿Por qué no utilizamos un lenguaje especializado en el manejo estadístico? En este sentido, <em><strong>PL/R</strong></em> reúne todas las condiciones para convertirse en el lenguaje de programación elegido. Una prueba de la potencia de este lenguaje, en este caso <strong>R</strong> lo podemos ver con este ejemplo.

<strong>Implementación en R</strong>
Tenemos la población activa (valores en miles) de todas las provincias españolas, obtenidas del INE, en el año 1996, disponibles en un objeto vector:
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">c(126.4,137.9,570.9,192.1,409.7,60.2,251.3,325.9,2109.1,145.8,159.4,425.5,207.8,194.3,
172.5,301.5,443,69,252.4,300,55.2,296.4,158.2,84.5,244.3,200.3,141.7,174.2,2180.7,
475.8,438,224.6,161.4,71.2,320.8,390,104.7,144.3,328.3,56.5,646.5,36.3,242,52.8,196.5,
919.6,211.6,473.9,70.2,356.7,50)</td>
</tr>
</tbody>
</table>
En una sesión de R se crearía de esta manera
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">poblacion.1996 &lt;- c(126.4,137.9,570.9,192.1,409.7,60.2,251.3,325.9,2109.1,145.8,159.4,
425.5,207.8,194.3,172.5,301.5,443,69,252.4,300,55.2,296.4,158.2,84.5,244.3,200.3,141.7,
174.2,2180.7,475.8,438,224.6,161.4,71.2,320.8,390,104.7,144.3,328.3,56.5,646.5,36.3,
242,52.8,196.5,919.6,211.6,473.9,70.2,356.7,50)</td>
</tr>
</tbody>
</table>
Y se podría ver simplemente escribiendo:

[shell]poblacion.1996[/shell]


Resultado
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">[1]  126.4  137.9  570.9  192.1  409.7   60.2  251.3  325.9 2109.1  145.8
[11]  159.4  425.5  207.8  194.3  172.5  301.5  443.0   69.0  252.4  300.0
[21]   55.2  296.4  158.2   84.5  244.3  200.3  141.7  174.2 2180.7  475.8
[31]  438.0  224.6  161.4   71.2  320.8  390.0  104.7  144.3  328.3   56.5
[41]  646.5   36.3  242.0   52.8  196.5  919.6  211.6  473.9   70.2  356.7
[51]   50.0</td>
</tr>
</tbody>
</table>
Para ver un sumario estadístico básico de dicho objeto escribimos:

[shell]summary(poblacion.1996)[/shell]

Con este resultado:
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">Min. 1st Qu.  Median    Mean 3rd Qu.    Max.
36.3   139.8   207.8   320.8   342.5  2181.0</td>
</tr>
</tbody>
</table>
Entrando ya en materia, vamos a calcular 5 clases distribuidas por el método de intervalos iguales, es decir, <em>(valor máximo- valor mínimo) / nº intervalos</em>:

[shell]seq(min(poblacion.1996),max(poblacion.1996),length.out=(5+1))[/shell]


Resultado
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">[1]   36.30  465.18  894.06 1322.94 1751.82 2180.70</td>
</tr>
</tbody>
</table>
Es decir, las clases quedarían agrupadas de esta forma:
<ul>
	<li>clase 1: 36.30 - 465.18</li>
	<li>clase 2: 465.18 - 894.06</li>
	<li>clase 3: 894.06 - 1322.94</li>
	<li>clase 4: 1322.94 - 1751.82</li>
	<li>clase 5: 1751.82 - 2180.70</li>
</ul>
Este método, aunque sencillo de implementar (incluso desde la parte cliente) ofrece una clasificación muy distorsionada cuando los valores son muy dispares, por lo que necesitamos recurrir a otros métodos más eficientes, como son los cuantiles. En R este cálculo queda reducido a la mínima expressión:

[shell]quantile(poblacion.1996)[/shell]


Resultado:
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">0%    25%    50%    75%   100%
36.3  139.8  207.8  342.5 2180.7</td>
</tr>
</tbody>
</table>
Esta sería nuestra clasificación
<ul>
	<li>1º quantil: 36.3 - 139.8</li>
	<li>2º quantil: 139.8 - 207.8</li>
	<li>3º quantil: 207.8 - 342.5</li>
	<li>4º quantil: 342.5 - 2180.7</li>
</ul>
¡Más sencillo imposible!

El problema de trabajar directamente en R estriba en la dificultad para obtener datos desde otras fuentes, aunque siempre podemos recurrir a paquetes externos para traer los datos de shapefiles o tablas de PostgreSQL a R

<strong>Implementación en PostgreSQL</strong>

Como ya hemos visto en anteriores artículos sobre PL/R, esta lenguaje procedural es muy sencillo de utilizar. Sólo hay que copiar el código de R y realizar pequeños cambios para adaptarlo a la sintaxis de PL/R.

Un aspecto que debemos tener muy claro es que casi todos los cálculos en R de una distribución deben de estar dispuestos en forma de un vector. Como es lógico, PostgreSQL no soporta este tipo de dato, pero el lenguaje PL/R si que nos permite una conversión hacia el tipo de datos "array", ofreciéndonos además una función que realiza todo el trabajo (<em>array_accum</em>)

Por ejemplo, si queremos obtener un array con los datos de la poblacion activa en 1996 (columna t1996t1) de la tabla ine.poblacion_activa, esta es la forma de conseguirlo con SQL:

[sql]SELECT array_accum(t1996t1) FROM ine.poblacion_activa;[/sql]


Y este el resultado:
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">{126.4,137.9,570.9,192.1,409.7,60.2,251.3,325.9,2109.1,145.8,
159.4,425.5,207.8,194.3,172.5,301.5,443,69,252.4,300,55.2,296.4,
158.2,84.5,244.3,200.3,141.7,174.2,2180.7,475.8,438,224.6,161.4,
71.2,320.8,390,104.7,144.3,328.3,56.5,646.5,36.3,242,52.8,196.5,
919.6,211.6,473.9,70.2,356.7,50}</td>
</tr>
</tbody>
</table>
Con esta pequeña aclaración ya podemos utilizar cualquier columna con datos numéricos para realizar funciones de clustering. Las siguientes funciones de PL/R reproducen los cálculos realizados anteriormente con R:

Función para el método de intervalos iguales:

[sql]
CREATE OR REPLACE FUNCTION r_equal(double precision[], integer)
RETURNS double precision[] AS
$BODY$
sort(seq(min(arg1),max(arg1),length.out=(arg2+1)))

$BODY$
LANGUAGE 'plr' VOLATILE STRICT
COST 100;
[/sql]


Vamos a analizar esta función.

Tiene dos parámetros de entrada:
<ul>
	<li>double precision[]: El array con los datos de entrada que queremos utilizar para calcular los intervalos, utilizando la función array_accum</li>
	<li>integer: el nº de intervalos de clase que deseamos obtener</li>
</ul>
El resultado (RETURNS) será siempre una estructura de tipo array.
El cuerpo de la función se reduciría a una sola línea que recibe los dos argumentos de la función (arg1 y arg2):
<em>sort(seq(min(arg1),max(arg1),length.out=(arg2+1)))</em>

El uso de esta función es muy sencillo:

[sql]SELECT r_equal( array_accum(t1996t1), 5 ) FROM ine.poblacion_activa;[/sql]


Nos devolverá el siguiente array:
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">{36.3,465.18,894.06,1322.94,1751.82,2180.7}</td>
</tr>
</tbody>
</table>
Para aquellos más versados en PLPg/SQL habrán advertido que realizar la función en este lenguaje requiere de algunas líneas más de código, que pueden ser bastantes más si queremos utilizar otros métodos, como el de cuantiles, que reproducimos a continuación.

[sql]
CREATE OR REPLACE FUNCTION r_quantile(double precision[], integer)
RETURNS double precision[] AS
$BODY$
quantile(arg1,probs = seq(0, 1, 1/arg2))
$BODY$
LANGUAGE 'plr' VOLATILE STRICT
COST 100;
[/sql]


Ejemplo de uso para 5 intervalos de clase:

[sql]
SELECT r_quantile(array_accum(t1996t1),5) FROM ine.poblacion_activa;[/sql]


Devuelve:
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="576" valign="top">{36.3,104.7,174.2,251.3,409.7,2180.7}</td>
</tr>
</tbody>
</table>
Ahora que tenemos implementado un método para calcular los intervalos de clase ya estamos en condiciones de desarrollar un auténtico motor de cartografía temática (corológica) en un ambiente web que combine la capacidad de cálculo de PL/R con la semiología cartográfica implementada en PHPcolorBrewer que sea capaz de "pintar" geometrias de una tabla espacial de PostGIS. El resultado de esta coctelera explosiva lo podeis ver en las siguientes demostraciones on-line:

<a name="demos"><strong>EJEMPLOS</strong></a>

<strong>Test 1: <a href="http://ec2-54-228-172-21.eu-west-1.compute.amazonaws.com/demos/j3m/plr/test1.php">Distribución de la población activa por provincias (valores en miles de personas)</a></strong>
Este sencillo ejemplo nos permite ver las posibilidades de integración de varias piezas de software (PostGIS, PL/R. PHP MapScripts y PHPcolorBrewer). Los cálculos de los intervalos de clase utilizan el método "Kmeans", y el usuario tiene la posibilidad de cambiar la paleta de colores y el nº de clases, así como el año que desea consultar.

<strong>Test 2: <a href="http://ec2-54-228-172-21.eu-west-1.compute.amazonaws.com/demos/j3m/plr/test2.php">Distribución de la tasa de ocupación/desempleo por Comunidades Autónomas(valores en miles de personas)</a></strong>
Similar al anterior, pero aplicado a CC.AA. Ofrece la posibilidad de múltiples consultas (número de ocupados/desempleados diferenciando hombre, mujeres y total)

<strong>Test 3: <a href="http://ec2-54-228-172-21.eu-west-1.compute.amazonaws.com/demos/j3m/plr/test3.php">Distribución de la población activa por provincias (valores en miles de personas) según diferentes métodos de <em>clustering</em></a></strong>
Este ejemplo es similar al primero, pero hemos añadido la posibilidad de que el usuario selecciones el método de cálculo de intervalos de clase desea. Es sin duda, un buen ejemplo de la potencia que tiene PL/R al permitirnos elegir hasta 8 métodos diferentes:
<ul>
	<li><a href="http://ec2-54-228-172-21.eu-west-1.compute.amazonaws.com/demos/j3m/plr/test3.php?paletas=YlOrRd&amp;numInt=5&amp;field=t2000t1&amp;method=quantile">quantile</a></li>
	<li><a href="http://ec2-54-228-172-21.eu-west-1.compute.amazonaws.com/demos/j3m/plr/test3.php?paletas=YlOrRd&amp;numInt=5&amp;field=t2000t1&amp;method=equal">equal</a></li>
	<li><a href="http://ec2-54-228-172-21.eu-west-1.compute.amazonaws.com/demos/j3m/plr/test3.php?paletas=YlOrRd&amp;numInt=5&amp;field=t2000t1&amp;method=kmeans">kmeans</a></li>
	<li><a href="http://ec2-54-228-172-21.eu-west-1.compute.amazonaws.com/demos/j3m/plr/test3.php?paletas=YlOrRd&amp;numInt=5&amp;field=t2000t1&amp;method=hclust.complete">hclust.complete</a></li>
	<li><a href="http://ec2-54-228-172-21.eu-west-1.compute.amazonaws.com/demos/j3m/plr/test3.php?paletas=YlOrRd&amp;numInt=5&amp;field=t2000t1&amp;method=hclust.single">hclust.single</a></li>
	<li><a href="http://ec2-54-228-172-21.eu-west-1.compute.amazonaws.com/demos/j3m/plr/test3.php?paletas=YlOrRd&amp;numInt=5&amp;field=t2000t1&amp;method=bclust">bclust</a></li>
	<li><a href="http://ec2-54-228-172-21.eu-west-1.compute.amazonaws.com/demos/j3m/plr/test3.php?paletas=YlOrRd&amp;numInt=5&amp;field=t2000t1&amp;method=fisher">fisher</a></li>
	<li><a href="http://ec2-54-228-172-21.eu-west-1.compute.amazonaws.com/demos/j3m/plr/test3.php?paletas=YlOrRd&amp;numInt=5&amp;field=t2000t1&amp;method=jenks">jenks</a></li>
</ul>
<strong>Test 4: <a href="http://ec2-54-228-172-21.eu-west-1.compute.amazonaws.com/demos/j3m/plr/test4.php">Varios mapas temáticos en una sóla página con todos los métodos anteriores</a></strong>
El objeto de este ejemplo es demostrar de una sóla vez como se representa un mismo fenómeno desde diferentes métodos de clustering, para ver cual es el que mejor se adapta a nuestras necesidades.

<strong>Test 5: <a href="http://ec2-54-228-172-21.eu-west-1.compute.amazonaws.com/demos/j3m/plr/test5.php">Mapa temático municipal sensible con HTML dinámico</a></strong>
Se trata del ejemplo más completo de todos, que nos permite seleccionar una provincia y ver la distribución de la población (total, hombres o mujeres) por municipio. También permite seleccionar todos los métodos indicados, así como la paleta de colores y nº de intervalos.

<strong>Test 6: <a href="http://ec2-54-228-172-21.eu-west-1.compute.amazonaws.com/demos/j3m/plr/test6.php">Todas las paletas PHPcolorBrewer aplicadas a un mapa temático</a></strong>
El objetivo de este ejemplo es demostrar en una sóla página como un mismo fenómeno es representado con todas las paleta de PHPcolorBrewer, para que el usuario seleccione la que más le convenga.