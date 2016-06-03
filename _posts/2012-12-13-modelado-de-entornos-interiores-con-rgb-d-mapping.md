---
ID: 2247
post_title: >
  Modelado de entornos interiores con
  RGB-D mapping
author: Jorge Piera Llodrá
post_date: 2012-12-13 21:07:12
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2012/12/13/modelado-de-entornos-interiores-con-rgb-d-mapping/
published: true
---
Los modelos de entornos en 3D se usan intensamente en numerosas actividades reales, como por ejemplo la navegación. Crear modelos tridimensionales en tiempo real facilita la construcción de autómatas que necesitan tener un conocimiento del entorno cambiante en el que tienen que desarrollar su función.

El problema es que el hardware necesario para crear modelos 3D es caro y no está al alcance del público en general. Pero hay una excepción a esta afirmación: las cámaras RGB-D. Este tipo de cámaras son capaces de capturar imágenes en RGB y asociar a cada uno de los puntos tomados información de profundidad.
<p style="text-align: left;"><img class="aligncenter size-full wp-image-2249" src="http://www.gisandchips.org/wp-content//depth.png" alt="" width="438" height="166" />
Con esta información se pueden crear modelos tridimensionales que serán tan precisos como precisa sea la información tomada por la cámara y preciso sea el proceso utilizado para crear el modelo.</p>
La mayoría de los sistemas de creación de modelos mediante imágenes tienen 3 componentes principales: primero, el que se encarga de alinear los frames consecutivos; segundo, el que se encarga de detectar los bucles cerrados; y tercero, el que consigue un alineamiento global de toda la secuencia de datos.

Este artículo trata de un algoritmo concreto de creación de modelos 3D a partir de los datos de una cámara RGB-D. El algoritmo se describió en el artículo <em>RGB-D mapping: Using Kinect-style depth cameras for dense 3D modeling of indoor environments</em> [1] que se presentó en la revista <em>The International Journal of Robotics Research</em> el 10 de Febrero del 2012. En él se describe cómo ha resuelto cada uno de los tres problemas que tiene un sistema de este tipo. Todas las imágenes de este artículo han sido extraídas del artículo original. Para obtener más información se puede ir directamente a la <a href="https://www.cs.washington.edu/node/3544/">web de los autores</a>

En este primer apartado se introducen algunos conceptos generales que ayudarán al lector a entender mejor el algoritmo. En el apartado segundo se detalla el algoritmo objeto del estudio. En el tercer apartado se comentan los resultados obtenidos al aplicar el algoritmo en pruebas reales. En el cuarto se finaliza con un aparatado de conclusiones.

<!--more-->
<h2>El RGB-D mapping</h2>
<em>RGB-D mapping</em> es el nombre que le han dado los autores del artículo estudiado a su proceso para generar modelos a partir de una cámara RGB-D. Según cuentan los mismos, se trata de un proceso novedoso que consigue unos resultados más precisos que algunos otros procesos de obtención de modelos.

Para el estudio se ha utilizado una camára RGB-D que es capaz de tomar imágenes de 640 x 480 píxeles almacenando la profundidad para cada píxel y que captura 30 frames por segundo. Se trata de una cámara muy económica, que únicamente es capaz de tomar información de profundidad a menos de 5 metros con un error de 3 cm cada 3 metros y un ángulo de visión de unos 60º.

Hay cámaras mucho más precisas en el mercado pero no son tan económicas. Con estas limitaciones hardware la precisión del modelo 3D depende única y exclusivamente del algoritmo utilizado en la creación del modelo.

