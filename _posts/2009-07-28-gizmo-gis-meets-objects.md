---
ID: 93
post_title: 'Gizmo: GIS Meets Objects!'
author: josetomas
post_date: 2009-07-28 09:33:08
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2009/07/28/gizmo-gis-meets-objects/
published: true
syntaxhighlighter_encoded:
  - "1"
---
<p style="text-align: justify">De acuerdo, el título parece un tanto pretencioso. Hace años que la tecnología GIS y la programación orientada a objetos confluyeron para hacernos la vida más fácil. Sin embargo, si al empezar vuestro próximo proyecto les decís a vuestros compañeros "¡eh, esta vez no hay modelo entidad-relación!" seguro que provoca sorpresa. Efectivamente, alguno de vosotros ya se lo imagina: os voy a hablar de bases de datos geográficas orientadas a objetos.<!--more--> Las bases de datos orientadas a objetos (OODB) constituyen uno de los campos de investigación y desarrollo más activos en la actualidad, y de hecho son, junto con la interoperabilidad, la clave que determina el futuro de la tecnología GIS. Existen diversas alternativas de OODB de código abierto. Probablemente el proyecto más activo y difundido sea <a title="DB4O" href="http://www.db4o.com/" target="_blank">DB4O</a>. No renuncio a explorar a fondo este gestor más adelante, sobre todo teniendo en cuenta que la comunidad empieza a construir utilidades para el almacenamiento de datos espaciales como <a title="MapMe" href="http://code.google.com/p/mapme/" target="_blank">MapMe</a>. Sin embargo os confieso que en algún punto se cruzó en mi camino <a title="Perst" href="http://www.mcobject.com/perst" target="_blank">Perst</a>, otro OODB bastante popular, y al indagar acerca de los métodos de indexación y comprobar que implementa R-Trees me picó la curiosidad. Muchos de vosotros sabéis que un experimento lleva a otro, y luego a otro, y así sucesivamente hasta que sin darnos cuenta tenemos un API que ... ¡sirve para algo! De modo que aquí os presento <strong>Gizmo</strong>, un API con el que podéis diseñar vuestra propia geodatabase orientada a objetos. Como su propio nombre indica, Gizmo es un trasto, es rudimentario, no hace muchas cosas, pero garantiza un rato de diversión a los curiosos dispuestos a escribir unas pocas líneas en C#.
Para descargar el código fuente de Gizmo podéis hacer un <em>checkout </em>del siguiente repositorio Subversion de GIS&amp;Chips:</p>
<p style="padding-left: 30px"><code>svn co http://www.gisandchips.org/svn/gizmo/release_0.1</code></p>
<p style="text-align: justify">Obtendréis una solución <a title="SharpDevelop" href="http://www.icsharpcode.net/OpenSource/SD/Default.aspx" target="_blank">SharpDevelop</a> en C# con el proyecto del API <strong>Gizmo</strong>. El proyecto compila sobre el framework .NET 2.0 e incluye los ensamblados de referencia de <a title="Perst" href="http://www.mcobject.com/perst" target="_blank">Perst.NET</a>, <a title="NetTopologySuite" href="http://code.google.com/p/nettopologysuite/" target="_blank">NetTopologySuite</a> y <a title="ProjNET" href="http://projnet.codeplex.com/" target="_blank">ProjNET</a>.</p>
<p style="text-align: justify"></p>

<h4>Cómo diseñar nuestro propio modelo</h4>
<p style="text-align: justify">En pocas palabras, Gizmo es un conjunto de interfaces y clases que facilitan al desarrollador la definición de su propio modelo de objetos geográficos de forma que éstos puedan ser almacenados en una base de datos Perst. Como ya he dicho, esto no deja de ser un experimento: no esperéis una herramienta CASE o un interfaz amigable de diseño a base de diagramas. De momento se trata de que podáis definir vuestras clases siguiendo una lógica coherente de derivación e implementación de métodos y propiedades sin preocuparos excesivamente de cómo funciona Perst. Recordad que, al fin y al cabo, el objetivo es que podáis persistir/serializar en un fichero binario cada objeto geográfico que instanciéis, para poder recuperarlo más adelante mediante criterios espaciales gracias a los índices R-Tree. No hay tablas, ni campos, ni registros. No hay <em>Object Relational Mapping</em>. Se trata de vuestros propios objetos, tal y como los hayáis definido en vuestro modelo, con el estado en que se encontraban al serializarlos.
Pero veamos un ejemplo básico de aplicación en línea de comandos con C#. Para nuestro proyecto necesitaremos sendas referencias a Gizmo y Perst.NET. Para mayor comodidad incluid el proyecto de Gizmo en vuestra solución y añadidlo como referencia en vuestro proyecto de aplicación de consola. Respecto a Perst.NET, disponéis de una copia del ensamblado en el directorio <em>lib</em> del proyecto Gizmo (<em>lib\Perst4.NET\PerstNetGenerics.dll) </em>que podéis usar para agregarlo como referencia en vuestro proyecto.</p>
<p style="text-align: justify">Construiremos un modelo muy simple en el que vamos a representar parcelas y conducciones. Esta podría ser la definición de nuestra clase <em>Parcela</em>:</p>

