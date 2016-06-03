---
ID: 1886
post_title: Proveedor de datos SQLite
author: josetomas
post_date: 2011-11-13 00:25:15
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2011/11/13/proveedor-de-datos-spatialite/
published: true
---
<p align="justify">Si habéis intentado alguna vez acceder con C# a una base de datos <a href="http://www.sqlite.org/">SQLite</a>, ya sabréis que existe un proveedor ADO.NET de código abierto, <a href="http://system.data.sqlite.org">System.Data.SQLite</a>. Sin embargo, a los que trabajamos en Linux, Mono ya nos ofrece un proveedor integrado en la propia plataforma (<a href="http://www.mono-project.com/SQLite">Mono.Data.Sqlite</a>), que de hecho deriva del anterior. Mejor aún, las distros más populares incorporan SQLite por defecto. Y si, además, hablamos de una geodatabase, no hay ningún problema: hay paquetes de <a href="http://www.gaia-gis.it/spatialite/">SpatiaLite</a> a nuestra disposición. Todo parece un camino de rosas hasta que, a pesar de tenerlo todo felizmente instalado, uno se encuentra con que no es posible cargar la librería de SpatiaLite: al ejecutar el comando "SELECT load_extension('libspatialite.so.2')" obtenemos un lacónico "Not authorized". Afortunadamente hay soluciones. La que aquí os propongo consiste simplemente en que uséis el proveedor Org.Gisandchips.Sqlite.</p><!--more-->

<h2>El problema de las extensiones de SQLite</h2>

<p align="justify">SpatiaLite es fundamentalmente una librería (libspatialite) que actúa como extensión de SQLite: explota el índice R-Tree de SQLite y añade el tipo Geometry y una serie de funciones que a su vez invocan a las de GEOS y PROJ. El resultado, al igual que el tándem PostgreSQL / PostGIS, es una implementación de la especificación "Simple Features for SQL" del OGC pero sin necesidad de una infraestructura cliente-servidor. Este tipo de librerías, que contienen funciones que extienden las propias de SQLite, se cargan dinámicamente, es decir, es el desarrollador el encargado de enlazarlas en tiempo de ejecución en el ámbito de una conexión de base de datos. El problema es que este mecanismo de extensión constituye a su vez un agujero de seguridad, un "coladero" de software malicioso. Así que, por defecto, las distros de Linux proporcionan el binario de SQLite compilado de forma que no se autoriza la carga dinámica de extensiones. Supongo que al final, quienes lo necesitan, optan por compilar su propio binario de SQLite o utilizar el que proporciona SpatiaLite, donde la extensión está estáticamente enlazada. Los que trabajáis con Mono tenéis ahora otra opción más.</p>

<h2>Org.Gisandchips.Sqlite</h2>

<p align="justify">Podéis descargar el código fuente de este proveedor ADO.NET para SQLite desde el repositorio SVN de Gis&amp;Chips:</p>

<code>svn co http://www.gisandchips.org/svn/sqliteprovider</code>

<p align="justify">Para vuestra comodidad, os proporciono el fichero de proyecto MonoDevelop. Yo compilo sin problemas en una máquina Ubuntu 11.04, con Mono 2.10, MonoDevelop 2.6 y bajo el framework .NET 4.</p>
<p align="justify">Si echáis un vistazo al código comprobaréis que se trata del mismo proveedor Mono.Data.Sqlite (obtenido de la master branch del repositorio público de Mono en github) con los siguientes añadidos:</p>
<ul>
	<li>Wrappers para las funciones enable_load_extension() and load_extension() del API de sqlite3.</li>
	<li>Método público SqliteConnection.LoadExtension() que permite cargar dinámicamente extensiones de sqlite3.</li>
</ul>

<p align="justify">El siguiente código de ejemplo es suficientemente explicativo:</p>
[csharp]
using (var cn = new SqliteConnection (&quot;Data Source=path/file.sqlite&quot;)) {
  cn.Open ();
  cn.LoadExtension (&quot;libspatialite.so.2&quot;);
  //your sql commands follow
}
[/csharp]

<p align="justify">Respecto a la licencia, me remito a la de Mono (MIT), que es la más permisiva y entiendo que compatible con las instrucciones del autor original de System.Data.SQLite, Robert Simpson, quien liberó este proveedor como de dominio público. Que lo disfrutéis.</p>