---
ID: 2308
post_title: >
  Predicción del crecimiento de núcleos
  urbanos mediante autómatas celulares
author: Jorge Piera Llodrá
post_date: 2013-05-28 17:30:03
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2013/05/28/prediccion-del-crecimiento-de-nucleos-urbanos-mediante-automatas-celulares/
published: true
---
La predicción del crecimiento de núcleos urbanos es un trabajo de investigación muy complejo que ha avanzado mucho en los últimos años con el aumento en la velocidad de cómputo y el avance en las tecnologías GIS. El principal problema de la predicción del crecimiento de los núcleos urbanos es que además de un problema espacial, tiene una componente temporal que dificulta enormemente su trabajo con GIS convencionales. Por este motivo se ha tenido que buscar una tecnología que se adapte mejor a este modelo espacial y temporal de las ciudades: los autómatas celulares.

En la actualidad no existe un método consensuado para determinar los patrones de crecimiento de las ciudades. Ni existe un método, ni existe un consenso en los parámetros que se tienen que usar para poder predecir la forma que tendrá la ciudad dentro de unos años.

El problema de capturar el dinamismo de los sistemas urbanos es uno de los problemas más complicados del modelado urbanístico. En este modelado intervienen variables de todos los tipos, incluyendo variables territoriales, económicas, sociales, culturales, etc. que complican enormemente la elaboración de un modelo preciso.

La aparición de los Sistemas de Información Geográfica (GIS) ha dado lugar a una nueva herramienta que facilita enormemente los estudios para tratar de entender el desarrollo de los núcleos urbanos. Pero al ser un problema de características espaciales y temporales, es muy complicado resolverlo utilizando un modelo clásico de GIS <a href="http://www.sciencedirect.com/science/article/pii/S0198971599000150">[1]</a>.

Debido al problema del dinamismo, a mediados del siglo pasado se empezaron a buscar otras herramientas de modelado que se adaptasen mejor a este tipo de problema. Tobler <a href="http://www.geog.ucsb.edu/~tobler/publications/pdf_docs/geog_analysis/ComputerMovie.pdf2080656,d.ZG4&amp;cad=rja">[2]</a>, en el año 1970, realizó el primer intento de modelado urbanístico mediante autómatas celulares dando lugar a una línea de desarrollo que no ha parado hasta la actualidad.

Este artículo es un estado del arte del modelado de núcleos urbanos mediante autómatas celulares. Primero se define en detalle qué es un autómata celular genérico para posteriormente definir qué es un autómata celular geográfico. En el siguiente apartado se comentan los dos casos concretos de uso de autómatas celulares que se han encontrado en los artículos de investigación.

En los siguientes apartados se mostrarán ejemplos concretos de los modelados urbanísticos más recientes y por último se detallarán las conclusiones de este trabajo.

[caption id="attachment_2322" align="aligncenter" width="350"]<a href="http://upload.wikimedia.org/wikipedia/commons/e/e5/Gospers_glider_gun.gif"><img class="size-full wp-image-2322" alt="" src="http://upload.wikimedia.org/wikipedia/commons/e/e5/Gospers_glider_gun.gif" width="350" height="180" /></a> Ejemplo de autómata celular (fuente: Wikipedia)[/caption]

<!--more-->
<h2>Autómatas celulares</h2>
El primer autor en introducir los autómatas celulares (en adelante CA) fue Von Newnam en 1940 <a href="http://cba.mit.edu/events/03.11.ASE/docs/VonNeumann.pdf">[3]</a> pero el sistema no se hizo famoso hasta la publicación en 1977 del Juego de la Vida <a href="http://conwaysgameoflife.appspot.com/">[4]</a> por parte de John Horton Conway.

Los CA fueron desarrollados originalmente para resolver problemas de física y de biología y en un estudio posterior, Wolfram <a href="http://gbic.biol.rug.nl/~rbreitling/dagstuhlpublications/006_Wolfram1984.pdf">[5]</a> demostró que se podían modelar complejos fenómenos naturales con CA.

