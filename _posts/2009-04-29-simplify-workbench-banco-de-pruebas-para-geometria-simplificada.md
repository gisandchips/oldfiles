---
ID: 22
post_title: 'Simplify Workbench: banco de pruebas para geometría simplificada'
author: josetomas
post_date: 2009-04-29 11:40:18
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2009/04/29/simplify-workbench-banco-de-pruebas-para-geometria-simplificada/
published: true
syntaxhighlighter_encoded:
  - "1"
---
<p style="text-align: justify">Optimizar nuestro cliente GIS, tanto para web como para escritorio, implica entre otras cosas afinar al máximo los rangos de escala de visualización para cada una de nuestras capas. Sin embargo puede darse el caso de que este mecanismo no sea aplicable. Ante esta situación, es importante que no renunciemos al ahorro en los tiempos de proceso, y en este sentido puede ser útil recurrir a la simplificación o generalización de la geometría.</p>
<p style="text-align: justify">Lo que aquí os ofrecemos es una herramienta que, dada una determinada entidad PostgreSQL/PostGIS, permite explorar rápidamente cuál es la relación entre tamaño de imagen, extensión del mapa, degradación visual y tiempo de proceso <!--more-->para un máximo de 4 valores de tolerancia designados por el usuario. Se trata de una aplicación sencilla, escrita en C#, en la que el área de trabajo se divide en cuatro paneles. En cada panel podemos asignar una tolerancia, visualizar la geometría simplificada resultante, echar un vistazo a la sentencia SQL ejecutada en PostgreSQL y comprobar el tiempo de proceso transcurrido en realizar la transacción en el servidor y generar la imagen resultante en el cliente. Además, en todo momento conocemos cuál es la dimensión en píxeles de la imagen y el ancho del mapa. La instantánea inferior muestra una sesión de Simplify Workbench en la que se compara la geometría simplificada de los municipios de España procedentes de la Base de Datos de Líneas Límite a escala 1:1.000.000 del <a title="IDEE" href="http://www.idee.es" target="_blank">IDEE</a>. Se puede observar cómo el tiempo de proceso del panel C es casi 4 veces menor que el del panel A, mientras que el impacto en la visualización es casi inapreciable. Por el contrario, la reducción del tiempo de proceso en el panel D conlleva una degradación visual que, para la resolución de la imagen, no es aceptable.</p>


