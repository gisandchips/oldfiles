---
ID: 2233
post_title: >
  Crear teselaciones (repitiendo formas
  hasta decir ¡basta!)
author: benizar
post_date: 2012-11-21 17:18:32
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2012/11/21/crear-teselaciones-repitiendo-formas-hasta-decir-%c2%a1basta/
published: true
---
Según la  <a title="Wikipedia" href="http://es.wikipedia.org/wiki/Teselado" target="_blank">Wikipedia</a>, un teselado o teselación es una regularidad o patrón de figuras que cubre completamente una superficie, sin dejar huecos y sin que las figuras se superpongan. Los que habéis trabajado mucho con SIG ya tenéis una buena idea de a que me estoy refiriendo...

En este post quiero hablaros un poco más de las teselaciones, comentar algunas aplicaciones y proponeros algo de código de ejemplo. Utilizando software libre, por supuesto ;)

<img class="size-full wp-image-2197 aligncenter" title="Figura1" src="http://www.gisandchips.org/wp-content//image02.png" alt="" width="355" height="271" />

Además, en la wikipedia se puede ver que existen numerosos tipos de teselaciones. De un modo general, éstas se pueden clasificar en teselaciones regulares, semirregulares e irregulares. Generalmente, en SIG se suelen utilizar distintas teselaciones regulares e irregulares para el análisis de zonas geográficas... Las teselaciones irregulares (Voronoi y Delaunay, de todo tipo) dan mucho juego. Podemos encontrar libros y publicaciones científicas de lo más interesantes (por ejemplo, podrías dedicar meses a leer y releer el libro <a title="Spatial Tessellations" href="http://www.amazon.es/Spatial-Tessellations-Applications-Probability-Statistics/dp/0471986356" target="_blank">Spatial Tessellations</a> ). Sin embargo, en este post nos centraremos en las teselaciones regulares más simples y las variaciones que pueden experimentar (ver la figura de arriba).
<h2><!--more--></h2>
<h2>¿Para qué sirven?</h2>
Las aplicaciones de las teselaciones en Geografía y SIG son muchas. Por mencionar algunas de las más habituales:
<ul>
	<li><strong>Estudios del paisaje a distintas escalas.</strong> En las siguiente figura podéis apreciar lo que sucedería si calculásemos el valor medio de los píxeles dentro de cada hexágono. Si tratásemos de contar las “celdas” que están mayormente ocupadas por el amarillo más intenso, obtendríamos valores distintos. Este tipo de consideraciones son muy tenidas en cuenta en los estudios de Ecología del Paisaje. Es necesario seleccionar la forma del grid, sus dimensiones y también su orientación. Esto nos dará una idea de la estructura espacial del paisaje. En la bibliografía se habla de “escalogramas” para estudiar los efectos del grid (<a title="Wu et al., 2002" href="http://www.google.es/url?sa=t&amp;rct=j&amp;q=&amp;esrc=s&amp;source=web&amp;cd=3&amp;ved=0CCgQFjAC&amp;url=http%3A%2F%2Fleml.asu.edu%2Fjingle%2FWeb_Pages%2FWu_Pubs%2FPDF_Files%2FWu_scalograms1_2002.pdf&amp;ei=hF6rUMPAOczWsgaE-YGwAg&amp;usg=AFQjCNEfF95Ca5czUvtNfCW2CvbX-E8qnA" target="_blank">Wu et al., 2002</a>).</li>
</ul>
<p style="text-align: center;"><img class="size-medium wp-image-2198" title="image2" src="http://www.gisandchips.org/wp-content//image12-300x211.jpg" alt="" width="214" height="150" /><img class="size-medium wp-image-2199" title="image03" src="http://www.gisandchips.org/wp-content//image01-300x202.jpg" alt="" width="226" height="153" /></p>
<p style="text-align: left;"><a title="http://www.cnfer.on.ca/SEP/patchanalyst/overview.htm" href="http://www.cnfer.on.ca/SEP/patchanalyst/overview.htm">http://www.cnfer.on.ca/SEP/patchanalyst/overview.htm</a></p>

