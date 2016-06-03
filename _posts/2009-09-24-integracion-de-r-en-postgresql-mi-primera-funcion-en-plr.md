---
ID: 165
post_title: >
  Integración de R en PostgreSQL. Mi
  primera función en pl/R.
author: benizar
post_date: 2009-09-24 10:49:58
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2009/09/24/integracion-de-r-en-postgresql-mi-primera-funcion-en-plr/
published: true
syntaxhighlighter_encoded:
  - "1"
---
En primer lugar, y para que ninguno de vosotros tenga impresión que no voy a hablar nunca de GIS, me gustaría tranquilizaros, y recordaros que en próximos artículos, yo o a quien le apetezca participar en este hilo, pasaremos a comentar usos de todo esto que estamos viendo orientado al tratamiento de la información espacial. Si hay alguien que quiera compartir sus experiencias con temas de R será bien recibido.

Este segundo artículo sobre tecnologías útiles para explotar toda la potencia de R se centra en el uso de R desde el PostgreSQL.

Como todos sabéis, tradicionalmente los Sistemas Gestores de Bases de Datos (SGBD), como PostgreSQL, Oracle, MySQL o Access han sido diseñados para guardar y gestionar gran cantidad de datos, por ello las posibilidades de análisis y salidas gráficas se han delegado mayormente en otros softwares como hojas de cálculo, paquetes estadísticos como SPSS, R o en nuestro caso en softwares SIG... pero eso se acabó  :-)  .

pl/R es un lenguaje de programación procedural que permite contar con toda la potencia de cálculo y salidas gráficas del paquete estadístico R, así como de su lenguaje de programación desde dentro de funciones de PostgreSQL.

<!--more-->

Los <span style="text-decoration: underline">requisitos básicos</span> para comenzar a trabajar con pl/R son, como no, tener instalado PostgreSQL (he trabajado sin demasiados problemas con las versiones 8.2 y 8.3), tener pl/R instalado, y crear una base de datos en PostgreSQL donde añadimos el nuevo lenguaje procedural.

Todo lo que necesitamos para iniciarnos lo podemos encontrar en la página WEB del creador de pl/R, <strong>Joe Conway</strong> (<a href="http://www.joeconway.com/plr/">http://www.joeconway.com/plr/</a> ), donde nos descargaremos los ficheros para la instalación tanto para Windows como para Linux. También hay documentación para dar los primeros pasos y unos cuantos más. Esta presentación os ayudará bastante (<a href="http://www.joeconway.com/oscon-pres-2003-1.pdf">http://www.joeconway.com/oscon-pres-2003-1.pdf</a> ). Os recomiendo leer todo lo que podáis, incluso su Curriculum que está en la página principal no tiene desperdicio.

Hay mucha documentación en Internet y ejemplos de funciones básicas. Nosotros para trabajar con algo que ya conocemos, volveremos a utilizar la función hist() de R para obtener el histograma de cualquier campo numérico que podamos tener en una tabla de nuestra base de datos.

Cualquiera que haya trabajado bien con R, bien con funciones de PostgreSQL en general encontrará rápidamente los primeros problemas o desventajas, si es que se pueden llamar así, de usar pl/R. En cualquiera de los dos casos, normalmente tendremos que aprender a usar una nueva herramienta (PostgreSQL o R), lo cual es un nuevo mundo. Depurar las funciones de pl/R puede ser engorroso si se hace directamente, y normalmente los procesos serán más lentos, proporcionalmente, que si trabajásemos solamente en uno de los dos lados del espejo.  Pero ahora os diré las ventajas que puedo destacar más rápidamente:
<ul>
	<li>Podemos ejecutar nuestras nuevas funciones mediante una      sencilla expresión de SQL,</li>
	<li>las funciones estarán siempre junto con los datos      (acabando con la importación-exportación),</li>
	<li>tenemos la posibilidad de usar las funciones desde PHP      y</li>
	<li>un potente SGBD se encargará de gestionar el manejo de      los datos, lo cual es muy bueno cuando se trabaja con grandes volúmenes de      información.</li>
</ul>
Hay más ventajas, pero creo que para empezar no está nada mal.
<h3>Mi primera función en pl/R</h3>
Una vez lo tenemos todo instalado, os recomiendo empezar probando que todo funciona bien desde el GUI de R, ya que desde pl/R nos será más complicada la depuración.

La función que paso a detallar está pensada para usarla en local y que la probéis con vuestros propios datos (numéricos; si no lo son veréis un mensaje de error). Se trata de crear un histograma y guardarlo como imagen.

Como os podéis dar cuenta parece una función de pl/pgSQL solo que cambiamos el lenguaje al final y metemos directamente código de R entre $BODYs o comillas simples, aunque resulta más práctico el $ por si dentro de la función necesitamos usar comillas de dos tipos.

Los comentarios de la función pueden ir con # igual que en R.

A continuación tenemos el código para crear la nueva función:

[sql]

CREATE OR REPLACE FUNCTION _plr_hist(text, text)
RETURNS text AS
$BODY$
select = 'select '
campo = arg1
from = ' from '
tabla = arg2
selection = paste(select, campo, from, tabla, sep='');

## Ejecutamos la consulta y la almacenamos en el objeto &quot;sql&quot;;
sql &lt;- pg.spi.exec (selection);

## Lo que obtengamos a partir de aquí se imprimirá en un pdf...
dir='c:/temp/';
nam= arg1;
ext='.png';
newfile=paste(dir,nam,ext, sep='')
png(newfile);

hist(sql[,1], xlab = campo, ylab='frecuencias', main= campo, col='blue', ylim=c(0,20));

## Hasta aquí
dev.off();

## Si trabajamos en linux querremos hacer algo como lo siguiente
## para asignar privilegios de lectura a todos los usuarios de linux
##sys1='chmod go+r'
##sys2=paste(sys1, '', newfile)
##system(sys2)

## Esto devuelve un mensaje al terminar la función
print1='¡¡¡Ya está!!!, ¡¡¡mira que gráfico más chulo en'
print2=paste(print1, '', newfile, sep='')
print (print2);

$BODY$
LANGUAGE 'plr' VOLATILE

[/sql]

Dentro de la función hay que destacar el empleo de pg.spi.exec(),  para uso de los desarrolladores de PostgreSQL, que ejecuta una expresión dada.

Una vez creada nuestra función la podremos ejecutar de un modo muy sencillo. Fijaos que bueno que es esto  :-)   :

