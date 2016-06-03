---
ID: 1847
post_title: Business card de GISchips (en LaTeX)
author: benizar
post_date: 2011-11-06 20:41:06
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2011/11/06/business-card-de-gischips-en-latex-3/
published: true
---
<p>La semana pasada estaba previsto que varios miembros de GIS&amp;Chips presentáramos un taller en una Jornada de Geografía 3.0. Además, algunos asistimos al posterior congreso nacional de geografía. Finalmente, no se pudo realizar el taller por falta de horarios, pero esperamos poder organizar algún evento propio solamente para la difusión de conocimientos de TIG libre.</p>
<p>En estos eventos es habitual el intercambio de tarjetas de presentación y como no teníamos una para darnos a conocer decidimos hacer una prueba en LaTeX, que no salió del todo mal. En este post compartimos esta primera versión para que la podáis personalizar o simplemente ver como se ha hecho.</p>
<p>[caption id="attachment_1825" align="aligncenter" width="305" caption="Cara tarjeta G&amp;C, generado con el fichero &quot;G&amp;C_businessCard_front.tex&quot;."]<img class="size-full wp-image-1825" src="http://www.gisandchips.org/wp-content//GC_businessCard_front1.jpg" alt="" width="305" height="152" />[/caption]</p>
<p style="text-align: center">
<p><!--more--></p>
<h3>Características a considerar en el diseño</h3>
<p>Hace mucho tiempo di un par de cursos para el diseño de marca. Ilustrator, Freehand y Photoshop para el diseño de tarjetas, trípticos, carteles, carátulas de cds, etc. Hay que reconocer que todo esto es una profesión en sí y hay muchos detalles por conocer. En tal caso, lo que hacemos nosotros aquí es un “apaño”, por lo que nadie espere grandes cosas :-)</p>
<p>Las características más a tener en cuenta en el diseño son el tamaño de la tarjeta, el logo, el fondo y la gama de colores. Otra cuestión a tener en cuenta es la problemática vector-raster.</p>
<p>El <strong>tamaño</strong> de la tarjeta se puede determinar por una serie de estándares, supongo que según el tamaño de los billetes de cada país :-) (la cartera donde guardamos las tarjetas también depende de esto). Nuestra tarjeta (90x45mm) no se ajusta a ninguno de estos estándares aunque la podríamos adaptar muy fácilmente. En <a title="http://en.wikipedia.org/wiki/Business_card" href="http://en.wikipedia.org/wiki/Business_card" target="_blank">http://en.wikipedia.org/wiki/Business_card</a> encontramos los tamaños más utilizados.</p>
<p>El <strong>logo</strong> es el  que figura en el blog, diseñado por Miguel. No obstante, eliminamos el texto pues lo incluiremos entre los otros elementos de la tarjeta.</p>
<p>El <strong>fondo de la cara frontal</strong> es una nube de tags basada en la de esta página, después en Gimp le hemos añadido un degradado lineal de negro a blanco. El<strong> fondo del reverso</strong> es un mapa de <a href="http://www.openstreetmap.org/">http://www.openstreetmap.org/</a> que indica la ubicación de la sede. Para completar la ubicación se podría añadir la dirección en texto.</p>
<p>El nombre de la asociación, el lema y los datos de contacto (web y Email) aparecen en la cara frontal. El contraste blanco/negro es uno de los más recurridos y elegantes aunque esto dependerá del gusto de cada uno, además son los colores que aparecen en el blog.</p>
<p>Finalmente, la problemática raster-vector hace que no todos los programas sean igual de prácticos al combinar imágenes con texto en un marco tan pequeño. El texto podría verse mal al rasterizarse en Gimp o las imágenes verse mal por no estar bien escaladas al tamaño del papel. Para evitar problemas es recomendable preparar todos los componentes del diseño en un marco de 90x45mm. Además, conviene que el texto sea siempre vectorial.</p>
<p>[caption id="attachment_1828" align="aligncenter" width="331" caption="Reverso de la tarjeta G&amp;C, generado con el fichero &quot;G&amp;C_businessCard_back.tex&quot;."]<img class="size-full wp-image-1828 " src="http://www.gisandchips.org/wp-content//GC_businessCard_back.jpg" alt="" width="331" height="165" />[/caption]</p>
<h3>¿Porqué LaTeX?</h3>
<p>Seguramente, podríamos haber creado la tarjeta solamente con Gimp, Impress, Inkscape o cualquier otro software comercial… incluso en un editor de textos cualquiera. O con una máquina de escribir… No obstante, para crear la tarjeta hemos utilizado Gimp y LaTeX.</p>
<p>Básicamente, prefiero esta opción porqué es FOSS y estándar. No necesitamos conocer demasiados botones y LaTeX lleva una larga trayectoria de más de 20 años que hace pensar que la plantilla que creemos seguirá siendo funcional en el futuro. Además, ya utilizo LaTeX para hacer pósters, artículos, presentaciones, materiales docentes, el currículum y mi tesis. De este modo, la pregunta sería ¿porqué aprender otros programas?</p>
<p>Desgraciadamente, no puedo aquí hacer un tutorial de LaTeX para geógrafos pero, si necesitáis escribir un documento estructurado de grandes dimensiones (tesis) o muchos de otro tipo (informes) os animo a que le echéis un vistazo a este proyecto tan interesante pues a medio plazo os ahorrará muchos dolores de cabeza.</p>
<h3>Como está hecha</h3>
<p>Las plantillas que he encontrado en la web (buscando “Business card; LaTeX”) son más bien sosas con un estilo próximo al de la máquina de escribir y además son bastante complejas. Para evitar estos problemas hemos utilizado la clase “Beamer” de LaTeX, que permite entre otras cosas añadir una imagen de fondo y formatear el espacio de la tarjeta apropiadamente.</p>
<p>Primero, hemos utilizado Gimp para crear los fondos de la tarjeta como dos ficheros PNG. A continuación, LaTeX se ha utilizado para la composición, añadir textos y dibujos vectoriales (líneas). En el reverso de la tarjeta hemos añadido un mapa de <a href="http://www.openstreetmap.org/">http://www.openstreetmap.org/</a> para aportar una localización más detallada, aunque también se podría añadir la dirección postal.</p>
<p>La plantilla se compone de tres ficheros TeX, uno para cada cara de la tarjeta y, tras compilar las dos caras, el tercero para crear una composición de 12 tarjetas a dos caras.</p>
<p><img class="size-full wp-image-1830 alignnone" src="http://www.gisandchips.org/wp-content//GC_businessCard_x12_Página_1.jpg" alt="" width="214" height="303" /><img class="alignnone size-full wp-image-1831" src="http://www.gisandchips.org/wp-content//GC_businessCard_x12_Página_2.jpg" alt="" width="214" height="303" /></p>
<p>Esta plantilla es altamente mejorable, es una versión 0.1 para salir del paso, pero os puede dar ideas. La idea de aprovechar OSM para la localización (idea de José) es interesante y evita tener que pensar en hacer un croquis como he visto en algunas tarjetas. Además, creo que los detalles de la tarjeta (la nube de tags como fondo, el mapa…) además de quedar bastante bien aportan información extra sobre nosotros si se saben leer.</p>
<h3>Código en latex</h3>
<p>Finalmente, os paso el código de los tres ficheros de LaTeX necesarios para crear esta tarjeta. Si leéis el código veréis donde podéis introducir vuestras propias imágenes de fondo o el logo que queráis. Espero que os sea de ayuda y os genere otras ideas.</p>
<p><strong>G&C_businessCard_front.tex</strong></p>
<p>[tex]
% GIS&amp;Chips nice business card
% By Benito M. Zaragozí
% Version 0.1 released 02/11/2011
% Further releases in: www.gisandchips.org
% This template is composed by three .tex files (businessCard.tex, businessCard_back.tex and businessCardx10.tex) for preparing a basic layout.

