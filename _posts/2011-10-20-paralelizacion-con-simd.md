---
ID: 1744
post_title: Paralelización con SIMD
author: josetomas
post_date: 2011-10-20 18:31:37
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2011/10/20/paralelizacion-con-simd/
published: true
Hide SexyBookmarks:
  - "0"
Hide OgTags:
  - "0"
---
<p style="text-align: justify">En este artículo os presento un test en el que medimos los tiempos que un programa escrito en C# emplea para procesar una serie de modelos digitales del terreno. El objetivo final es comparar el rendimiento de un código que hace un cálculo secuencial frente a otro que hace uso de una tecnología de paralelización de datos por hardware denominada SIMD, disponible en la mayoría de microprocesadores que usamos hoy en día. Y el resultado es muy interesante.[caption id="attachment_1801" align="aligncenter" width="300" caption="Test de rendimiento SIMD con Mono.Simd.Vector8s"]<a href="http://www.gisandchips.org/wp-content//SIMD_performance.png"><img src="http://www.gisandchips.org/wp-content//SIMD_performance-300x131.png" alt="SIMD performance test" width="300" height="131" class="size-medium wp-image-1801" /></a>[/caption]<!--more--></p>
<p style="text-align: justify">Las técnicas de computación paralela encuentran en los sistemas de información geográfica una de sus aplicaciones más elocuentes dada la naturaleza masiva de los datos geográficos, en particular si hablamos de estructuras de datos raster. No en vano, allá por el año 1997, Richard Healey, junto con otros compañeros del Dpto. de Geografía de la Universidad de Edimburgo, editó "Parallel Processing Algorithms For GIS". El profesor Healey es un geógrafo que ya a principios de los 90 planteaba a sus alumnos problemas de geoprocesamiento paralelo que éstos habían de resolver con ADA, el único lenguaje accesible que por aquel entonces prefiguraba el paradigma OOP y permitía definir tareas concurrentes. Desde entonces se han producido muchos avances en paralelización de procesos, tanto a nivel de software como de hardware. Sin embargo, existe un tipo de paralelización que no parece haber alcanzado demasiada notoriedad en el campo de los GIS: se trata del conjunto de instrucciones <a href="http://en.wikipedia.org/wiki/SIMD">SIMD</a> (Single Instruction, Multiple Data) que permite realizar operaciones aritméticas entre pares de vectores mediante una sola instrucción. No se trata por tanto de concurrencia de procesos, sino de paralelismo a nivel de datos. Lo más curioso es que, desde 1996, los fabricantes de microprocesadores han ido incorporando extensiones que soportan en mayor o menor medida este juego de instrucciones.</p>
<p style="text-align: justify">Cualquier PC o videoconsola actual permite ejecutar instrucciones SIMD. Evidentemente los desarrolladores de juegos hace tiempo que explotan este tipo de paralelización. Sin embargo, encontrar referencias recientes a SIMD en el ámbito de la Geomática resulta más complicado: <a href="http://gisws.media.osaka-cu.ac.jp/grass04/viewpaper.php?id=15">aquí tenéis una interesante ponencia</a> de la conferencia de usuarios de GRASS del 2004. Seguramente habrá más trabajos publicados, así que hacednos saber lo que encontréis.</p>
<p style="text-align: justify">Yo sinceramente no sabía nada de esto hasta que leí <a href="http://tirania.org/blog/archive/2008/Nov-03.html">este post de Miguel de Icaza</a>, anunciando la publicación de <a href="http://docs.go-mono.com/monodoc.ashx?link=N%3aMono.Simd">Mono.Simd</a>. Este API es un paso en la estrategia de Mono por entrar de lleno en la industria del "gaming". ¿Y para las desarrolladores de GIS? En mi opinión significa un abanico de posibilidades a la hora de paralelizar geoprocesos que queda pendiente de explorar.</p>
<p style="text-align: justify">En este artículo sólo quiero dejar constancia de un test para un caso extremo de optimización con SIMD. En la práctica carece de utilidad, mi única intención es medir diferencias de tiempo en un escenario absolutamente favorable a la paralelización mediante vectores. Dado un grid de elevaciones, donde cada valor de altitud puede representarse como un entero de 16 bits, se trata simplemente de doblar el valor de cada pixel para "exagerar" el modelo del terreno. Estas son las especificaciones de la máquina en que he realizado el test:</p>

<ul>
	<li>Intel Q8300 @ 2.50GHz</li>
	<li>3.9 GiB RAM</li>
	<li>Ubuntu 11.04 i686</li>
	<li>Mono 2.10.5</li>
</ul>
<p style="text-align: justify">El test se ha realizado sobre 4 ficheros <a href="http://en.wikipedia.org/wiki/Esri_grid">ARC/INFO ASCII GRID</a> de distintos tamaños. Para cada fichero se han tomado 5 muestras del tiempo transcurrido en el cálculo aritmético sin aceleración de hardware, pixel a pixel, tomando finalmente la mediana. La misma metodología de muestreo se ha empleado para obtener los tiempos con aceleración de hardware, es decir, empleando Mono.Simd para multiplicar los pixels de 8 en 8 mediante una única instrucción. Para ello es necesario agrupar previamente los valores de altitud y almacenar cada vector en una lista de tipo Mono.Simd.Vector8s. Estos son los resultados:</p>