Existen muchas definiciones para los CA pero una de las más comúnmente aceptadas es definirlos como un sistema discreto en el espacio y en el tiempo que opera en un espacio de celdas uniforme mediante reglas [<a href="http://www.geocomputation.org/2005/Alkheder.pdf">6</a>]. En detalle, Un CA es un proceso computacional interactivo formado por 4 componentes:
<ul>
	<li>Celdas: son las celdas que forman el espacio del autómata y que no son más que una cuadrícula de celdas que dividen el espacio de forma discreta.</li>
	<li>Estados (S): cada uno de los posibles estados que puede tomar una celda. Normalmente los estados definen cómo se tiene que visualizar la celda.</li>
	<li>Vecinos (N): conjunto de vecinos que tiene una celda y que puede variar en función del tipo de autómata. Dos de los conjuntos de vecinos más usados son los que tienen en cuenta sólo las celdas izquierda/derecha/arriba/abajo (vecinos de Von Neumann) y los que tratan las diagonales (vecinos de Moore). La siguiente imagen muestra un ejemplo.</li>
	<li>Reglas de transición (T): reglas que se aplican a cada celda del autómata en un instante de tiempo y que en función del estado de la celda y de los estados de sus celdas vecinas obtiene un nuevo valor de estado para la celda.</li>
</ul>
[caption id="attachment_2321" align="aligncenter" width="526"]<a href="http://www.gisandchips.org/wp-content//cel1.png"><img class="size-full wp-image-2321" alt="Celdas de una autómata celular. Ejemplo de los dos conjuntos de vecinos más utilizados." src="http://www.gisandchips.org/wp-content//cel1.png" width="526" height="212" /></a> Celdas de una autómata celular. Ejemplo de los dos conjuntos de vecinos más utilizados.[/caption]

El estado de de una celda concreta en un momento determinado se denomina Automatón (A) y se define a partir del conjunto de estados del autómata, una función de transición y los vecinos de la siguiente forma [<a href="http://www.tandfonline.com/doi/abs/10.1080/13658810512331325139?journalCode=tgis20#.UYaXA0kcAQ8">7</a>]:
<p style="text-align: center">At+1 = Ft (S, T, N )</p>
Supongamos un CA muy sencillo de tres filas y tres columnas con dos estados Blanco y Negro que definen el color de una celda. Supongamos además una función de transición que cambia al estado Negro si el vecino superior es Negro y en cualquier otro caso, cambia al estado Blanco. Para un estado inicial en el que sólo la celda de la primera fila y segunda columna se encuentra en estado Negro el CA evolucionará tal y como muestra la siguiente imagen:

[caption id="attachment_2322" align="aligncenter" width="463"]<a href="http://www.gisandchips.org/wp-content//cel2.png"><img class="size-full wp-image-2322" alt="" src="http://www.gisandchips.org/wp-content//cel2.png" width="463" height="127" /></a> Evolución de un autómata celular sencillo. La función de transición cambia a estado Negro si el vecino superior es Negro y en cualquier otro caso, cambia al estado Blanco.[/caption]

El CA del ejemplo anterior es muy sencillo y se ha definido con el propósito de entender qué es un CA. Los CA utilizados para modelar los núcleos urbanos son infinitamente más complejos y se definen en detalle en el siguiente apartado.
<h2>Autómatas celulares geográficos</h2>
Los primeros estudios de modelados de núcleos urbanos ya definieron un conjunto de estados que tenían que tener los CA. [<a href="http://www.geog.ucsb.edu/~tobler/publications/pdf_docs/geog_analysis/CellularGeog.pdf">8</a>] definió los posibles estados del autómata como Residencial (R), Comercial (C), Industrial (I), Público (P) y Agrícola (A). Además de los estados, [8] describe cómo deberían ser las funciones de transición. Por ejemplo, la siguiente imagen muestra la evolución de una celda en estado agrícola a una celda en estado comercial.

[caption id="attachment_2323" align="aligncenter" width="384"]<a href="http://www.gisandchips.org/wp-content//cel3.png"><img class="size-full wp-image-2323" alt="Denicion de una función de transición de un autómata celular geográfico" src="http://www.gisandchips.org/wp-content//cel3.png" width="384" height="131" /></a> Definición de una función de transición de un autómata celular geográfico.[/caption]