[csharp]
using System;
using Gizmo.Core;

namespace TestGizmo
{
 public class Parcela : GizmoObject
 {
  string _id = string.Empty;
  string _cultivo = string.Empty;

  public string Id {
   get { return _id; }
  }

  public string Cultivo {
   get { return _cultivo; }
  }

  public Parcela(string wkt, int srid,
                 string id, string cultivo) : base(wkt, srid)
  {
   _id = id;
   _cultivo = cultivo;
  }
 }
}
[/csharp]

Y esta la definición de la clase <em>Conduccion:</em>

[csharp]
using System;
using Gizmo.Core;

namespace TestGizmo
{
 public class Conduccion : GizmoObject
 {
  string _id = string.Empty;
  string _categoria = string.Empty;

  public string Id {
   get { return _id; }
  }

  public string Categoria {
   get { return _categoria; }
  }

  public Conduccion(string wkt, int srid,
                    string id, string categoria) : base(wkt, srid)
  {
  _id = id;
  _categoria = categoria;
  }
 }
}
[/csharp]
<p style="text-align: justify">Ya tenemos la primera clave para implementar nuestro modelo con Gizmo: nuestros tipos de entidad geográfica deben derivar de la clase <em>GizmoObject</em> y pasar en el constructor dos parámetros comunes a cualquier objeto geográfico en Gizmo, a saber, la geometría (en nuestro caso en WKT) y el identificador del Sistema de Referencia espacial.</p>
<p style="text-align: justify">El siguiente paso es crear el catálogo de índices de nuestro modelo. Esto nos permitirá implementar búsquedas eficientes por atributos. Para ello definamos una clase <em>Catalogo</em>:</p>

[csharp]
using System;
using Gizmo.Catalogue;
using Perst;

namespace TestGizmo
{
  public class Catalogo : GizmoCatalogue
  {
    FieldIndex&lt;string, Parcela&gt; _parcelaIdIdx = null;
    FieldIndex&lt;string, Parcela&gt; _parcelaCultIdx = null;
    FieldIndex&lt;string, Conduccion&gt; _conduccionIdIdx = null;
    FieldIndex&lt;string, Conduccion&gt; _conduccionCatIdx = null;

   public FieldIndex&lt;string, Parcela&gt; ParcelaIdIdx {
     get { return _parcelaIdIdx; }
   }

   public FieldIndex&lt;string, Parcela&gt; ParcelaCultIdx {
     get { return _parcelaCultIdx; }
   }

   public FieldIndex&lt;string, Conduccion&gt; ConduccionIdIdx {
     get { return _conduccionIdIdx; }
   }

   public FieldIndex&lt;string, Conduccion&gt; ConduccionCatIdx {
     get { return _conduccionCatIdx; }
   }

   public Catalogo(Storage db, string name) : base(db, name)
   {
     _parcelaIdIdx = db.CreateFieldIndex&lt;string, Parcela&gt;(&quot;Id&quot;, true);
     _parcelaCultIdx = db.CreateFieldIndex&lt;string, Parcela&gt;(&quot;Cultivo&quot;, false);
     _conduccionIdIdx = db.CreateFieldIndex&lt;string, Conduccion&gt;(&quot;Id&quot;, true);
     _conduccionCatIdx = db.CreateFieldIndex&lt;string, Conduccion&gt;(&quot;Categoria&quot;, false);
   }

   public Catalogo()
   {
   }
 }
}
[/csharp]
<p style="text-align: justify">Como podeis ver aquí la clave está en derivar de la clase <em>GizmoCatalogue</em>, crear en el constructor un índice simple de Perst para cada una de las propiedades de nuestras clases <em>Parcela</em> y <em>Conduccion</em> e implementar los <em>getters </em>correspondientes. Alguien se preguntará "¿y qué hay del índice espacial?": bueno, precisamente de esa tarea se ocupan Gizmo y Perst en última instancia.</p>
<p style="text-align: justify">Finalicemos nuestro modelo agregando una clase <em>layer</em> para cada uno de nuestros tipos de entidad geográfica. Esta es la clase <em>CapaParcelas</em>:</p>

[csharp]
using System;
using Gizmo.Core;

namespace TestGizmo
{
 public class CapaParcelas : GizmoLayer
 {
  public Catalogo _root = null;

  protected override Type CatalogueType
  {
   get { return typeof(Catalogo); }
  }

  protected override Type TargetType
  {
   get { return typeof(Parcela); }
  }