La siguiente figura  muestra un diagrama de componentes del proceso RGB-D mapping. Se parte de la información RGB y de profundidad que proporciona la cámara RGB-D.
<p style="text-align: left;"><img class="aligncenter size-full wp-image-2249" src="http://www.gisandchips.org/wp-content//structure.png" alt="" width="432" height="129" /></p>
<p style="text-align: left;">La información de profundidad proporciona una nube de puntos densa ({\em Dense Point Cloud}) que representa la profundidad que hay en cada punto. Y con la información de profundidad y con las imágenes RGB se crean las características dispersas (<em>Sparse Features</em>) que no es más que un conjunto de puntos que tienen asociados la información de color y de profundidad.</p>
Estos dos modelos de datos son los datos de entrada del componente RGBD-ICP que es el encargado de alinear los distintos frames y de detectar los posibles bucles que puedan haber. Recordemos que el alineamiento de frames es uno de los tres problemas que hay que resolver para tener un modelo 3D a partir de una cámara RGB-D.
<h3>Alineamiento de frames</h3>
Lo primero que se ejecuta es el algoritmo <em>RANSAC RGB-D ICP</em>. <em>RANSAC</em> es el acrónimo de <em>RANdom SAmple Consensus</em> que es un método iterativo que trata de estimar los parámetros de un modelo matemático a partir de un conjunto de datos de entrada.

En el caso concreto de este algoritmo, los parámetros de entrada son un frame con información de RGB-D <em>Ps</em>, otro frame a comparar <em>P_t</em> y una transformación <em>Tp</em> que inicialmente tiene el valor de la transformación identidad.
<p style="text-align: center;"><a href="http://www.gisandchips.org/wp-content//icp.png"><img class="size-full wp-image-2258 aligncenter" src="http://www.gisandchips.org/wp-content//icp.png" alt="" width="449" height="271" /></a></p>
Lo primero que se hace es, para cada uno de los dos frames extraer las características visuales de ambos y asociarlas con la información 3D con el fin de obtener características visuales en 3D (líneas 1 y 2). El algoritmo no indica qué características visuales hay que utilizar, pero comenta que ellos han usado las SIFT features descritas en [2]. La siguiente imagen muestra un ejemplo de ello.
<p style="text-align: center;"><img class="aligncenter size-full wp-image-2260" src="http://www.gisandchips.org/wp-content//features.png" alt="" width="474" height="174" /></p>
Con esos dos conjuntos de características, se aplica <em>RANSAC</em> para obtener la mejor transformación <em>T*</em> entre esos dos conjuntos y el conjunto de pares de puntos <em>Af</em> que han generado la mejor transformación (línea 3).

Si el conjuntos de pares de puntos que se han encontrado <em>Af</em> es menor que un cierto umbral (línea 4) no se puede afirmar que la transformación <em>T*</em> sea la adecuada, por lo tanto, se inicializa la transformación con el valor que tenía en la iteración anterior (línea 5) y se vacía el conjunto de pares de puntos <em>Af</em> (línea 6).

En la línea 8 empieza el proceso principal del algoritmo. La función <em>Compute_Closest_Point</em> aplica a cada uno de los puntos del frame de entrada <em>Ps</em> la transformación <em>T*</em> y para el punto obtenido trata de encontrar en el frame de salida <em>Pt</em> el punto más cercano. Para ello se pueden utilizar combinaciones de distancia euclídea, diferencia de color y diferencia en la forma.

El resultado de esta operación es el conjunto de asociaciones de pares de puntos  <em>Ad</em> que indica para cada punto del frame de entrada el punto del frame de salida que tiene más posibilidades de ser el mismo punto.

La función <em>Optimize_Alignment</em> de la línea 10 simplemente minimiza el error de alineamiento entre los puntos utilizando para ello la transformación<em> T*</em> y los dos conjuntos de pares de puntos <em>Af</em> y <em>Ad</em>. El objetivo es conseguir una nueva transformación <em>T*</em> que minimice los errores.

El proceso se repetirá hasta que la transformación no se modifique en una iteración o hasta que el número de iteraciones llegue a un umbral en cuyo caso se devolverá la última transformación calculada (líneas 11 y 12). Esta matriz es la que se puede aplicar para alinear correctamente dos frames consecutivos de la cámara por lo que el primer problema de los tres que se tenían que resolver ya está resuelto. Recordemos que los otros dos son la detección de bucles y la construcción global del modelo.
<h3>Detección de bucles</h3>
El alineamiento de frames no es perfecto y va introduciendo errores a medida que se avanza en la construcción del modelo global. Estos errores se van acumulando llegando incluso a representar la misma región del mundo real en diferentes partes del modelo. Esto es lo que se conoce como un bucle cerrado y es el segundo problema que hay que resolver.