<table style="height: 5px;width: 80%" border="1" cellspacing="1" cellpadding="1" align="center">
<thead>
<tr>
<th scope="col"><span class="Apple-style-span" style="font-weight: normal">File size (MiB)</span></th>
<th scope="col"><span class="Apple-style-span" style="font-weight: normal">Elapsed time (miliseconds)</span></th>
<th scope="col"><span class="Apple-style-span" style="font-weight: normal">Elapsed time with SIMD (miliseconds)</span></th>
<th scope="col"><span class="Apple-style-span" style="font-weight: normal">Time reduction</span></th>
<th scope="col"><span class="Apple-style-span" style="font-weight: normal">Performance gain</span></th>
</tr>
</thead>
<tbody>
<tr>
<td>12.0</td>
<td>49.7447</td>
<td>14.2542</td>
<td>71.3</td>
<td><span style="background-color: #ffd700"><strong>3.5x</strong></span></td>
</tr>
<tr>
<td>48.2</td>
<td>197.5251</td>
<td>56.149</td>
<td>71.6</td>
<td><span style="background-color: #ffd700"><strong>3.5x</strong></span></td>
</tr>
<tr>
<td>192.7</td>
<td>783.0778</td>
<td>216.305</td>
<td>72.4</td>
<td><span style="background-color: #ffd700"><strong>3.6x</strong></span></td>
</tr>
<tr>
<td>535.4</td>
<td>2180.8665</td>
<td>618.2225</td>
<td>71.7</td>
<td><span style="background-color: #ffd700"><strong>3.5x</strong></span></td>
</tr>
</tbody>
</table>
<p style="text-align: justify">Como veis, en todos los casos, con SIMD el proceso de cálculo se completa en torno a 3,5 veces más rápido. Esto es una buena noticia, pero no significa que este tipo de paralelismo sea la panacea. En mi opinión la conclusión es que, si con un escenario favorable se puede reducir el tiempo de proceso en más de un 70%, merece la pena explorar otros escenarios con verdadera utilidad práctica.</p>
<p style="text-align: justify">Si alguien se pregunta por la incidencia que puede tener esta mejora en el rendimiento si contabilizamos el proceso de lectura y carga en memoria de los datos, me atrevería a decir que escasa. En cualquier caso, los procesos de lectura y manejo de grandes volúmenes de información raster constituyen un problema distinto y existen diversas técnicas para su optimización.</p>
<p style="text-align: justify">Si os pica la curiosidad, podeis averiguar el conjunto de instrucciones SIMD que soporta vuestro procesador ejecutando el código de ejemplo que aparece en la documentación de la clase <a href="http://docs.go-mono.com/monodoc.ashx?link=T%3aMono.Simd.SimdRuntime">Mono.Simd.SimdRuntime</a> (es posible que tengáis que hacer algunas modificaciones). Para aquellos que quieran reproducir un test de rendimiento con sus propios ficheros ASCII GRID, aquí os dejo el código C# que he usado (por cierto, ni es óptimo ni puedo garantizar que esté libre de bugs):</p>

[csharp]
using System;
using System.IO;
using Mono.Simd;
using System.Collections.Generic;
using System.Globalization;
using System.Diagnostics;
using System.Linq;

namespace SIMDTest
{
 class MainClass
 {
  public static void Main (string[] args)
  {
   bool useSIMD = true;
   string path = string.Empty;
   CultureInfo formatProvider = null;
   try {
    if (args.Length == 0) {
     Console.WriteLine (&quot;Usage: SIMDTest [ASC GRID file name] [specific culture name]&quot;);
     return;
    }
    if (args.Length &gt; 0)
     path = args[0];
    if (args.Length &gt; 1)
     formatProvider = CultureInfo.GetCultureInfo (args[1]);
    Console.WriteLine (&quot;Use hardware acceleration? [y / n]? : &quot;);
    ConsoleKeyInfo key = Console.ReadKey ();
    while (key.Key != ConsoleKey.Y &amp;&amp; key.Key != ConsoleKey.N) {
     Console.WriteLine (&quot;Press 'y' to use hardware acceleration, 'n' otherwise: &quot;);
     key = Console.ReadKey ();
    }
    Console.WriteLine ();
    Console.WriteLine (&quot;Processing, please wait ...&quot;);
    useSIMD = (key.Key == ConsoleKey.Y);
    ElevationModel dem = new ElevationModel(path, formatProvider);
    Stopwatch stopwatch = new Stopwatch ();
    if (useSIMD) {
     Vector8s scale = new Vector8s (2);
     foreach (IList vPage in dem.EnumerateVectors ()) {
      stopwatch.Start ();
      for (int i = 0; i &lt; vPage.Count; i ++)
       vPage[i] = vPage[i] * scale;
      stopwatch.Stop ();
     }
    } else {
     foreach (IList page in dem.EnumerateData ()) {
      stopwatch.Start ();
      for (int i = 0; i &lt; page.Count; i ++)
       page[i] = (short) (page[i] * 2);
      stopwatch.Stop ();
     }
    }
    Console.WriteLine (&quot;Time elapsed: {0}&quot;, stopwatch.Elapsed);
   } catch (Exception ex) {
    Console.WriteLine (ex.Message);
   }
  }
 }

