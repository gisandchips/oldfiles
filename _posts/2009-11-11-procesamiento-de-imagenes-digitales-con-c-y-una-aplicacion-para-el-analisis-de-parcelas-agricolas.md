---
ID: 1038
post_title: 'Procesamiento de imágenes digitales con C# (y una aplicación para el análisis de parcelas agrícolas).'
author: benizar
post_date: 2009-11-11 14:05:28
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2009/11/11/procesamiento-de-imagenes-digitales-con-c-y-una-aplicacion-para-el-analisis-de-parcelas-agricolas/
published: true
---
<p style="text-align: justify;">En este artículo voy a exponer una aplicación de ejemplo que he realizado con C# utilizando Aforge.NET. Dicha aplicación trata (y a veces lo consigue  :-)  ) de distinguir si una parcela dada puede ser una plantación  agrícola arbórea mediante el análisis de la Transformada de Hough, y luego se posibilita el conteo automático de los árboles. Al tratarse de un programa con finalidad didáctica los análisis se realizan para una sola parcela y por pasos muy definidos. No obstante, cabe pensar que su mayor utilidad vendría de un análisis masivo de parcelas.</p>

<h4 style="text-align: center;"><a href="http://www.gisandchips.org/wp-content/hough_direcciones.jpg"><img class="size-full wp-image-1041 aligncenter" alt="hough1" src="http://www.gisandchips.org/wp-content/hough_direcciones.jpg" width="392" height="314" /></a>Imagen1</h4>
<p style="text-align: justify;"><!--more-->En primer lugar, Aforge.NET es un Framework con distintas librerías que abarcan un amplio rango de campos relacionados con el tratamiento digital de imágenes. Se trata de un proyecto GNU GPL disponible en <a href="http://code.google.com/p/aforge/">http://code.google.com/p/aforge/</a> . Es un proyecto consolidado pero también muy prometedor, en el que Andrew Kirillov tiene un gran trabajo realizado.</p>
<p style="text-align: justify;">Según vemos en la página Web entre sus librerías se encuentran:</p>

<ul style="text-align: justify;">
	<li><strong>AForge.Imaging</strong> - library with image processing routines and filters;</li>
	<li><strong>AForge.Vision</strong> - computer vision library;</li>
	<li><strong>AForge.Neuro</strong> - neural networks computation library;</li>
	<li><strong>AForge.Genetic</strong> - evolution programming library;</li>
	<li><strong>AForge.Fuzzy</strong> - fuzzy computations library;</li>
	<li><strong>AForge.MachineLearning</strong> - machine learning library;</li>
	<li><strong>AForge.Robotics</strong> - library providing support of some robotics kits;</li>
	<li><strong>AForge.Video</strong> - set of libraries for video processing</li>
	<li>etc…</li>
</ul>
<p style="text-align: justify;">Se me ocurre que alguien se podría plantear un <strong>AForge.GIS</strong>. Pensaremos en ello…</p>
<p style="text-align: justify;">Nosotros hemos trabajado únicamente con Aforge.Imaging, pero salta a la vista la potencialidad que prácticamente todas las librerías tendrían para realizar análisis espaciales.</p>

<h3 style="text-align: justify;">Introducción</h3>
<p style="text-align: justify;">Actualmente, al trabajar en cuestiones de teledetección aplicada a fotografías aéreas e imágenes de satélite se esta aplicando cada vez más el paradigma del Análisis Orientado a Objetos (AOO), pasando de hablar de los valores de los píxeles, a las propiedades de los objetos. Podríamos resumirlo como: teledetección orientada a objetos.</p>
<p style="text-align: justify;">No voy a tratar de hablar de análisis orientado a objetos, pues considero que hay abundante información en Internet.</p>
<p style="text-align: justify;">Solamente por sentar el ejemplo que vamos a trabajar en este artículo, “Objeto” sería una parcela agrícola que podría tener múltiples “propiedades” como un área, perímetro, otros índices de forma… pero también tendría propiedades basadas en la respuesta espectral de la superficie que encierra, por ejemplo los valores de un índice de vegetación. También serían propiedades muy descriptivas de la parcela aquellas que describan la estructura generada por la distribución de los píxeles. Así pues, para una sola parcela podríamos tener muchísimas propiedades que la describieran, claro está, siempre unas lo harían mejor que otras.</p>
<p style="text-align: justify;">En AForge.Imaging están disponibles distintas funciones de análisis para la extracción de estas propiedades o características a partir de una imagen, como por ejemplo la Transformada de Fourier, distintos algoritmos de detección de bordes y la que nos interesa aquí: La <strong>Transformada</strong><strong> de Hough</strong>.</p>
<p style="text-align: justify;">En definitiva, hemos elaborado una aplicación de escritorio que calcula la transformada de Hough para la imagen aérea de una parcela, y después de aplicar unas sencillas reglas para decidir si se trata de una parcela agrícola y arbórea, permite contar automáticamente los árboles de dicha parcela.</p>