<ul>
	<li><strong>Muestreos de la zona de estudio.</strong> Son muy utilizados en estudios biogeográficos donde se determina la presencia-ausencia de una determinada especie. Por ejemplo, el código que presento en este post fue desarrollado para un estudio de fototrampeo .</li>
</ul>
<img class="aligncenter size-full wp-image-2200" title="image04" src="http://www.gisandchips.org/wp-content//image04.png" alt="" width="261" height="241" />
<ul>
	<li><strong>Diseño de cartografía.</strong> El ejemplo más típico de uso de una cuadrícula es el del mapa turístico donde se utiliza un grid para localizar de manera relativa los elementos de interés. También se suelen crear cuadrículas UTM o se crean "<a title="Mapbooks" href="http://www.esri.com/news/arcuser/0702/dsmapbook1of2.html" target="_blank">Mapbooks</a>" que organizan una serie de mapas.</li>
</ul>
<img class="size-full wp-image-2201 alignleft" title="image06" src="http://www.gisandchips.org/wp-content//image06.jpg" alt="" width="233" height="175" /><img class="size-full wp-image-2202 aligncenter" title="image05" src="http://www.gisandchips.org/wp-content//image05.gif" alt="" width="200" height="168" />

&nbsp;
<ul>
	<li><strong>Interpolación espacial.</strong> En este caso predomina el uso de teselaciones no regulares. Por ejemplo, las triangulaciones de Delaunay son una alternativa muy utilizada para la creación de modelos digitales de elevaciones detallados. Igualmente, los polígonos de Voronoi se han utilizado en múltiples contextos, por ejemplo, para determinar la necesidad de instalar nuevos observatorios meteorológicos...</li>
</ul>
<img class="aligncenter size-medium wp-image-2203" title="image09" src="http://www.gisandchips.org/wp-content//image09-300x119.jpg" alt="" width="365" height="144" />
<h2>Software de ejemplo</h2>
Si tuviese que mencionar un software interesante para realizar teselaciones os recomendaría echarle un vistazo al <a title="Repeating Shapes for ArcGIS" href="http://www.jennessent.com/arcgis/repeat_shapes.htm" target="_blank">Repeating Shapes for ArcGIS</a>. Se trata de una extensión de ArcGIS, creada por el biólogo <strong>Jeff Jenness</strong>, que permite crear las teselaciones básicas, modificándolas según distintos parámetros (forma, dimensiones y ángulos). El manual que aparece en la web resulta muy explicativo para entender las teselaciones más básicas. Sin embargo, esta extensión es muy limitada si tenemos la necesidad de generar muchos grids, o añadir nuevos parámetros. Además, como hemos dejado entrever, existen muchas otras configuraciones posibles que este software no contempla.
<h2></h2>
<h2>Propuesta de software</h2>
He decidido ordenar un poco mis ideas y he creado un software que calcula teselaciones simples (rectángulos, triángulos y hexágonos). Se trata de una aplicación de consola que solicita una serie de parámetros para calcular la teselación. Esta aplicación utiliza la NetTopologySuite v1.11 y funciona bien sobre la versión 4.0 de la plataforma Mono.

<img class="aligncenter size-full wp-image-2204" title="image13" src="http://www.gisandchips.org/wp-content//image13.png" alt="" width="474" height="318" />

Como podéis ver los parámetros son los siguientes:
<ol>
	<li>un fichero que contenga una geometría en formato “Well-Known Text” (se pide un fichero ya que una geometría completa nos ocuparía toda la pantalla).</li>
	<li>el alto y ancho de cada “celda”.</li>
	<li>el tipo de forma deseada (rectángulo, triángulo o hexágono).</li>
	<li>el ángulo de rotación de la teselación y,<span style="text-decoration: underline;"> en caso de tratarse de hexágonos</span>,</li>
	<li>un radio, que es una medida que define el estiramiento de los polígonos (no creo que sea la manera habitual de crear estas geometrías, pero da mucha flexibilidad).</li>
