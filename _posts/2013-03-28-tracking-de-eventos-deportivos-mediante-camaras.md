---
ID: 2333
post_title: >
  Tracking de eventos deportivos mediante
  cámaras
author: Jorge Piera Llodrá
post_date: 2013-03-28 00:00:58
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2013/03/28/tracking-de-eventos-deportivos-mediante-camaras/
published: true
---
Definimos tracking como la capacidad de conocer la posición en la que se encuentra un objeto en tiempo real. Con el abaratamiento en los últimos años de los dispositivos GPS y de la telefonía móvil, el tracking ha adquirido mucha mayor relevancia ya que a priori, un teléfono móvil es suficiente para poder hacer el seguimiento de un objeto.

Pero este tipo de soluciones basadas en asociar un dispositivo físico con el objeto a seguir no pueden utilizarse en todos los escenarios. Aplicado al mundo de los eventos deportivos, existen deportes como el ciclismo o la vela en la que se puede utilizar esta aproximación pero hay otras como el fútbol o los deportes indoor en los que la opción de enlazar un dispositivo con cada competidor no es viable.

En estos casos se tiene que utilizar otro tipo de tecnología que sea capaz de hacer el tracking de los competidores desde fuera del terreno de juego. Las soluciones basadas en cámaras es una de las más utilizadas en la actualidad existiendo soluciones comerciales como <a href="http://www.dartfish.com/en/sports/team-sports/index.htm">[1]</a>.

El tracking de personas es un tema que ha sido estudiado desde muchos otros campos como los sistemas de ambientes inteligentes, visual servoing, interacción hombre-máquina, compresión de vídeo o robótica. Aunque muchos de estos estudios se basan en aproximaciones basadas en cámaras simples, existen problemas como el de la oclusión que hacen que este tipo de sistemas sencillos no funcionen correctamente. Sistemas basados en múltiples cámaras pueden ayudar a resolver estos problemas <a href="http://www.sciencedirect.com/science/article/pii/S0888613X09000346">[2]</a>.

Este artículo es un estado del arte del tracking de eventos deportivos mediante cámaras de vídeo.

[caption id="attachment_2357" align="aligncenter" width="520"]<a href="http://www.gisandchips.org/wp-content//rugby.png"><img class="size-full wp-image-2357  " alt="" src="http://www.gisandchips.org/wp-content//rugby.png" width="520" height="75" /></a> La figura muestra distintas capturas de pantalla de un evento de fútbol<br />americano [5][/caption]<!--more-->

En el siguiente apartado se van a enumerar los problemas encontrados a la hora de hacer seguimiento de competidores en eventos deportivos. En el apartado 2 se proponen para cada uno de los problemas encontrados, algunas de las soluciones propuestas por algunos autores en las revistas de investigación. Para concluir, en el apartado 3, se detallan algunas conclusiones de este trabajo.

Este documento no pretende entrar en detalle en ninguna de las soluciones propuestas. Para más información es más que recomendable utilizar directamente las referencias enlazadas.
<h2>1. Problemática</h2>
Este apartado presenta los distintos problemas que tiene que resolver un sistema de tracking mediante cámaras. No se van a comentar soluciones concretas que dan los autores para solucionar los problemas sino que se van a exponer los problemas encontradas en todos los artículos de investigación estudiados.

El el siguiente apartado se comentarán una por una las soluciones propuestas para resolver cada uno de los problemas.
<h3>1.1. Georeferenciación de las imágenes</h3>
Las cámaras son capaces de capturar información en forma de imágenes que no son más que matrices de píxeles que se caracterizan por sus coordenadas <em>x</em> e<em> y</em> (y <em>z</em>) y el valor <em>RGB</em> capturado. El problema es que estas coordenadas no son coordenadas del mundo real y para poder realizar el tracking de los objetos es necesario asociar puntos de la imagen con coordenadas del mundo real (o al menos con un sistema de referencia del terreno de juego donde se está disputando el evento). Esta asociación se conoce como georeferenciación.

La siguiente figura <a href="http://www.sciencedirect.com/science/article/pii/S1077314209001027">[4]</a> muestra la relación que existe entre un punto tomado en el sistema de coordenadas de la cámara <em>(p(x, y))</em> y un punto en el sistema de coordenadas del mundo real <em>(P(X, Y, Z))</em>.

