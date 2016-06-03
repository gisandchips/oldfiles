---
ID: 470
post_title: >
  Cómo definir y usar modelos de
  precisión fija en NTS
author: josetomas
post_date: 2009-11-03 14:14:29
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2009/11/03/como-definir-y-usar-modelos-de-precision-fija-en-nts/
published: true
syntaxhighlighter_encoded:
  - "1"
---
En <a title="JTS Topology Suite" href="http://www.vividsolutions.com/JTS/JTSHome.htm" target="_blank">JTS Topology Suite</a> el modelo de precisión  es un mecanismo central que determina la robustez de los cálculos geométricos y topológicos. En las <a title="especificaciones técnicas" href="http://www.vividsolutions.com/jts/bin/JTS%20Technical%20Specs.pdf" target="_blank">especificaciones técnicas</a> de este API libre para el análisis espacial encontrareis una detallada exposición de las implicaciones del modelo de precisión, principalmente en lo que se refiere a situaciones de colapso dimensional. Por ejemplo, podemos encontrarnos una situación de colapso dimensional cuando a partir de una geometría topológicamente válida calculada sobre un modelo de doble precisión (i.e. un polígono calculado mediante un objeto <em>Polygonizer</em>) obtenemos su representación en WKT y al intentar renderizarlo en una aplicación externa obtenemos un error de validación topológica (i.e. el polígono contiene una autointersección).

Cuando nos vemos en esta coyuntura, es decir, en la necesidad de transferir geometría entre aplicaciones mediante un formato interoperable pero de precisión limitada (i.e. DXF, WKT), hay que plantearse el uso de un modelo de precisión fija, a saber, un modelo que nos permita definir un "grid" al que siempre estarán referidas nuestras coordenadas, tanto las de origen como las derivadas del cálculo. En este breve artículo tenéis ejemplos de código fuente que os orientarán sobre cómo aplicar la clase <em>PrecisionModel</em> para trabajar con un "grid" de referencia en <a title="NetTopologySuite" href="http://code.google.com/p/nettopologysuite/" target="_blank">NetTopologySuite</a> (NTS), el porting de JTS a C#, en su versión 1.2.<!--more-->

Definir un "grid" es sencillo. Basta con crear una instancia de la clase <em>PrecisionModel</em> y asignarlo al objeto <em>GeometryFactory</em> mediante el cual construyamos nuestras geometrías. Cada geometría así instanciada heredará una referencia a un "grid" que determinará cualquier cálculo en el que dicha geometría intervenga posteriormente. Pero, ¿cómo se especifica la precisión de dicho  "grid"? Asignando un <strong>factor de escala</strong> que equivale al denominador de la fracción deseada de la unidad principal. Por ejemplo, si la unidad principal es el metro asignaremos 100 para trabajar en centímetros (coordenadas con un máximo de 2 decimales), 1000 para trabajar en milímetros (coordenadas con un máximo de 3 decimales), 200 para trabajar con una precisión de 0.5 centímetros, ...

[c language="#"]
using System;
using GisSharpBlog.NetTopologySuite.Geometries;

namespace Grids
{
 class Program
 {
  public static void Main(string[] args)
  {
   //Definición de varios &quot;grids&quot; asumiendo que trabajamos en metros
   PrecisionModel mGrid = new PrecisionModel(PrecisionModels.Fixed); //precisión métrica
   PrecisionModel mGrid2 = new PrecisionModel(1); //equivale al anterior
   PrecisionModel cmGrid = new PrecisionModel(100); //precisión centimétrica
   PrecisionModel mmGrid = new PrecisionModel(1000); //precisión milimétrica
   PrecisionModel halfcmGrid = new PrecisionModel(200); //0.5 cm de precisión
  }
 }
}
[/c]

Cuando se parte de geometrías expresadas como <a title="WKT" href="http://en.wikipedia.org/wiki/Well-known_text" target="_blank">WKT</a>, la aplicación del "grid" a la imagen en memoria de las nuevas geometrías es transparente para el programador. Basta con asignar al objeto <em>WKTReader</em> el correspondiente objeto <em>GeometryFactory</em>. Esto significa que si probáis este código:

[c language="#"]
using System;
using GisSharpBlog.NetTopologySuite.Geometries;
using GisSharpBlog.NetTopologySuite.IO;

namespace Grids
{
 class Program
 {
  public static void Main(string[] args)
  {
   string wkt = &quot;POLYGON ((253722.078898558 4292685.50730101, 253722.07889856 4292685.50730102, 253735.282733042 4292698.56531273, 253738.905778592 4292696.94218745, 253741.840165249 4292695.62758151, 253735.276565443 4292675.86615228, 253717.38 4292636.98, 253700.54 4292646.23, 253722.078898558 4292685.50730101))&quot;;
   PrecisionModel grid = new PrecisionModel(1000); //precisión milimétrica
   GeometryFactory gf = new GeometryFactory(grid); //vinculamos el &quot;grid&quot; con un objeto GeometryFactory
   WKTReader r = new WKTReader(gf); //asociamos el objeto GeometryFactory con el intérprete de WKT
   Console.WriteLine(r.Read(wkt).ToText()); //de WKT a Geometry y de nuevo a WKT
   Console.Write(&quot;Press any key to continue . . . &quot;);
   Console.ReadKey(true);
  }
 }
}
[/c]