[caption id="attachment_42" align="aligncenter" width="300" caption="Geometría simplificada de los municipios de España en Simplify Workbench. Fuente de datos: IDEE"]<a href="http://www.gisandchips.org/wp-content/simplifyworkbench3.png"><img class="size-medium wp-image-42" src="http://www.gisandchips.org/wp-content/simplifyworkbench3-300x290.png" alt="Geometría simplificada de los municipios de España en Simplify Workbench. Fuente de datos: IDEE" width="300" height="290" /></a>[/caption]
<p style="text-align: center"></p>
<p style="text-align: justify">Podéis disponer del código fuente de Simplify Workbench bajo licencia GPL haciendo un <em>checkout </em>del siguiente repositorio Subversion de GIS&amp;Chips:</p>
<p style="text-align: justify;padding-left: 30px"><code>svn co http://www.gisandchips.org/svn/simplifyworkbench</code></p>
<p style="text-align: justify">Para los que queráis probarlo de inmediato, tened en cuenta que los datos de conexión a PostgreSQL y entidad PostGIS se configuran en el fichero XML <strong>app.config</strong> que acompaña al ejecutable. Por defecto, os proporcionamos una conexión al servidor PostgreSQL de GIS &amp; Chips para que podáis hacer tests con los municipios del IDEE. Si vais a realizar pruebas con vuestros propios datos tened en cuenta que la función de simplificación de PostGIS (<em>simplify</em> o <em>st_simplify</em>) no opera sobre geometrías en coordenadas geográficas (e.g. SRID = 4326), por lo que la aplicación no realizará el tratamiento esperado. Tampoco opera sobre entidades con sistema de referencia espacial indeterminado en la tabla <em>geometry_columns </em>(i.e. SRID = -1).</p>
<p style="text-align: justify">Si eres desarrollador y te vas a 'zambullir' en el código seguramente te interesará conocer que Simplify Workbench no es más que una extensión de <a title="SharpMap" href="http://www.codeplex.com/SharpMap" target="_blank">SharpMap</a> con un interfaz gráfico en Windows.Forms. Formalmente es una solución desarrollada en el IDE de código abierto <a title="SharpDevelop" href="http://www.icsharpcode.net/OpenSource/SD/" target="_blank">SharpDevelop</a>, escrita en C# sobre el framework 2.0 de .NET. Integra tres proyectos:  <strong>SharpMap </strong>(versión 0.9), <strong>SharpMapSimplifyExtension</strong> y <strong>Simplify</strong>.</p>
<p style="text-align: justify">Seguramente alguno de vosotros ya conozca <a title="SharpMap" href="http://www.codeplex.com/SharpMap" target="_blank">SharpMap</a>. Desde que <a title="Morten Nielsen" href="http://www.sharpgis.net/" target="_blank">Morten Nielsen</a>, su creador, publicó allá por 2005-2006  este versátil renderizador de cartografía basado en GDI+, una pequeña revolución se ha producido en el ámbito de los clientes GIS libres sobre plataforma .NET y de hecho el proyecto no sólo continúa muy activo sino que ha conseguido integrar un equipo consolidado de desarrolladores en torno a un ecosistema de proyectos interrelacionados. El proyecto <strong>SharpMap</strong> de Simplify Workbench corresponde a los fuentes de la versión 0.9, la versión estable, aunque observaréis que el namespace SharpMap.Data.Providers incluye el proveedor de PostGIS (<a title="PostGIS.cs" href="http://sharpmap.codeplex.com/Wiki/View.aspx?title=PostGIS" target="_blank">PostGIS.cs</a>) y una referencia a <a title="Npgsql" href="http://npgsql.projects.postgresql.org/" target="_blank">Npgsql</a>, la librería de acceso a datos de PostgreSQL en su versión 2.0.4.</p>
<p style="text-align: justify">El proyecto <strong>SharpMapSimplifyExtension</strong> consta de dos clases: <strong>PostGISSimplify</strong> y <strong>VectorSimplifiedLayer</strong>. La primera es una derivación de la clase SharpMap.Data.Providers.PostGIS que modifica el método <em>GetGeometriesInView</em> de forma que la sentencia SQL subyacente invoque la función <em>simplify</em> de PostGIS con la tolerancia de entrada. También añade un delegado que permite recuperar mediante un evento tanto la sentencia SQL final como el ancho del <em>bounding box</em>. La segunda deriva de la clase SharpMap.Layers.VectorLayer, sobrecarga el constructor para instanciar capas vectoriales especificando una tolerancia de simplificación y modifica el método <em>Render</em>. Con estas dos simples extensiones podemos definir  una capa vectorial que muestre geometría simplificada de PostGIS sin variar sustancialmente la lógica habitual de trabajo con SharpMap, tal y como se muestra en este ejemplo:</p>

[csharp]
//
//
//
string cnstr = &quot;Server=your.pgsql.server;Port=5432;User Id=user;Password=secret;Database=yourDB&quot;;
string tableName = &quot;yourPolygons&quot;;
string geomFieldName = &quot;geometry&quot;;
string oidFieldName = &quot;oid&quot;;
//Instance of data provider for PostGIS simplified geometry
PostGISSimplify src = new PostGISSimplify(cnstr, tableName, geomFieldName, oidFieldName);
//Instance of vector layer to hold simplified PostGIS geometry
//Third parameter is an integer corresponding to tolerance in Douglas-Peucker vertex reduction algorithm
VectorSimplifiedLayer lyr = new VectorSimplifiedLayer(&quot;SIMPLIFIED_LAYER&quot;, src, 1000);
lyr.Style.EnableOutline = true;
lyr.Style.Fill = Brushes.Transparent;
//Map object, the layer container and rendering logic construct in SharpMap
Map m = new Map(new System.Drawing.Size(350, 275));
m.BackColor = Color.White;
m.Layers.Add(lyr);
//Zoom to extents internally invokes the modified GetGeometriesInView method in PostGISSimplify class
m.ZoomToExtents();
//Retrieve map image
System.Drawing.Image img = m.GetMap();
//
[/csharp]
<p style="text-align: justify">El proyecto <strong>Simplify</strong> incluye el formulario estructurado en cuatro paneles comentado más arriba y una clase de conveniencia, <strong>MapRenderer</strong>, que encapsula la lógica de lectura del fichero app.config, definición de layer, obtención de la imagen, recuperación de la sentencia SQL y cálculo del tiempo de proceso. En el directorio de este proyecto encontraréis también el fichero de solución de SharpDevelop (Simplify.sln).</p>
<p style="text-align: justify">Para terminar, y puesto que muchos de nosotros aquí en GIS &amp; Chips somos fans del framework de código abierto Mono, os animamos a que probéis a compilar usando esta plataforma. Sabemos positivamente que SharpMap 0.9, con pequeñas modificaciones, es compilable sobre Mono. Creemos, aunque no lo hemos comprobado, que el código de Simplify Workbench también lo es. Si alguno lo consigue nos alegrará que nos lo comente.</p>
<p style="text-align: justify">¡Disfrutad con Simplify Workbench!</p>