[caption id="attachment_2334" align="aligncenter" width="408"]<a href="http://www.gisandchips.org/wp-content//camera.png"><img class="size-full wp-image-2334" alt="" src="http://www.gisandchips.org/wp-content//camera.png" width="408" height="402" /></a> La figura muestra la relación entre el sistema de coordenadas de la cámara<br />y el sistema de coordenadas del mundo real [4][/caption]
<h3>1.2. Identificación de competidores</h3>
Es necesario identificar a los distintos competidores con el fin de poder personalizar los datos obtenidos de un competidor. El objetivo final de tracking es poder crear estadísticas en tiempo real del juego tales como los kilómetros recorridos, tiempo de ataque, tiempo en defensa... y para ello es necesario asociar cada uno de los objetos en movimiento con personas del mundo real.
<h3>1.3. Alineamiento de frames</h3>
Las cámaras de vídeo capturan una secuencia de frames a medida que van gravando. El problema es que la cámara se mueve y es necesario alinear estos frames con el fin de poder generar un modelo continuo del ambiente que se está gravando.

Si se pueden establecer las equivalencias entre las secuencias de frames, se podrá conocer el movimiento de la cámara y en consecuencia, se sabrá dónde está enfocando la cámara.
<h3>1.4. El movimiento de los competidores</h3>
Es un problema similar al de alineamiento de los frames pero para competidores con la dificultad añadida de que estos se encuentran en constante movimiento. Es necesario establecer correspondencias de un competidor entre dos frames consecutivos con el fin de poder realizar el tracking de un competidor.

Este problema presenta la dificultan añadida de que los competidores entran y salen del foco de la cámara constantemente.
<h3>1.5. Las oclusiones</h3>
Las oclusiones se producen cuando un competidor se coloca entre la cámara y otro competidor, dejando a este último momentáneamente en un estado de oclusión. La siguiente figura muestra un ejemplo de un competidor que se encuentra en un estado de oclusión.

[caption id="attachment_2347" align="aligncenter" width="238"]<a href="http://www.gisandchips.org/wp-content//oclusion1.png"><img class="size-full wp-image-2347  " alt="" src="http://www.gisandchips.org/wp-content//oclusion1.png" width="238" height="156" /></a> La figura muestra el ángulo de visión de una cámara que está siguiendo a dos competidores. Cuando el competidor 1 se posiciona entre la cámara y el competidor 2, este último deja de estar visible.[/caption]

Los deportes indoor se suelen practicar en campos pequeños. Por ejemplo, en un deporte como el baloncesto si se utiliza una única cámara para seguir a todos los competidores, se estarán produciendo continuas oclusiones que el sistema deberá ser capaz de resolver.

Ligado a este problema está el problema de diferenciar a dos competidores en el momento en el que uno de ellos se encuentra en un estado de oclusión parcial. Es sistema debe ser capaz de detectar este estado y diferenciar a los dos competidores. En general para un deporte como el baloncesto y con una cámara podrían estar en estado de oclusión parcial los 10 competidores.

Además está el problema de las oclusiones cuando un competidor entra y sale de la escena. No todas las cámaras enfocan a todos los competidores y constantemente los competidores entran y salen del foco de la cámara. Detectar que un nuevo competidor ha entrado o ha salido de una escena es otro tipo de oclusión.
<h2>2. Soluciones propuestas</h2>
En este apartado se presentan algunas propuestas de solución a los problemas descritos en el apartado anterior.
<h3>2.1. Georeferenciación de las imágenes</h3>
La forma de realizar un cambio de sistema de coordenadas es mediante una homografía que no es más que una matriz de transformación de $3x3$ que al multiplicarla por un punto del sistema de coordenadas de origen nos da un punto del sistema de coordenadas destino. En el caso del tracking mediante cámaras, la homografía proyecta los puntos de las imágenes tomadas en coordenadas del mundo real.

Para obtener un punto del sistema de coordenadas real a partir del sistema de coordenadas de la imagen simplemente hay que utilizar la siguiente fórmula:
<p align="center">P(X, Y) = M^t p(x, y)</p>
En la fórmula superior estamos asumiendo que los dos sistemas de coordenadas son planos: estamos tratando de mapear los puntos de la cámara con puntos del terreno de juego. De momento se asume que no hay jugadores en la imagen.