 class ElevationModel
 {
  string _path;
  IFormatProvider _formatProvider = CultureInfo.CurrentCulture;
  short _noData = -9999;
  int BOD = 0;

  public ElevationModel (string path, IFormatProvider formatProvider)
  {
   if (string.IsNullOrEmpty (path))
    throw new ArgumentNullException ();
   else if (!File.Exists (path))
    throw new FileNotFoundException ();
   else {
    _path = path;
    if (formatProvider != null)
     _formatProvider = formatProvider;
    this.SetMetadata ();
   }
  }

  public ElevationModel (string path) : this(path, null)
  {
  }

  void SetMetadata ()
  {
   using (StreamReader reader = new StreamReader (_path)) {
    //skip GRID header
    for (int i = 0; i &lt; 5; i++)
     reader.ReadLine ();
    _noData = short.Parse (reader.ReadLine ().Split (' ')[1], _formatProvider);
    reader.BaseStream.Position = 0;
    reader.DiscardBufferedData ();
    char[] buffer = new char[512];
    reader.Read(buffer, 0, buffer.Length);
    BOD = this.GetBeginOfData (buffer);
   }
  }

  int GetBeginOfData (char[] source)
  {
   int i = 0;
   bool isNewHeaderLine = false;
   bool isNewLine = true;
   foreach (char c in source) {
    if (char.IsLetter (c) &amp;&amp; isNewLine) {
     isNewHeaderLine = true;
     isNewLine = false;
    } else if ((char.IsDigit (c) || c == '-') &amp;&amp; isNewLine &amp;&amp; !isNewHeaderLine) {
     break;
    } else if (char.IsControl (c)) {
     isNewLine = true;
     isNewHeaderLine = false;
    }
    i++;
   }
   return i;
  }

  IList&lt;short&gt; GetPage (char[] source, int sourceIndex, int sourceLength, IList pendingDigits)
  {
   IList&lt;short&gt; page = new List&lt;short&gt; ();
   for (int i = sourceIndex; i &lt; sourceLength; i++) {
    if (char.IsDigit (source[i]) || source[i] == '-') 
     pendingDigits.Add (source[i]);
    else if (pendingDigits.Count &gt; 0 &amp;&amp; (char.IsWhiteSpace (source[i]) || char.IsControl (source[i]))) {
     short n = _noData;
     string s = new string(pendingDigits.ToArray ());
     if (!short.TryParse(s, NumberStyles.Integer, _formatProvider, out n))
      Console.WriteLine (&quot;Error parsing {0}&quot;, s);
     page.Add (n);
     pendingDigits.Clear ();
    }
   }
   return page;
  }

  public IEnumerable&lt;IList&lt;short&gt;&gt; EnumerateData ()
  {
   using (StreamReader reader = new StreamReader (_path)) {
    char[] buffer = new char[25000 * 1024];
    int beginOfData = 0;
    IList pendingDigits = new List ();
    int charCount = 0;
    while ((charCount = reader.Read (buffer, 0, buffer.Length)) &gt; 0) {
     if (beginOfData == 0) {
      beginOfData = this.BOD;
      yield return this.GetPage (buffer, beginOfData, charCount, pendingDigits);
     } else
      yield return this.GetPage (buffer, 0, charCount, pendingDigits);
    }
   }
  }

  IList&lt;Vector8s&gt; GetVectorPage(IList page, IList pendingValues)
  {
   List vPage = new List ();
   foreach (short n in page) {
    pendingValues.Add (n);
    if (pendingValues.Count == 8 ) {
     vPage.Add (new Vector8s (pendingValues[0], pendingValues[1],
      pendingValues[2], pendingValues[3], pendingValues[4],
      pendingValues[5], pendingValues[6], pendingValues[7]));
     pendingValues.Clear ();
    }
   }
   return vPage;
  }

  public IEnumerable&lt;IList&lt;Vector8s&gt;&gt; EnumerateVectors ()
  {
   int lastPageSize = -1;
   IList pendingValues = new List ();
   foreach (IList page in this.EnumerateData ()) {
    IList vPage = this.GetVectorPage (page, pendingValues);
    // complete last vector in last page
    if (lastPageSize != -1 &amp;&amp; vPage.Count &lt; lastPageSize) {
     if (pendingValues.Count &gt; 0 &amp;&amp; pendingValues.Count &lt; 8 ) {
      for (int i = pendingValues.Count; i &lt; 8; i++)
	pendingValues.Add(_noData);
      vPage.Add (new Vector8s (pendingValues[0], pendingValues[1],
       pendingValues[2], pendingValues[3], pendingValues[4],
       pendingValues[5], pendingValues[6], pendingValues[7]));
     }
    }
    lastPageSize = vPage.Count;
    yield return vPage;
   }
  }
 }
}
[/csharp]