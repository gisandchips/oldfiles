---
ID: 2723
post_title: ¡¡De SIOSE a PostGIS en 4 sesiones!!
author: benizar
post_date: 2014-03-25 21:17:39
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2014/03/25/de-siose-a-postgis-en-4-sesiones/
published: true
---
Hola a todos, en este post quiero compartir la presentación de un seminario temático que impartí dentro de la asignatura “SIG aplicado a la Ordenación del Territorio” en el Grado de Geografía de la Universidad de Alicante.

<iframe style="border: 1px solid #CCC; border-width: 1px 1px 0; margin-bottom: 5px; max-width: 100%;" src="http://www.slideshare.net/slideshow/embed_code/32638696" width="427" height="356" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" allowfullscreen="allowfullscreen"> </iframe>
<div style="margin-bottom: 5px;"><strong> <a title="De SIOSE a PostGIS en cuatro sesiones" href="https://www.slideshare.net/BeniZaragoz/de-siose-a-postgis-en-cuatro-sesiones" target="_blank">De SIOSE a PostGIS en cuatro sesiones</a> </strong> from <strong><a href="http://www.slideshare.net/BeniZaragoz" target="_blank">Benito M. Zaragozi</a></strong></div>
El curso tuvo lugar hace un par de meses, durante 4 sesiones de 2 horas:
<ol>
	<li>En la primera sesión hice una introducción en la que me refería a las ventajas de saber utilizar SQL y tener un conocimiento de ciertos estándares para explotar la información geográfica.</li>
	<li>En una segunda sesión expliqué la utilidad de la base de datos del SIOSE (Sistema de información sobre Ocupación del Suelo de España), ya que ésta nos puede ahorrar mucho trabajo en proyectos SIG que desarrollemos por toda España.</li>
	<li>En la tercera sesión exploramos una solución en la que se combinan SQL y Xpath para generar reclasificaciones personalizadas del SIOSE desde PostgreSQL/PostGIS.</li>
	<li>Finalmente los alumnos tuvieron algo de tiempo para practicar y generar una reclasificación del SIOSE para separar aquellos usos del suelo susceptibles de albergar un vertedero de residuos de todos los usos del suelo que no lo admitirían. Todo esto como parte de una asignatura práctica.</li>
</ol>
Los que hayáis trabajado con la capa del SIOSE probablemente revisaréis la presentación y este post con interés, ya que las alternativas son generalmente muy trabajosas. Espero que a los demás también os resulte interesante ;)<!--more-->
<h1>Información Geográfica y lenguajes de programación</h1>
Hace muy poco que tuvimos en las redes sociales un debate muy interesante sobre la necesidad de que los Geógrafos dominemos algún lenguaje de programación. Un compañero de GIS&amp;Chips hizo algunos comentarios sobre esta cuestión en <a title="este post" href="http://www.gisandchips.org/2014/01/10/ser-geografo-tambien-implica-saber-leer-y-escribir-codigo/" target="_blank">este post</a>. No hace falta decir que estos razonamientos se pueden aplicar a cualquier otra disciplina que analice la información geográfica.

En Twitter, yo voté que los geógrafos deberíamos adquirir unos conocimientos básicos de programación con SQL y un lenguaje interpretado, como Python o R. Lamentablemente, un tweet se queda corto y la mejor manera de apoyar mi punto de vista es con casos de uso como éste del SIOSE. Se trataba de obtener una clasificación de usos del suelo a partir de la capa del SIOSE y con el menor trabajo posible.

Básicamente, la capa del SIOSE es una base de datos relacional que contiene tres elementos fundamentales:
<ol>
	<li>Un campo, nombrado por defecto <em><strong>the_geom,</strong></em> de tipo Geometry que describe los polígonos de usos del suelo.</li>
	<li>Un campo <em><strong>code_2009</strong></em> que contiene un código alfanumérico que describe los usos del suelo de cada polígono en menos de 256 caracteres.</li>
	<li>Un último campo <em><strong>xml_2009</strong></em> asociado, que contiene una estructura XML (eXtensible Markup Language) donde se almacenan las coberturas y porcentajes. Esto tiene muy poco que ver con un ESRI Shapefile y creo que ningún SIG de escritorio permite interpretar un XML dentro de una tabla… Sí, para manejar esta base de datos es necesario saber de bases de datos, saber programar o tener muchas ganas de trabajar :)</li>
</ol>
<a href="http://www.gisandchips.org/wp-content//siose_campos.png"><img class="aligncenter size-medium wp-image-2732" src="http://www.gisandchips.org/wp-content//siose_campos-300x130.png" alt="siose_campos" width="300" height="130" /></a>