Si la cámara tuviera información de profundidad, la fórmula anterior podría utilizar las tres dimensiones con lo que se podrían modelar terrenos de juego mucho más complejos ya que se dispondría de una tercera dimensión.
<p align="center">P(X, Y, Z) = M^t p(x, y, z)</p>
El problema de la georeferenciación es el de obtener la matriz de transformación. Éste es un problema matemático que necesita de unos valores iniciales que relacionan los dos sistemas de coordenadas. Estos valores iniciales se obtienen con la calibración de la cámara.

La cámara se puede calibrar introduciendo valores iniciales (correspondencias entre puntos de la cámara y puntos del mundo real) y su posición respecto al terreno de juego de forma que al moverse la cámara detecta puntos del terreno de juego y puede crear la matriz de transformación adecuada.

Otra forma de calibrado es mediante sensores de orientación <a href="http://ieeexplore.ieee.org/xpl/login.jsp?tp=&amp;arnumber=6392583&amp;url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D6392583">[6]</a> que le indican a la cámara en todo momento la posición en la que se encuentra.

Existe una problemática adicional que no se ha comentado hasta el momento y que consiste en la multiplicidad de las cámaras. Normalmente en un evento deportivo indoor hay diversas cámaras y todas ellas tienen que estar calibradas entre sí para poder crear el mismo modelo.

<a href="http://ieeexplore.ieee.org/xpl/articleDetails.jsp?reload=true&amp;arnumber=4379605">[7]</a> propone un sistema para calibrar múltiples cámaras basado en el reconocimiento de los mismos patrones por distintas cámaras. <a href="http://crcv.ucf.edu/papers/handoffHUMO2000.pdf">[8]</a> propone otra aproximación sin reconocimiento de características basada en la detección de los campos de visión de cada una de las cámaras mediante la visualización directa de las otras cámaras.

Por otra parte <a href="http://ieeexplore.ieee.org/xpl/login.jsp?tp=&amp;arnumber=897380&amp;url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D897380">[9]</a> propone otra forma de calibrado de múltiples cámaras basada en el reconocimiento de los individuos que se mueven por una determinada escena.
<h3>2.2. Identificación de competidores</h3>
Los competidores se tienen que poder identificar para poder mostrar estadísticas. El la práctica los sistemas se centran en poder diferenciar a unos competidores de otros y es trabajo del administrador del sistema asociar los distintos competidores del sistema con sus nombres reales.

De ese modo el problema de la identificación se divide en dos problemas: reconocer a un competidor en la imagen y diferenciarlo del resto de competidores.

El primer problema es diferenciar en una imagen lo que es competidor de lo que es pista. A lo largo del evento el competidor puede presentarse ante las cámaras en muchas posiciones distintas por lo que establecer un patrón de posición de un competidor es en la práctica inviable. La siguiente figura muestra las distintas imágenes tomadas por las cámaras en una carrera de patinaje sobre hielo.

[caption id="attachment_2349" align="aligncenter" width="457"]<a href="http://www.gisandchips.org/wp-content//skaters.png"><img class="size-full wp-image-2349 " alt="" src="http://www.gisandchips.org/wp-content//skaters.png" width="457" height="235" /></a> La figura muestra las distintas imágenes que toman las cámaras que siguen una carrera de patinaje sobre hielo [3][/caption]
<p style="text-align: left">Para poder reconocer a un competidor algunos autores aplican la técnica de diferencias con la imagen de referencia que consiste en calcular la diferencia de imágenes entre una imagen del entorno sin competidores y otra imagen con competidores. Obviamente la diferencia será una imagen con competidores.</p>
<a href="http://ieeexplore.ieee.org/xpl/login.jsp?tp=&amp;arnumber=868677&amp;url=http%3A%2F%2Fieeexplore.ieee.org%2Fiel5%2F34%2F18808%2F00868677">[10]</a> propone un método para realizar las diferencias con la imagen de referencia en el que se evalúan los píxeles con una distribución Gausiana con el fin de discriminar los píxeles del terreno de juego de los píxeles de los competidores.