Teniendo en cuenta que se han definido 5 estados diferentes y que cada regla considera únicamente 5 celdas como parámetros de entrada se tienen un total de Sn = 55 = 3125 posibles funciones de transición para un autómata definido de esta forma. Estas funciones de transición se podrán simplificar ya que hay muchas que son simétricas y muchas otras que se podrán unir ya que no todos los estados vecinos tienen relevancia en la función de transición.

Esta primera aproximación de CA sirve como punta de partida para entender el concepto de autómata celular geográfico pero se ha simplificado tanto el problema que su uso en el mundo real lo hace del todo todo impreciso. Asumir que un núcleo urbano se puede definir como una cuadrícula perfecta en la que cada celda pertenece a un estado no se corresponde con la realidad y tener en cuenta únicamente los vecinos siguiendo una aproximación de Von Neumann es demasiada simplificación.

El problema de la simplificación del espacio en celdas se resuelve a día de hoy mediante ortofotos. La mayoría de autores realizan estudios con imágenes ráster y asocian celdas del autómata con píxeles de la fotografía que se intentan categorizar en estados mediante técnicas de teledetección de forma que en el estado inicial, las celdas del autómata son los píıxeles de la ortofoto. La asociación entre un píıxel y un estado queda fuera del alcance de este trabajo.

El problema de tener en cuenta únicamente los vecinos más próximos de una celda concreta se resuelve con nuevos grupos de vecinos más complejos como el Circular usado por [<a href="http://www.geocomputation.org/2005/Alkheder.pdf">9</a>] que define buffers circulares como el mostrado en la siguiente imagen:

[caption id="attachment_2324" align="aligncenter" width="208"]<a href="http://www.gisandchips.org/wp-content//cel4.png"><img class="size-full wp-image-2324" alt="Buffer circular que tiene en cuenta muchos mas vecinos que las aproximaciones sencillas" src="http://www.gisandchips.org/wp-content//cel4.png" width="208" height="178" /></a> Buffer circular que tiene en cuenta muchos más vecinos que las aproximaciones sencillas.[/caption]

Hasta ahora hemos asumido que el territorio se puede categorizar en un estado que evolucionara en función de los estados de sus vecinos. Pero en un núcleo urbano hay otro tipo de objetos móviles como las personas, automóviles, etc. que están relacionados con el desarrollo de las ciudades.  Teniendo en cuenta la movilidad de algunos de los componentes del sistema, [7] define un nuevo autómata denominado Sistema de Autómata Geográfico.
<h3>Sistema de autómata geográfico</h3>
Un Sistema de Autómata Geográfico (GAS) es un autómata formado por 7 componentes:
<p style="text-align: center">Gs (K; S; TS; L;ML;M;RN)</p>
K es el conjunto de tipos de objetos del autómata que pueden ser fijos o no fijos. Los objetos geográficos fijos son aquellos que representan a objetos que no cambian su localización, como viviendas, parques, etc. Los no fijos son los que representan a objetos móviles como ciudadanos, vehículos, etc. Los fijos son los que presentan una analogía con los CA y son los únicos que se van a tratar en este documento.

El resto de la fórmula son 3 pares de valores. En el primer par de valores S representa en conjunto de estados del autómata y TS representa el conjunto de funciones de transición. El segundo par representa información de localización. L define la convención de geolocalización del autómata en el espacio mientras que ML define las reglas de movimiento de los objetos móviles (que se aplica a los objetos no fijos). El último par de valores está formado por N, que representan a los vecinos de la celda y por RN que representa el conjunto de reglas entre vecinos.

Con esto podemos definir un autómata geográfico como una autómata que usando tres funciones de transición diferentes TS, ML y RN, un conjunto de objetos K, y a partir de un conjunto de estados S, un conjunto de localizaciones L y un conjunto de vecinos N, obtiene el estado en el siguiente instante de tiempo.
<h2>Reglas de transición</h2>
La parte más complicada a la hora de diseñar un CA es escoger las reglas de transición entre estados que se van a aplicar en la simulación del sistema. Una vez escogidas las reglas de transición, sólo hay que aplicarlas al autómata celular parar obtener los mapas futuros por lo que la complejidad se encuentra en escoger las reglas de transición más adecuadas.