\documentclass{beamer}
\usepackage[utf8x]{inputenc}
\usepackage[spanish]{babel}
\usepackage{hyperref}
\usepackage[absolute,showboxes,verbose,overlay]{textpos}
\usepackage{geometry}
\geometry{paperwidth=90mm, paperheight=45mm, layoutwidth=90mm, layoutheight=45mm, left=0mm, top=0mm, right=0mm, bottom=0mm}

\usetheme{default}

\setbeamertemplate{navigation symbols}{}
\setbeamertemplate{background canvas}{\includegraphics[height=\paperheight]{background.png}}

\begin{document}

\frame{
\begin{columns}
	\begin{column}{35mm}
		\includegraphics[width=44mm]{gischips_logo.png}
	\end{column}

	\begin{column}{40mm}
		\color{white}{Asociación\\ \vspace{1mm} \huge{GIS \&amp;amp; Chips}}
		\normalsize
		\vspace{1mm}
		\line(1,0){80}
		\vspace{-1mm}
		\color{white}{Geografía útil para llevar}
	\end{column}
\end{columns}

\vspace{-1mm}
\normalsize
\color{black}{\hspace{5mm}\line(1,0){220}\\ \hspace{2mm} \url{www.gisandchips.org} \hspace{3mm} \url{info@gisandchips.org}}
}
\end{document}
[/tex]</p>
<p><strong>G&C_businessCard_back.tex</strong></p>
<p><strong></strong>[tex]
% GIS&amp;Chips nice business card
% By Benito M. Zaragozí
% Version 0.1 released 02/11/2011
% Further releases in: www.gisandchips.org
% This template is composed by three .tex files (businessCard.tex, businessCard_back.tex and businessCardx10.tex) for preparing a basic layout.