</ol>
Finalmente, el programa calcula la teselación y pregunta si quereis crear otra teselación más (no creo que resulte adictivo... :) El resultado que genera el software es una WKT GeometryCollection, que podemos visualizar en distintos SIG de escritorio (yo utilizo OpenJump).

Una vez definidos los parámetros de entrada, hay algunas cuestiones que pueden resultar interesantes sobre el programa.

El código se divide en dos clases (más la del interfaz de usuario, claro). Existe una clase (1) <em>MyGeometricShapeFactory</em> que calcula triángulos y hexágonos a partir de los parámetros de entrada. Esta clase hereda de la <em>GeometricShapeFactory</em> de NTS, que se ocupa de crear los rectángulos. Por otro lado, existe otra clase (<em>GraticuleBuilder</em>; 2) que se ocupa de la repetición de figuras (haciendo servir la clase 1). Esta clase (2) permite crear un grid mucho mayor que la zona de estudio y filtrarlo con operadores espaciales para obtener otro “grid ajustado” a la zona de estudio.

En la siguiente figura se puede ver como se calcula la zona que debe cubrir el grid mayor. Por supuesto, habrá otros métodos de definir el área mayor, pero esta es bastante sencilla de entender y programar con NTS... :)

<img class="aligncenter size-full wp-image-2205" title="image07" src="http://www.gisandchips.org/wp-content//image07.png" alt="" width="508" height="380" />
<h2>Algunos ejemplos utilizando esto:</h2>
Para que os hagáis una idea de lo que permite el software he preparado algunos ejemplos interesantes:
<ul>
	<li>Ancho de celda: 5000 m, Alto de celda: 5000 m, Forma: rectangle, Ángulo: 0 y Radio: 0 (<strong>Completo</strong>).</li>
</ul>
<img class="aligncenter size-medium wp-image-2206" title="image00" src="http://www.gisandchips.org/wp-content//image00-300x298.png" alt="" width="300" height="298" />
<ul>
	<li>Ancho de celda: 2500 m, Alto de celda: 2500 m, Forma: rectangle, Ángulo: 0 y Radio: 0 (<strong>Ajustado</strong>).</li>
</ul>
<img class="aligncenter size-medium wp-image-2207" title="image11" src="http://www.gisandchips.org/wp-content//image11-300x241.png" alt="" width="300" height="241" />
<ul>
	<li>Ancho de celda: 5000 m, Alto de celda: 1000 m, Forma: rectangle, Ángulo: 0 y Radio: 0 (<strong>Ajustado</strong>).</li>
</ul>
<img class="aligncenter size-medium wp-image-2212" title="Captura de pantalla de 2012-11-20 13:55:05" src="http://www.gisandchips.org/wp-content//Captura-de-pantalla-de-2012-11-20-135505-300x229.png" alt="" width="300" height="229" />
<ul>
	<li>Ancho de celda: 5000 m, Alto de celda: 2500 m, Forma: triangle, Ángulo: 45 y Radio: 0 (<strong>Ajustado</strong>).</li>
</ul>
<img class="aligncenter size-medium wp-image-2208" title="image03" src="http://www.gisandchips.org/wp-content//image03-300x232.png" alt="" width="300" height="232" />
<ul>
	<li>Ancho de celda: 5000 m, Alto de celda: 5000 m, Forma: hexagon, Ángulo: 45 y Radio: 1000 m (<strong>Ajustado</strong>).</li>
</ul>
<img class="aligncenter size-medium wp-image-2209" title="image08" src="http://www.gisandchips.org/wp-content//image08-300x256.png" alt="" width="300" height="256" />
<ul>
	<li>Ancho de celda: 5000 m, Alto de celda: 5000 m, Forma: hexagon, Ángulo: 0 y Radio: 2000 m (<strong>Ajustado</strong>).</li>
</ul>
<img class="aligncenter size-medium wp-image-2210" title="image10" src="http://www.gisandchips.org/wp-content//image10-300x250.png" alt="" width="300" height="250" />
<h2></h2>
<h2>Código comentado:</h2>
A continuación os adjunto el código que hace todo esto posible. No incluyo lo referente al UI, pero comprobareis que tampoco hay que hacer nada del otro mundo para utilizarlo (new GraticuleBuilder). Además, os tendréis que crear un enumerado con las formas (rectangle, triangle y hexagon).