  public CapaParcelas(GizmoStore gs, string name) : base(gs, name)
  {
   this._root = (Catalogo) base.Root;
  }

  public Parcela FindById(string id)
  {
   return _root.ParcelaIdIdx.Get(id);
  }

  public Parcela[] FindByCultivo(string cultivo)
  {
   return _root.ParcelaCultIdx.Get(cultivo, cultivo);
  }
 }
}
[/csharp]
<p style="text-align: justify">Y aquí tenéis la clase <em>CapaConducciones</em>:</p>

[csharp]
using System;
using Gizmo.Core;

namespace TestGizmo
{
 public class CapaConducciones : GizmoLayer
 {
  Catalogo _root = null;

  protected override Type CatalogueType
  {
   get { return typeof(Catalogo); }
  }

  protected override Type TargetType
  {
   get { return typeof(Conduccion); }
  }

  public CapaConducciones(GizmoStore gs, string name) : base(gs, name)
  {
   this._root = (Catalogo) base.Root;
  }

  public Conduccion FindById(string id)
  {
   return _root.ConduccionIdIdx.Get(id);
  }

  public Conduccion[] FindByCategoria(string categoria)
  {
   return _root.ConduccionCatIdx.Get(categoria, categoria);
  }
 }
}
[/csharp]
<p style="text-align: justify">Las clases <em>layer</em> son el mecanismo para explotar objetos geográficos del mismo tipo. La clave está en designar el tipo asociado mediante la propiedad <em>TargetType</em> e implementar métodos de recuperación de objetos (<em>FindBy...</em>) mediante los correspondientes índices del catálogo.</p>
<p style="text-align: justify"></p>

<h4 style="text-align: justify">¡Hagamos una prueba!</h4>
<p style="text-align: justify">Ya podemos editar la función <em>Main</em> de nuestra aplicación de consola. No olvidéis añadir el correspondiente  <em>using Gizmo.Core; </em>en la cabecera de vuestro fichero <em>Program.cs</em>. En primer lugar creamos nuestra base de datos geográfica (<em>gizmo.dbs)</em>, la abrimos y creamos las capas de parcelas y conducciones.</p>

[csharp]
GizmoStore gs = new GizmoStore(&quot;gizmo.dbs&quot;);
gs.Open();
CapaParcelas cp = new CapaParcelas(gs, &quot;parcelas&quot;);
CapaConducciones cc = new CapaConducciones(gs, &quot;conducciones&quot;);
[/csharp]
<p style="text-align: justify">A continuación almacenamos un  par de parcelas y una conducción a partir de geometrías en WKT en el sistema de proyección UTM 30N ED50 (SRID = 23030).</p>

[csharp firstline="5"]
string wktP0 = &quot;POLYGON((10 10, 10 20, 20 20, 20 10, 10 10))&quot;;
string wktP1 = &quot;POLYGON((30 30, 30 40, 40 40, 40 30, 30 30))&quot;;
cp.Add(new Parcela(wktP0, 23030, &quot;P0&quot;, &quot;CEREAL&quot;));
cp.Add(new Parcela(wktP1, 23030, &quot;P1&quot;, &quot;FRUTAL&quot;));
string wktC0 = &quot;LINESTRING(5 5, 45 45)&quot;;
cc.Add(new Conduccion(wktC0, 23030, &quot;C0&quot;, &quot;PRIMARIA&quot;));
[/csharp]

Como podéis ver,  almacenamos/persistimos nuestras parcelas y conducciones a través de la <em>layer</em> correspondiente invocando el método <em>Add</em>. En la <em>layer</em> <em>CapaParcelas </em>tenemos la parcela <strong>P0</strong> con cultivo de <strong>CEREAL</strong> y la <strong>P1</strong> con cultivo de <strong>FRUTAL</strong>. Y en la <em>layer CapaConducciones </em>tenemos la tubería <strong>C0</strong> de categoría <strong>PRIMARIA</strong>. Hasta aquí todo resulta bastante sencillo. Pero ¿y si queremos saber por qué parcelas pasa la tubería <strong>C0</strong>? Pues bien, esta es la respuesta en 2 líneas:

[csharp firstline="11"]

Conduccion c0 = cc.FindById(&quot;C0&quot;);
IGizmoObject[] query = cp.FindCrossedBy(c0);

[/csharp]

Y si imprimimos las parcelas resultantes,

[csharp firstline="13"]

foreach (IGizmoObject p in query)
{
 Console.WriteLine((p as Parcela).Id);
}
gs.Close();
[/csharp]

el resultado que debemos obtener es <strong>P0</strong> y <strong>P1</strong>.

Por cierto: ¡no olvidéis cerrar vuestra base de datos! (línea de código 17).

Eso es todo. Espero que disfrutéis un rato con Gizmo. En futuros artículos comentaré más aspectos de este proyecto.