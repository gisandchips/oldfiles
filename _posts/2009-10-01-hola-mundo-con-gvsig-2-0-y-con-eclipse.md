---
ID: 210
post_title: Hola Mundo con gvSIG 2.0 (y con Eclipse)
author: Jorge Piera Llodrá
post_date: 2009-10-01 13:52:22
post_excerpt: ""
layout: post
permalink: >
  http://www.gisandchips.org/2009/10/01/hola-mundo-con-gvsig-2-0-y-con-eclipse/
published: true
syntaxhighlighter_encoded:
  - "1"
---
Este tutorial es una guía para desarrollar el famoso "Hola Mundo" en <a href="http://www.gvsig.gva.es">gvSIG</a> 2.0. Se va a partir de un sistema operativo en el que se ha instalado previamente el entorno de desarrollo <a href="http://www.eclipse.org/">Eclipse</a> con el plugin para SVN <a href="http://www.eclipse.org/subversive/">Subversive</a> y a partir de este entorno, se va a describir paso por paso las acciones que se tienen que realizar para añadir una opción en el menú "Ayuda" de gvSIG llama "Saludo" que al pulsar sobre ella abrirá una ventana con el clásico "Hola Mundo".

Este documento no pretende ser una guía de desarrollo de gvSIG 2.0.  Si se quiere disponer de toda la información de desarrollo es aconsejable visitar la <a href="http://www.gvsig.org">web de documentación del proyecto</a>.<!--more-->

El primer paso es crear una carpeta en nuestro sistema donde vamos a situar todos los proyectos que nos van a hacer falta, que serán tanto nuestro proyecto del Hola Mundo como todos los proyectos mínimos necesarios para poder generar un gvSIG 2.0. Otra opción que no está contemplada en este tutoria es la de poder instalar nuestra extensión en una versión de gvSIG previemente instalada desde un instalador.

Una vez creada la carpeta hay que abrir el Eclipse e indicarle establecer la carpeta que hemos creado como workspace. Para ello tendremos que ir a la opción "File-&gt;Switch Workspace-&gt;Other" y seleccionar la carpeta que hemos creado anteriormente.