Primero tenéis la clase que crea la tesela:

[csharp]
//GraticuleBuilder class for creating tessellations.
//Copyright (C) 2012 Benito M. Zaragozi
//Authors: Benito M. Zaragozi­ (www.gisandchips.org)
//Send comments and suggestions to benito.zaragozi@ua.es

//This program is free software: you can redistribute it and/or modify
//it under the terms of the GNU General Public License as published by
//the Free Software Foundation, either version 3 of the License, or
//(at your option) any later version.
//
//This program is distributed in the hope that it will be useful,
//but WITHOUT ANY WARRANTY; without even the implied warranty of
//MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
//GNU General Public License for more details.
//
//You should have received a copy of the GNU General Public License
//along with this program.  If not, see &lt;http://www.gnu.org/licenses/&gt;.

using System;
using System.Collections;
using System.Collections.Generic;

using NetTopologySuite.IO;
using NetTopologySuite.Geometries;
using NetTopologySuite.Utilities;
using GeoAPI.Geometries;

namespace RepeatShapes
{
	public class GraticuleBuilder
	{
		public GraticuleBuilder (IGeometry studyArea,int cellWidth, int cellHeight, ShapeType shp, int angle, int radius)
		{
			_studyArea=studyArea;
			GetBigArea (_studyArea,cellWidth,cellHeight);
			_origin=GetOrigin (_bigArea);

			if(shp== ShapeType.rectangle)
			{
				_bigGrid = rectRepeat(cellWidth,cellHeight,_bigGridRows,_bigGridCols);
			}
			else if(shp==ShapeType.triangle)
			{
				_bigGrid = triRepeat(cellWidth,cellHeight,_bigGridRows,_bigGridCols);
			}
			else if(shp==ShapeType.hexagon)
			{
				_bigGrid = hexRepeat(cellWidth,cellHeight,_bigGridRows,_bigGridCols, radius);
			}

			_bigGrid = rotateGrid (_bigGrid, _studyArea, angle);
			_filteredGrid=gridFilter(_bigGrid,_studyArea);
		}

		IGeometry _studyArea;
		IGeometry _bigArea;
		IPoint _origin;
		GeometryCollection _bigGrid;
		GeometryCollection _filteredGrid;
		int _bigGridRows=0;
		int _bigGridCols=0;

		public IGeometry StudyArea {
			get { return _studyArea; }
		}

		public GeometryCollection BigGrid
		{
			get{return _bigGrid;}
		}

		public GeometryCollection FilteredGrid
		{
			get{return _filteredGrid;}
		}

		/// &lt;summary&gt;
		/// Calculates the left-bottom coordinate/point of a geometry.
		/// &lt;/summary&gt;
		/// &lt;returns&gt;
		/// The origin point for the grid.
		/// &lt;/returns&gt;
		/// &lt;param name='bigAreaEnvelope'&gt;
		/// A big area that encloses the study area.
		/// &lt;/param&gt;
		private Point GetOrigin(IGeometry bigAreaEnvelope)
		{
			double minX=0;
			double minY=0;

			foreach(Coordinate c in bigAreaEnvelope.Coordinates)
			{
				minX = c.X;
				minY = c.Y;

				if(c.X &lt; minX)
				{
					minX=c.X;
				}
				if(c.Y&lt;minY)
				{
					minY=c.Y;
				}
			}

			Point p = new Point(minX,minY);
			return p;
		}

