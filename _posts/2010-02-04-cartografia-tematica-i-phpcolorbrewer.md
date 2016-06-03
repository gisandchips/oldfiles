---
ID: 1360
post_title: 'Cartografía temática I: PHPcolorBrewer'
author: jose
post_date: 2010-02-04 15:51:30
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2010/02/04/cartografia-tematica-i-phpcolorbrewer/
published: true
---
La creación de cartografía coroplética es quizás uno de los aspectos más recurrentes que podemos encontrar en un ambiente webgis. Todos hemos visto o realizado alguna vez con nuestro programa de GIS favorito un mapa de distribución de la población o cualquier otra variable por municipios, provincias, autonomías o estados. En su preparación siempre intervienen tres elementos: una base cartográfica con datos asociados para representar, un método estadístico para agrupar los datos en intervalos de clase o conjunto de datos, y finalmente una simbología cartográfica aplicada a dichos grupos que represente de una manera clara el fenómeno que deseamos destacar. Los datos a tratar siempre serán de tipo numérico y harán referencia a datos cuantitativos.

[caption id="attachment_1363" align="aligncenter" width="300" caption="PHPcolorBrewer palettes"]<a title="PHPcolorBrewer palette" href="http://www.gisandchips.org/demos/j3m/plr/PHPcolorBrewer/examples.palettes.PHPcolorBrewer.php" target="_blank"><img class="size-medium wp-image-1363" src="http://www.gisandchips.org/wp-content/paletas.phpColorBrewer-300x204.jpg" alt="" width="300" height="204" /></a>[/caption]

<strong>NOTA</strong>: La segunda parte de este artículo lo puede consultar en: <a href="/2010/02/04/cartografia-tematica-ii-calculo-de-intervalos-de-clase-con-plr/"> Cartografía temática II: Cálculo de intervalos de clase con Pl/R
</a>
<!--more-->

Sobre este tipo de cartografía siempre han circulado muchos tópicos del tipo: <em>"la mejor manera de representar los datos es con un mapa"</em>, con una clara alusión al uso de mapas en detrimento de largos listados o tablas cuya lectura resulta menos intuitiva que un mapa con colores degradados. También encontramos declaraciones con una percepción negativa sobre los mapas temáticos, <em>"la mejor forma de ocultar datos es con un mapa"</em>, aludiendo esta vez al hecho incuestionable de que un fenómeno dado puede dar diferentes lecturas en función del mayor o menor acierto en la elección de los intervalos de clase, en el sentido de que muchos intervalos dificultan la lectura y pocos generalizan demasiado los datos.

Cuando trabajamos con aplicaciones de escritorio GIS nuestra labor se reduce a indicar el fenómeno a representar, el número de intervalos y como mucho elegir una de las gamas de paletas, para finalmente obtener una distribución más o menos aceptable donde es raro que el usuario intervenga en el proceso estadístico que implica decidir las rupturas de clase. De igual forma ocurre con la gama de colores, donde solemos aceptar los predefinidos, o como mucho definimos una degradación de colores.

Quizás sea por deformación profesional, pero suelo advertir una cierta dejadez del usuario respecto al tratamiento de los colores que intervienen en una distribución normalizada en mucha de la cartografía coroplética existente, utilizando, en la mayoría de los casos, los valores que por defecto nos proporciona la aplicación. Para cualquiera que haya estudiado cartografía, sabe que la simbolización del color es uno de los temás más complejos, que muchas veces delegamos en la aplicación de escritorio, en la que intervienen factores tales como la representatividad de los colores, legibilidad, idoneidad en función de su uso (por ejemplo en un mapa pensado para su impresión no todos los colores son imprimibles), adecuación del número de intervalos al fenómeno estudiado, capacidad del usuario para distinguir colores, y por supuesto la elección de un sistema de agrupamiento (clustering) adecuado.

La doctora Cynthia A. Brewer, profesora del Departamento de Geografía en la universidad estatal de Pennsylvania (<a href="http://www.personal.psu.edu/cab38/">http://www.personal.psu.edu/cab38/</a>) ha dedicado su faceta profesional al estudio del color en la cartografía coroplética, dejando como legado una serie de paletas de colores adecuadas para la representación cartográfica que han sido bautizadas con su nombre y utilizadas en muchas publicaciones de renombre, como el Census Atlas of the United States (<a href="http://www.census.gov/population/www/cen2000/censusatlas/">http://www.census.gov/population/www/cen2000/censusatlas/</a>) e implementadas en muchas otras aplicaciones de GIS (ArcGIS)

Basándonos en su trabajo hemos elaborado una sencilla clase en PHP que nos permite hacer uso de dichas paletas de una manera sencilla para el usuario, susceptibles de ser utilizadas en la elaboración de cartografía coroplética.

<a title="Descarga PHPcolorBrewer" href="http://www.gisandchips.org/demos/j3m/plr/PHPcolorBrewer.tar.gz">Descargar PHPcolorBrewer</a>

Esta utilidad está sujeta a la <a href="http://www.gisandchips.org/demos/j3m/plr/PHPcolorBrewer/LICENSE-2.0.txt">Apache License 2.0</a>

Esta clase consta de los siguientes métodos:
<ul>
	<li>listColors: Listar colores de una paleta y un nº de intervalos o clases</li>
	<li>getPalettes: Obtiene un array de paletas</li>
	<li>getPaletteIntervals: Obtiene un array de intervalos de una paleta. Los valores siempre estarán entre de 3 a 9</li>
	<li>getPalIntColors: Obtiene un array de intervalos de una paletas. Siempre será de 3 a 9</li>
	<li>getColor: Obtiene un array con el color de una paleta, un intervalo y un número.</li>
</ul>
Dada la vocación web de esta clase se han creado estos métodos:
<ul>
	<li>comboPalettes: Utilidad HTML para listar paletas en forma de combo</li>
	<li>comboNumInt: Utilidad HTML para listado del nº de intervalos de una paleta en forma de combo</li>
	<li>rgb2html:  Convierte un color en formato RGB a hexadecimal para su uso en web.</li>
	<li>tableColor: Crea una tabla HTML dada una paleta y un intervalo (de 3 a 9)</li>
</ul>
Con el objeto de facilitar al usuario su implementación se han añadido dos scripts con ejemplos de uso:
1. Listado de todas las paletas de colores: <a title="Paletas" href="/demos/j3m/plr/PHPcolorBrewer/examples.palettes.PHPcolorBrewer.php">examples.palettes.colorBrewer.php</a>
2. Ejemplos de uso: <a title="Ejemplos de uso" href="http://www.gisandchips.org/demos/j3m/plr/PHPcolorBrewer/examples.PHPcolorBrewer.php">examples.colorBrewer.php</a>

Finalmente, y como ejemplo de cartografía temática les mostramos algunas demostraciones que utilizan esta utilidad, incluidos en el siguiente post: <a href="/2010/02/04/cartografia-tematica-ii-calculo-de-intervalos-de-clase-con-plr/#demos">Cartografía temática II: Cálculos de intervalos de clase con PL/R</a>