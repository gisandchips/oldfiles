---
ID: 2066
post_title: 'Geomondrian: un servidor OLAP con soporte espacial'
author: Jorge Piera Llodrá
post_date: 2012-03-23 19:51:02
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2012/03/23/geomondrian-un-servidor-olap-con-soporte-espacial/
published: true
---
OLAP es el acrónimo en inglés de <em>procesamiento analítico en línea</em> (<em>On-Line Analytical Processing</em>). Es una solución utilizada en el campo de la llamada Inteligencia Artificial (o <em>Business Intelligence</em>) cuyo objetivo es agilizar la consulta de grandes cantidades de datos (<a href="http://es.wikipedia.org/wiki/OLAP"><em>Wikipedia</em></a>). El objetivo es generar tablas, gráficas u otro tipo de documentos que puedan ser usados para facilitar la toma de decisiones.

<a href="http://www.spatialytics.org/projects/geomondrian/"><em></em>Geomondrian</a> es un servidor OLAP con soporte espacial, lo que permite utilizar la componente espacial a la hora de realizar consultas en el almacén.

Este artículo describe los pasos necesarios que hay que realizar para instalar y probar un servidor <em>Geomondrian</em>  en un computador. El objetivo no es tener un sistema en producción, sino que se pueda instalar fácilmente el sistema y se pueda "jugar" un poco para ver la potencia que puede llegar a tener.

<!--more-->
<h2>Instalación</h2>
Partimos de una máquina con una Ubuntu 11.10 de 64 bits. Instalamos el <em>Tomcat 6</em> en <em>Ubuntu</em>. Para ello ejecutamos el siguiente comando:

[bash]
sudo apt-get install tomcat6 tomcat6-admin
[/bash]

Editamos el fichero <em>/etc/tomcat6/tomcat-users.xml</em> y añadimos el usuario <em>admin</em> con contraseña <em>admin</em>:

[xml]
 &lt;role rolename=&quot;manager&quot;/&gt;
 &lt;role rolename=&quot;admin&quot;/&gt;
 &lt;user username=&quot;admin&quot; password=&quot;admin&quot; roles=&quot;manager,admin&quot;/&gt;
[/xml]

Reiniciamos el tomcat:

[bash]
sudo /etc/init.d/tomcat6 restart
[/bash]

En este punto ya tenemos una instancia de <em>Tomcat</em> corriendo en nuestra máquina junto con la aplicación de administración que se va a utilizar para instalar el <em>Geomondrian</em>.

Descargamos el <em>Geomondrian 1.0</em> desde su <a href="http://sourceforge.net/projects/geomondrian/files/geomondrian-1.x/1.0/geomondrian.war/download">página web</a>. Se trata de un archivo con extensión war, que puede ser fácilmente desplegado en <em>Tomcat</em>.