El siguiente paso será descargarnos el proyecto "build" desde el repositorio de gvSIG. Este proyecto es necesario para poder construir nuestro sistema y es el único proyecto que hay que bajar manualmente. Para ello hay que abrir la vista de SVN del Eclipse seleccionando la opción "Window -&gt; Open Perspective -&gt; Other" y hay que seleccionar la opción "SVN Repository Exploring". Si no aparece esta opción es porque no se ha instalado ningún plugin de SVN.
<p style="text-align: center"><a href="http://www.gisandchips.org/wp-content/svn.png"><img class="size-large wp-image-212 aligncenter" src="http://www.gisandchips.org/wp-content/svn-1024x604.png" alt="Abrir la vista de SVN" width="500" height="300" /></a></p>
A continuación se tiene que añadir manualmente el repositorio de gvSIG desde el que hay que descargar el proyecto "build". Para ello hay que pulsar con el botón derecho del ratón sobre la ventana "SVN Repositories" y se abrirá una nueva ventana donde hay que introducir la dirección del repositorio de gvSIG (http://subversion.gvsig.org/gvSIG). Pulsaremos en "Finish" y si todo ha funcionado correctamente deberá aparecer el repositorio en la lista de servidores.

La lista de servidores es una lista en forma de árbol que podremos ir desplegando haciendo un doble "click" en cada uno de los nodos del árbol. Hay que abrir el servidor que hemos añadido y tendremos que buscar la ruta "<img src="///tmp/moz-screenshot.jpg" alt="" />branches -&gt; v2_0_0_prep -&gt; build". Una vez estemos en ella, pulsamos con el botón derecho del ratón sobre la carpeta y seleccionamos la opción "Check Out" que descargará el proyecto en nuestro workspace.

<a href="http://www.gisandchips.org/wp-content/chechout.png"><img class="aligncenter size-full wp-image-218" src="http://www.gisandchips.org/wp-content/chechout.png" alt="Check Out del proyecto build" width="500" height="300" /></a>

Ahora hay que ir a la vista de Java para poder ver el proyecto que acabamos de bajar. Para ello hay que seleccionar la opción "Window -&gt; Open Perspective -&gt; Java". Si todo ha ido bien el proyecto "build" se encontrará en la vista "Package".

A continuación vamos a abrir la vista de Ant desde la cual vamos a poder ejecutar algunos objetivos automáticamente. Para ello hay que ir al menú "Window -&gt; Show View -&gt; Ant" y se abrirá una ventana nueva llamada "Ant". El proyecto build tiene en su carpeta raíz un fichero llamado "build.xml" que contiene todos los objetivos que podemos ejecutar. Si pulsamos sobre ese fichero y sin soltar lo arrastramos y lo dejamos caer sobre la ventana de "Ant" automáticamente aparecerán los objetivos definidos en el fichero.
<p style="text-align: center"><a href="http://www.gisandchips.org/wp-content/ant.png"><img class="aligncenter size-full wp-image-219" src="http://www.gisandchips.org/wp-content/ant.png" alt="Obetivos de Ant" width="500" height="300" /></a></p>
<p style="text-align: left">Lo primero que tenemos que hacer es modificar la configuración del Eclipse con algunos parámetros que necesitamos. Se puede hacer manualmente, pero para facilitar la configuración se ha creado un objetivo de Ant que lo hace todo por nosotros. Para ello tenemos que ir a la ventana de Ant y hacer un doble "click" en la opción "mvn-configure-eclipse-workspace". Si todo ha funcionado correctamente se abrirá una ventana donde tendremos que escribir la ruta donde se encuentra el workspace que queremos configurar, que por defecto será el workspace actual. Pulsaremos en OK y esperaremos a que se ejecute el comando con éxito utilizando para ello la vista "Console".</p>
<p style="text-align: left">Una vez que ya tenemos el workspace configurado correctamente vamos a bajarnos los fuentes del gvSIG. Para ello debemos desplegar el proyecto "build" e ir a la carpeta "projects" donde veremos unas cuantas carpetas que se corresponden a grupos de proyectos de gvSIG. Nosotros necesitamos lo mínimo para poder arrancar un gvSIG, así que tendremos que desplegar la carpeta "gvsig-base", pinchar con el botón izquierdo del ratón sobre el archivo "build.xml" y arrastrarlo hasta la vista de Ant.</p>
<p style="text-align: left">Ahora tendremos en la vista de Ant dos configuraciones diferentes (gvsig-build-config y gvsig-group-base), por lo que hay que ir con precaución a la hora de seleccionar un objetivo. Seleccionaremos gvsig-group-base y a continuación haremos un doble "click" en el objetivo "svm.checkout.all". Se abrirá una ventana que servirá para poder configurar el servidor desde el que vamos a descargar los fuentes. Por defecto no hay que modificar nada, salvo la versión del conector de SVN que tenemos instalada en el Eclipse. Si no se conoce  la versión, se puede consultar en "Window -&gt; Preferences -&gt; Team -&gt; SVN -&gt; SVN Connector".</p>
<p style="text-align: left"><a href="http://www.gisandchips.org/wp-content/checkout.png"><img class="aligncenter size-full wp-image-237" src="http://www.gisandchips.org/wp-content/checkout.png" alt="checkout" width="500" height="300" /></a></p>
<p style="text-align: left">Pulsamos en "Ok" y si todo ha funcionado correctamente veremos en la ventana "Console" como la aplicación empieza a descargarse diversos proyectos. Una vez que haya terminado tendremos que importar los proyectos descargados. Para ello tenemos que pulsar con el botón derecho sobre la ventana de "Package" y seleccionar la opción "Import". A continuación debemos seleccionar "General -&gt; Existing Projects into Workspace" y pulsa en "Next". Seleccionamos la carpeta de nuestro workspace y veremos como aparece una lista de proyectos en la ventana. Pulsamos en "Finish" y automáticamente aparecen todos los proyectos que forman el gvSIG base en la ventana "Package".</p>
Ahora vamos a compilar todos los fuentes y vamos a arrancar gvSIG. Para ello hay que ir a la vista de Ant y seleccionando "gvSIG-group-base" ejecutaremos el objetivo "mvn-install". Este objetivo compilará todos los proyectos que nos hemos bajado y creará una instalación de gvSIG en la carpeta "build -&gt; product". En la ventana "console" podremos ver el progreso de la compilación. En la actualidad gvSIG 2.0 está en fase de desarrollo y es posible que el comando anterior falle porque ha fallado alguno de los tests unitarios. Si es el caso hay que repetir el proceso anterior pero ejecutando "mv-install-without-tests", que hace lo mismo que el install pero sin generar ni ejecutar los tests unitarios.

Si todo ha funcionado correctamente ya estamos en disposición de poder ejecutar gvSIG 2.0 desde el Eclipse. Para eso tenemos que seleccionar la opción "Run -&gt; Debug Configurations -&gt; Java Application"  y seleccionar el lanzador de nuestro sistema operativo. Al pulsar en "Debug" debería arrancar gvSIG. Si no lo hace y sale un mensaje diciendo que no se encuentra alguna librería, es porque Eclipse "no se ha dado cuenta" de que se ha instalado la versión de gvSIG en la carpeta "build/product". Refrescamos esta carpeta y volvemos a arrancar la aplicación.
<p style="text-align: center"><a href="http://www.gisandchips.org/wp-content/run.png"><img class="aligncenter size-full wp-image-262" src="http://www.gisandchips.org/wp-content/run.png" alt="run" width="500" height="300" /></a></p>
Ahora que ya tenemos la aplicación vamos a crear nuestra extensión para gvSIG. Primero, deberemos decidir si lo que vamos a crear es una librería o una extensión. La diferencia principalmente es que las librerías no tienen interfaz de usuario y no dependen de Andami mientras que las extensiones sí que dependen de Andami. Nuestra idea es mostrar el "Hola Mundo" en una nueva ventana por lo que vamos a generar una extensión.

Para ellos seleccionamos la opción "Run -&gt; External Tools -&gt; External Tools configurations"  y en la opción "Ant Build" seleccionamos y ejecutamos "create extension". Se abrirá una ventana dónde tendremos que insertar algunos de los parámetros de configuración de "Maven". El único que nos interesa en estos momentos en el "Maven artifactid", cuyo valor corresponderá con el nombre del proyecto de Eclipse que se va a crear. en nuestro caso escribiremos "org.gvsig.holamundo" y pulsaremos el botón "OK". En la ventana "Console" podremos seguir el proceso de creación del nuevo proyecto.

Una vez que tenemos el proyecto creado hay que importarlo del mismo modo que importamos anteriormente todos los proyectos que nos bajamos del repositorio. El proyecto que se ha creado ha tomado como base la extensión de centrar una vista en un punto por lo que tiene algunas clases que no vamos a utilizar. Podemos eliminar todas las clases del proyecto.

Ahora vamos a crear una clase llamada "HolaMundoExtension" que herede de "org.gvsig.andami.plugins.Extension" en el paquete "org.gvsig.holamundo" que será nuestro punto de entrada cuando el usuario seleccione la opción de menú "Hola Mundo" desde la aplicación.  Tendremos que hacer que los métodos "isVisible" e "isEnabled" devuelvan "true" y en el método "execute"pondremos el código que muestre una ventana con un "Hola Mundo". La clase quedará del siguiente modo:

[java]

import java.awt.Component;
import javax.swing.JOptionPane;
import org.gvsig.andami.PluginServices;
import org.gvsig.andami.plugins.Extension;

public class HolaMundoExtension extends Extension {

public void execute(String actionCommand) {
JOptionPane.showMessageDialog((Component)PluginServices.getMainFrame(), &quot;Hola Mundo&quot;);
}

public void initialize() {

}

public boolean isEnabled() {
return true;
}

public boolean isVisible() {
return true;
}
}

[/java]

Ahora tenemos que "enganchar" una opción de menú con nuestra clase editando el fichero "src/main/resources/config.xml" del modo que se muestra a continuación:

[xml]

&lt;?xml version=&quot;1.0&quot; encoding=&quot;ISO-8859-1&quot;?&gt;
&lt;plugin-config&gt;
&lt;libraries library-dir=&quot;lib&quot;/&gt;
&lt;depends plugin-name=&quot;org.gvsig.app&quot;/&gt;
&lt;resourceBundle name=&quot;text&quot;/&gt;
&lt;extensions&gt;
&lt;extension class-name=&quot;org.gvsig.helloworld.HolaMundoExtension&quot;
description=&quot;Extensión que hace un hola mundo&quot;
active=&quot;true&quot;&gt;
&lt;menu text=&quot;Prueba/Hola Mundo&quot;
tooltip=&quot;Hola Mundo&quot;
action-command=&quot;&quot;/&gt;
&lt;/extension&gt;
&lt;/extensions&gt;
&lt;/plugin-config&gt;

[/xml]

Para poder probar nuestra extensión, antes debemos compilarla e instalarla en gvSIG. Para ello debemos de seleccionar nuestro proyecto y a continuación ejecutar el objetivo "mvn install" que encontraremos en "Run -&gt; External Tools -&gt; External Tools configurations -&gt; Ant Build". Si todo funciona correctamente, deberemos arrancar gvSIG del mismo modo que lo habíamos hecho anteriormente y aparecerá una nueva entrada en la barra de menú llamana "Prueba" desde la que podremos ejecutar nuestra extensión.
<p style="text-align: center"><a href="http://www.gisandchips.org/wp-content/helloworld.png"><img class="aligncenter size-full wp-image-263" src="http://www.gisandchips.org/wp-content/helloworld.png" alt="helloworld" width="500" height="300" /></a></p>
<p style="text-align: left">Si se quiere profundizar un poco más para entender qué contiene un fichero "config.xml" o cómo se crean extensiones más complejas se puede consultar la web de documentación del proyecto <a href="http://www.gvsig.org">gvSIG</a>.</p>
<p style="text-align: left"></p>
<p style="text-align: left"></p>
<p style="text-align: left"></p>
<p style="text-align: left"></p>