Existen varios métodos para escoger estas reglas, pero en este trabajo se han destacado los basados en algoritmos genéticos y los basados en redes neuronales
<h3>Algoritmos genéticos</h3>
Según [<a href="http://web.cecs.pdx.edu/~mm/EvDens.pdf">10</a>], los algoritmos genéticos son métodos de búsqueda inspirados en la evolución biológica [11]. En un algoritmo genético típico, las muestras de un problema se codifican como cadenas de bits y inicialmente se eligen aleatoriamente un subconjunto de ellas dando lugar a la primera generación de individuos. A continuación esa generación pasa iterativamente por 3 estados:
<ul>
	<li>Cruzamiento: se emparejas distintas muestras del problema dando lugar a nuevos individuos en el espacio de las soluciones.</li>
	<li>Mutación: se añaden mutaciones aleatorias en los resultados para abrir nuevas posibilidades no contempladas simplemente con el cruzamiento.</li>
	<li>Selección: se escogen las muestras que hayan presentado mejores resultados en el sistema dando lugar a la siguiente generación de soluciones que puede volver a entrar en proceso iterativo.</li>
</ul>
Un ejemplo de estudio utilizando esta tecnología aplicada a la predicción de crecimiento urbanístico es el de [<a href="http://link.springer.com/content/pdf/10.1007%2F978-1-4471-1281-5_16.pdf#page-1">12</a>] que modelo los cambios de usos del suelo en Roma (Italia) utilizando algoritmos genéticos para producir un conjunto de reglas para un autómata celular.

Suponiendo el ejemplo de los estados propuestos por [<a href="http://www.geog.ucsb.edu/~tobler/publications/pdf_docs/geog_analysis/CellularGeog.pdf">8</a>], los diferentes estados se podrán codificar en binario tal y como muestra el cuadro siguiente:
<table>
<tbody>
<tr>
<th width="100px">Estado</th>
<th width="100px">Codificación</th>
</tr>
<tr>
<td>Residencial (R)</td>
<td>000</td>
</tr>
<tr>
<td>Comercial (C)</td>
<td>001</td>
</tr>
<tr>
<td>Industrial (I)</td>
<td>010</td>
</tr>
<tr>
<td>Publico (P)</td>
<td>011</td>
</tr>
<tr>
<td>Agrcola (A)</td>
<td>100</td>
</tr>
</tbody>
</table>
Suponiendo además un autómata que usa vecinos de Von Neumann (por simplificar), se pueden codificar las reglas de transición del autómata como cadenas de bits de tal forma que se representan los 5 vecinos de entrada y los 5 vecinos de salida como una lista binaria de dígitos, tal y como muestra la siguiente imagen:

[caption id="attachment_2325" align="aligncenter" width="494"]<a href="http://www.gisandchips.org/wp-content//cel5.png"><img class="size-full wp-image-2325" alt="Representación binaria de una regla de transición" src="http://www.gisandchips.org/wp-content//cel5.png" width="494" height="171" /></a> Representación binaria de una regla de transición.[/caption]

Estas reglas codificadas en binario serán las entradas del algoritmo genético cuya salida sera el conjunto de reglas más representativo de la población inicial que será el utilizado para predecir el crecimiento del núcleo urbano en un futuro.

El problema ahora es cómo obtener las reglas de transición iniciales. Para ello, es necesario disponer de varias ortofotos de una misma zona tomadas en distintos instantes de tiempo en las que se han definido previamente los estados de los píxeles (o celdas del autómata). Una vez que tengamos las celdas de dos o más imágenes categorizadas, se pueden obtener las reglas de transición comparando las imágenes dos a dos y a continuación, se pueden convertir a binario para que puedan usarse como entrada del algoritmo genético.
<h3>Redes neuronales artificiales</h3>
Una red neuronal artificial (ANN) es una estructura matemática flexible que es capaz de identificar complejas relaciones no lineales entre un conjunto de entradas [<a href="http://iahs.info/hsj/470/hysj_47_06_0865.pdf">13</a>]. Está basado en el modelo de neuronas de nuestro cerebro y es muy eficiente en problemas que son difíciles de expresar mediante ecuaciones físicas.

