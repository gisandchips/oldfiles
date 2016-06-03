---
ID: 2037
post_title: >
  Una librería de código libre y abierto
  para el cálculo de métricas del
  paisaje
author: benizar
post_date: 2012-03-13 14:02:17
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2012/03/13/una-libreria-de-codigo-libre-y-abierto-para-el-calculo-de-metricas-del-paisaje/
published: true
---
Hola a todos,

Os presento un artículo de investigación que algunos miembros de GIS&amp;Chips publicamos el mes pasado en la revista “Environmental modelling &amp; software”. Trata de la creación de un API libre y abierto para el cálculo de las métricas del paisaje.
<p style="text-align: center;"><img class="size-full wp-image-2060 aligncenter" title="Póster landmetrics DIY" src="http://www.gisandchips.org/wp-content//PosterDIY_jul_2010.jpg" alt="" width="300" height="400" /></p>
<!--more-->

La utilidad de las métricas del paisaje en distintas aplicaciones ha sido generalmente aceptada y ha provocado que existan muchos paquetes de software diseñados para proporcionar cálculos y análisis de los patrones estructurales del paisaje. Es posible obtener más información <a href="http://www.umass.edu/landeco/research/fragstats/documents/fragstats_documents.html">aqui</a>

Tras examinar detenidamente las herramientas más utilizadas (Fragstats, V-Late, PA4, etc), se ha podido extraer una serie de puntos fuertes y débiles con la finalidad de crear una lista de las características deseables en este tipo de software. Tras dicho análisis se consideró necesario el diseño de un API sin limitaciones en los datos de entrada, capaz de calcular a partir de datos vectoriales o raster, etc. Este API debería facilitar no sólo la construcción de aplicaciones propias, sino que también debería permitir añadir nuevas métricas y la investigación de nuevos paradigmas relacionados con las métricas del paisaje. Con estas premisas se ha comenzado a desarrollar una propuesta, basada en estándares abiertos y software libre, que se ha denominado landmetrics-DIY (“Do It Yourself”). Podéis encontrar una versión alfa junto con un sencillo interfaz de usuario haciendo checkout en el siguiente repositorio subversión:
<p style="text-align: center;"><a title="http://www.gisandchips.org/svn/landmetrics_diy" href="http://www.gisandchips.org/svn/landmetrics_diy">http://www.gisandchips.org/svn/landmetrics_diy</a></p>
Tendréis que volver a añadir las referencias a NTS y os daréis cuenta de que el desarrollo está aún muy verde. Era solamente una primera propuesta, pero creemos que las directrices definidas en el artículo deberían marcar los criterios a seguir en el desarrollo de proyectos geoespaciales científicos de este tipo.

Por el momento, este API puede calcular unas 40 métricas del paisaje a partir de ficheros vectoriales Shapefile de ESRI, pero estamos trabajando para completar su contenido, siguiendo las líneas que se explica en el artículo.

Aquellos que estéis interesados en saber más, podéis ver el poster que hemos añadido al principio del post o podréis encontrar el artículo en <a href="http://www.sciencedirect.com/science/article/pii/S1364815211002209">ScienceDirect</a>

Hasta pronto.