Éste es el resultado que deberíais obtener:
<pre>POLYGON ((253722.079 4292685.507, 253722.079 4292685.507,
          253735.283 4292698.565, 253738.906 4292696.942,
          253741.84 4292695.628, 253735.277 4292675.866,
          253717.38 4292636.98, 253700.54 4292646.23,
          253722.079 4292685.507))</pre>
Sin embargo, puede ocurrir que nuestros datos de entrada se presenten como lista de pares (x,y). En este caso, el programador es responsable de hacer que cada coordenada se ajuste previamente a la precisión del "grid" de referencia. Esto se consigue de manera efectiva invocando al método <em>MakePrecise</em> del correspondiente objeto <em>PrecisionModel</em>. El siguiente ejemplo ilustra una solución para el polígono del ejemplo anterior, dada su lista de coordenadas.

[c language="#"]
using System;
using GisSharpBlog.NetTopologySuite.Geometries;
using GisSharpBlog.NetTopologySuite.IO;

namespace Grids
{
 class Program
 {
  public static void Main(string[] args)
  {
   List&lt;double[]&gt; coords = new List&lt;double[]&gt;();
   coords.Add(new double[]{253722.078898558, 4292685.50730101});
   coords.Add(new double[]{253722.07889856, 4292685.50730102});
   coords.Add(new double[]{253735.282733042, 4292698.56531273});
   coords.Add(new double[]{253738.905778592, 4292696.94218745});
   coords.Add(new double[]{253741.840165249, 4292695.62758151});
   coords.Add(new double[]{253735.276565443, 4292675.86615228});
   coords.Add(new double[]{253717.38, 4292636.98});
   coords.Add(new double[]{253700.54, 4292646.23});
   coords.Add(new double[]{253722.078898558, 4292685.50730101});

   PrecisionModel grid = new PrecisionModel(1000); //precisión milimétrica
   GeometryFactory gf = new GeometryFactory(grid); //vinculamos el &quot;grid&quot; con un objeto GeometryFactory
   Geometry g = GetPolygon(coords, gf); //instanciamos el polígono a partir de la lista de coordenadas y el objeto GeometryFactory
   Console.WriteLine(g.ToText()); //mostramos el resultado en WKT
   Console.Write(&quot;Press any key to continue . . . &quot;);
   Console.ReadKey(true);
  }

  static Polygon GetPolygon(List&lt;double[]&gt; coords, GeometryFactory gf)
  {
   List&lt;Coordinate&gt; fixedCoords = new List&lt;Coordinate&gt;();
   foreach (double[] xyPair in coords)
   {
    fixedCoords.Add(GetFixedCoordinate(xyPair[0], xyPair[1], gf));
   }
   return gf.CreatePolygon(gf.CreateLinearRing(fixedCoords.ToArray()), null);
  }

  static Coordinate GetFixedCoordinate(double x, double y, GeometryFactory gf)
  {
   Coordinate c = new Coordinate(x, y);
   gf.PrecisionModel.MakePrecise(ref c); //cálculo de coordenada precisa, es decir, referida al &quot;grid&quot; del modelo de precisión milimétrica que lleva aparejado el objeto GeometryFactory
   return c;
  }
 }
}
[/c]

Si ejecutáis este código deberíais obtener un resultado idéntico al anterior.
<h2>¿Y si no funciona ...?</h2>
Como  dije al principio, podéis usar NTS 1.2 para probar estos ejemplos. Si preferís usar NTS 1.7 seguramente necesitaréis efectuar pequeños cambios en el código. De todos modos, tanto con la versión 1.2 como con la 1.7.1 hay un "bug" que hace que, independientemente del factor de escala, el modelo de precisión  fija quede referido siempre a la unidad principal, es decir, a coordenadas enteras. Este "bug" está corregido en la versión 1.7.3. Lo bueno de trabajar con software libre es que en la mayoría de casos (y éste es uno de ellos) la solución está a nuestro alcance. Así que, los que trabajéis con la versión 1.2, podéis descargar el código fuente <a title="NTS 1.2 download" href="http://sourceforge.net/projects/nts/files/NetTopologySuite/NetTopologySuite%201.2%20Final%20Release/NetTopologySuite_20feb06.zip/download">aquí</a> y modificar la línea <strong>351</strong> del fichero <strong>PrecisionModel.cs</strong>. Donde dice:
<pre>return Math.Floor(((val * scale) + 0.5d) / scale);</pre>
simplemente debe decir:
<pre>return Math.Floor((val * scale) + 0.5d) / scale;</pre>