Una vez identificados los competidores, se puede aplicar una técnica de clasificación utilizando vectores de características para poder clasificar a los distintos competidores que se encuentran en la imagen <a href="http://server.cs.ucf.edu/~vision/papers/trackECCV02.pdf">[11]</a>.

En otro tipo de deportes como las carreras de patinaje sobre hielo, todos los competidores visten de la misma forma por lo que el reconocimiento mediante vector de características del cuerpo resulta menos eficiente. En este caso <a href="http://www.sciencedirect.com/science/article/pii/S1077314209001027">[3]</a> propone una solución fijándose en el casco de los competidores como elemento de mayor discriminación.

Un problema asociado al reconocimiento de los competidores es el de reconocimiento de las sombras. Mediante la técnica de diferencias con la imagen de referencia la imagen resultante incluye las sombras de los competidores. Estas sombras tienen que ser eliminadas para poder diferenciarlas del competidor.

Para poder eliminar las sombras <a href="http://server.cs.ucf.edu/~vision/papers/trackECCV02.pdf">[11]</a> propone un modelo que se basa en explotar las diferencias entre la sombra y la imagen del competidor: los píxeles de la sombra son más oscuros que los de la imagen de referencia y siempre mantienen información de la textura y el color de la imagen original. Explotando estas dos características se puede discriminar una sombra y eliminarla del procesamiento.
<h3>2.3. Alineamiento de frames</h3>
Para poder alinear dos frames lo más sencillo es buscar vectores de características entre dos frames consecutivos. Con esos dos vectores de características se puede aplicar el algoritmo <i>RANSAC</i> que devuelve una matriz de transformación entre los dos frames <a href="http://www.cs.washington.edu/node/4819/">[13]</a> que se utiliza para asociar cada uno de los puntos del primer frame con cada uno de los puntos del segundo frame.

El cálculo de vectores de características es muy costoso y hay otros autores que utilizan otras aproximaciones. En <a href="http://research.microsoft.com/apps/pubs/default.aspx?id=155416">[14]</a> se realiza el alineamiento utilizando el algoritmo ICP Iterative Closest Point que trata de buscar los puntos más cercanos entre los píxeles de ambos frames. Este algoritmo es mucho más eficiente que el de calcular los vectores de características.

Las dos soluciones propuestas se utilizan para alinear las imágenes del campo de juego pero ninguna de las dos tiene en cuenta el movimiento de los competidores. Antes de realizar el alineamiento de los frames es necesario diferenciar qué parte de la imagen es competidor y qué parte de la imagen es terreno de juego. Los algoritmos de alineamiento se aplican únicamente sobre esta última descartando por completo la parte de la imagen que contiene a los jugadores.

El alineamiento de frames indica información del movimiento que ha sufrido la cámara en un momento dado. Si la cámara está correctamente calibrada, se podrá obtener el movimiento preciso que ha realizado el operador para seguir el movimiento de los jugadores.
<h3>2.4. El movimiento de los competidores</h3>
Una de las mayores problemáticas del tracking de eventos deportivos es el movimiento de los jugadores. Hasta este momento se ha comentado cómo es posible detectar e identificar a los competidores en una imagen estática. Una vez hecho esto, se han tenido que eliminar de las imágenes capturadas la parte de la imagen donde aparecen los jugadores para poder realizar el alineamiento de los frames. Pero el tracking se basa en poder detectar el movimiento de los competidores y este es un problema que hasta este apartado ha sido ignorado.

<a href="http://cil.cs.ucf.edu/pdf/junejo_ICCV07_pathModeling.pdf">[15]</a> propone un modelo de tracking que se basa en el reconocimiento de la cabeza y de los pies de un competidor. Para identificar estas posiciones es necesario calcular el centro de masas y en momento de segundo orden de la parte de superior e inferior de la región que contiene la silueta del competidor <a href="http://ieeexplore.ieee.org/xpl/login.jsp?tp=&amp;arnumber=1544942&amp;url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D1544942">[12]</a>.

La siguiente figura muestra dos frames consecutivos que enfocan a un mismo competidor. Si el competidor está identificado y se detectan los puntos de la cabeza y de los pies se puede mediante una translación determinar el movimiento que ha realizado el competidor. Obviamente hay que tener en cuenta además el movimiento de la cámara.