<h3 style="text-align: justify;">La Transformada de Hough</h3>
<p style="text-align: justify;">Se trata de una técnica utilizada para extraer elementos, con una forma particular, a partir de una imagen. Es comúnmente aplicada para encontrar y describir líneas rectas en una imagen, aunque también se pueden hallar círculos y otras formas. En la carpeta de ejemplos de AForge podéis encontrar el ejemplo en el que me he basado, en el cual se aplica Hough para hallar líneas y círculos dentro de una imagen.</p>
<p style="text-align: justify;">Aquí no vamos a explicar los fundamentos de cálculo, sino la interpretación que podemos hacer de los resultados en nuestro caso del análisis de parcelas.</p>
<p style="text-align: justify;">En concreto uno de los datos que obtenemos aplicando Hough es la inclinación de cada una de todas las líneas rectas halladas, como también la intensidad de cada línea (en nuestro caso tendrán mayor intensidad las líneas que más árboles atraviesen).</p>
<p style="text-align: justify;">Solamente viendo la <strong>imagen1</strong> podríamos intuir cuales serían las inclinaciones/direcciones de las dos líneas de mayor intensidad halladas por Hough, pero como se ve en la imagen no hay una coincidencia exacta debido a pequeños detalles (los árboles no están homogéneamente separados, la parcela no es cuadrada…).</p>
<p style="text-align: justify;"><strong>Nota: las líneas rojas no son las de mayor intensidad, solamente muestran las direcciones.</strong></p>
<p style="text-align: justify;">En la <strong>Imagen2</strong> vemos como las direcciones principales son muy similares, y esto nos indica que difícilmente la parcela tendrá la estructura necesaria para realizar el conteo de árboles.</p>

<h4 style="text-align: center;"><a href="http://www.gisandchips.org/wp-content/hough_direcciones2.jpg"><img class="size-full wp-image-1043 aligncenter" alt="hough_direcciones2" src="http://www.gisandchips.org/wp-content/hough_direcciones2.jpg" width="417" height="274" /></a>Imagen2</h4>
<h3 style="text-align: justify;">Reglas de decisión</h3>
<p style="text-align: justify;">Una vez calculado Hough usamos algunas estadísticas de las líneas halladas para crear reglas de decisión que ayuden a distinguir automáticamente la estructura de la parcela.</p>
<p style="text-align: justify;">De un modo arbitrario a partir de las pocas parcelas vistas he decidido que serán agrícolas – arbóreas aquellas parcelas cumplan lo siguiente:</p>

<ul style="text-align: justify;">
	<li>aquellas parcelas que tengan una diferencia angular entre las dos direcciones principales comprendida entre 80 y 120,</li>
	<li>o que el % de líneas en la 1ª dirección no sea mucho mayor que el % de la 2ª (&lt;2x)</li>
</ul>
<p style="text-align: justify;">Estas reglas deberían ser más complejas y basadas en algún clasificador estadístico o matemático. Pero para ser didáctico es más que suficiente.</p>

<h3 style="text-align: justify;">Rough Agricultural Plots IDentifier (RAPID)</h3>
<p style="text-align: justify;">RAPID es el nombre que le hemos dado a nuestra aplicación de ejemplo. Es un identificador “basto” de parcelas agrícolas. No hay que esperar maravillas, pero veréis que acierta bastante.</p>
<p style="text-align: center;"><a href="http://www.gisandchips.org/wp-content/RAPID.jpg"><img class="aligncenter size-medium wp-image-1448" alt="" src="http://www.gisandchips.org/wp-content/RAPID-300x176.jpg" width="300" height="176" /></a></p>

<h4 style="text-align: center;">Imagen3</h4>
<p style="text-align: justify;">La aplicación muestra una barra de tareas donde se secuencian los pasos de análisis y a medida que se realiza cada paso se activan nuevos botones.</p>
<p style="text-align: justify;">Los botones son:</p>

<ul style="text-align: justify;">
	<li>OpenImage: permite añadir imágenes propias.</li>
	<li>Binarize: binariza la imagen aplicando el umbral especificado en el cuadro de texto.</li>
	<li>Calc Hough: calcula la transformada de Hough para la imagen binaria y muestra algunas estadísticas en es cuadro de la derecha. También muestra un mensaje sobre la adecuación, o no, de la parcela.</li>
	<li>Count Trees: realiza el recuento de árboles de la imagen binaria y muestra el resultado en el cuadro “Trees estimation”. Este último no debería activarse en caso de que no se cumplieran las condiciones establecidas en nuestras reglas, pero se activa para facilitar todo tipo de pruebas.</li>