		/// &lt;summary&gt;
		/// Create a big rectangle containing the study area.
		/// &lt;/summary&gt;
		/// &lt;param name=&quot;studyArea&quot;&gt;&lt;/param&gt;
		/// &lt;param name=&quot;widthDivBy&quot;&gt;&lt;/param&gt;
		/// &lt;param name=&quot;heightDivBy&quot;&gt;&lt;/param&gt;
		private void GetBigArea(IGeometry studyArea, int widthDivBy, int heightDivBy)
		{
			//circumscribed circle for obtaining its envelope and create the bigArea.
			NetTopologySuite.Algorithm.MinimumBoundingCircle mbc =
				new NetTopologySuite.Algorithm.MinimumBoundingCircle(studyArea);

			_bigArea = mbc.GetCircle ().Envelope.Boundary;
			double diameter = mbc.GetRadius ()*4;

			//get the total num of cols and rows
			_bigGridCols=(int)diameter/widthDivBy;
			_bigGridRows=(int)diameter/heightDivBy;
		}

		/// &lt;summary&gt;
		/// Rotates the bigGrid.
		/// &lt;/summary&gt;
		/// &lt;returns&gt;
		/// The rotated grid.
		/// &lt;/returns&gt;
		/// &lt;param name='bigGrid'&gt;
		/// The grid that overlaps the bigArea.
		/// &lt;/param&gt;
		/// &lt;param name='studyArea'&gt;
		/// The region of interest.
		/// &lt;/param&gt;
		/// &lt;param name='degree'&gt;
		/// A rotation angle in decimal degrees.
		/// &lt;/param&gt;
		private GeometryCollection rotateGrid(GeometryCollection bigGrid, IGeometry studyArea, int degree)
		{
			NetTopologySuite.Geometries.Utilities.AffineTransformation trans =
				NetTopologySuite.Geometries.Utilities.AffineTransformation.RotationInstance(
					Degrees.ToRadians (degree),studyArea.Centroid.X, studyArea.Centroid.Y);
			return (GeometryCollection)trans.Transform (bigGrid);
		}

		/// &lt;summary&gt;
		/// Filter the bigGrid for achieving a grid adjusted to the region of interest.
		/// &lt;/summary&gt;
		/// &lt;returns&gt;
		/// A grid adjusted to the region of interest.
		/// &lt;/returns&gt;
		/// &lt;param name='bigGrid'&gt;
		/// A grid that exceeds the region of interest.
		/// &lt;/param&gt;
		/// &lt;param name='studyArea'&gt;
		/// The region of interest.
		/// &lt;/param&gt;
		private GeometryCollection gridFilter(GeometryCollection bigGrid, IGeometry studyArea)
		{
			List&lt;IGeometry&gt;filteredGrid=new List&lt;IGeometry&gt;();

			foreach(IPolygon p in bigGrid.Geometries)
			{
				if(p.Intersects(studyArea)==true)
				{
					filteredGrid.Add(p);
				}
			}
			return new GeometryCollection(filteredGrid.ToArray ());
		}

		/// &lt;summary&gt;
		/// Creates a rectangular tessellation.
		/// &lt;/summary&gt;
		/// &lt;returns&gt;
		/// A rectangular grid.
		/// &lt;/returns&gt;
		/// &lt;param name='cellWidth'&gt;
		/// Cell width.
		/// &lt;/param&gt;
		/// &lt;param name='cellHeight'&gt;
		/// Cell height.
		/// &lt;/param&gt;
		/// &lt;param name='numRows'&gt;
		/// Number of rows.
		/// &lt;/param&gt;
		/// &lt;param name='numColumns'&gt;
		/// Number of columns.
		/// &lt;/param&gt;
		private GeometryCollection rectRepeat(int cellWidth, int cellHeight, int numRows, int numColumns)
		{
			List&lt;IGeometry&gt; bigGrid = new List&lt;IGeometry&gt;();
			double x = _origin.X;
			double y = _origin.Y;

			GeometricShapeFactory gsf = new GeometricShapeFactory();
			gsf.Height=Convert.ToDouble(cellHeight);
			gsf.Width=Convert.ToDouble(cellWidth);
			gsf.NumPoints=4;

			for (int i = 1; i &lt;= numColumns; i++)
			{
				for (int j = 1; j &lt;= numRows; j++)
				{
					gsf.Base=new Coordinate(x, y);
					IPolygon newpol = gsf.CreateRectangle();

					newpol.UserData=i+&quot; - &quot;+j;
					bigGrid.Add(newpol);
					y += cellHeight;
				}
				if(i!=0)
				{
					x += cellWidth;
				}
				y = _origin.Y;
			}
			return new GeometryCollection(bigGrid.ToArray ());
		}