Evidentemente, para consultar una base de datos y procesar un XML, podríamos utilizar casi cualquier lenguaje de programación. Para la mayoría habrá APIs que trabajen con geometrías OGC y estructuras XML. Sin embargo, en la siguiente figura se puede ver una clasificación de lenguajes de programación según productividad (resultados por tiempo invertido) y versatilidad (cosas que podemos hacer con un mismo lenguaje).

<a href="http://www.gisandchips.org/wp-content//programming_languages.png"><img class="aligncenter size-medium wp-image-2730" src="http://www.gisandchips.org/wp-content//programming_languages-265x300.png" alt="programming_languages" width="265" height="300" /></a>
Como se puede ver, he situado al SQL como el primer lenguaje justo por debajo de los “Query Builder”, que son los entornos de consulta típicos de los SIG de escritorio. Estos entornos de consulta permiten explorar los datos con una versión reducida del SQL, por lo que este lenguaje debería ser bastante accesible para usuarios habituales de SIG. Además, utilizando SQL podemos explotar casi toda la potencia de PostgreSQL y PostGIS. Esto nos dará mucho juego a la hora de hacer todo tipo de consultas y operaciones espaciales (y no espaciales). Con SQL podemos utilizar todas las funciones espaciales de PostGIS, alrededor de un millar de funciones de análisis con geometrías vectoriales, ráster, 3D, LiDAR, etc… todo eso son muchos botones :)
<h1>Sistema de Información sobre la Ocupación del Suelo de España (SIOSE)</h1>
No pretendo repetir una descripción detallada del SIOSE, que se puede encontrar en su <a title="documentación" href="http://www.siose.es/siose/documentacion.jsp" target="_blank">documentación</a>, sino que quiero relatar la sensación que me produjo mi primer contacto con el SIOSE. La sensación de que trabajar con estos datos no es tan fácil como algunos dicen.

Os recomiendo leer <a title="esta comunicación" href="http://www.sigte.udg.edu/jornadassiglibre2012/uploads/articulos_12/art17.pdf " target="_blank">esta comunicación</a> presentada a las Jornadas de SIG Libre de 2012. Entre otras cuestiones, trata de los problemas de memoria y ancho de banda a la hora de compartir la capa del SIOSE mediante protocolos estándar. Sin embargo lo más interesante es todo lo relacionado con el modelo de datos, del cual admiten que es más complejo de lo habitual en SIG. En esta comunicación también se reconoce que los servicios de visualización y explotación del SIOSE tienen que mejorar con ayuda de la comunidad de usuarios. Creo que estaremos todos de acuerdo en que el objetivo de una IDE es solamente proporcionar acceso a los datos y que deben ser los usuarios los que los exploten.
<h2>Clasificación Orientada a Objetos</h2>
La primera cuestión interesante es la buena publicidad de la que goza el SIOSE. Se trata de una clasificación de usos del suelo Orientada a Objetos (OO). Esto significa que es una clasificación muy flexible, que permite que reclasifiquemos el SIOSE de acuerdo a nuestras necesidades. Por ejemplo, no es lo mismo decir que un lugar/polígono tiene un uso “Forestal, Matorral Denso y Viviendas Aisladas”, que “Forestal 50%, Matorral Denso 48% y Viviendas Aisladas 2%”. En algunos proyectos podríamos querer reclasificar todo el polígono como Monte Alto o cualquier otro adjetivo/categoría que nos interese. En cambio, si fuese un 20% de cobertura de Viviendas Aisladas, la cosa cambia mucho.

Todo esto de la clasificación OO está muy bien. Sin embargo, estamos hablando de dos paradigmas diferentes, los SIG más utilizados funcionan con un paradigma <strong>Relacional</strong>, mientras que la clasificación es <strong>Orientada a Objetos</strong>.
<h2>¿Cómo se come esto?</h2>
Los SIG habituales tratarán de desplegarnos la información geográfica en una tabla-relacional y pintarán las geometrías en otra ventana, pero los datos OO no se prestan a esto, sino que se parecen más bien a un árbol jerárquico. Entonces, ¿cómo gestionar una clasificación Orientada a Objetos dentro de una base de datos Relacional? La respuesta es que no lo haces, guardas la clasificación del polígono en una celda y después utilizas alguna herramienta distinta para leer la estructura OO. En este caso, alguna herramienta que permita gestionar ficheros o estructuras XML.

A continuación podemos ver un ejemplo de clasificación OO de los usos del suelo (<em><strong>xml_2009</strong></em>) de una parcela al azar:

