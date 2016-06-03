---
ID: 66
post_title: 'Integración de R en aplicaciones de escritorio. R, Rcom y C#'
author: benizar
post_date: 2009-09-21 14:06:06
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2009/09/21/integracion-de-r-en-aplicaciones-de-escritorio-r-rcom-y-c/
published: true
syntaxhighlighter_encoded:
  - "1"
---
Hola a todos. Con este artículo espero escribir el primero de una larga serie de artículos sobre el mundo de R. A nuestro juicio, y el de mucha más gente éste es uno de los proyectos GNU con mayor repercusión y también uno de los más prometedores para los usuarios de información geográfica (sigue siendo prometedor, aunque ya es toda una realidad).

Para aquellos que no conozcan R ( <a href="http://cran.r-project.org/">http://cran.r-project.org/</a> ), podemos decir que es un lenguaje de programación a la vez que un paquete estadístico de software, desarrollado bajo licencia GNU GPL de la Free Software Foundation. Lo más interesante de R es que se puede adaptar fácilmente a nuestros intereses mediante la instalación de nuevos paquetes. Hay muchos, y suelen englobar funciones comunes a una misma área de conocimiento. En nuestro caso, existen paquetes relacionados con el tratamiento de la información espacial y más directamente, otros que se relacionan con software GIS. Por poner algunos ejemplos, podemos interactuar desde el GUI de R con SAGA GIS, Grass6, GDAL... y utilizar muchas de sus herramientas, o bien trabajar con otros paquetes propios de R.

En este artículo no trabajaremos desde el GUI de R. Queremos mostrar como integrar R en nuestras aplicaciones de escritorio de .NET mediante objetos COM.

<!--more-->

Para poder comenzar, deberemos instalar un servidor COM disponible en <a href="http://cran.r-project.org/">http://cran.r-project.org/</a>. Entramos en la sección de software/otros/ e instalamos  <a href="http://cran.r-project.org/contrib/extra/dcom/">R-(D)COM Interface</a>. Por supuesto, también necesitamos tener instalado R 2.8.1. Por otra parte puede ser conveniente instalar un paquete de R llamado rcom con todas sus dependencias.

A partir de aquí, <span style="text-decoration: underline">este artículo está dividido en tres partes</span> por motivos didácticos. Primero se habla de cómo se trabaja con Rcom, después describimos cómo se usa el visor de histogramas (que es el ejemplo propuesto), y por último entramos en algunos detalles del código.

Para descargar el código fuente de R2csharp  podéis hacer un <em>checkout </em>del siguiente repositorio Subversion de GIS&amp;Chips:
<p style="padding-left: 30px"><code>svn co http://www.gisandchips.org/svn/r2csharp</code></p>
<p style="padding-left: 30px"></p>

<h3>Uso básico de Rcom</h3>
Una vez instalado el servidor COM agregamos todas las referencias a un proyecto WindowsForm  en nuestro IDE de C# y sus correspondientes using (aquí hemos trabajado con Visual C# Express Edition).
<p class="MsoNormal" style="text-align: justify;line-height: 150%"></p>

[csharp]
using STATCONNECTORCLNTLib;
using StatConnectorCommonLib;
using STATCONNECTORSRVLib;
[/csharp]

Con lo que podemos crear nuestro primer objeto de conexión a R, e iniciar una sesión

[csharp]

//Conexión a R
StatConnector sc1 = new STATCONNECTORSRVLib.StatConnectorClass();
sc1.Init(&quot;R&quot;);

[/csharp]

A partir de aquí trabajaremos con tres métodos para interactuar con la sesión de R. Estos son:
<ul>
	<li>sc1.EvaluateNoReturn("código de R");</li>
</ul>
<ul>
	<li>sc1.Evaluate("código de R");</li>
</ul>
<ul>
	<li>sc1.GetSymbol("nombre de un objeto de R");</li>
</ul>
He observado que en ocasiones es indistinto el uso de las dos primeras, pero también podemos recibir una excepción en tiempo de ejecución, por lo que utilizaremos la primera para ejecutar código de R que no tiene porqué devolver ningún resultado (por ejemplo setwd('C:/') , asigna el directorio de trabajo de R), mientras que el resultado de sc1.evaluate() habitualmente querremos almacenarlo en un objeto para poder consultarlo (no tiene sentido hacer un getwd() para después no mostrar su valor, sc1.Evaluate("wd&lt;-getwd()"), con esto almacenamos el wd en un objeto, el cual podemos consultar desde nuestra aplicación).

Por último, sc1.GetSymbol("wd") nos sirve para consultar un objeto creado en nuestra sesión de R y manipularlo. Por ejemplo, podríamos querer mostrar en una etiqueta lblWD la ruta del directorio de trabajo.

[csharp]

string wd = (string)sc1.GetSymbol(&quot;wd&quot;);
lblWD.Text = wd;

[/csharp]

Hasta aquí ya hemos visto todo lo necesario para interactuar con R. Pero la complejidad puede ser mayor debido a las diferencias entre tipos de datos de R y de .NET. Mientras que en R hablamos entre otros de vectores, listas, matrices y dataframes (de números, caracteres...), en .NET tendremos arrays de distintos tipos o tipos simples. Deberemos consultar el tipo antes de almacenar los datos en estructuras de .NET. Esto lo podemos hacer mediante:

[csharp]

txtType.Text = sc1.GetSymbol(&quot;wd&quot;).GetType().ToString();

[/csharp]
<p class="MsoNormal" style="text-align: justify;line-height: 150%"><span style="font-size: 11pt;line-height: 150%;font-family: Arial">donde averiguamos que la función getwd() devuelve un string de .NET. Utilizaremos GetType() muy amenudo. Solamente hay que pensar que estructuras como un dataframe tendrán una estructura en .NET de array de arrays de... por lo que deberemos tener muy en cuenta los tipos de cada vector miembro del dataframe (podemos entender un dataframe como un conjunto de vectores relacionados; cada vector puede ser un campo de la "tabla" que es un dataframe).</span></p>

<h3>R2csharp_histoviewer</h3>
Nuestra aplicación es algo más que un "!Hola mundo ¡ ". Tiene ejemplos básicos de distintas tareas habituales en R, pero en este caso realizadas desde nuestra aplicación. En esta demostración creamos un visor de histogramas con Windows Forms, que organiza la información de R de un modo claro y permite crear, mostrar, editar y eliminar dataframes de R, ejecutar una sencilla función de R (<em>"hist</em>( )<em>"</em>), y también crear y eliminar ficheros en el sistema operativo (imágenes, directorios...).

Podemos simular una sesión de trabajo con este visor siguiendo estos pasos (también vemos la correspondencia de las acciones en comandos de R):

1. Seleccionar cargar objeto para añadir uno a uno los tres ficheros de ejemplo  ( bichos.txt, spatstat.txt y usos.txt)
<p align="center"><strong>read.table()</strong></p>
<p align="center"><strong> </strong></p>
2. Seleccionar el objeto bichos y borrarlo <strong> </strong>
<p align="center"><strong>rm(bichos)</strong></p>
3. Seleccionar un campo en el segundo listbox y crear su histograma      presionando el botón Hist.
<p align="center"><strong>hist(objeto$campo)</strong> / nos ahorra hacer un <strong>attatch()</strong> en R</p>
4. Editar una celda del campo "incendio" del objeto usos. Cambiar algún 0 por    100, y creamos de nuevo el histograma. Vemos como ha cambiado el objeto de R sin necesidad de conocer la posición del dato dentro del dataframe. (para   saber que estamos editando en el row del datagridview deberá verse un lapiz sobre el "row head")

[caption id="attachment_76" align="aligncenter" width="272" caption="Vista de la aplicación en funcionamiento"]<a href="http://www.gisandchips.org/wp-content/rcom_histoviewer.jpg"><img class="size-full wp-image-76" src="http://www.gisandchips.org/wp-content/rcom_histoviewer.jpg" alt="Vista de la aplicación en funcionamiento" width="272" height="271" /></a>[/caption]

Evidentemente, sin que el usuario lo sepa se están realizando varios list(), se aplican filtros, se obtienen los rownames() y colnames(), y otras operaciones habituales en R que aquí no se necesitará teclear y cuyos resultados estarán siempre visibles. Lo cual hace que nuestro interfaz sea más amigable solo por esto.
<h3>Algunas notas sobre el código:</h3>
Entre las tareas más interesantes en el código podemos ver un ejemplo de lectura y escritura de un dataframe en un datagridview, teniendo en cuenta los dos tipos de datos devueltos (enteros y dobles). El fichero de ejemplo spatstat.txt intercala columnas de datos dobles y enteros, claro que también podría haber String [] u otros, pero confiamos en que estas mejoras las puedan realizar nuestros lectores.

En el flujo de trabajo habitual hay dos tipos de objetos, los que contienen nuestros datos y los que utilizamos para interactuar con R. Estos últimos los ocultamos al usuario mediante condiciones "if".

A la hora de crear imágenes temporales podemos hacer que se muestren en el device de R si no especificamos ningún dispositivo de salida para hist(). Aunque me ha parecido más interesante integrar la gráfica en el formulario. Para esto hay que decir que existe un problema al usar el portapapeles de Windows para almacenar un metafile, que es el único formato que R nos permite. Se trata de una incompatibilidad del SO que no nos permitirá recuperar la imagen del clipboard para mostrarla en el picbox (podemos consultar algo acerca de esta cuestión en <a href="http://support.microsoft.com/?scid=kb%3Ben-us%3B323530&amp;x=11&amp;y=12">http://support.microsoft.com/?scid=kb%3Ben-us%3B323530&amp;x=11&amp;y=12</a> ) Por este motivo, recurrimos a crear nuestro propio directorio temporal, el cual eliminamos al cerrar el formulario.

Hay que decir que Rcom aporta muchas posibilidades más directas para mostrar o trabajar con los datos de R. No obstante, considero que ya es mucho trabajo conocer un lenguaje de .NET y tener conocimientos de R, como para también especializarse en todas las opciones de Rcom. Para empezar está muy bien, pero en caso de trabajar a menudo con estas herramientas debería explorarse los ejemplos de Rcom, por si resultan más funcionales.

Una consideración que puedo transmitir a los lectores es que no existe ningún "binding" para C# /.NET que nos evite tener que embeber código de R en nuestra aplicación, así como existe rJava (<a href="http://cran.r-project.org/web/packages/rJava/index.html">http://cran.r-project.org/web/packages/rJava/index.html</a> ). En caso de disponer de tal posibilidad la hubiésemos utilizado por múltiples motivos, sobretodo por el uso de un solo lenguaje de programación, lo que facilita la comprensión y sencillez del código y también la depuración de la aplicación.

No es necesario advertir que un buen conocimiento de R y las funciones que se quiera aplicar es fundamental para diseñar el mejor interfaz de usuario posible. Pero en determinados casos donde se debe efectuar análisis exploratorios de los datos el esfuerzo se verá compensado por la comodidad y la rapidez en el trabajo. Incluso pensándolo bien, en equipos de trabajo donde haya investigadores o técnicos que no dominen R la creación de un interfaz de este tipo puede ser más que interesante.
<h3>Algunas referencias:</h3>
En <a href="http://www.codeproject.com/">www.CodeProject.com</a> existe un ejemplo de aplicación para C# (<a href="http://www.codeproject.com/KB/cs/RtoCSharp.aspx">http://www.codeproject.com/KB/cs/RtoCSharp.aspx</a> ) donde se repasan las posibilidades básicas para interactuar con R. También hay varios ejemplos de código disponibles para VB6 en la carpeta donde se nos instala el COM Server (por defecto "C:\Archivos de programa\R\(D)COM Server\samples"). Por último, la lista de distribución de Rcom donde se tratan muchas cuestiones relacionadas está en <a href="http://mailman.csd.univie.ac.at/pipermail/rcom-l/">http://mailman.csd.univie.ac.at/pipermail/rcom-l/</a> .
<h3>Datos de ejemplo:</h3>
<a href="http://www.gisandchips.org/wp-content/usos.txt">Descargar usos.txt</a>

<a href="http://www.gisandchips.org/wp-content/spatstat.txt">Descargar spatstat.txt</a>

<a href="http://www.gisandchips.org/wp-content/bichos.txt">Descargar bichos.txt</a>

-------------------------------------------------------------------

Si quereis contactar podeis enviarme un email (asunto: gisandchips):

Benito M. Zaragozí

benito.zaragozi@ua.es