[sql]

select _plr_hist('un campo numérico', 'tabla');

[/sql]

Y obtenemos nuestra gráfica en la carpeta especificada.
<h3>Algunos comentarios sobre todo esto:</h3>
Viéndolo ya funcionar parece que nada “malo” pueda pasarnos. A este respecto, uno de los aspectos que más problemas me creó en su día es lo relacionado con los derechos del usuario Postgres en el SO, tanto en Linux como en Windows. La solución en linux viene dada y comentada en el código de más arriba. Con “chmod go+r” damos derechos de lectura a todos los usuarios. Lo que no me imaginaba nunca es que en Windows también iba a tener problemas. Resulta que el usuario que ejecuta el postmaster de Postgresql es “postgres”, que no dispone de derechos de escritura (no se si hay otros problemas) en “c:\”. Cuando se trata de crear una salida gráfica a c:\, el interprete de pl/R indica que no se puede iniciar el device(). Esto me llevó algún dolor de cabeza pues en Windows suelo hacer todas mis primeras pruebas directamente en c:\. En el ejemplo de antes trabajamos en un directorio creado a propósito. Sobre esto último podemos consultar en <a href="http://pgfoundry.org/pipermail/plr-general/2009-June/000264.html">http://pgfoundry.org/pipermail/plr-general/2009-June/000264.html</a> .
<h3>Referencias:</h3>
Solamente recordaros lo esencial de la página de Joe Conway (<a href="http://www.joeconway.com/plr/">http://www.joeconway.com/plr/</a> ). También os recomiendo entrar en <a href="http://www.bostongis.com/">http://www.bostongis.com/</a> donde hay un hilo de 3 artículos explicando distintos temas relacionados con pl/R. El primero es una introducción bastante detallada, el segundo es trata la creación de polígonos de Voronoi en PostgreSQL usando PostGIS y pl/R (muy recomendable, y avanzado), y el último es un ejemplo de uso de RGDAL.

-------------------------------------------------------------------

Si quereis contactar podeis enviarme un email (asunto: gisandchips):

Benito M. Zaragozí

benito.zaragozi@ua.es