Un ANN consiste en 3 componentes principales: la capa de entrada, la capa oculta y la capa de salida (siguiente imagen). La capa oculta es el motor de la red neuronal y es la encargada de procesar las entradas del sistema [<a href="http://www.ccsenet.org/journal/index.php/jgg/article/view/15215">16</a>]. Los detalles de cómo funciona la capa oculta de la red quedan fuera del alcance del estudio de este artículo.

[caption id="attachment_2326" align="aligncenter" width="519"]<a href="http://www.gisandchips.org/wp-content//cel6.png"><img class="size-full wp-image-2326" alt="Ejemplo de una red neuronal articial genérica con una salida de una neurona" src="http://www.gisandchips.org/wp-content//cel6.png" width="519" height="237" /></a> Ejemplo de una red neuronal artificial genérica con una salida de una neurona.[/caption]

Uno de los primeros trabajos usando redes neuronales para encontrar las reglas de transición para un autómata celular lo realizó [<a href="http://www.envplan.com/abstract.cgi?id=a33210">14</a>] en 2001 utilizando para ello dos estados en el autómata (urbano/no urbano). Los mismos autores continuaron con su investigación aumentado los estados del autómata en [<a href="http://www.tandfonline.com/doi/abs/10.1080/13658810210137004#.UYaZHUkcAQ8">15</a>].
<h2>Ejemplos de autómatas celulares geográficos</h2>
Hasta aquí hemos descrito a grandes rasgos los conceptos teóricos para entender cómo funcionan los autómatas celulares geográficos. En este apartado vamos a comentar un caso de uso reciente de cada uno de los dos tipos de autómatas celulares.
<h3>Usando algoritmos genéticos</h3>
El último estudio encontrado de autómata celular es el realizado por [<a href="http://gis.vsb.cz/GIS_Ostrava/GIS_Ova_2012/sbornik/papers/foroutan.pdf">17</a>] donde los autores hacen un estudio de la ciudad de Isfahan (Irán). Los datos que han utilizado para calibrar el sistema son dos ortofotos raw de 28.5 metros de resolución de los años 1990 y 2001 respectivamente (siguiente imagen).

[caption id="attachment_2327" align="aligncenter" width="486"]<a href="http://www.gisandchips.org/wp-content//cel7.png"><img class="size-full wp-image-2327" alt="Ortofotos de la ciudad de Isfahan en 1990 y en 2001 [17]" src="http://www.gisandchips.org/wp-content//cel7.png" width="486" height="213" /></a> Ortofotos de la ciudad de Isfahan en 1990 y en 2001 [17].[/caption]A continuación se han denido 5 estados para categorizar los píxeles y se han creado dos nuevas imágenes categorizadas (siguiente imagen). Los 5 estados son urbano (urban), calle (street), vegetación (vegetation), agua (water) y no urbano (barren). Para realizar los cálculos se ha utilizado la aproximación mediante vecinos de Moore y se han realizado varias simulaciones con tamaños de celdas de 3x3, 5x5 y 7x7. Para cada caso, se han calculado las reglas de transición mediante un algoritmo genético y con los resultados, se ha aplicado una transformación de la imagen original de 1990 para obtener una predicción de la imagen de 2001 y poder compararla con la imagen original del mismo año.

[caption id="attachment_2328" align="aligncenter" width="488"]<a href="http://www.gisandchips.org/wp-content//cel8.png"><img class="size-full wp-image-2328" alt="Imagenes clasicadas de la ciudad de Isfahan en 1990 y en 2001 [17]" src="http://www.gisandchips.org/wp-content//cel8.png" width="488" height="234" /></a> Imágenes clasificadas de la ciudad de Isfahan en 1990 y en 2001 [17][/caption]&nbsp;

