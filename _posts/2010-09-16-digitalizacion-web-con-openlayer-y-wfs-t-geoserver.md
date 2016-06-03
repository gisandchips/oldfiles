---
ID: 1654
post_title: >
  Digitalización web con OpenLayer y
  WFS-T (Geoserver)
author: jose
post_date: 2010-09-16 18:06:40
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2010/09/16/digitalizacion-web-con-openlayer-y-wfs-t-geoserver/
published: true
---
OBJETIVO:
En el contexto de la Web 2.0. cada vez son más los casos de cartografía interactiva donde continuamente se está actualizando la información. El caso más espectacular es el de OpenStreetMap, donde una legión de "mappers" interactuan con el sistema. Nuestro objetivo es mucho más modesto, pero también más fácil de implementar, y todo gracias al uso de estándares y servicios de mapa libres. Perseguimos, en definitiva, una digitalización <em>on-line</em>, en todos los aspectos: creación de nuevos elementos, modificación geométrica de los ya existentes, actualización de atributos, etc. Todo ello con casi todas las ventajas de las aplicaciones de escritorio, pero con la singularidad de que cualquier usuario pueda intervenir sin apenas conocimientos. 

Estos son los ingredientes para la receta:
<ul>
	<li>capas geográficas almacenadas en tablas PostgreSQL/PostGIS</li>
	<li>servicio de mapas WFS-T con GeoServer</li>
	<li>Diseño web con OpenLayers</li>
</ul>
[caption id="attachment_1677" align="aligncenter" width="300" caption="Digitalizacion WFS-T"]<a href="http://www.gisandchips.org/wp-content/digitalizacion.png"><img src="http://www.gisandchips.org/wp-content/digitalizacion-300x298.png" alt="Digitalizacion WFS-T" width="300" height="298" class="size-medium wp-image-1677" /></a>[/caption]
<!--more-->
En definitiva, nuestro objetivo es ofrecer un servicio de digitalización de elementos geométricos en Internet, utilizando como framework a OpenLayers, que carga una serie de capas servidas por un servicio de mapas de elementos transaccional (WFS-T) almacenadas en una geodatabase PostGIS.