<script src="https://gist.github.com/benizar/9739541.js" type="text/javascript"></script>La otra opción es utilizar el código alfanumérico de usos del suelo, almacenado en el campo <em><strong>code_2009</strong></em>, pero esto da mucho trabajo ya que hay centenares de combinaciones de todo tipo. Aún siendo un gran usuario de SQL, habría que esforzarse por hacer clasificaciones que tuvieran en cuenta los porcentajes y todos los niveles de agregación. Además, habría que consultar la documentación para encontrar la descripción de cada etiqueta (ver <a title="este documento" href="http://www.ign.es/siose/Documentacion/Modelo_de_datos_SIOSE/Doc_ModeloDatos_Rotulo%20SIOSE%202005%20v2.pdf" target="_blank">este documento</a>). Aquí va un ejemplo de código alfanumérico “sencillito”:

<strong>R(50LFNfzrr_40CNFpl_10SDNfc)</strong>

Cobertura compuesta en “Mosaico regular” formado por tres clases simples:

<ul>
	<li style="text-align: left;">50% Frutales. No cítricos; atributos “forzado” y ”regadío regado”</li>
	<li style="text-align: left;">40% Coníferas; atributo “plantación”</li>
	<li style="text-align: left;">10% Suelo desnudo; atributo “función cortafuegos”</li>
</ul>
<h2>¿Puedo reclasificar el SIOSE con las herramientas que conozco?</h2>

Bueno, dependerá de las herramientas que conozcas :)

En la web de documentación del SIOSE vemos que hay herramientas y extensiones SIG (ArcGIS y Geomedia) que se utilizan en la creación y mantenimiento del SIOSE, pero éstas no parecen estar disponibles para descarga y, lo más importante, no son FOSS.

Si esta pregunta se la hace un experto en SIG con conocimientos de programación, estoy seguro de que encontrará una manera de que esta tarea no se convierta en un suplicio. En cambio, si esta pregunta se la hace un analista SIG que no conoce ningún lenguaje de programación y que solamente utiliza las opciones de un “Query Builder”, lo más probable es que trate de reclasificar los usos del suelo casi manualmente, ya que no hay herramientas en los SIG que procesen un XML, ni otras funciones que trabajen con strings y distingan los textos de los porcentajes numéricos.

<h1>Reclasificar el SIOSE con PostgreSQL+PostGIS</h1>

Al descargar los datos del <a title="Terrasit" href="http://terrasit.gva.es/es/descargas" target="_blank">Terrasit</a>, la descarga se hace por municipios, es más actualizada pero requiere un mayor preprocesado. En cambio, si descargamos los datos del <a title="CNIG" href="http://centrodedescargas.cnig.es/CentroDescargas/catalogo.do" target="_blank">CNIG</a> podemos descargar toda una provincia de golpe y ahorrarnos el paso de unir los municipios.

En la presentación veréis que realizamos varias tareas de preprocesado de la capa del SIOSE-Terrasit. Importamos los datos con el comando shp2pgsql, los unimos en una misma capa, detectamos y eliminamos los polígonos repetidos, recortamos los polígonos según la zona de estudio y finalmente hicimos una clasificación de aquellos polígonos recortados en el borde de la zona de estudio. Esta clasificación sirve para asegurarnos de que la clasificación sea ajustada en los bordes de la zona, ya que al recortar los polígonos la clasificación del SIOSE pierde su representatividad. Podeis ver el resultado en la siguiente figura:

<a href="http://www.gisandchips.org/wp-content//siose2postgis1.png"><img class="aligncenter size-medium wp-image-2733" src="http://www.gisandchips.org/wp-content//siose2postgis1-300x225.png" alt="siose2postgis1" width="300" height="225" /></a>

Todas estas tareas de preprocesado resultan muy sencillas con SQL y se pueden automatizar fácilmente para trabajar otras zonas de estudio. En este punto, llegamos a la tarea verdaderamente importante: reclasificar el SIOSE con el mínimo trabajo posible.

Al final de la diapositiva nº33 de la presentación se muestra un ejemplo de cómo sería utilizar expresiones regulares para trocear el campo “<strong><em>code_2009</em></strong>” reclasificar los datos. ¡¡Menudo dolor de cabeza!! Menos mal que PostgreSQL es tremendamente potente y, entre muchas otras herramientas (recordad pl/R o PostGIS, sin ir más lejos), permite evaluar expresiones de Xpath para procesar estructuras XML, todo esto desde una consola de SQL... Viendo las alternativas, ¡¡esto es alucinante!!

