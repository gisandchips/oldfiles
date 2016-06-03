---
ID: 1303
post_title: >
  Extracción de características
  estructurales de una imagen ( II )
  (Semi-variograma de una imagen usando
  R).
author: benizar
post_date: 2010-01-28 13:45:08
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2010/01/28/extraccion-de-caracteristicas-estructurales-de-una-imagen-ii-semi-variograma-de-una-imagen-usando-r-2/
published: true
---
En este artículo usaremos una librería de R (“fields”), que contiene métodos para calcular el semi-variograma empírico de una imagen, lo cual quiere decir que calcularemos el semi-variograma de una gran cantidad de puntos. Y a partir de ahí, analizaremos la existencia de patrones espaciales en tres parcelas agrícolas arbóreas.
<h5><strong>Figura 1: Parcelas estudiadas en este artículo.</strong></h5>
<strong><a href="http://www.gisandchips.org/wp-content/parcelas.jpg"><img class="aligncenter size-medium wp-image-1238" src="http://www.gisandchips.org/wp-content/parcelas-300x115.jpg" alt="" width="415" height="158" /></a></strong>

<!--more-->

Espero que este sea el primero de varios posts donde aprendamos a estudiar las ortoimagenes, extrayendo características de distintos tipos como ya hicimos en un post anterior (<a href="../2009/11/11/procesamiento-de-imagenes-digitales-con-c-y-una-aplicacion-para-el-analisis-de-parcelas-agricolas/">http://www.gisandchips.org/2009/11/11/procesamiento-de-imagenes-digitales-con-c-y-una-aplicacion-para-el-analisis-de-parcelas-agricolas/</a> ). En aquel caso, se extrajeron algunas sencillas frecuencias a partir del cálculo de la Transformada de Hough que nos proporcionaba AForge.NET, lo cual nos servía para automatizar la decisión de si una parcela era una plantación arbórea y realizar un conteo automático del número de árboles que contenía. En dicho post, se ofrece una aplicación llamada RAPID, que incorpora una galería de imágenes de parcelas agrícolas. Hemos extraído tres de ellas para realizar los siguientes análisis. Son las siguientes:

Al estudiar las 3 parcelas que aparecen en la <strong>Figura 1</strong> con el RAPID: La primera es identificada totalmente como agrícola y nos permite contar con bastante certidumbre el número de árboles de un modo automático. En el caso de la segunda, su estructura en hileras no permite el conteo de árboles pues están muy juntos. Y por último, en los olivos de la tercera no se cumplía la regla de una estructura normal, aunque sí que podíamos contar los árboles. Este último caso era el único donde RAPID no podía “acertar” pues identificaba la parcela como no agrícola debido a que la diferencia angular entre las direcciones principales de la Transformada de Hough no se aproximaba a 90º.

<strong>¿Qué es un variograma experimental?</strong> (una breve explicación para entender los resultados)

Los variogramas se utilizan para caracterizar la posible estructura espacial de un conjunto de datos. Podemos distinguir dos tipos de variogramas, el experimental y el modelizado. De estos nos interesa más el experimental. Este último se usa para describir los datos de una muestra, habitualmente una nube de puntos (xyz)

Matemáticamente el semi-variograma, o variograma/2, se puede definir como la mitad del promedio de las diferencias al cuadrado.
<h5><strong><strong>Figura 2: Fórmula para calcular el semi-variograma</strong></strong></h5>
<strong><strong><a href="http://www.gisandchips.org/wp-content/formula_vario.jpg"><img class="aligncenter size-full wp-image-1342" src="http://www.gisandchips.org/wp-content/formula_vario.jpg" alt="" width="254" height="46" /></a>
</strong></strong>

Donde Np(h) es el número de pares a la distancia h, h es el incremento.

Z(xi) son los valores experimentales.

xi localizaciones donde son medidos los valores z(xi).

Aplicado a imágenes, el semi-variograma (o variograma/2), mide el grado de correlación espacial entre los píxeles de una imagen. Podemos comentar varios conceptos (ver <strong>Figura 3</strong>):
<ul>
	<li>Sill o meseta: representa la varianza máxima.</li>
	<li>Range, rango o alcance: muestra la distancia (en nuestro caso en píxeles) a la que el semi-variograma alcanza la meseta.</li>
	<li>Nugget o “efecto pepita”, es la discontinuidad en el origen. Ésta es debida a que el semivariograma, en la práctica, no es nulo en el origen.</li>
</ul>
<h5><strong><strong>Figura 3: Interpretación del semivariograma</strong></strong></h5>
<strong><strong><a href="http://www.gisandchips.org/wp-content/variogram.jpg"><img class="aligncenter size-medium wp-image-1343" src="http://www.gisandchips.org/wp-content/variogram-300x184.jpg" alt="" width="300" height="184" /></a>
</strong></strong>

<strong>Ejemplos de cálculo en R</strong>

Os aviso de que las imágenes referidas en el código estan en formato *.ppm aunque R tiene muchas librerías capaces de convertir a este formato. También podéis utilizar OpenOffice.

En este post, aplicamos el siguiente código para calcular el semi-variograma empírico de cada una de estas imágenes:

[shell]
&lt;pre&gt;# Instalo los packages necesarios:

#-------------------------------------------------

install.packages(&quot;pixmap&quot;, dependencies= T)

install.packages(&quot;fields&quot;, dependencies= T)

# Cargo las librerias:
#-------------------------------------------------
library(pixmap)

library(fields)

# Hago una lista con las imagenes de las parcelas y las cargo todas:
#-------------------------------------------------
imag_dir &lt;- list.files(&quot;C:\\ ... \\Articulos Gisandchips\\ parcelas_ppm\\&quot;, full.names=T)

parcela1 &lt;- read.pnm(imag_dir[1])

parcela2 &lt;- read.pnm(imag_dir[2])

parcela3 &lt;- read.pnm(imag_dir[3])

# Aislamos una banda...
#-------------------------------------------------
matriz1&lt;-parcela1@green

matriz2&lt;-parcela2@green

matriz3&lt;-parcela3@green

# Calculamos el semivariograma (si queremos podemos acabar antes cambiando el alcance de 40  a 10, por ejemplo, pero los resultados pueden cambiar y quedará menos bonito (-: )
#-------------------------------------------------
vgram1_40&lt;-vgram.matrix( matriz1, R=40) # esto llevará un poco de tiempo

vgram2_40&lt;-vgram.matrix( matriz2, R=40) # esto llevará un poco de tiempo

vgram3_40&lt;-vgram.matrix( matriz3, R=40) # esto llevará un poco de tiempo

# Añadimos al Plot las matrices que acabamos de calcular
#-------------------------------------------------
plot.vgram.matrix(vgram1_40)# La matriz del variograma

plot.vgram.matrix(vgram2_40)# La matriz del variograma

plot.vgram.matrix(vgram3_40)# La matriz del variograma

# Creamos una curva que ajuste bien sobre la muestra y la añadimos al variograma
#-------------------------------------------------
polyfit1_40_20 &lt;- lm(vgram1_40$vgram ~ poly(vgram1_40$d, 20));

plot(vgram1_40$d, vgram1_40$vgram)

lines(sort(vgram1_40$d), polyfit1_40_20$fit[order(vgram1_40$d)], col=2, lwd=4)

polyfit2_40_20 &lt;- lm(vgram2_40$vgram ~ poly(vgram2_40$d, 20));

plot(vgram2_40$d, vgram2_40$vgram)

lines(sort(vgram2_40$d), polyfit2_40_20$fit[order(vgram2_40$d)], col=2, lwd=4)

polyfit3_40_20 &lt;- lm(vgram3_40$vgram ~ poly(vgram3_40$d, 20));

plot(vgram3_40$d, vgram3_40$vgram)

lines(sort(vgram3_40$d), polyfit3_40_20$fit[order(vgram3_40$d)], col=2, lwd=4)
[/shell]

El plot que obtenemos es el siguiente, o similar si es que hemos preferido cambiar algún parámetro:
<h5><strong><strong>Figura 4: Panel donde mostramos los resultados</strong></strong></h5>
<strong><strong><a href="http://www.gisandchips.org/wp-content/variograma_3parcelas_polyfit.jpg"><img class="aligncenter size-medium wp-image-1344" src="http://www.gisandchips.org/wp-content/variograma_3parcelas_polyfit-299x299.jpg" alt="" width="327" height="327" /></a>
</strong></strong>

En la imagen vemos las parcelas, la matriz de su variograma y el variograma, con una curva que ajustamos mucho al los datos. Es fácil interpretar que en la estructura de las parcelas se llega a apreciar cierta "ciclicidad", ya que encontramos mesetas a distintas distancias. Cada meseta se  corresponde aproximadamente a las hileras de árboles. El alcance es el que hemos definido (40, en este caso 40 píxeles; puede que unos 20 m. en la realidad).

Viendo estas imágenes nos podemos dar cuenta de que es más fácil determinar una regla de clasificación que cuando lo hacíamos en el caso del RAPID. En este caso, nos fijamos en el número de máximos relativos de la función polinómica de ajuste del variograma. A simple vista, la primera parcela tiene unos 4, la segunda apenas 1 y la última 3 o 4. <strong>En este caso, podríamos citar una nueva regla para nuestro programa según la que a partir de 2 o 3 máximos relativos una parcela puede ser considerada una plantación arbórea.</strong>

Como nos interesa automatizar la tarea creamos una función que nos cuente los máximos relativos. Aquí viene el ejemplo aplicado a la primera parcela:

[shell]

# Encontrar máximos relativos en la función 1

#--------------------------------------------------

maxRelativos1=0

hMaxRelativos1=0

for(i in 1:(length(polyfit1_40_20$fitted.values)-1))

{

if (polyfit1_40_20$fitted.values[i] &gt; polyfit1_40_20$fitted.values[i+1] &amp;&amp; polyfit1_40_20$fitted.values[i] &gt; polyfit1_40_20$fitted.values[i-1])

{maxRelativos1[i]&lt;- polyfit1_40_20$fitted.values[i]

hMaxRelativos1[i]&lt;- vgram1_40$d[i]}

}

MaxRelativos1&lt;-data.frame(hMaxRelativos1,maxRelativos1)

MaxRelativos1&lt;- na.omit(MaxRelativos1)

NumMaxRelat1_40&lt;-length(rownames(MaxRelativos1))

[/shell]

Consultamos los vértices de la curva que hemos dibujado mediante un bucle, de modo que si el vértice <strong>i</strong> tiene un valor superior al vértice anterior y también al vértice posterior entonces es considerado un máximo relativo y queda almacenado en la lista.
<h3>Algunos comentarios sobre todo esto:</h3>
(1) El semivariograma implica un gran esfuerzo de cálculo por parte del ordenador. Esto hace que aún habiendo varias librerías en R que obtienen el semivariograma de una nube de puntos, sean significativamente más lentas que <strong>Fields. </strong>Además, <strong>Fields</strong> hace directamente lo que necesitábamos. No obstante, en un futuro no muy lejano, intentaré desarrollar una librería en C# para reproducir un análisis de este tipo y otras cosillas relacionadas.

(2) Por cuestiones de tiempo no he tratado de crear funciones para las distintas etapas de la demostración. Puede ser un buen ejercicio tratar de implementar todo esto en funciones que permitan entre sus parámetros especificar la banda con la que trabajar, el alcance del semivariograma…

(3) Existen más posibilidades a la hora de establecer reglas, por ejemplo podríamos explorar la distancia que separa los máximos relativos para distinguir los cultivos más intensivos de los más tradicionales. Se me ocurren muchas más posibilidades.

(4) Por último, aunque no le doy mucha importancia, deciros que el número de máximos relativos no es del todo correcto pues el cero siempre aparece en el conteo (sale 5, 2, 5 y no 4, 1, 4 como he dicho más arriba), esto es porqué no he trabajado bastante el bucle, pero es fácil restar 1 a la lista final.  Mi idea principal es la de explorar el concepto y dar unos ejemplos de código a modo de ideas.
<h3>Referencias:</h3>
Para entender bien todo esto creo que es interesante ver mi artículo anterior...

Solamente os remito al siguiente tutorial de Surfer. Es bastante didáctico. <a href="http://www.goldensoftware.com/variogramTutorial.pdf">http://www.goldensoftware.com/variogramTutorial.pdf</a>

De todos modos la red está llena de materiales, apuntes de clase, libros en PDF… No tendréis problemas en encontrar información sobre los variogramas.

-------------------------------------------------------------------

Si quereis contactar podeis enviarme un email (asunto: gisandchips):

Benito M. Zaragozí

benito.zaragozi@ua.es