		/// &lt;summary&gt;
		/// Creates a triangular tessellation.
		/// &lt;/summary&gt;
		/// &lt;returns&gt;
		/// A triangular tessellation.
		/// &lt;/returns&gt;
		/// &lt;param name='cellWidth'&gt;
		/// Cell width.
		/// &lt;/param&gt;
		/// &lt;param name='cellHeight'&gt;
		/// Cell height.
		/// &lt;/param&gt;
		/// &lt;param name='numRows'&gt;
		/// Number of rows.
		/// &lt;/param&gt;
		/// &lt;param name='numColumns'&gt;
		/// Number of columns.
		/// &lt;/param&gt;
		private GeometryCollection triRepeat(int cellWidth, int cellHeight, int numRows, int numColumns)
		{
			List&lt;IGeometry&gt; bigGrid = new List&lt;IGeometry&gt;();
			double x = _origin.X-cellWidth;
			double y = _origin.Y;

			MyGeometricShapeFactory gsf = new MyGeometricShapeFactory();
			gsf.Height=cellHeight;
			gsf.Width=cellWidth;
			gsf.NumPoints=4;

			for (int i = 1; i &lt;= numColumns*2; i++)
			{
				for (int j = 1; j &lt;= numRows; j++)
				{
					if (i % 2 == 0)
					{
						gsf.Centre=new Coordinate(x, y);
						gsf.Envelope=new Envelope(x,x+cellWidth,y,y+cellHeight);
						IPolygon newpol = gsf.CreateTriangle();

						newpol.UserData=i+&quot; - &quot;+j;
						bigGrid.Add(newpol);
					}
					else
					{
						gsf.Centre=new Coordinate(x, y);
						gsf.Envelope=new Envelope(x+cellWidth/2,x+cellWidth/2+cellWidth,y,y+cellHeight);
						IPolygon newpol = gsf.CreateInvertedTriangle();

						newpol.UserData=i+&quot; - &quot;+j;
						bigGrid.Add(newpol);
					}
					y += cellHeight;
				}
				if(i % 2!=0)
				{
					x += cellWidth;
				}
				y = _origin.Y;
			}
			return new GeometryCollection(bigGrid.ToArray ());

		}

		/// &lt;summary&gt;
		/// Creates an hexagonal tessellation.
		/// &lt;/summary&gt;
		/// &lt;returns&gt;
		/// The hexagonal tessellation
		/// &lt;/returns&gt;
		/// &lt;param name='cellWidth'&gt;
		/// Cell width.
		/// &lt;/param&gt;
		/// &lt;param name='cellHeight'&gt;
		/// Cell height.
		/// &lt;/param&gt;
		/// &lt;param name='numRows'&gt;
		/// Number of rows.
		/// &lt;/param&gt;
		/// &lt;param name='numColumns'&gt;
		/// Number of columns.
		/// &lt;/param&gt;
		/// &lt;param name='radius'&gt;
		/// A proportion that defines the longitude of the hexagon's base.
		/// A larger radius is relate with a narrower base of the hexagon.
		/// &lt;/param&gt;
		private GeometryCollection hexRepeat(int cellWidth, int cellHeight, int numRows, int numColumns, int radius)
		{
			List&lt;IGeometry&gt; bigGrid = new List&lt;IGeometry&gt;();
			double x = _origin.X-cellWidth;
			double y = _origin.Y;

			MyGeometricShapeFactory gsf = new MyGeometricShapeFactory();
			gsf.Height=cellHeight;
			gsf.Width=cellWidth;
			gsf.NumPoints=4;

			for (int i = 1; i &lt;= numColumns*2; i++)
			{
				for (int j = 1; j &lt;= numRows; j++)
				{
					if (i % 2 == 0)
					{
						gsf.Centre=new Coordinate(x, y);
						gsf.Envelope=new Envelope(x,x+cellWidth,y,y+cellHeight);
						IPolygon newpol = gsf.CreateHexagon(radius);

						newpol.UserData=i+&quot; - &quot;+j;
						bigGrid.Add(newpol);
					}
					else
					{
						gsf.Centre=new Coordinate(x, y);
						gsf.Envelope=new Envelope(x+(cellWidth-radius),x+cellWidth+(cellWidth-radius),y+(cellHeight/2),y+cellHeight+(cellHeight/2));
						IPolygon newpol = gsf.CreateHexagon(radius);

						newpol.UserData=i+&quot; - &quot;+j;
						bigGrid.Add(newpol);
					}
					y += cellHeight;
				}
				if(i % 2!=0)
				{
					x += cellWidth+(cellWidth-2*radius);
				}
				y = _origin.Y;
			}
			return new GeometryCollection(bigGrid.ToArray ());
		}
	}
}
[/csharp]

