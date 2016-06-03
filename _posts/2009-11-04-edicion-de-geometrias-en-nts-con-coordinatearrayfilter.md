---
ID: 747
post_title: >
  Edición de geometrías en NTS con
  CoordinateArrayFilter
author: josetomas
post_date: 2009-11-04 12:09:09
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2009/11/04/edicion-de-geometrias-en-nts-con-coordinatearrayfilter/
published: true
---
<a title="Java Topology Suite" href="http://www.vividsolutions.com/jts/JTSHome.htm" target="_blank">Java Topology Suite</a> (JTS) es una librería que resuelve problemas complejos y en la que sus arquitectos han recurrido al uso exhaustivo de <a title="patrones de diseño de software" href="http://en.wikipedia.org/wiki/Design_pattern_(computer_science)" target="_blank">patrones de diseño de software</a> como medio de gestionar eficientemente dicha complejidad. Como muestra baste mencionar que los desarrolladores debemos asumir desde el principio el <a title="factory pattern" href="http://en.wikipedia.org/wiki/Factory_pattern" target="_blank"><em>factory pattern</em></a> que implementa la clase <em>GeometryFactory</em> que empleamos en la construcción de objetos geométricos. El hecho de que JTS haga uso de patrones de diseño es una muestra más de su fiabilidad, aunque en ocasiones (sobretodo si hay carencia de documentación) también implica un esfuerzo extra por nuestra parte a la hora de entender cómo se solucionan determinados casos de uso. Uno de esos casos en los que la solución no se evidencia a primera vista es la edición de los vértices de una geometría, así que en este  artículo os muestro código de ejemplo para el porting de JTS a C#, <a title="NetTopologySuite" href="http://code.google.com/p/nettopologysuite/" target="_blank">NetTopologySuite</a> (NTS) en su versión 1.2, que os permitirá introduciros en el manejo del interface <em>ICoordinateFilter </em>para la manipulación geométrica.<!--more-->

Quizá me equivoque, pero creo que no se puede calificar JTS/NTS como un "framework" para el desarrollo de herramientas de  manipulación de geometrías (e.g. aplicaciones tipo CAD). El diseño de este API promueve más bien la idea de inmutabilidad de un objeto geométrico a lo largo de su ciclo de vida (aunque no hay un mecanimo que fuerze dicha inmutabilidad), y se centra en la modelización a bajo nivel de las tareas de cálculo topológico y análisis espacial, y en la exactitud de los resultados derivados de un geoproceso, todo ello en función de las recomendaciones del <a title="OGC Simple Feature Access specification" href="http://www.opengeospatial.org/standards/sfa" target="_blank">Open Geospatial Consortium</a>.

Pero es probable que, en algún punto de una secuencia de geoprocesamiento, necesitemos manipular las coordenadas de una geometría (por ejemplo, forzar una corrección de forma transparente para el  usuario). Asumiendo que el array de coordenadas almacena la secuencia de vértices actual en lugar de una copia (que la propiedad <em>Coordinates </em>sea o no una copia depende de la secuencia de coordenadas empleada en la construcción de la geometría; tenéis más información sobre esto en el capítulo 8 de la <a title="JTS Developer's Guide" href="http://www.vividsolutions.com/jts/bin/JTS%20Developer%20Guide.pdf" target="_blank">guía del desarrollador de JTS</a>), nada nos impide editar diréctamente los valores en ese array. También podríamos copiar dicho array, operar sobre los vértices en cuestión, instanciar una nueva geometría y desechar la antigua. Discutir sobre la idoneidad de estas soluciones no es objeto de este artículo. Lo que interesa es que sepamos que el propio API proporciona otras vías específicas para la edición de vértices.

En concreto, si tan sólo necesitamos operar con las coordenadas existentes (no añadir ni borrar), NTS proporciona el interfaz <em>ICoordinateFilter</em>, a partir del cual pueden derivarse clases que implementen el <a title="visitor pattern" href="http://en.wikipedia.org/wiki/Visitor_pattern"><em>visitor pattern</em></a> para operar con la lista de coordenadas de cualquier tipo de geometría. Cada desarrollador puede implementar sus propias operaciones o filtros de coordenadas a conveniencia, pero NTS ya proporciona unos cuantos: <em>UniqueCoordinateArrayFilter</em>, <em>CoordinateCountFilter </em>y, el más simple y objeto de nuestro interés,  <em>CoordinateArrayFilter</em>, que da acceso al array completo de coordenadas para su manipulación.
<h2>Veamos un ejemplo</h2>
Suponed que tenemos el siguiente LineString:
<pre>LINESTRING (100 100,  150 180,  200 210,
            210 140,  100 110,  100 100)</pre>
Si lo visualizárais en vuestro cliente favorito observaríais algo similar a esto:
<p style="padding-left: 90px"><img class="size-medium wp-image-632" src="http://www.gisandchips.org/wp-content/CoordinateArrayFilter1-300x222.png" alt="LineString con autointersección" width="300" height="222" /></p>
Esta geometría presenta una autointersección que NTS permite detectar de diversas formas. Nosotros nos centraremos en corregirla aplicando un criterio muy simple: intercambiaremos los valores de las coordenadas en las posiciones 0 y 4. Aquí tenéis el código fuente de esta operación de desplazamiento de vértices:

[csharp]
using System;
using GisSharpBlog.NetTopologySuite.Geometries;
using GisSharpBlog.NetTopologySuite.IO;
using GisSharpBlog.NetTopologySuite.Utilities;