[caption id="attachment_2353" align="aligncenter" width="453"]<a href="http://www.gisandchips.org/wp-content//junejo.png"><img class="size-full wp-image-2353 " alt="" src="http://www.gisandchips.org/wp-content//junejo.png" width="453" height="340" /></a> La figura muestra la relación que existe entre dos frames consecutivos en los que aparece un competidor [15][/caption]
<p style="text-align: left">Este cálculo lo realizan todas las cámaras en paralelo de forma que aunque una cámara no pueda determinar las posiciones de la cabeza o de los pies de un competidor, es muy posible que otra cámara si que las haya podido obtener.</p>

<h3>2.5. Las oclusiones</h3>
Disponer de múltiples cámaras que están capturando datos en paralelo minimiza considerablemente el problema de las oclusiones. Las cámaras deben estar calibradas correctamente generando entre todas ellas una representación del mismo espacio geográfico. Cuando un competidor está siendo oclusionado por otro competidor desde el punto de vista de una cámara concreta, es muy posible que haya otra cámara que no sufra esta oclusión y que pueda identificar al competidor oclusionado <a href="http://www.sciencedirect.com/science/article/pii/S0888613X09000346">[2]</a>.

El mismo razonamiento se podría aplicar al oclusionamiento de escena: puede que una cámara no esté enfocando a un competidor pero es muy probable que otra cámara sí que lo esté haciendo. En este caso el sistema conoce la posición del competidor que no aparece en la escena y podría en un momento dado detectar cunado un competidor va a entrar en el foco de una cámara en función de los movimientos del competidor y de los movimientos de la cámara.

Independientemente de la solución basada en múltiples cámaras, también existen propuestas que tratan de solucionar el problema para una única cámara. En una de estas propuestas <a href="http://www.sciencedirect.com/science/article/pii/S0165168409004563">[16]</a> propone un modelo basado en campos aleatorios de Markov. Cuando no hay oclusiones, el sistema está capturando constantemente características de los individuos que se encuentran en movimiento y cuando hay una oclusión, utiliza el conjunto de características para separar las partes de la imagen que representan a cada competidor.

[caption id="attachment_2337" align="aligncenter" width="540"]<a href="http://www.gisandchips.org/wp-content//col.png"><img class="size-full wp-image-2337 " alt="" src="http://www.gisandchips.org/wp-content//col.png" width="540" height="72" /></a> La figura muestra el reconocimiento de personas en movimiento realizado<br />según el algoritmo propuesto por [16][/caption]
<h2>3. Conclusiones</h2>
En este artículo se han enumerado los principales problemas que existen en el tracking de eventos deportivos mediante cámaras. Aunque la investigación se ha centrado en los eventos deportivos, la mayoría de problemas encontrados son aplicables al tracking de objetos móviles mediante cámaras.

Para cada problema se han citado varias de las soluciones dadas por los algunos autores en sus respectivos artículos de investigación. Vistos los resultados que han obtenido los autores y a falta de realizar una prueba empírica, se puede concluir que el tracking de eventos deportivos mediante cámaras se puede realizar con la tecnología hardware actual  utilizando algunos de los algoritmos propuestos para resolver los distintos problemas que presenta.