Y ahora la clase que crea las geometrías simples:

[csharp]
//MyGeometricShapeFactory class for creating tessellations.
//Copyright (C) 2012 Benito M. Zaragozi­
//Authors: Benito M. Zaragozi­ (www.gisandchips.org)
//Send comments and suggestions to benito.zaragozi@ua.es

//This program is free software: you can redistribute it and/or modify
//it under the terms of the GNU General Public License as published by
//the Free Software Foundation, either version 3 of the License, or
//(at your option) any later version.
//
//This program is distributed in the hope that it will be useful,
//but WITHOUT ANY WARRANTY; without even the implied warranty of
//MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
//GNU General Public License for more details.
//
//You should have received a copy of the GNU General Public License
//along with this program.  If not, see &lt;http://www.gnu.org/licenses/&gt;.

using System;

using NetTopologySuite.Utilities;
using GeoAPI.Geometries;
using NetTopologySuite.Geometries;

namespace RepeatShapes
{
	/// &lt;summary&gt;
	/// This class inherites from NTS's GeometricShapeFactory.
	/// See that rectangles are created using the parent class.
	/// &lt;/summary&gt;
	public class MyGeometricShapeFactory:GeometricShapeFactory
	{
		public MyGeometricShapeFactory ()
		{
		}

		private readonly Dimensions _dim = new Dimensions();
        private int _nPts = 100;

		/// &lt;summary&gt;
		/// Creates a triangular polygon.
		/// &lt;/summary&gt;
		/// &lt;returns&gt;
		/// The triangle.
		/// &lt;/returns&gt;
        public IPolygon CreateTriangle()
        {
            int i;
            int ipt = 0;
            int nSide = _nPts / 100;
            if (nSide &lt; 1) nSide = 1;
            double XsegLen = this.Envelope.Width/nSide;
            double YsegLen = this.Envelope.Width/nSide;

            Coordinate[] pts = new Coordinate[3 * nSide + 1];
            Envelope env = (Envelope)this.Envelope;

            for (i = 0; i &lt; nSide; i++)
            {
                double x = env.MinX + i * XsegLen;
                double y = env.MinY;
                pts[ipt++] = (Coordinate)CreateCoord(x, y);
            }
            for (i = 0; i &lt; nSide; i++)
            {
                double x = env.MaxX;
                double y = env.MinY + i * YsegLen;
                pts[ipt++] = (Coordinate)CreateCoord(x, y);
            }
            for (i = 0; i &lt; nSide; i++)
            {
            	double x = env.MaxX - XsegLen/2;
                double y = env.MaxY;
                pts[ipt++] = (Coordinate)CreateCoord(x, y);
            }
//            for (i = 0; i &lt; nSide; i++)
//            {
//                double x = env.MinX;
//                double y = env.MaxY - i * YsegLen;
//                pts[ipt++] = (Coordinate)CreateCoord(x, y);
//            }
            pts[ipt++] = new Coordinate(pts[0]);

            ILinearRing ring = GeomFact.CreateLinearRing(pts);
            IPolygon poly = GeomFact.CreatePolygon(ring, null);
            return poly;
        }