La primera aproximación que se podría realizar para resolver este problema es ejecutar el algoritmo de <em>RANSAC</em> entre el frame actual y todos los frames anteriores que haya capturado la cámara. Aunque esto resolvería el problema, es computacionalmemte prohibitivo, por lo que hay que utilizar otra aproximación.

La solución a este problema ha sido definir <em>keyframes</em> que no son más que unos frames 'especiales' que se obtienen a medida que se van alineando los frames. Cada vez que se procesa un frame, se intenta asociar también con el último <em>keyframe</em> de forma que si se consigue alinear, el frame procesado pasará a ser un <em>keyframe</em>. La idea es tener un conjunto de frames especiales que no se puedan alinear entre si.

Cada vez que se crea un nuevo <em>keyframes</em>, se intenta alinear con los anteriores <em>keyframes</em>. Aunque no es necesario aplicar el costosísimo <em>RANSAC</em> en todos, ya que se puede aplicar un proceso previo que puede descartar dos frames rápidamente.

Si dos <em>keyframes</em> son candidatos, se aplicará el <em>RANSAC</em> entre ellos con el fin de averiguar si se trata de dos frames que puedan ser alineados. En caso afirmativo, se detectará un bucle y se asociarán los dos frames al mismo espacio del modelo. Ahora ya sólo queda resolver el tercer problema para crear el modelo completo que es la construcción global del modelo.
<h3>Construcción global del modelo</h3>
El la imagen de los componentes del sistema aparecen a la derecha del todo 3 componentes de los que no hemos hablado todavía y que tienen que ver con la construcción global del modelo. El primero de ellos, el <em>Global Optimization,</em> consiste en representar los frames y las relaciones entre frames en forma de una grafo que permite detectar posibles bucles. Para ello se usa el <em>TORO</em>, que es un algoritmo desarrollado en [3] y [4].

La segunda estrategia de optimización es utilizar SBA ([5], [6] y [7]) para minimizar el error en la reproyección de puntos entre los frames. Ninguno de estos dos algoritmos va a ser comentado en este documento ya que no es el objetivo del mismo.

Cada frame de una cámara RGB-D tiene 640 x 480 = 250.000 puntos. Si a esto le sumamos que al ir alineando frames se van añadiendo puntos, es fácil observar que el modelo resultante tendrá un número de puntos muy elevado. Si queremos representar el modelo habrá que convertir este conjunto de puntos en una estructura de datos más apropiada para su visualización.

En este ejemplo se ha utilizado el método de los <em>surfels</em> ([8] y [9]). Un <em>surfel</em> consiste en una localización, una orientación de la superficie, un tamaño y un color. Las nubes de puntos se pueden añadir, borrar y modificar de esta forma de representar los modelos que, es mucho más eficiente y está mucho más próxima a una representación para su visualización
<h2 style="text-align: left;">Experimentos</h2>
<p style="text-align: left;">Una vez comentado de forma general cómo funciona el algoritmo de creación de modelos tridimensionales, es necesario realizar algunas pruebas empíricas que lo pongan a prueba y que ofrezcan algún tipo de resultado que muestre lo bueno o malo que es el algoritmo.</p>
<p style="text-align: left;">Las pruebas se han realizado en 2 entornos cerrados: <em>las oficinas de los laboratorios de Intel en Seattle</em> y el <em>Centro Paul G. Allen para la Informática de la Universidad de Washington</em>. Durante la captura de frames, la cámara ha sido llevada por una sola persona que enfocaba en la dirección en la que caminaba.
<img class="aligncenter size-full wp-image-2262" src="http://www.gisandchips.org/wp-content//models.png" alt="" width="469" height="429" /></p>
Las dos imágenes de la izquierda  muestran los mapas obtenidos en estas dos zonas con una gran cantidad de bucles. El modelo de la imagen superior consiste en 906 frames tomados en una distancia de 71 metros mientras que el modelo de la imagen inferior se ha creado a partir de 716 frames para una distancia de 114 m.