El siguiente cuadro [17] muestra los porcentajes de aciertos de cada una de las simulaciones.
<table>
<tbody>
<tr>
<th width="100">Ventana</th>
<th width="100">Acierto</th>
</tr>
<tr>
<td>3x3</td>
<td>86,28</td>
</tr>
<tr>
<td>5x5</td>
<td>87,17</td>
</tr>
<tr>
<td>7x7</td>
<td>86,49</td>
</tr>
</tbody>
</table>
Como puede observarse en la tabla, la simulación de 5x5 ha sido la que ha presentado mejores tasas de acierto. Este ejemplo muestra que no por aumentar los tamaños de la ventana se van a mejorar los resultados del algoritmo ya que si dos píxeles están muy lejos, es muy probable que no exista relación entre ellos.
<h3>Usando redes neuronales artificiales</h3>
Para el caso de autómatas celulares utilizando redes neuronales se ha encontrado un artículo del 2012 en el que se hace un estudio de la ciudad de Lagos (Nigeria) [16]. Los datos que se han utilizado son 3 imágenes Landsat de 1978, 1984 y 2000 respectivamente y otra imagen del 1968.

En el estudio se compararon tres períodos de grandes cambios de la ciudad (1963-1978, 1978-1984, y 1984-2000). Para cada uno de estos períodos se eligieron 1000 puntos de entrenamiento y se hizo una simulación mediante el método de k-fold cross-validation con k = 10. La elección del número de neuronas óptimo para el funcionamiento del algoritmo también se desconocía por lo que se hicieron varias simulaciones para poder obtenerlo. Otra variable importante en este caso es el número de iteraciones en la red: también se hicieron varias simulaciones para distintos números de iteraciones.

Una vez escogido en número óptimo de neuronas para cada caso, se hizo una simulación a partir de las imágenes de partida originales para poder comparar los resultados con las ortofotos. Las tasas se acierto se pueden observar en el siguiente cuadro [16]
<table>
<tbody>
<tr>
<th width="100">Período</th>
<th width="100">Acierto</th>
</tr>
<tr>
<td>1963-1978</td>
<td>81</td>
</tr>
<tr>
<td>1978-1984</td>
<td>89</td>
</tr>
<tr>
<td>1984-2000</td>
<td>86</td>
</tr>
</tbody>
</table>
Una representación gráfica de los resultados se puede ver en la siguiente imagen:

[caption id="attachment_2329" align="aligncenter" width="503"]<a href="http://www.gisandchips.org/wp-content//cel9.png"><img class="size-full wp-image-2329" alt="Resultados de las simulaciones en la ciudad de lagos. TN = aciertos no urbanos, FP = errores urbanos, FN = errores no urbanos, TP = aciertos urbanos [16]" src="http://www.gisandchips.org/wp-content//cel9.png" width="503" height="180" /></a> Resultados de las simulaciones en la ciudad de lagos. TN = aciertos no<br />urbanos, FP = errores urbanos, FN = errores no urbanos, TP = aciertos urbanos [16][/caption]
<h2>Conclusiones</h2>
El uso de los autómatas celulares aplicado a la predicción del crecimiento de los núcleos urbanos es una tecnología que se ha ido definiendo en la última década y que cada vez está siendo más utilizada.

La principal dificultad que tienen los autómatas celulares está en encontrar el conjunto de reglas de transición que generen el modelo futuro minimizando el error.

Existen varios métodos para poder obtener unas buenas reglas de transición, destacando el método basado en algoritmos genéticos y el basado en redes neuronales.

