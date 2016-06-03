---
ID: 1629
post_title: >
  La ciencia de los suelos (I). Creación
  de triángulos texturales directamente
  desde una Geodatabase.
author: benizar
post_date: 2010-09-08 06:44:54
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2010/09/08/la-ciencia-de-los-suelos-i-creacion-de-triangulos-texturales-directamente-desde-una-geodatabase/
published: true
---
Llevo un tiempo ocupado pero ya es hora de enseñar en GIS&amp;Chips algunas de las demos que realicé para mi Trabajo Fin de Master (TFM) sobre SIG.

En este artículo os muestro la demo nº2 que consiste en la creación de triángulos texturales directamente desde una <strong>Geodatabase </strong>que diseño y creo previamente dentro del TFM. Pero más adelante añadiré algunos comentarios de las otras demos.

[caption id="attachment_1630" align="aligncenter" width="267" caption="Triángulo textural"]<a href="http://www.gisandchips.org/wp-content/clasificacion_textural.png"><img class="size-medium wp-image-1630" src="http://www.gisandchips.org/wp-content/clasificacion_textural-267x300.png" alt="" width="267" height="300" /></a>[/caption]
<h3><!--more-->El TFM</h3>
Es interesante comentar que el TFM es una pequeña parte de otra investigación que estoy realizando en el contexto de la comarca de la Marina Baja (Alicante-España) y los datos que se muestra en las distintas demos son parte de un muestreo para entender mejor el potencial nutritivo de los suelos de la comarca.

El título del TFM es “Diseño de una base de datos geográfica orientada al estudio de los suelos”. Evidentemente, en el espacio de un TFM no cabía desarrollar toda la idea, que podría ser muy trabajosa para una sola persona, del mismo modo que aquí no cabe desarrollar todo el TFM ya que hay algunas partes que no tienen que ver con lo que queremos contar en G&amp;C.