Para comprobar la consistencia de los modelos generados, se han generado otros dos mapas en 2D utilizando un escáner láser (imágenes de la derecha). Se puede observar que ambos modelos modelan un entorno similar, por lo que parece ser que el método funciona y que es capaz de detectar bucles en los frames.

En cuanto a los <em>surfels</em>, cuántas más imágenes tenga de una zona, más <em>surfels</em> será capaz de añadir al modelo y más parecido será este a la realidad. Aunque muchas veces no haga falta tener una gran cantidad de frames para poder alinearlos, si tenemos muchos frames conseguiremos una representación final del modelo mucho más parecida a la realidad.
<h2>Conclusiones</h2>
Cuando en la primera parte del artículo se comentan las características de la cámara utilizada (640 x 480 píxeles, con un error de 3 cm cada 3 metros y un ángulo de visión de unos 60º) parece complicado pensar que van a conseguir buenos modelos. A priori, parece muy complicado poder encontrar características comunes en dos imágenes de 640 x 480.

Y por si esto fuera poco, los dos entornos que han utilizado para hacer las pruebas son entornos muy similares a medida que se avanza con la cámara por ellos. Con estos datos de entrada, a priori, todo parece indicar que va a ser muy complicado enlazar dos frames y más todavía, poder detectar bucles.

Pero cuando ves los dos modelos obtenidos y entiendes un poco la complejidad del algoritmo te das cuenta de la complejidad del trabajo que ha hecho este grupo de investigación. Mejorando la complejidad del algoritmo de creación de modelos tridimensionales, se pueden fabricar dispositivos relativamente económicos (la Kinect es un ejemplo de ellos) e integrarlos en cualquier aparato hardware que lo requiera.

Y si a esto le añadimos un mejor hardware que llegará y se abaratará con el tiempo, estamos en un presente que augura un futuro de autómatas que serán capaces de interactuar con el entorno a un coste relativamente barato.
<h2>Referencias</h2>
[1] Peter Henry, Michael Krainin, Eva Herbst, Xiaofeng Ren and Dieter Fox (2012): RGB-D mapping: Using Kinect-style depth cameras for dense 3D modeling of indoor environments. http://ijr.sagepub.com/content/early/2012/02/10/0278364911434148.full.pdf+html
[2] Wu, C. (2007). SiftGPU: A GPU implementation of scale invariant feature transform (SIFT). http://cs.unc.edu/ccwu/siftgpu
[3] Grisetti G, Grzonka S, Stachniss C, Pfaff P and Burgard W (2007a): Estimation of accurate maximum likelihood maps in 3D. In Proceedings of the IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS).
[4] Grisetti G, Stachniss C, Grzonka S and Burgard W (2007b): A tree parameterization for efficiently computing maximum likelihood maps using gradient descent. In Proceedings of Robotics: Science and Systems (RSS).
[5] Triggs B, McLauchlan P, Hartley R and Fitzgibbon A (2000): Bundle adjustment—a modern synthesis. Vision Algorithms: Theory and Practice 1883: 153–177.
[6] Lourakis M and Argyros A (2009): SBA: A software package for generic sparse bundle adjustment. ACM Transactions on Mathematical Software 36: 1–30.
[7] Konolige K (2010b): Sparse sparse bundle adjustment. In Proceedings of the British Machine Vision Conference (BMVC).
[8] Pfister H, Zwicker M, van Baar J and Gross M (2000): Surfels: Surface elements as rendering primitives. In: SIGGRAPH 2000, Proceedings of the 27th Annual Conference on Computer Graphics, pp. 335–342.
[9] Krainin M, Henry P, Ren X and Fox D (2011): Manipulator and object tracking for in-hand 3D object modeling. The International Journal of Robotics Research 30(11): 1311–1327.