</ul>
<p style="text-align: justify;">Por último, y para ahorrar tiempo se ha añadido una galería de imágenes de parcelas extraídas del visor del SIGPAC (<a href="http://sigpac.mapa.es/fega/visor/">http://sigpac.mapa.es/fega/visor/</a> ). He intentado que haya cierta variedad. Hay algunas parcelas donde resulta fácil el recuento (1, 2, 6, 8 ), pero también es muy interesante ver como se rechazan las otras parcelas. En algunos casos el conteo de árboles sería difícil incluso manualmente sobre la imagen.</p>
En el caso de la parcela 8 de los ejemplos, vemos que RAPID hace un recuento bastante preciso de los olivos de la parcela 8 de los ejemplos (SIGPAC = 148; RAPID +/- 150, según el Threshold). Por supuesto que seríamos más precisos si elimináramos los ruidos que los bordes de la parcela introducen en el análisis.

<a href="http://www.gisandchips.org/wp-content/parcela8_sigpac.jpg"><img class="size-medium wp-image-1449   alignleft" alt="" src="http://www.gisandchips.org/wp-content/parcela8_sigpac-242x300.jpg" width="199" height="246" /></a><a href="http://www.gisandchips.org/wp-content/parcela8_rapid.jpg"><img class="aligncenter size-medium wp-image-1450" alt="" src="http://www.gisandchips.org/wp-content/parcela8_rapid-241x300.jpg" width="202" height="251" /></a>
<h5 style="text-align: center;">Imagenes 4 y 5</h5>
<p style="text-align: justify;">Podéis ajustar el umbral, y veréis como el recuento mejora en algunas imágenes. Esto también se podría automatizar.</p>
<p style="text-align: justify;">Otra cuestión es que hemos trabajado sobre una imagen RGB, pero es evidente que la distinción de los árboles mejoraría mucho si dispusiéramos de una banda de infrarrojo próximo y la combináramos con las otras antes de realizar la binarización.</p>

<h3 style="text-align: justify;">Algunas cosas sobre el código</h3>
<p style="text-align: justify;">La mayor parte del código que he escrito se corresponde con aspectos del UI, lo que da una idea de cómo de bueno es AForge.</p>
<p style="text-align: justify;">He organizado el código en un fórmulario y tres clases. Hay una clase para binarizar una imagen aplicando unos pocos filtros, otra para obtener las estadísticas de Hough y la última realiza el recuento de “blobs”, en este caso árboles.</p>
<p style="text-align: justify;">Solamente me interesa destacar el uso de FilterSequence que permite predefinir el uso de varios filtros, lo cual resulta muy práctico.</p>

[csharp]

// binarization filtering sequence

FiltersSequence filter = new FiltersSequence(

new ContrastCorrection(),

new Mean(),

new GrayscaleBT709(),

new Threshold()

);

[/csharp]
<p style="text-align: justify;">Por otra parte, si usando filtros y otras herramientas notáis que hacen justo lo contrario de lo que deberían, es porque justamente están haciendo lo contrario :-) . Deberéis aplicar un Invert() al Bitmap. Esto me pasaba con el BlobCounter(), pues me devolvía la cuenta de todo lo que no eran árboles. También deberéis invertir la imagen si queréis usar filtros del tipo Erosion() o Dilatation().</p>
<p style="text-align: justify;"><strong>El código fuente lo podéis obtener haciendo un clon del siguiente repositorio GIT: </strong></p>
<p style="text-align: justify;"><a title="https://github.com/benizar/rapid.git" href="https://github.com/benizar/rapid.git" target="_blank">https://github.com/benizar/rapid.git</a></p>

<h3 style="text-align: justify;">Referencias</h3>
<ul style="text-align: justify;">
	<li>De nuevo la página del proyecto: <a href="http://code.google.com/p/aforge/">http://code.google.com/p/aforge/</a></li>
</ul>
<ul style="text-align: justify;">
	<li>Para entender mejor esta técnica podéis consultar <a href="http://en.wikipedia.org/wiki/Hough_transform">http://en.wikipedia.org/wiki/Hough_transform</a></li>
</ul>
<ul style="text-align: justify;">
	<li>Antes del verano asistí a un curso en Valencia (Esp.) donde aprendí bastante de este tema. Como me gustó bastante os adjunto la referencia por si lo repiten de nuevo el año que viene: Teledetección aplicada a la actualización de cartografía de ocupación del suelo: técnicas de clasificación orientada a objetos. Curso teórico-práctico. <a href="http://cgat.webs.upv.es/bigfiles/c_objetos/index.html">http://cgat.webs.upv.es/bigfiles/c_objetos/index.html</a></li>
</ul>
-------------------------------------------------------------------

Si quereis contactar podeis enviarme un email (asunto: gisandchips):

Benito M. Zaragozí

benito.zaragozi@ua.es