Tengo que reconocer que en un primer momento intenté realizar este juego con un servidor WFS-T poco conocido, pero muy ligero (funciona como CGI, al igual que MapServer), y trabaja con PostGIS, con lo que me las prometía felices, pero las pruebas fueron satisfactorias  sólo mientras  modificase o crease geometría, pero en ningún momento pude subir o modificar atributos. Para los estudiosos este motor se llama TinyOWS . Seguro que más de uno lo habrá usado satisfactoriamente, con lo que se agradecerían comentarios de como hacerlo . Me gustaba porque la instalación era muy sencilla (colocar el ejecutable en el directorio donde se ejecutan los scripts CGI, generalmente en "cgi-bin", sin necesidad de tener instaladas otras herramientas. En este enlace tienes un precioso ejemplo de esta implementación: <a href="http://dev4.mapgears.com/bdga/bdgaWFS-T.html">http://dev4.mapgears.com/bdga/bdgaWFS-T.html</a>

Ante este dilema opté por instalar GeoServer (versión 2.0.1) cuya experiencia está más que demostrada para tener servicios de mapa interoperables. Sí quieres ver las características de nuestro servicio consúltalas aquí

<a href="http://www.gisandchips.org:8080/geoserver/wfs?service=wfs&amp;VERSION=1.1.1&amp;REQUEST=GetCapabilities">http://www.gisandchips.org:8080/geoserver/wfs?service=wfs&amp;VERSION=1.1.1&amp;REQUEST=GetCapabilities</a>

De esta manera me propuse crear un servicio WFS-T sobre tres capas, cada una de una tipología distinta ( puntos, polígonos y líneas), almacenadas en sus correspondientes tablas de PostGIS:
<ul>
	<li>tabla "golf" para puntos de campos de golf. Ver <a href="http://www.gisandchips.org:8080/geoserver/wfs?service=WFS&amp;version=1.0.0&amp;request=DescribeFeatureType&amp;TypeName=cite:golf">características.</a></li>
	<li>tabla "municipios" para polígonos de términos municipales Ver <a href="http://www.gisandchips.org:8080/geoserver/wfs?service=WFS&amp;version=1.0.0&amp;request=DescribeFeatureType&amp;TypeName=cite:municipios">características.</a></li>
	<li>tabla "vias" para línea de una red de carreteras Ver <a href="http://www.gisandchips.org:8080/geoserver/wfs?service=WFS&amp;version=1.0.0&amp;request=DescribeFeatureType&amp;TypeName=cite:vias">características.</a></li>
</ul>
Para ver su potencial desarrollé tres páginas web para cada uno de los tipos de digitalización
<ul>
	<li>demo 1: <a href="http://www.gisandchips.org/demos/j3m/wfs/wfs_golf.html">Digitalización de Campos de golf</a></li>
	<li>demo2: <a href="http://www.gisandchips.org/demos/j3m/wfs/wfs_municipios.html">Digitalización de municipios</a></li>
	<li>demo 3: <a href="http://www.gisandchips.org/demos/j3m/wfs/wfs_vias.html">Digitalización de vías</a></li>
</ul>
<em>NOTA: Puedes probar sin miedo cualquiera de estas demos. Los datos se vuelven a cargar cada día. No te desesperes si es un poco lento el servidor web. No da más de sí.</em>

En cada una de las demos he querido incorporar una barra de iconos para hacer uso de las siguientes herramientas:

[caption id="attachment_1673" align="aligncenter" width="202" caption="panel digitalización"]<a href="http://www.gisandchips.org/wp-content/panel.png"><img class="size-full wp-image-1673" src="http://www.gisandchips.org/wp-content/panel.png" alt="" width="202" height="29" /></a>[/caption]
<ul>
	<li>Consulta y edición de atributos</li>
	<li>Digitalización</li>
	<li>Mover elementos (o vértices en el caso de municipios y vías)</li>
	<li>Borrar elementos</li>
	<li>Guardar las transacciones</li>
	<li>Navegación en modo Pan-zoom</li>
</ul>
El código está implementado de tal forma que la digitalización forma parte de una sesión, y  sólo se validan los cambios (transacción) cuando hacemos clic en el botón "salvar". De esta forma tengo recogido todo el abanico de posibilidades de la digitalización.

OPENLAYERS
Este framework, de sobra conocido es el responsable de articular todos los flujos entre el cliente y la base de datos utilizando WFS-T servido por GeoServer. El código fuente de cada ejemplo lo puedes obtener del propio navegador, por lo que me centraré en desmenuzar las partes:

ESTILO HTML PARA MENÚS
Estilo para la barra de iconos

[html]
    &lt;style&gt;
        .customEditingToolbar {
            float: right;
            right: 0px;
            height: 30px;
            width: 220px;
        }
        .customEditingToolbar div {
            float: right;
            margin: 5px;
            width: 24px;
            height: 24px;
        }
        .olControlNavigationItemActive {
            background-image: url(&quot;../../js/OpenLayers-2.8/theme/default/img/editing_tool_bar.png&quot;);
            background-repeat: no-repeat;
            background-position: -103px -23px;
        }
        .olControlNavigationItemInactive {
            background-image: url(&quot;../../js/OpenLayers-2.8/theme/default/img/editing_tool_bar.png&quot;);
            background-repeat: no-repeat;
            background-position: -103px -0px;
        }
        .olControlDrawFeaturePointItemInactive {
            background-image: url(&quot;../../js/OpenLayers-2.8/theme/default/img/editing_tool_bar.png&quot;);
            background-repeat: no-repeat;
            background-position: -77px 0px;
        }
        .olControlDrawFeaturePointItemActive {
            background-image: url(&quot;../../js/OpenLayers-2.8/theme/default/img/editing_tool_bar.png&quot;);
            background-repeat: no-repeat;
            background-position: -77px -23px ;
        }
        .olControlModifyFeatureItemActive {
            background-image: url(../../js/OpenLayers-2.8/theme/default/img/move_feature_on.png);
            background-repeat: no-repeat;
            background-position: 0px 1px;
        }
        .olControlModifyFeatureItemInactive {
            background-image: url(../../js/OpenLayers-2.8/theme/default/img/move_feature_off.png);
            background-repeat: no-repeat;
            background-position: 0px 1px;
        }
        .olControlDeleteFeatureItemActive {
            background-image: url(../../js/OpenLayers-2.8/theme/default/img/remove_point_on.png);
            background-repeat: no-repeat;
            background-position: 0px 1px;
        }
        .olControlDeleteFeatureItemInactive {
            background-image: url(../../js/OpenLayers-2.8/theme/default/img/remove_point_off.png);
            background-repeat: no-repeat;
            background-position: 0px 1px;
        }
        .olControlSelectFeatureItemInactive {
            background-image: url(&quot;../../js/OpenLayers-2.8/theme/default/img/info_off.png&quot;);
            background-repeat: no-repeat;
            background-position: 0px 1px;
        }
        .olControlSelectFeatureItemActive {
            background-image: url(&quot;../../js/OpenLayers-2.8/theme/default/img/info_on.png&quot;);
            background-repeat: no-repeat;
            background-position: 0px 1px;
        }

    &lt;/style&gt;
[/html]

Aplico en el código de OpenLayers los estilos que tendrá la capa WFS que vamos a cargar

[javascript]
	    // Styles
            var styles = new OpenLayers.StyleMap({
                &quot;default&quot;: new OpenLayers.Style(null, {
                    rules: [
                        new OpenLayers.Rule({
                            symbolizer: {
                                &quot;Point&quot;: {
                                    pointRadius: 5,
                                    graphicName: &quot;square&quot;,
                                    fillColor: &quot;red&quot;,
                                    fillOpacity: 0.25,
                                    strokeWidth: 1,
                                    strokeOpacity: 1,
                                    strokeColor: &quot;#333333&quot;
                                }
                            }
                        })
                    ]
                }),
                &quot;select&quot;: new OpenLayers.Style({
                    strokeColor: &quot;#00ccff&quot;,
                    strokeWidth: 4
                }),
                &quot;temporary&quot;: new OpenLayers.Style(null, {
                    rules: [
                        new OpenLayers.Rule({
                            symbolizer: {
                                &quot;Point&quot;: {
                                    pointRadius: 5,
                                    graphicName: &quot;square&quot;,
                                    fillColor: &quot;white&quot;,
                                    fillOpacity: 0.25,
                                    strokeWidth: 1,
                                    strokeOpacity: 1,
                                    strokeColor: &quot;#333333&quot;
                                }

                            }
                        })
                    ]
                })
            });
[/javascript]


En definitivo asigno un estilo para el elemento geométrico, y su comportamiento cuando es seleccionado, cuando está temporal (digitalización), o su valor por defecto.

CAPA WFS
Definimos una capa WFS con Strategy (para indicar que conjunto de datos deben de ser cargados -ej. todos los datos, por BBOX, en clusters, por página, por filtro-)

[javascript]
// Definimos una capa WFS con St
           var saveStrategy = new OpenLayers.Strategy.Save();

           wfs = new OpenLayers.Layer.Vector(&quot;golf&quot;, {
               //isBaseLayer:true,
               strategies: [new OpenLayers.Strategy.BBOX(), saveStrategy],
               projection: new OpenLayers.Projection(&quot;EPSG:23030&quot;),
               styleMap: styles,
               protocol: new OpenLayers.Protocol.WFS({
                   version: &quot;1.0.0&quot;,
                   srsName: &quot;EPSG:23030&quot;,
                   url: &quot;http://www.gisandchips.org:8080/geoserver/wfs&quot;,
                   featureType: &quot;golf&quot;,
                   featureNS: &quot;http://www.opengeospatial.net/cite&quot;,
                   geometryName: &quot;geometria&quot;,
                   schema: &quot;http://www.gisandchips.org:8080/geoserver/wfs?service=WFS&amp;version=1.0.0&amp;request=DescribeFeatureType&amp;TypeName=cite:golf&quot;,
               })
            });
 // Añadimos la capa anterior y los WMS que queramos de fondo
            map.addLayers([tiled,wfs]);
[/javascript]

RESALTADO DE ELEMENTOS

El siguiente código controla el resaltado de los elementos cuando se lanzan eventos del ratón (mouseover, selección)

[javascript]
		/* HIGHLIGHT */

	    // inicio highlight mouseover
            var report = function(e) {
                OpenLayers.Console.log(e.type, e.feature.id);
            };

            var highlightCtrl = new OpenLayers.Control.SelectFeature(wfs, {
                hover: true,
                highlightOnly: true,
                renderIntent: &quot;temporary&quot;,
                eventListeners: {
                    beforefeaturehighlighted: report,
                    featurehighlighted: report,
                    featureunhighlighted: report
                }
            });

            var selectCtrl = new OpenLayers.Control.SelectFeature(wfs,
                {clickout: true}
            );

            map.addControl(highlightCtrl);
            map.addControl(selectCtrl);

            highlightCtrl.activate();
            selectCtrl.activate();

	    /* END HIGHLIGHT */
[/javascript]

SNAP O TOLERANCIAS DE CAZADO Y SPLIT (CORTES)

En ocasiones puede ser útil que la digitalización no sea a mano alzada (donde hagamos clic con el ratón), sino que tenga establecidas tolerancias de cazado para que cuando nos acerquemos a una línea o un polígono pueda pillar el nodo, vértice o eje. En este <a href="http://openlayers.org/dev/examples/snapping.html">enlace</a> verás un ejemplo más completo de este tema.
En el caso del Split sólo tiene sentido cuando los elementos son polígonos o líneas. Esto nos permite por ejemplo crear una ruta (R1) desde cualquier punto de otra ruta existente (R2), y mientras con el snapping obtenemos la sujección al vértice o eje, la ruta ya existente (R2) se parte en dos partes.

[javascript]
		/* SNAPPING */
            // Configuración del Snapping
            var snap = new OpenLayers.Control.Snapping({layer: wfs});
            map.addControl(snap);
            snap.activate();

            // Configuración del Split
            var split = new OpenLayers.Control.Split({
                layer: wfs,
                source: wfs,
                tolerance: 1000,
                deferDelete: true,
                eventListeners: {
                    aftersplit: function(event) {
                        var msg = &quot;Split resulted in &quot; + event.features.length + &quot; features.&quot;;
                        flashFeatures(event.features);
                    }
                }
            });
            map.addControl(split);
            split.activate();
[/javascript]

PANEL Y CONTROLES

Como ya hemos comentado es preferible tener los iconos a mano en el mapa en vez de los típicos radio button, ya que resultan más elegantes.

[javascript]
/* PANEL */
// crear un panel y añadirle iconos con funcionalidad
var panel = new OpenLayers.Control.Panel(
    {displayClass: 'customEditingToolbar'}
);

// Control para digitalizar puntos (draw)
var draw = new OpenLayers.Control.DrawFeature(
    wfs, OpenLayers.Handler.Point,
    {
        title: &quot;Draw Feature&quot;,
        displayClass: &quot;olControlDrawFeaturePoint&quot;,
        handlerOptions: {freehand: false, multi: false},
        // Muy importante: Si la capa postGIS es POINT poner multi:false.
        // Si es MULTIPOINT poner multi:true
        featureAdded: onFeatureInsert,
        //onFeatureInsert es el método que se ejecuta una vez que hemos terminado de digitalizar
    }
);

/*
En el caso de digitalizar polígonos este sería el código
            var draw = new OpenLayers.Control.DrawFeature(
                wfs, OpenLayers.Handler.Polygon,
                {
                    title: &quot;Draw Feature&quot;,
                    displayClass: &quot;olControlDrawFeaturePolygon&quot;,
                    handlerOptions: {multi: true},
                    featureAdded: onFeatureInsert,
                }
            );
*/

// Control para modificar elementos. En el caso de puntos sólo se pueden desplazar
// a otra posición
modify = new OpenLayers.Control.ModifyFeature(
    wfs, {displayClass: &quot;olControlModifyFeature&quot;, title: &quot;Modify Feature&quot;}
);

// Control para borrar elementos
var del = new DeleteFeature(wfs, {title: &quot;Delete Feature&quot;});

// Control para salvar la sesión de digitalización
var save = new OpenLayers.Control.Button({
    title: &quot;Save Changes&quot;,
    trigger: function() {
        if(modify.feature) {
            modify.selectControl.unselectAll();
        }
        saveStrategy.save();
    },
    displayClass: &quot;olControlSaveFeatures&quot;
});

/*
Control de Información. Es el típico control que una vez que seleccionas un elementos obtenemos los atributos, en este caso englobado en un popUp. Aquí hemos pensado reutilizar este formulario para actualizar datos de cada elemento (UPDATE de atributos).
*/
 selectControl = new OpenLayers.Control.SelectFeature(wfs,
 {
   onSelect: onFeatureInsert,
   onUnselect: onFeatureUnselect,
   displayClass: &quot;olControlSelectFeature&quot;,
   title: &quot;Info&quot;,
});

// Añadimos los controles al panel
panel.addControls([
    new OpenLayers.Control.Navigation({title: &quot;Pan and zoom&quot;}),
    save, del, modify, draw, selectControl
]);

// Definimos el control de navegación como el activo por defecto
panel.defaultControl = panel.controls[0];
map.addControl(panel);
[/javascript]

FUNCIONES DE CADA CONTROL
Borrar elementos

[javascript]
 var DeleteFeature = OpenLayers.Class(OpenLayers.Control, {
      initialize: function(layer, options) {
          OpenLayers.Control.prototype.initialize.apply(this, [options]);
          this.layer = layer;
          this.handler = new OpenLayers.Handler.Feature(
              this, layer, {click: this.clickFeature}
          );
      },
      clickFeature: function(feature) {
          // sí no tiene fid eliminarlo
          if(feature.fid == undefined) {
              this.layer.destroyFeatures([feature]);
          } else {
              feature.state = OpenLayers.State.DELETE;
              this.layer.events.triggerEvent(&quot;afterfeaturemodified&quot;,
                                             {feature: feature});
              feature.renderIntent = &quot;select&quot;;
              this.layer.drawFeature(feature);
          }
      },
      setMap: function(map) {
          this.handler.setMap(map);
          OpenLayers.Control.prototype.setMap.apply(this, arguments);
      },
      CLASS_NAME: &quot;OpenLayers.Control.DeleteFeature&quot;
  });
[/javascript]

Información de los elementos, actualización de atributos e inserción de nuevos elementos (geometría + atributos):

[javascript]
// Funciones para cerrar el popUp
  function onPopupClose(evt) {
      selectControl.unselect(selectedFeature);
  }

// Función para cuando se quita la selección
 function onFeatureUnselect(feature) {
      map.removePopup(feature.popup);
      feature.popup.destroy();
      feature.popup = null;
  }
[/javascript]

Función para insertar un nuevo elemento o actualizar atributos en uno ya existente

[javascript]
// PopUp para insert/update
  function onFeatureInsert(feature)
  {
  		selectedFeature = feature;
                // mapeo de los atributos
		var fid = selectedFeature.id;
     	        var nombre = selectedFeature.attributes['nombre'];
     	        var municipio = selectedFeature.attributes['municipio'];
		var codive = selectedFeature.attributes['codive'];
		var hoyos = selectedFeature.attributes['hoyos'];
                // Compruebo si existe un valor para el atributo nombre.
                // Sí no tiene atributos es un &quot;INSERT&quot; y muestro el formulario para cumplimentar
		if (nombre == null)
		{
 			// formulario
	            htmlForm = &quot;&lt;div style='font-size:.8em'&gt;&quot;+
	            &quot;&lt;H2&gt;&lt;b&gt;A&amp;Ntilde;ADIR CAMPO DE GOLF&lt;/b&gt;&lt;/h2&gt;\n&quot; +
	            &quot;FID: &quot;+ fid + &quot;&lt;br/&gt;\n&quot; +
	            &quot;&lt;input type='hidden' name='fid' id='fid' value='&quot;+ fid +&quot;'&gt;&quot; +
	            //&quot;GEOMETRIA: &quot;+ feature.geometry + &quot;&lt;br/&gt;\n&quot; +
	            &quot;UBICACION: &quot;+ feature.geometry + &quot;&lt;br/&gt;\n&quot; +
	            &quot;Rellene los siguientes datos: &lt;br/&gt; \n&quot; +
	            &quot;NOMBRE:&lt;input type='text' name='nombre' id='nombre' value='Test Campo de Golf' size=20 style='background-color: #E6E6FA; color: #7F7F7F; font-style: italic;' &gt; &lt;br/&gt;\n&quot; +
	            &quot;HOYOS:&lt;input type='text' name='hoyos' id='hoyos' value='18' size=2 style='background-color: #E6E6FA; color: #7F7F7F; font-style: italic;' &gt; &lt;br/&gt;\n&quot; +
	            &quot;MUNICIPIO:&lt;input type='text' name='municipio' id='municipio' value='Test Municipio' size=20 style='background-color: #E6E6FA; color: #7F7F7F; font-style: italic;' &gt; &lt;br/&gt;\n&quot; +
		    &quot;IVE:&lt;input type='text' name='codive' id='codive' value='12345' size=5 style='background-color: #E6E6FA; color: #7F7F7F; font-style: italic;' &gt; &lt;br/&gt;\n&quot; +
	            &quot;&lt;button onclick='onTriggerInsertar()'&gt;A&amp;ntilde;adir&lt;/button&gt;&quot; +
	            &quot;&lt;button onclick='onTriggerDelNewPoint()'&gt;Eliminar&lt;/button&gt;&lt;/div&gt;&quot;;
             }
/* Sí nombre tiene valor ofrezco los atributos en el formulario para modificarlos, añadiendo al popUp
    un botón para &quot;Actualizar&quot; el registro.
*/
                 else
            {
	         htmlForm = &quot;&lt;div style='font-size:.8em'&gt;&quot;+
	         &quot;&lt;H2&gt;&lt;b&gt;ACTUALIZAR CAMPO DE GOLF&lt;/b&gt;&lt;/h2&gt;\n&quot; +
	         &quot;FID: &quot;+ fid + &quot;&lt;br/&gt;\n&quot; +
	         &quot;&lt;input type='hidden' name='fid' id='fid' value='&quot;+ fid +&quot;'&gt;&quot; +
	         &quot;Rellene los siguientes datos: &lt;br/&gt; \n&quot; +
	         &quot;NOMBRE:&lt;input type='text' name='nombre' id='nombre' value='&quot;+ nombre + &quot;' size=20 style='background-color: #E6E6FA;color: #7F7F7F; font-style: italic;' &gt; &lt;br/&gt;\n&quot; +
	         &quot;HOYOS:&lt;input type='text' name='hoyos' id='hoyos' value='&quot; + hoyos + &quot;' size=2 style='background-color: #E6E6FA; color:#7F7F7F; font-style: italic;' &gt; &lt;br/&gt;\n&quot; +
	         &quot;MUNICIPIO:&lt;input type='text' name='municipio' id='municipio' value='&quot; + municipio +&quot;' size=20 style='background-color:#E6E6FA; color: #7F7F7F; font-style: italic;' &gt; &lt;br/&gt;\n&quot; +
		 &quot;IVE:&lt;input type='text' name='codive' id='codive' value='&quot; + codive +&quot;' size=5 style='background-color:#E6E6FA; color: #7F7F7F; font-style: italic;' &gt; &lt;br/&gt;\n&quot; +
	         &quot;&lt;button onclick='onTriggerUpdate()'&gt;Actualizar&lt;/button&gt;&lt;/div&gt;&quot;;
      }
// Defino el objeto popUp
      popup = new OpenLayers.Popup.FramedCloud(&quot;info&quot;,
    		feature.geometry.getBounds().getCenterLonLat(),
         null,
         htmlForm,
         null,
         true,
         onPopupClose);

      feature.popup = popup;
      map.addPopup(popup);

  }

   // Trigger y función para pasar atributos del formulario
	var btnInsert = new OpenLayers.Control.Button({trigger: onTriggerInsertar});

	function onTriggerInsertar(fid)
	// se le pasa el fid como parametro para seleccionarlo mediante getFeatureById
	{
      //alert(&quot;has llegado a trigger insertar&quot;);
      var fid =  OpenLayers.Util.getElement('fid').value;// recojo el fid del formulario
      var miFeature = wfs.getFeatureById(fid);// hago la selección de la feature a partir del fid del form
      //alert(miFeature.id);
      // Paso atributos del formulario al elemento
		miFeature.attributes.nombre = OpenLayers.Util.getElement('nombre').value;
		miFeature.attributes.municipio = OpenLayers.Util.getElement('municipio').value;
		miFeature.attributes.hoyos = OpenLayers.Util.getElement('hoyos').value;
		miFeature.attributes.codive = OpenLayers.Util.getElement('codive').value;
		// elimina el popup
		onFeatureUnselect(miFeature);
	}
	// fin trigger insert

   // Trigger y función para pasar atributos del formulario en el update
	var btnUpdate = new OpenLayers.Control.Button({trigger: onTriggerUpdate});

	function onTriggerUpdate()
	{
      // defino miFeature como el array de elementos seleccionados, miFeature[0] será el único elemento del array
      miFeature = [selectedFeature]; //alert(miFeature.length);
      // Paso atributos del formulario al elemento
                var fid =  OpenLayers.Util.getElement('fid').value;
                miFeature[0].id = fid;
		miFeature[0].attributes.nombre = OpenLayers.Util.getElement('nombre').value;
		miFeature[0].attributes.municipio = OpenLayers.Util.getElement('municipio').value;
		miFeature[0].attributes.hoyos = OpenLayers.Util.getElement('hoyos').value;
		miFeature[0].attributes.codive = OpenLayers.Util.getElement('codive').value;
		// comprobar que se le pasan los atributos al feature. Descomentar para comprobación
		/*alert (  miFeature[0].id + &quot;,&quot; +
				   miFeature[0].attributes.nombre + &quot;,&quot; +
					miFeature[0].attributes.municipio  + &quot;,&quot; +
					miFeature[0].attributes.hoyos  + &quot;,&quot; +
					miFeature[0].attributes.codive );
               */

		miFeature[0].state = OpenLayers.State.UPDATE;
		// elimina el popup
		map.removePopup(miFeature[0].popup);
                selectControl.unselectAll();
                miFeature[0].popup = null;
	}
	// fin trigger
[/javascript]

Faltan algunos elementos más, pero en esencia aquí está la lógica de la digitalización. Observa cada caso, puntos, polígonos y líneas para ver las diferencias existentes.

CONCLUSIÓN:
Quizás lo más relevante de este tema es que con un sólo fichero HTML hemos abordado la complejidad de la digitalización, utilizando siempre servicios y tecnologías estándar y abiertas. Con este tipo de soluciones tan sencillas podemos acercar a un número mayor de usuarios, menos especializados quizás, pero que de otra forma se hubieran visto imposibilitados de actuar por razones técnicas, económicas o de formación.