Estas técnicas asumen un modelo de objetos estáticos de un núcleo urbano: la realización de modelos que traten los objetos dinámicos es un trabajo que está todavía por desarrollar.
<h2>Referencias</h2>
<a href="http://www.sciencedirect.com/science/article/pii/S0198971599000150">[1]</a> Batty, M., Xie, Y., Sun, Z.:  Modeling urban dynamics through GIS-based cellular automata. In: Computers, Environment and Urban Systems 23. 205-233, 1999.
<a href="http://www.geog.ucsb.edu/~tobler/publications/pdf_docs/geog_analysis/ComputerMovie.pdf">[2]</a> Tobler, W. R.:A computer movie simulating urban growth in the Detroit region. In: Economic geography, 46, 234-240, 1970.
<a href="http://cba.mit.edu/events/03.11.ASE/docs/VonNeumann.pdf">[3]</a> Von Neumann, J.: Theory of Self-Reproducing Automata. In: University of Illinois Press, Illinois. Edited and completed by A. W. Burks, 1966.
<a href="http://conwaysgameoflife.appspot.com/">[4]</a> El Juego de la Vida, http://conwaysgameoflife.appspot.com/
<a href="http://gbic.biol.rug.nl/~rbreitling/dagstuhlpublications/006_Wolfram1984.pdf">[5]</a> Wolfram, S.: Cellular automata as models of complexity. In: Nature, 311(5985), 419-424, 1984. okwuashi1
<a href="http://www.geocomputation.org/2005/Alkheder.pdf">[6]</a> Alkheder, S., Shan, J.: Cellular Automata Urban Growth Simulation and Evaluation - A Case Study of Indianapolis. In: Proceedings of the 8th International Conference on GeoComputation. University of Michigan, 2005.
<a href="http://www.tandfonline.com/doi/abs/10.1080/13658810512331325139?journalCode=tgis20#.UYaXA0kcAQ8">[7]</a> Torrens, P. M., Benenson, I. : Geographic Automata Systems. In International Journal of Geographical Information Science. Vol 19, No 4, 2005
<a href="http://www.geog.ucsb.edu/~tobler/publications/pdf_docs/geog_analysis/CellularGeog.pdf">[8]</a> Tobler, W. R.:Cellular Geography, 1979.
<a href="http://www.geocomputation.org/2005/Alkheder.pdf">[9]</a> Alkheder, S., Shan, J.: Cellular Automata Urban Growth Simulation and Evaluation - A Case Study of Indianapolis, 2005.
<a href="http://web.cecs.pdx.edu/~mm/EvDens.pdf">[10]</a> Crutcheld, J. P., Mitchell, M., Das, R.: The Evolutionary Design of Collective Computation in Cellular Automata. In: Evolutionary Dynamics|Exploring the Interplay of Selection, Neutrality, Accident, and Function. New York: Oxford University Press, 2002.
<a>[11]</a> Holland, J. H.: Adaptation in Natural and Articial Systems. MIT Press, Cambridge, MA, Second edition, 1992.
<a href="http://link.springer.com/content/pdf/10.1007%2F978-1-4471-1281-5_16.pdf#page-1">[12]</a> Colonna, A., DiStephano, V., Lombardo, S., Papini, L., Rabino, A.: Learning urban cellular automata in a real world: The case study of Rome metropolitan area. In: The ACRI'98 Third Conference on Cellular Automata for Research and Industry, Trieste, 1999.
<a href="http://iahs.info/hsj/470/hysj_47_06_0865.pdf">[13]</a> Hsu, K., Gupta, H. V., Sorooshian, S.: Articial Neural Network Modeling of the Rainfall-Runo Process. In: Water Resources Research. Vol 31, no. 10, P. 2517, 1995.
<a href="http://www.envplan.com/abstract.cgi?id=a33210">[14]</a> Li, X.; Yeh, A. G.: Calibration of cellular automata by using neural networks for the simulation of complex urban systems. Environmental and Planning A, v. 33, p. 1445-1462, 2001.
<a href="http://www.tandfonline.com/doi/abs/10.1080/13658810210137004#.UYaZHUkcAQ8">[15]</a> Li, X.; Yeh, A. G.: Neural-network-based cellular automata for simulating multiple land use changes using GIS. International Journal of Geographical Information Science, v. 16, p. 323-343, 2002.
<a href="http://www.ccsenet.org/journal/index.php/jgg/article/view/15215">[16]</a> Okwuashi, O., Isong, M., Eyo, E., Eyoh, A., Nwanekezie, O., Olayinka, D. N., Udoudo, D. O., Ofem, B.: GIS Cellular Automata Using Articial Neural Network for Land Use Change Simulation of Lagos, Nigeria. In: Journal of Geography and Geology; Vol. 4, No. 2, 2012.
<a href="http://gis.vsb.cz/GIS_Ostrava/GIS_Ova_2012/sbornik/papers/foroutan.pdf">[17]</a> Foroutan, E., Delavar, M. R.: Urban Growth Modeling Using GEnetic Algorithms and Cellular Automata: A case study of Isfahan Metropolitan Area, Iran. In: GIS Ostrava - Surface models for geosciences, 2012