		/// &lt;summary&gt;
		/// Creates an inverted triangle (up-down).
		/// &lt;/summary&gt;
		/// &lt;returns&gt;
		/// The inverted triangle.
		/// &lt;/returns&gt;
        public IPolygon CreateInvertedTriangle()
        {
            int i;
            int ipt = 0;
            int nSide = _nPts / 100;
            if (nSide &lt; 1) nSide = 1;
            double XsegLen = this.Envelope.Width/nSide;
            double YsegLen = this.Envelope.Width/nSide;

            Coordinate[] pts = new Coordinate[3 * nSide + 1];
            Envelope env = (Envelope)this.Envelope;

            for (i = 0; i &lt; nSide; i++)
            {
                double x = env.MinX + i * XsegLen;
                double y = env.MaxY;
                pts[ipt++] = (Coordinate)CreateCoord(x, y);
            }
            for (i = 0; i &lt; nSide; i++)
            {
            	double x = env.MinX + XsegLen/2;
                double y = env.MinY;
                pts[ipt++] = (Coordinate)CreateCoord(x, y);
            }
            for (i = 0; i &lt; nSide; i++)
            {
                double x = env.MaxX;
                double y = env.MaxY + i * YsegLen;
                pts[ipt++] = (Coordinate)CreateCoord(x, y);
            }

            pts[ipt++] = new Coordinate(pts[0]);

            ILinearRing ring = GeomFact.CreateLinearRing(pts);
            IPolygon poly = GeomFact.CreatePolygon(ring, null);
            return poly;
        }

        /// &lt;summary&gt;
        /// Creates an hexagon adjusted by a radius parameter.
        /// &lt;/summary&gt;
        /// &lt;returns&gt;
        /// The hexagon.
        /// &lt;/returns&gt;
        /// &lt;param name='radius'&gt;
        /// A proportion that defines the longitude of the hexagon's base.
		/// A larger radius is relate with a narrower base of the hexagon.
        /// &lt;/param&gt;
        public IPolygon CreateHexagon(int radius)
        {
            int i;
            int ipt = 0;
            int nSide = _nPts / 100;
            if (nSide &lt; 1) nSide = 1;
            double XsegLen = this.Envelope.Width/nSide;
            double YsegLen = this.Envelope.Height/nSide;

            Coordinate[] pts = new Coordinate[6 * nSide + 1];
            Envelope env = (Envelope)this.Envelope;

             for (i = 0; i &lt; nSide; i++)
            {
                double x = env.MinX + radius + i * XsegLen;
                double y = env.MinY;
                pts[ipt++] = (Coordinate)CreateCoord(x, y);
            }
            for (i = 0; i &lt; nSide; i++)
            {
                double x = env.MaxX - radius;
                double y = env.MinY + i * YsegLen;
                pts[ipt++] = (Coordinate)CreateCoord(x, y);
            }
            for (i = 0; i &lt; nSide; i++)
            {
                double x = env.MaxX;
                double y = env.MinY+(env.Height/2) + i * YsegLen;
                pts[ipt++] = (Coordinate)CreateCoord(x, y);
            }
            for (i = 0; i &lt; nSide; i++)
            {
                double x = env.MaxX - radius;
                double y = env.MaxY + i * YsegLen;
                pts[ipt++] = (Coordinate)CreateCoord(x, y);
            }
            for (i = 0; i &lt; nSide; i++)
            {
                double x = env.MinX + radius + i * XsegLen;
                double y = env.MaxY;
                pts[ipt++] = (Coordinate)CreateCoord(x, y);
            }
            for (i = 0; i &lt; nSide; i++)
            {
                double x = env.MinX;
                double y = env.MinY+(env.Height/2) + i * YsegLen;
                pts[ipt++] = (Coordinate)CreateCoord(x, y);
            }

            pts[ipt++] = new Coordinate(pts[0]);

            ILinearRing ring = GeomFact.CreateLinearRing(pts);
            IPolygon poly = GeomFact.CreatePolygon(ring, null);
            return poly;
        }
	}
}
[/csharp]