Abrimos un navegador y vamos a la página de administración de <em>Tomcat</em> (http://localhost:8080/manager/html). La primera vez que accedemos tenemos que escribir el nombre de usuario y la contraseña que hemos creado previamente.

Buscamos la opción <em>WAR file to deploy</em> y seleccionamos el archivo <em>geomondrian.war</em> que nos hemos descargado previamente. A continuación hacemos click en <em>deploy</em>.
<p style="text-align: center"><img class="aligncenter size-full wp-image-2074" src="http://www.gisandchips.org/wp-content//geomondrian-war2.png" alt="" width="434" height="314" /></p>
Si todo ha ido bien aparecerá una nueva entrada en la lista de aplicaciones que tiene el Tomcat instaladas:
<p style="text-align: center"><img class="aligncenter size-full wp-image-2075" src="http://www.gisandchips.org/wp-content//installed.png" alt="" width="434" height="314" /></p>
Hacemos click en el enlace o bien escribimos la url http://localhost:8080/geomondrian/ y ya podemos acceder a la ventana principal del <em>Geomondrian</em>.

Por defecto la aplicación incluye una base de datos embebida que contiene algunos ejemplos sin información espacial. En la web de <em>Geomondrian</em> hay un script de creación de una base de datos con información espacial pero se trata de una base de datos <em>Postgis</em>, por lo que tenemos que instalar previamente <em>Postgres</em> y <em>Postgis</em>.

Para ello ejecutamos desde una consola el siguiente comando:

[bash]
 sudo apt-get -install postgresql-9.1 postgresql-contrib-9.1 postgis postgresql-9.1-postgis
 [/bash]

Cambiamos el password del usuario <em>postgres</em> a <em>postgres</em>.

[bash]
 sudo -u postgres psql
 postgres=# ALTER USER postgres WITH ENCRYPTED password 'postgres';
 [/bash]

A continuación vamos a crear una base de datos plantilla y vamos a instalar <em>Postgis</em> en ella.

[bash]
sudo su postgres
createdb postgistemplate
psql -d postgistemplate -f /usr/share/postgresql/9.1/contrib/postgis-1.5/postgis.sql
psql -d postgistemplate -f /usr/share/postgresql/9.1/contrib/postgis-1.5/spatial_ref_sys.sql
[/bash]

Una vez que ya tenemos la base de datos plantilla con todas las funciones de <em>Postgis</em> instaladas vamos a crear una base de datos a partir de esta plantilla que será usada por el <em>Geomondrian</em>.

[bash]
 createdb -T postgistemplate simple_geofoodmart
 [/bash]

Ahora vamos a instalar el script creación de la base de datos OLAP que hemos tenido que <a href="http://sourceforge.net/projects/geomondrian/files/geomondrian-1.x/1.0/simple_geofoodmart_sql_postgres.zip/downloadhttp://">descargar</a> previamente desde la web de <em>Geomondrian</em>. Una vez descargado, vamos al directorio donde lo hemos descargado y ejecutamos lo siguiente:

[bash]
 psql -h localhost -U postgres -W -d simple_geofoodmart -f simple_geofoodmart.sql
 [/bash]

A continuación tenemos que editar la cadena de conexión para que el <em>Geomondrian</em> pueda conectar con la base de datos espacial. Para ello abrimos el fichero <em>/var/lib/tomcat6/webapps/geomondrian/WEB-INF/web.xml</em>, buscamos el servlet <em>GeoMDXQueryServlet</em> y editamos la propiedad <em>conectString</em>:

[xml]
 &lt;servlet&gt;
 &lt;servlet-name&gt;GeoMDXQueryServlet&lt;/servlet-name&gt;
 &lt;servlet-class&gt;
 mondrian.web.servlet.MdxQueryServlet
 &lt;/servlet-class&gt;
 &lt;init-param&gt;
 &lt;param-name&gt;connectString&lt;/param-name&gt;
 &lt;param-value&gt;Provider=mondrian;Jdbc=jdbc:postgresql_postGIS://localhost/simple_geofoodmart?user=postgres&amp;amp;password=postgres;Catalog=/WEB-INF/queries/simple_geofoodmart.xml;JdbcDrivers=org.postgis.DriverWrapper;&lt;/param-value&gt;
 &lt;/init-param&gt;
 &lt;/servlet&gt;
 [/xml]

Llegados a este punto ya tenemos un servidor SOLAP en funcionamiento que está conectado con una base de datos con soporte espacial.
<h2>Estructura de la base de datos</h2>
<p style="text-align: left">Antes de empezar a probar la base de datos, hay que entenderla. Existe poca información en la web sobre la estructura de la misma, pero se puede consultar el xml que utiliza el <em>Geomondrian</em> para explotarla que se encuentra en el fichero <em>/var/lib/tomcat6/webapps/geomondrian/WEB-INF/queries/simple_geofoodmart.xml</em>.</p>
Se trata de una base de datos de ventas de supermercados que tiene un único cubo Ventas (Sales) para el que se han definido 5 dimensiones: Tienda (Store), Producto (Product), Promociones (Promotions), Clientes (Customers) y Tiempo (Time). El objetivo es estudiar las ventas en función de estas dimensiones:
<p style="text-align: center"><img class="aligncenter size-full wp-image-2078" src="http://www.gisandchips.org/wp-content//cubos.png" alt="" width="504" height="335" /></p>
El cubo ventas, tiene 3 medidas que son Ventas (Store Sales), Coste (Store Cost) y Unidades (Unit Sales) que son las variables que van a ser estudiadas.

La dimensión Tienda establece una jerarquía espacial de objetos (Tienda -&gt; Ciudad -&gt;Estado -&gt; País) que tiene un atributo de tipo geométrico. Este atributo representa el polígono que describe el contorno del objeto y será estudiado en un apartado posterior.
<h2>Consulta de la base de datos</h2>
Desde la ventana principal del <em>Geomondrian</em>, hay que seleccionar la opción <em>JPivot pivot table with simple_geofoodmart spatial cube</em> para acceder a la consola de administración de  <em>Geomondrian</em> que contiene la base de datos espacial.

Lo primero que se muestra es una ventana en la que se han seleccionado dos dimensiones (Store y Product) y en la que se muestran las tres medidas que hay en el cubo.
<p style="text-align: center"><img class="aligncenter size-full wp-image-2079" src="http://www.gisandchips.org/wp-content//main.png" alt="" width="434" height="341" /></p>
Seleccionando el primer botón de la derecha se abre el navegador de OLAP que sirve para configurar el cubo.

<img class="aligncenter size-full wp-image-2080" src="http://www.gisandchips.org/wp-content//dimension_selector.png" alt="" width="160" height="209" />

En esta ventana se configuran las columnas y las filas que se quieren mostrar y los filtros que hay que aplicar en alguna de las dimensiones. Los iconos que hay delante de las dimensiones las mueven de un lugar a otro. Por ejemplo, si queremos realizar un análisis por cliente y queremos que aparezca como una columna en la tabla, simplemente seleccionaremos el icono de columna que aparece delante de <em>Customers</em>.

En la figura se puede observar como en <em>Time</em> aparece un  <em>Year=1997</em>, lo que indica que se están filtrando registros que no sean del año 1997. Se puede pulsar en en nombre de la dimensión y se abrirá una nueva ventana donde se pueden cambiar las condiciones del filtro.

Sobre la tabla que muestra los resultados se pueden realizar operaciones de drill down y drill up de forma que cada vez vemos más o menos detalle. La siguiente imagen se ha generado haciendo operaciones de drill down hasta que se han visualizado las ventas de bebida de la tienda 15 de Seattle.
<p style="text-align: center"><img class="aligncenter size-full wp-image-2081" src="http://www.gisandchips.org/wp-content//drill.png" alt="" width="407" height="383" /></p>
En la imagen se puede ver como no hay datos de ejemplo para las tiendas de México y Canadá.
<h2>El lenguaje MDX</h2>
Otra opción interesante es la de MDX, que se puede configurar seleccionado el segundo botón de la derecha de la barra de menú. Las expresiones multidimensionales (MDX es el acrónimo de MultiDimensional eXpressions) es un lenguaje de consulta para bases de datos multidimensionales sobre cubos OLAP, se utiliza en Business Intelligence para generar reportes para la toma de decisiones basados en datos históricos, con la posibilidad de cambiar la estructura, o permitiendo rotar el cubo (<a href="http://es.wikipedia.org/wiki/Expresiones_multidimensionaleshttp://">wikipedia</a>).

Conociendo la sintaxis de MDX, se pueden crear cubos a medida sin necesidad de utilizar el interfaz de usuario. Además, existe una equivalencia entre la consulta que hay en la ventana de MDX y el interfaz gráfico de forma que si se modifica uno, el otro se actualiza inmediatamente.

La siguiente imagen muestra un ejemplo de consulta MDX desde el interfaz web de la aplicación. El interfaz permite la edición manual de la consulta.
<p style="text-align: center"><img class="aligncenter size-full wp-image-2082" src="http://www.gisandchips.org/wp-content//mdx.png" alt="" width="556" height="397" /></p>

<h2>Gráficas</h2>
También se pueden añadir gráficas para una consulta determinada. Las gráficas son interactivas, de forma que a medida que el usuario navega haciendo operaciones de drill up y del drill down, la gráfica se refresca con las dimensiones seleccionadas.

La siguiente imagen muestra un ejemplo de ello. Se han ido haciendo operaciones de drill down y en la gráfica han ido apareciendo nuevas columnas dinámicamente.
<p style="text-align: center"><img class="aligncenter size-full wp-image-2084" src="http://www.gisandchips.org/wp-content//graphic.png" alt="" width="349" height="410" /></p>
Existen varios tipos de gráficas que el usuario puede configurar, como la gráfica de barras, la de tartas, etc.
<h2>El soporte espacial</h2>
Todo lo que se ha comentado hasta este apartado es tan válido para el <em>Geomondrian</em> como para el <em>Mondrian</em> ya que, hasta este momento, no se ha hecho ninguna mención al soporte espacial en los datos.

En el apartado anterior se comentó la importancia del lenguaje MDX y su mapeo con el interfaz gráfico de la aplicación. En realidad, la diferencia que aporta <em>Geomondrian</em> frente al <em>Mondrian</em> es que extiende el lenguaje MDX para que se puedan utilizar operadores espaciales. A este nuevo lenguaje lo llama GeoMDX.

Ya hemos visto que existen algunas tablas de la base de datos que tienen un atributo de tipo geometría. Ese tipo de datos no existe en <em>Postgres</em>, y es la extensión espacial <em>Postgis</em> la que lo añade. La siguiente imagen muestra una consulta sobre la tabla de ciudades realizada con el <a href="http://www.pgadmin.org/">pgAdmin</a>.
<p style="text-align: center"><img class="aligncenter size-full wp-image-2085" src="http://www.gisandchips.org/wp-content//geometry.png" alt="" width="555" height="355" /></p>
Se puede observar como existe una columna <em>geometry</em> que tiene un valor binario que no representa nada. Si se intenta utilizar un atributo de este tipo en un <em>Mondrian</em> lo que puede ocurrir es que o bien no procese el campo al no reconocer su tipo o bien lo muestre con su valor en binario que, para un observador humano, no dice nada.

A continuación vamos a intentar visualizar el mismo atributo desde el <em>Geomondrian</em>. Para ello vamos a seleccionar que nos muestre los atributos de las tablas de dimensiones y vamos a hacer drill down hasta llegar al nivel de ciudad. La siguiente figura muestra el resultado de San Diego y San Francisco.
<p style="text-align: center"><img class="aligncenter size-full wp-image-2087" src="http://www.gisandchips.org/wp-content//geometries.png" alt="" width="560" height="338" /></p>
Puede observarse que no aparece un valor binario en la columna de la geometría, sino que aparece el valor de un multipolígono, que es una de las primitivas de dibujo que soporta <em>Postgis</em>. Además se puede ver cómo se pueden leer fácilmente las coordenadas de los puntos que forman el polígono que se utiliza para describir a la ciudad.

Este formato de mostrar las geometrías se conoce como <a href="http://en.wikipedia.org/wiki/Well-known_text">WKT</a>  y es un estándar utilizado por muchas aplicaciones GIS. Con una geometría en WKT se podría abrir una aplicación GIS y visualizar de forma gráfica las geometrías obtenidas con el <em>Geomeondrian</em>.

En cuando a visualización ya hemos visto como <em>Geomeondrian</em> es capaz de convertir atributos geométricos en representaciones textuales legibles por un ser humano. Pero la extensión que se hace sobre el MDX permite además poder utilizar operadores espaciales en la consulta de la base de datos.

La figura siguiente muestra un ejemplo de consulta para mostrar las ventas en las tiendas de estados unidos y su área de venta en kilómetros cuadrados. Para ello se utilizan algunas funciones espaciales sobre un campo de tipo geometría.
<p style="text-align: center"><img class="aligncenter size-full wp-image-2088" src="http://www.gisandchips.org/wp-content//spatial.png" alt="" width="536" height="353" /></p>
La función ST_textunderscore Area devuelve el área de un polígono. La función ST_textunderscore Transform hace una conversión de sistema de coordenadas desde el sistema de coordenadas origen a un sistema de coordenadas destino que es el que necesita la función del área para operar. La función ST_textunderscore UnionAgg realiza la unión de varios campos de tipo geometría en una única geometría.
<h2>Conclusiones</h2>
El <em>Geomondrian</em> es un servidor SOLAP que tiene un proceso de  instalación muy sencillo (simplemente hay que desplegar un war en el <em>Tomcat</em>).

La aplicación extiende el MDX añadiendo soporte de operaciones espaciales que abren la posibilidad de mostrar cualquier dato que se pueda calcular mediante primitivas geométricas. Los límites son los que la semántica del GeoMDX ofrezca.

En la implementación actual se echa de menos un visor espacial para poder visualizar en un mapa las geometrías mostradas en WKT. Aunque esta tarea es aparentemente sencilla y se podría añadir con poco esfuerzo utilizando aplicaciones web de GIS existentes que ya son capaces de representar geometrías en WKT.