Si entráis en la página principal del la Web de demostraciones podéis leer un poco más sobre la intención y contenidos del TFM ( <a href="http://www.gisandchips.org/demos/suelosTFM/">http://www.gisandchips.org/demos/suelosTFM/</a> ). Además en esta página se explican los detalles de la tecnología utilizada y del método de trabajo (<strong>PostgreSQL</strong>, <strong>PostGIS</strong>, R, <strong>pl/R</strong>...).

Considero que las demos 2 y 3 son más interesantes, puesto que la primera es solo una función de clasificación que cada uno podría hacer a su gusto, pero si a alguien le interesa la explicaré más adelante.

[caption id="attachment_1645" align="aligncenter" width="300" caption="Página de la Demo"]<a href="http://www.gisandchips.org/wp-content/tritex_1.jpg"><img class="size-medium wp-image-1645" src="http://www.gisandchips.org/wp-content/tritex_1-300x232.jpg" alt="" width="300" height="232" /></a>[/caption]
<h3>Los triángulos texturales</h3>
El estudio de los suelos es una disciplina tremendamente amplia, por lo que aquí nos limitaremos a dar, de un modo más breve, las explicaciones necesarias para entender los gráficos resultantes.

Los <strong>triángulos texturales</strong> son gráficos que facilitan la interpretación de la granulometría del suelo. Así pues, habitualmente la textura del suelo se puede representar en un diagrama ternario. En dicho diagrama ternario se emplean reglas trigonométricas para representar en el plano coordenadas tridimensionales que se refieren a distintas combinaciones de clases granulométricas (clases texturales) cuya suma es constante (100). Por último, las clases texturales son convenientes para representar a la textura del suelo en mapas ya que resulta más sencillo representar una variable que no tres, y su uso es bastante amplio en descripciones y clasificaciones del suelo.

A pesar de todas las ventajas de estos gráficos, aparece la dificultad de que existen gran número de triángulos texturales que permiten la clasificación de los suelos, lo que implica que en caso de querer incluir herramientas relacionadas en una BDGS (Geodatabase de suelos) se deba considerar tal variedad de criterios. En este caso se ha creado un gráfico de texturas según las especificaciones del USDA (1951) en castellano.

Pese a que existe cierta variedad de programas pensados para la elaboración de los triángulos de texturas parece interesante que una BDGS que almacene los datos de textura de suelos sea capaz de generar este tipo de gráficos. Además, los otros software suelen ser difíciles de encontrar y poco flexibles. Aquí podemos modificar nuestro gráfico a placer.

[caption id="attachment_1643" align="aligncenter" width="300" caption="Página de resultados de la demo"]<a href="http://www.gisandchips.org/wp-content/tritex_results.jpg"><img class="size-medium wp-image-1643" src="http://www.gisandchips.org/wp-content/tritex_results-300x188.jpg" alt="" width="300" height="188" /></a>[/caption]
<h3>Demostración  Triángulo Textural</h3>
En la propia página Web (<a href="http://www.gisandchips.org/demos/suelosTFM/paginas/demo_tritex1.php">http://www.gisandchips.org/demos/suelosTFM/paginas/demo_tritex1.php</a> ) se explica como interactuar con la demo, seleccionando las muestras que se desee, si se desea etiquetar las muestras y con que símbolo se van a representar. Como se puede ver, los datos han sido previamente ordenados según una consulta espacial usando funciones de <strong>PostGIS</strong>, por lo que la gráfica que vamos a generar se ve enriquecida por esta posibilidad. A partir de los datos seleccionados se ejecuta una función escrita en pl/R que usa la librería "Plotrix" de R para generar un triángulo textural personalizado y guardarlo como un fichero PNG en el servidor. La función que se usa en este ejemplo puede ser personalizada en muchos sentidos para obtener otros triángulos texturales (hay decenas), en otros idiomas, clasificando los símbolos por algún atributo de la base de datos...

El uso de la función resulta muy sencillo,

[sql]
SELECT  _plr_clasif_textural ( 'arenas' , 'limos' , 'arcilla' , 'submuestra' , 'simbolo' , 'etiquetas') ;
[/sql]

y a continuación podemos ver el código que hemos usado para crearla, y algunos comentarios:

[sql]
-- Function: _plr_clasif_textural(text, text, text, text, text, text)
-- DROP FUNCTION _plr_clasif_textural(text, text, text, text, text, text);
CREATE OR REPLACE FUNCTION _plr_clasif_textural(text, text, text, text, text, text)
RETURNS text AS
$BODY$
[/sql]

[shell]
# Cargamos las librerias necesarias
library(Cairo)
library(plotrix)
# Definimos la funcion
clasif.textural = function (soiltexture = NULL, at = seq(0.1, 0.9, by = 0.1),
axis.labels = c(&quot;% arena (entre 0,05 y 2 mm)&quot;, &quot;% limo (entre 0,05 y 0,002 mm)&quot;, &quot;% arcilla (menor de 0,002  mm)&quot;),
tick.labels = list(l = seq(10, 90, by = 10), r = seq(10,
90, by = 10), b = seq(10, 90, by = 10)), show.names = TRUE,
show.lines = TRUE, col.names = &quot;black&quot;, bg.names = par(&quot;bg&quot;),
show.grid = TRUE, col.axis = &quot;black&quot;, col.lines = &quot;black&quot;,
col.grid = &quot;gray&quot;, lty.grid = 3, show.legend = FALSE, label.points = FALSE,
point.labels = '', col.symbols = &quot;blue&quot;, pch = par(&quot;pch&quot;),
...)
{
par(xpd = TRUE)
plot(0.5, type = &quot;n&quot;, axes = FALSE, xlim = c(0, 1), ylim = c(0,
1), main = NA, xlab = NA, ylab = NA)
triax.frame(at = at, axis.labels = axis.labels,
tick.labels = tick.labels, col.axis = col.axis, show.grid = show.grid,
col.grid = col.grid, lty.grid = lty.grid)
arrows(0.12, 0.41, 0.22, 0.57, length = 0.15)
arrows(0.78, 0.57, 0.88, 0.41, length = 0.15)
arrows(0.6, -0.1, 0.38, -0.1, length = 0.15)
if (show.lines) {
triax.segments &lt;- function(h1, h3, t1, t3, col) {
segments(1 - h1 - h3/2, h3 * sin(pi/3), 1 - t1 -
t3/2, t3 * sin(pi/3), col = col)
}
h1 &lt;- c(85, 70, 80, 52, 52, 50, 20, 8, 52, 45, 45, 65,
45, 20, 20)/100
h3 &lt;- c(0, 0, 20, 20, 7, 0, 0, 12, 20, 27, 27, 35, 40,
27, 40)/100
t1 &lt;- c(90, 85, 52, 52, 43, 23, 8, 0, 45, 0, 45, 45,
0, 20, 0)/100
t3 &lt;- c(10, 15, 20, 7, 7, 27, 12, 12, 27, 27, 55, 35,
40, 40, 60)/100
triax.segments(h1, h3, t1, t3, col.lines)
}
if (show.names) {
xpos &lt;- c(0.5, 0.7, 0.7, 0.73, 0.73, 0.5, 0.275, 0.275,
0.27, 0.27, 0.25, 0.135, 0.18, 0.07, 0.49, 0.72,
0.9)
ypos &lt;- c(0.66, 0.49, 0.44, 0.36, 0.32, 0.35, 0.43, 0.39,
0.3, 0.26, 0.13, 0.072, 0.032, 0.024, 0.18, 0.15,
0.06) * sin(pi/3)
snames &lt;- c(&quot;arcillosa&quot;, &quot;arcillosa&quot;, &quot;limosa&quot;, &quot;franco arcillosa&quot;, &quot;limosa&quot;,
&quot;franco arcillosa&quot;, &quot;arcillosa&quot;, &quot;arenosa&quot;, &quot;franco arcillosa&quot;, &quot;arenosa&quot;,
&quot;franco arenosa&quot;, &quot;arenosa&quot;, &quot;franca&quot;, &quot;arenosa&quot;, &quot;franca&quot;, &quot;franco limosa&quot;,
&quot;limosa&quot;)
boxed.labels(xpos, ypos, snames, border = FALSE, col = col.names,
cex=0.8, xpad = 0.5)
}
par(xpd = FALSE)
if (is.null(soiltexture))
return(NULL)
soilpoints &lt;- triax.points(soiltexture, show.legend = show.legend,
label.points = label.points, point.labels = point.labels,
col.symbols = col.symbols, pch = pch, ...)
invisible(soilpoints)
}
# Definimos la imagen
CairoPNG(&quot;…/images/clasificacion_textural.png&quot;, width=500, height=560)
# Construimos las consultas
select = 'select '
arena = paste(arg1, ',', sep=' ');
limo = paste(arg2, ',', sep=' ');
arcilla = paste(arg3, ' ');
from = ' from '
tabla = arg4
selection = paste(select, arena, limo, arcilla, from, tabla, sep='');
selectgids = paste('select gid', from, tabla, sep='');
# Ejecutamos las consultas
texturas &lt;- pg.spi.exec(selection)
gids&lt;- pg.spi.exec(selectgids)
# Ejecutamos la funcion creada mas arriba
soiltex.return&lt;-clasif.textural(texturas,
pch=arg5, point.labels = gids[,], label.points=arg6)
dev.off()
# Damos permisos de lectura
system('chmod go+r …/images/clasificacion_textural.png');
# Una comprobacion para asegurarnos de que se ha creado la grafica, pero no afecta para nada.
if (file.exists('…/images/clasificacion_textural.png')) {
print ('Grafico realizado. Se llama …/images/clasificacion_textural.png')};
[/shell]

[sql]
$BODY$
LANGUAGE 'plr' VOLATILE
COST 100;
ALTER FUNCTION _plr_clasif_textural(text, text, text, text, text, text) OWNER TO postgres;
[/sql]
<h3>Algunas notas sobre todo esto:</h3>
(1) En el TFM se apunta que en breve aparecerá una librería creada por Julien Moeys ( <a href="https://r-forge.r-project.org/projects/soiltexture/">https://r-forge.r-project.org/projects/soiltexture/</a> ) que hará innecesario tener que crear la función por nosotros mismos como hemos hecho en esta demostración. Será interesante comentar con los miembros del proyecto la posibilidad de compartir las funciones de su proyecto en una Web completamente dedicada al cálculo de triángulos texturales, donde los usuarios puedan añadir sus propios datos.

(2) Los datos de ejemplo están un poco modificados pero se aproximan a la realidad.

(3) La función tiene muchos aspectos mejorables, por lo menos en cuanto a la legibilidad. Otra posibilidad sería la de que no todos los argumentos de entrada fuesen de tipo texto. Esto lo veremos plasmado en el código de la demo 3.

(4) Si en la página  inicial de la demo no se ve una lista con las muestras disponibles (como en la segunda imagen) será porqué el servidor de la base de datos ha caído y lo deberemos reiniciar en breve. Solo ha pasado una vez, pero quien sabe…

(5) Este artículo destaca la potencia de pl/R a la hora de crear gráficos de lo más complejos.
<h3>Agradecimientos:</h3>
El TFM de que he estado hablando ha sido dirigido por Toni Hernández, en el Máster en Sistemas de Información Geográfica y Teledetección de la Universitat de Girona (UNIGIS), y fue presentado en Mayo de 2010. Les estoy muy agradecido a todos los miembros, tanto alumnos como profesores, por estos últimos años disfrutando y aprendiendo SIG.

Especialmente, quiero agradecer a los compañeros de G&amp;C por la ayuda con instalaciones, creación de la Web y otros dolores de cabeza.

——————————————————————

Si queréis contactar podéis enviarme un email (asunto: gisandchips):

Benito M. Zaragozí

benito.zaragozi@ua.es