namespace CoordFilter
{
 class Program
 {
  public static void Main(string[] args)
  {
   string wkt = &quot;LINESTRING (100 100,  150 180,  200 210, 210 140,  100 110,  100 100)&quot;;
   WKTReader r = new WKTReader();
   Geometry g = r.Read(wkt);
   //Creamos sendas copias de las coordenadas objeto de desplazamiento
   Coordinate c0 = new Coordinate(g.Coordinates[0]);
   Coordinate c4 = new Coordinate(g.Coordinates[4]);
   //Instanciamos el filtro; puesto que queremos acceso a todo el array
   //de coordenadas de nuestra geometría, indicamos el tamaño del array
   CoordinateArrayFilter filter = new CoordinateArrayFilter(g.NumPoints);
   //El método Apply vuelca el array de coordenadas de la geometría en
   //el array de coordenadas del filtro
   g.Apply(filter);
   //Realizamos la operación de desplazamiento de vértices
   //sobre el array de coordenadas del filtro
   filter.Coordinates[0].X = c4.X;
   filter.Coordinates[0].Y = c4.Y;
   filter.Coordinates[4].X = c0.X;
   filter.Coordinates[4].Y = c0.Y;
   filter.Coordinates[5].X = c4.X;
   filter.Coordinates[5].Y = c4.Y;
   //Notificamos la actualización para que el Envelope pueda recalcularse
   g.GeometryChanged();
   Console.WriteLine(g.ToText());
   Console.Write(&quot;Press any key to continue . . . &quot;);
   Console.ReadKey(true);
  }
 }
}
[/csharp]

Si ejecutáis el código, obtendréis el siguiente resultado en WKT:
<pre>LINESTRING (100 110,  150 180,  200 210,
            210 140,  100 100,  100 110)</pre>
Y este sería el resultado gráfico:
<p style="padding-left: 90px"><img class="size-medium wp-image-657" src="http://www.gisandchips.org/wp-content/CoordinateArrayFilter2-300x221.png" alt="LineString modificado con CoordinateArrayFilter" width="300" height="221" /></p>

<h2>¿Qué hay del modelo de precisión?</h2>
Lo cierto es que la clase CoordinateArrayFilter no toma en consideración el modelo de precisión de la geometría a modificar. Entonces ¿cómo podemos controlar que, ante un desplazamiento, las coordenadas se calculen sobre el "grid" de referencia asociado a la geometría? Una opción es utilizar la clase <em>GeometryEditor</em>, lo que implica generar una nueva geometría a partir de la antigua. Pero ya que hablamos de filtros y traslado de coordenadas, ¿por qué no derivar una clase de tipo ICoordinateFilter que, dado un desplazamiento (dx, dy), mueva una geometría al completo respetando su modelo de precisión? Aquí tenéis un ejemplo:

[csharp]
using System;
using GisSharpBlog.NetTopologySuite.Geometries;
using GisSharpBlog.NetTopologySuite.IO;

namespace CoordFilter
{
 class Program
 {
  public static void Main(string[] args)
  {
   string wkt = &quot;LINESTRING (100.24 100.5, 150.82 180.56, 200.06 210.1, 210.44 140.82, 100.24 100.5)&quot;;
   //Asumiendo que la unidad principal es el metro, asociamos un &quot;grid&quot; centimétrico
   //a nuestro objeto GeometryFactory.
   WKTReader r = new WKTReader(new GeometryFactory(new PrecisionModel(100)));
   Geometry g = r.Read(wkt);
   //Instanciamos el filtro; puesto que queremos acceso a todo el array
   //de coordenadas de la geometría a desplazar, indicamos el tamaño del array;
   //además indicamos el modelo de precisión de dicha geometría.
   GeometryMoveFilter filter = new GeometryMoveFilter(g.NumPoints, g.Factory.PrecisionModel);
   g.Apply(filter);
   filter.Move(5.342237891, 3.865754133);
   g.GeometryChanged();
   Console.WriteLine(g.ToText());
   Console.Write(&quot;Press any key to continue . . . &quot;);
   Console.ReadKey(true);
  }

  //Clase derivada de ICoordinateFilter para el desplazamiento
  //de geometrías acorde con un modelo de precisión. El código de esta
  //clase está basado en el de la clase CoordinateArrayFilter de NTS,
  //cuyo autor es Diego Guidi.
  class GeometryMoveFilter : ICoordinateFilter
  {
   Coordinate[] pts = null;
   int n = 0;
   PrecisionModel grid = null;

   //Al igual que la clase CoordinateArrayFilter, el constructor necesita
   //el tamaño del array de coordenadas. Adicionalmente se necesita el
   //objeto PrecisionModel que define el &quot;grid&quot; de referencia
   public GeometryMoveFilter(int size, PrecisionModel model)
   {
    pts = new Coordinate[size];
    grid = model;
   }

   //Este método aplica un desplazamiento en X y en Y sobre
   //cada coordenada
   public void Move(double xOffset, double yOffset)
   {
    for(int i = 0; i &lt; n; i++)
    {
     Coordinate c = pts[i];
     //Aplicamos el desplazamiento
     c.X += xOffset;
     c.Y += yOffset;
     //Ajustamos la coordenada al &quot;grid&quot; de referencia
     grid.MakePrecise(ref c);
    }
   }

   public void Filter(Coordinate coord)
   {
    pts[n++] = coord;
   }
  }
 }
}
[/csharp]

Si lo ejecutáis deberíais obtener el siguiente resultado:

LINESTRING (105.58 104.37, 156.16 184.43, 205.4 213.97, 215.78 144.69, 105.58 104.37)

Recordad que si estáis trabajando con NTS 1.2 hay que solucionar previamente un "bug" que afecta a la clase <em>PrecisionModel</em>. En <a title="modelos de precisión fija en NTS" href="http://www.gisandchips.org/2009/11/03/como-definir-y-usar-modelos-de-precision-fija-en-nts/" target="_blank">este artículo</a> encontraréis información sobre cómo resolverlo.