\documentclass{beamer}
\usepackage[utf8x]{inputenc}
\usepackage[spanish]{babel}
\usepackage{geometry}
\geometry{margin=2mm, paperwidth=90mm, paperheight=45mm}

\usetheme{default}
\setbeamertemplate{navigation symbols}{}
\setbeamertemplate{background canvas}{\hspace{2mm} \includegraphics[height=41.5mm]{mapbackground.png}}

\begin{document}
	\frame{
		\vspace{-1.5mm}
		\colorbox{white}{GIS\&amp;amp;Chips en la Universidad de Alicante}\\
		\vspace{31mm}
		\begin{flushright}
		\colorbox{white}{\tiny {\url{www.openstreetmap.org}}}
		\end{flushright}
	}
\end{document}
[/tex]</p>
<p><strong>G&C_businessCard_x12.tex</strong></p>
<p><strong></strong>[tex]
% GIS&amp;Chips nice business card
% By Benito M. Zaragozí
% Version 0.1 released 02/11/2011
% Further releases in: www.gisandchips.org
% This template is composed by three .tex files (businessCard.tex, businessCard_back.tex and businessCardx10.tex) for preparing a basic layout.

\documentclass[10pt,a4paper]{minimal}
\usepackage{graphicx}
\usepackage[margin=1cm]{geometry}

\begin{document}
\thispagestyle{empty}
\noindent

\begin{center}
\hspace{1mm}\includegraphics[scale=1]{businessCard.pdf} \includegraphics[scale=1]{businessCard.pdf} \vspace{1mm}

\hspace{1mm}\includegraphics[scale=1]{businessCard.pdf} \includegraphics[scale=1]{businessCard.pdf} \vspace{1mm}

\hspace{1mm}\includegraphics[scale=1]{businessCard.pdf} \includegraphics[scale=1]{businessCard.pdf} \vspace{1mm}

\hspace{1mm}\includegraphics[scale=1]{businessCard.pdf} \includegraphics[scale=1]{businessCard.pdf} \vspace{1mm}

\hspace{1mm}\includegraphics[scale=1]{businessCard.pdf} \includegraphics[scale=1]{businessCard.pdf} \vspace{1mm}

\hspace{1mm}\includegraphics[scale=1]{businessCard.pdf} \includegraphics[scale=1]{businessCard.pdf} \vspace{1mm}

\pagebreak
%%%%%%Back with map
\hspace{1mm}\includegraphics[scale=1]{businessCard_back.pdf} \includegraphics[scale=1]{businessCard_back.pdf} \vspace{1mm}

\hspace{1mm}\includegraphics[scale=1]{businessCard_back.pdf} \includegraphics[scale=1]{businessCard_back.pdf} \vspace{1mm}

\hspace{1mm}\includegraphics[scale=1]{businessCard_back.pdf} \includegraphics[scale=1]{businessCard_back.pdf} \vspace{1mm}

\hspace{1mm}\includegraphics[scale=1]{businessCard_back.pdf} \includegraphics[scale=1]{businessCard_back.pdf} \vspace{1mm}

\hspace{1mm}\includegraphics[scale=1]{businessCard_back.pdf} \includegraphics[scale=1]{businessCard_back.pdf} \vspace{1mm}

\hspace{1mm}\includegraphics[scale=1]{businessCard_back.pdf} \includegraphics[scale=1]{businessCard_back.pdf} \vspace{1mm}

\end{center}
\pagebreak
\end{document}
[/tex]</p>