Para más información es muy recomendable leer las referencias a todos los artículos aquí mencionados.
<h2>Referencias</h2>
<a href="http://www.dartfish.com/en/sports/team-sports/index.htm">[1]</a> http://www.dartfish.com/en/sports/team-sports/index.htm
<a href="http://www.sciencedirect.com/science/article/pii/S0888613X09000346">[2]</a> Mu~noz-Salinas, R., Medina-Carnicer, R., Madrid-Cuevas, F.J., Carmona-Poyato, Multi-camera people tracking using evidential filters. En: International Journal of Approximate Reasoning 50 732{749. 2009.
<a href="http://www.sciencedirect.com/science/article/pii/S0031320309001204">[3]</a> GuoJun, L., Tang, X., Cheng, H. D., Huang, J., Liu, J.: A novel approach for tracking high speed skaters in sports using a panning camera. En: Pattern Recognition 42, 2922 - 2935, 2009.
<a href="http://www.sciencedirect.com/science/article/pii/S1077314209001027">[4]</a> Chen, C., Yao, Y., Page, D., Abidi, B., Koschan, A., Abidi, M.: Camera handoff and placement for automated tracking systems with multiple omnidirectional cameras. En: Computer Vision and Image Understanding 114 179{197. 2010.
<a href="http://www.sciencedirect.com/science/article/pii/S1877705810003036">[5]</a> Gandhi, H., Collins, M., Chuang, M., Narasimhan, P.: Real-Time Tracking of Game Assets in American Football for Automated Camera Selection and Motion Capture. En: 8th Conference of the International Sports Engineering Association (ISEA). 2010.
<a href="http://ieeexplore.ieee.org/xpl/login.jsp?tp=&amp;arnumber=6392583&amp;url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D6392583">[6]</a> Miti, R., Pongthorn, A., Supakorn S., Thitiporn, C., Makoto, S.: Planar surface area calculation using camera and orientation sensor. En: Cyber Technology in Automation, Control, and Intelligent Systems (CYBER), IEEE. 2012.
<a href="http://ieeexplore.ieee.org/xpl/articleDetails.jsp?reload=true&amp;arnumber=4379605">[7]</a> Orekhov, V., Abidi, B., Broaddus, C., Abidi, M.: Universal camera calibration with automatic distortion model selection. En: Proceedings of the IEEE International Conference on Image Processing ICIP2007, vol. VI, San Antonio TX. August 2007.
<a href="http://crcv.ucf.edu/papers/handoffHUMO2000.pdf">[8]</a> Javed, O., Khan, S., Rasheed, Z., Shah, M.: Camera handoff: tracking in multiple uncalibrated stationary cameras. En: Workshop on Human Motion 2000. Pages 113-118. 2000.
<a href="http://ieeexplore.ieee.org/xpl/login.jsp?tp=&amp;arnumber=897380&amp;url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D897380">[9]</a> Junejo, I. N.: Camera calibration for uneven terrains by observing pedestrians. En: 19th International Conference on Pattern Recognition. Pages 1-4. 2008
<a href="http://ieeexplore.ieee.org/xpl/login.jsp?tp=&amp;arnumber=868677&amp;url=http%3A%2F%2Fieeexplore.ieee.org%2Fiel5%2F34%2F18808%2F00868677">[10]</a> Stauffer C., Grimson.: Learning Patterns of Activity using Real Time Tracking. En: IEEE PAMI, Vol .22, No. 8. 747-767. 2000.
<a href="http://server.cs.ucf.edu/~vision/papers/trackECCV02.pdf">[11]</a> Javed, O., Shah, M.: Tracking and object classification for automated surveillance. En: the seventh European Conference on Computer Vision. 2002.
<a href="http://ieeexplore.ieee.org/xpl/login.jsp?tp=&amp;arnumber=1544942&amp;url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D1544942">[12]</a> Krahnstoever, N., Mendonca, P. R. S.: Bayesiann autocalibration for surveillance. In: Tenth IEEE International Conference on Computer Vision. 2005.
<a href="http://www.cs.washington.edu/node/4819/">[13]</a> Henry, P., Krainin, M., Herbst, E., Ren, X., Fox, D.: RGB-D mapping: Using Kinect-style depth cameras for dense 3D modeling of indoor environments. En: The International Journal of Robotics Research. 2012.
<a href="http://research.microsoft.com/apps/pubs/default.aspx?id=155416">[14]</a> Izadi, S., Kim, D., Hilliges, O., Molyneaux, D., Newcombe, R., Kohli, P., Shotton, J., Hodges, S., Freeman, D., Davison, A., Fitzgibbon, A.: Kinect Fusion: Real-time 3D Reconstruction and Interaction Using a Moving Depth Camera. 2011.
<a href="http://cil.cs.ucf.edu/pdf/junejo_ICCV07_pathModeling.pdf">[15]</a> Junejo, I. N., Foroosh, H.: Trajectory Rectification and Path Modeling for Video Surveillance. En: IEEE 11th International Conference on Computer Vision. 2007.
<a href="http://www.sciencedirect.com/science/article/pii/S0165168409004563">[16]</a> Wua, M., Peng, X., Zhang, Q., Zhao, R.: Patches-based Markov random field model for multiple object tracking under occlusion. En: Signal Processing 90. 1518-1529. 2010.