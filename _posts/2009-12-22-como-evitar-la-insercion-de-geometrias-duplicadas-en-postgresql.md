---
ID: 1171
post_title: >
  Cómo evitar la inserción de
  geometrías duplicadas en PostgreSQL
author: josetomas
post_date: 2009-12-22 12:51:02
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2009/12/22/como-evitar-la-insercion-de-geometrias-duplicadas-en-postgresql/
published: true
Hide SexyBookmarks:
  - "0"
Hide OgTags:
  - "0"
---
Una de las tareas habituales en el tratamiento de inconsistencias cartográficas es la detección y corrección de geometrías duplicadas. En muchos casos eliminarlas es necesario para evitar incoherencias en nuestra base de datos geográfica, sobre todo en cuanto al cómputo de frecuencias, longitudes y superficies. Por otra parte la eliminación a posteriori, también plantea problemas de decisión que pueden complicar el código a desarrollar y desembocar en tiempos dilatados de proceso. De modo que, en ocasiones merece la pena plantearse la detección de geometrías duplicadas a priori. Implementar semejante mecanismo de control del lado del cliente puede resultar relativamente sencillo, pero lo que aquí os propongo es centralizar la lógica y evitar la inserción de geometrías duplicadas desde el lado del servidor PostgreSQL/PostGIS.<!--more-->

Planteado el problema, la primera solución que viene a la mente pasa por un "trigger" que haga la llamada a la correspondiente función de evaluación y lance un mensaje de error en caso necesario. Sin embargo, si lanzar el mensaje de error no es absolutamente necesario y lo que perseguimos es que una transacción de, por ejemplo, 1000 geometrías donde 50 son duplicadas, se complete con los 950 registros válidos insertados, entonces podemos usar una regla o <a title="RULE" href="http://www.postgresql.org/docs/8.4/static/rules.html" target="_blank">RULE</a>. Cada dos por tres estoy revisitando el sistema de reglas de PostgreSQL y no deja de sorprenderme por su versatilidad y elegancia. Echad un vistazo a la solución propuesta y decidid por vosotros mismos:

[sql]
CREATE OR REPLACE RULE skip_duplicate AS
 ON INSERT TO table_name
 WHERE 0 != (SELECT count(*)
 FROM table_name
 WHERE geom_field ~= new.geom_field) DO INSTEAD NOTHING;
[/sql]

Como podéis comprobar, hay 2 claves para interpretar esta regla:

1) Es una regla condicional (CREATE RULE ... <em>WHERE</em> ... DO ...), y en la condición integramos una subquery que emplea el operador de igualdad geométrica (<em>~=</em>) para evaluar si la geometría de entrada (<em>new.geom_field</em>) es un duplicado. Recordad que, para acelerar el proceso, los operadores de PostGIS explotan, si está presente, el índice GIST sobre el campo de geometría.

2) Con <em>INSTEAD NOTHING</em> indicamos al motor de reescritura de expresiones subyacente al sistema de reglas que, cuando se cumpla la condición anterior, simplemente vacíe la expresión entrante y no haga nada. Por tanto, la sentencia INSERT se omite y deja de formar parte de la transacción actual.

¡Eso es todo!