Según la Wikipedia, XPath (XML Path Language) es un lenguaje que permite construir expresiones que recorren y procesan un documento XML. La idea es parecida a las expresiones regulares para seleccionar partes de un texto sin atributos (plain text). XPath permite buscar y seleccionar teniendo en cuenta la estructura jerárquica del XML. XPath fue creado para su uso en el estándar XSLT, en el que se usa para seleccionar y examinar la estructura del documento de entrada de la transformación. A continuación, en este GIST os muestro un ejemplo básico para reclasificar la capa de usos del SIOSE a partir del campo <em><strong>xml_2009</strong></em>, que como veis es de tipo “xml” y no string:<script src="https://gist.github.com/benizar/9740379.js" type="text/javascript"></script>

Aunque me cuesta explicarlo brevemente, la función “xpath(expresión, campo xml)”, requiere un campo de tipo xml, como el campo <em><strong>xml_2009</strong></em> del SIOSE, y una expresión como: '//COBERTURA[@ID="EDF" and @Sup&gt;20]/@ID', que explora solamente el segundo nivel del XML (Leyéndolo como un árbol jerárquico, “/” es el primer nivel, “//” el segundo y así sucesivamente) y selecciona todos los registros donde el tipo de cobertura es mayor que 20EDF (Edificaciones recubriendo un 20%). Esta función devuelve un array de 0 o 1 elementos según si existe o no existe la cobertura EDF en este nivel jerárquico del XML.

En el ejemplo, se utilizan expresiones similares de Xpath para cada uso que nos interese reclasificar como apto para acoger un vertedero de residuos y, finalmente, aunque no es necesario, se hace lo contrario para no dejar celdas vacías.

El resultado final de la reclasificación se puede apreciar en la siguiente ampliación de la zona de estudio (en rojo las zonas no aptas):

<a href="http://www.gisandchips.org/wp-content//siose2postgis.png"><img class="aligncenter size-medium wp-image-2731" src="http://www.gisandchips.org/wp-content//siose2postgis-300x225.png" alt="siose2postgis" width="300" height="225" /></a>

Como se puede apreciar, seguramente hemos dejado de considerar usos no aptos, como el cementerio (código ECM), por lo que el objetivo de los alumnos era extender la consulta anterior para completar la reclasificación de modo razonado.

La consulta se puede optimizar mucho, pero esta aproximación resulta más didáctica, ya que parte de otra reclasificación que se hacía en la sesión anterior del curso.
<h1>Conclusiones y trabajo futuro</h1>
La principal conclusión es que solamente utilizando SQL ya se pueden acceder a herramientas muy potentes propias de los lenguajes de programación más versátiles (operaciones geométricas, expresiones regulares, estadísticas y, en este caso, procesado de XML). Además, el SQL es un lenguaje muy maduro utilizado en todo tipo de disciplinas, lo cual es una ventaja para su aprendizaje.

Por otro lado, el caso del SIOSE viene bien para volver a plantearnos la cuestión del paradigma de Orientación a Objetos, que ya había sido mencionado por <a title="josetomas en uno de sus primeros post en GIS&amp;Chips" href="http://www.gisandchips.org/2009/07/28/gizmo-gis-meets-objects/" target="_blank">josetomas en uno de sus primeros post en GIS&amp;Chips</a>. Bases de datos como la del SIOSE podrían servir como impulsoras del uso de las bases de datos OO que fueron planteadas ya a principios de los años ‘90 y que parece que estemos evitando en el mundillo de los GIS. Lo cierto es que sí que hay una necesidad. Cada vez hay más datos disponibles y cada vez son más complejos...

No es menos importante fijarse en la cantidad de trabajo que nos ahorra el SIOSE. A pesar de que su manejo requiera un cierto esfuerzo, fijaos en el detalle de la digitalización de los polígonos y el enorme rango de categorías que diferencia. Los que hemos digitalizado usos del suelo alguna vez, sabremos apreciar este recurso como se merece.

Este breve test con PostGIS y Xpath no debería terminar aquí. Desde hace escasamente una semana, he entrado a participar en un proyecto de investigación sobre Ecología y Paisaje donde los usos del suelo serán, como no, una capa de información imprescindible. Creo que trataré de desarrollar un plugin de QGIS que construya expresiones de Xpath y las ejecute en el servidor de PostgreSQL para generar consultas ágiles del campo XML del SIOSE. Os tendre informados si hago algo parecido ;)
<h1 style="text-align: center;"><strong>Gracias por vuestra atención!!
</strong></h1>