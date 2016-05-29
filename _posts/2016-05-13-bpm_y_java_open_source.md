---
title: "BPM y Java Open Source"
header:
  teaser: "post_teaser_process.jpg"
categories: 
  - Técnicas y Metodologías
tags:
  - BPM
  - Open Source
---

Tengo intención de publicar una serie de artículos sobre el modelado e implementación de procesos BPM con Java Open Source. En este primer post haré una breve introducción a BPM, qué es, para qué sirve, qué beneficios aporta a la organización y en qué consiste la notación gráfica BPMN.  También haré referencia a las características que debe tener una solución software BPM completa y qué alternativas tenemos para integrar un workflow de procesos en un proyecto Java usando APIs Open Source.

En próximos artículos hablaré del API que uso yo actualmente y porqué me decanté por él, y periódicamente iré publicando ejemplos de código e iré profundizando en temas que considero relevantes para iniciarse en los motores de procesos o workflow con notación BPMN.

Comencemos...

# ¿Qué es BPM?

Antes de empezar con BPM repasemos el concepto de 'proceso de negocio' (business process). Un proceso es un conjunto de actividades con algún tipo de lógica enfocado en lograr algún resultado específico. En un proceso de negocio la finalidad es generar productos o servicios utilizando recursos de la empresa.

Por ejemplo este podría ser un proceso básico para el pedido de una pizza:

![no-alignment]({{ site.url }}{{ site.baseurl }}/images/bpmn/bpmn_pizza_simple.jpg)

Muy fácil de comprender: primero se hace el pedido de la pizza, luego se cocina y finalmente se sirve al cliente.

Pero en el mundo real los procesos son mucho más detallados y complejos y pueden requerir la ejecución de tareas en paralelo, bifurcaciones según algún tipo de condición, repetición de tareas, comunicación entre procesos, reacción ante eventos, control de errores, distribución de tareas por roles, etc.

Este sería un ejemplo del proceso 'pizza' un poco más complejo pero más cercano a la realidad:

![no-alignment]({{ site.url }}{{ site.baseurl }}/images/bpmn/bpmn_pizza_complex.jpg)

(fuente: [www.bpmn.info](http://www.bpmn.info/tutorials/bpmn-order-process-for-pizza/))

Bien, pues BPM (Business Process Management -Gestión de Procesos de Negocio-) podría definirse como un *"enfoque disciplinado para identificar, diseñar, ejecutar, documentar, medir, monitorear y controlar procesos de negocio, con el objetivo de lograr resultados alineados con las metas estratégicas de la organización"* (definición obtenida del libro "BPM Common Body of Knowledge"). 

Si gestionamos los procesos de nuestro negocio aplicando BPM podremos obtener, entre otros, los siguientes **beneficios**:

- Reducción de costes.
- Incremento de calidad.
- Conseguir procesos más efectivos y eficientes.
- Reducción en tiempo y coste de formación del personal involucrado.
- Menos peticiones de soporte interno.
- Reducción de quejas de los clientes.
- Incremento en la exactitud de los pronósticos.
- Mejor comprensión de cómo funciona el negocio.
- Mejor comprensión de cómo se administran los recursos.

Esta disciplina se fundamenta en el siguiente **ciclo de vida**:

![no-alignment]({{ site.url }}{{ site.baseurl }}/images/bpmn/bpm_lifecycle.jpg)

- **Diseño del proceso**
Es la etapa donde se diseñan y construyen los diagramas de procesos usando la notación gráfica BPMN (Business Process Modeling Notation). Se trata de un diseño de alto nivel elaborado normalmente por gerentes y analistas de procesos.

- **Modelado (Implementación)**
El diseño teórico se amplía añadiendo lógica que permita que el proceso pueda ser automatizado y ejecutado por un sistema software. Se definen formularios de usuario, reglas de ejecución, acceso a datos, scripts, etc. En esta fase intervienen administradores de sistemas y desarrolladores.

- **Ejecución**
El proceso entra en ejecución y es utilizado por las personas, herramientas y/o sistemas. El diagrama diseñado y modelado en las etapas anteriores ahora es interpretado por un motor de procesos dentro de una solución software. Se forma al personal, se establecen metas y se obtienen resultados tangibles.

- **Monitorización**
Se realiza un seguimiento del proceso. Se recopila, analiza y procesa información para la toma de decisiones, detección de desviaciones y para proponer posibles mejoras.

- **Optimización**
A partir de la información obtenida en la fase de monitorización se aplican las correcciones y mejoras necesarias en el proceso.

# Notación y simbología BPMN

BPMN (Business Process Model and Notation -Modelo y Notación de Procesos de Negocio-) es una notación gráfica estandarizada por OMG (Object Management Group) que permite el modelado de procesos de negocio en un formato de flujo de trabajo (workflow). Su objetivo es ofrecer una notación estándar que sea entendible para todos los interesados del negocio (stakeholders): gerentes, analistas, administradores, desarrolladores, etc. Los procesos mostrados en el apartado anterior están diseñados con BPMN.

BPMN se centra en la siguiente simbología:

### Tareas
Son unidades de trabajo que representan una actividad desempeñada por un participante. Ese participante puede ser una persona, un script, una clase Java, etc.

![no-alignment]({{ site.url }}{{ site.baseurl }}/images/bpmn/bpmn_tareas.jpg)
	
### Eventos
Describen algo que sucede durante la ejecución del proceso. Hay multitud de eventos definidos en BPMN: para control de tiempos, cancelaciones, reacción ante errores, envío y recepción de mensajes, etc. Y, por otro lado, los eventos pueden iniciar un proceso, finalizarlo o participar durante el flujo del proceso (intermediate).

![no-alignment]({{ site.url }}{{ site.baseurl }}/images/bpmn/bpmn_eventos.jpg)

Aquí puedes ver una [tabla completa de eventos](http://training-course-material.com/images/4/40/TypesOfEvents.png).
	
### Conectores
Representan flujos de ejecución en el proceso.

![no-alignment]({{ site.url }}{{ site.baseurl }}/images/bpmn/bpmn_conectores.jpg)
	
### Compuertas
Una compuerta o gateway representa un punto de decisión donde a partir de unos datos de entradas se toma una determinada decisión y se canaliza la salida por uno o varios caminos.

![no-alignment]({{ site.url }}{{ site.baseurl }}/images/bpmn/bpmn_compuertas.jpg)

### Contenedores
Como su nombre indica sirven para contener otras actividades y agruparlas para un mismo objetivo, como por ejemplo agrupar actividades según el rol de la persona o personas que las va a ejecutar, o agrupar actividades en un subproceso

![no-alignment]({{ site.url }}{{ site.baseurl }}/images/bpmn/bpmn_contenedores.jpg)

### Artefactos
Son símbolos que ayudan a la comprensión y documentación del proceso pero que no tienen implicación directa sobre el flujo de ejecución. 

![no-alignment]({{ site.url }}{{ site.baseurl }}/images/bpmn/bpmn_artefactos.jpg)

Te recomiendo echar un vistazo a [este PDF](http://www.bpmb.de/images/BPMN2_0_Poster_ES.pdf) que resume toda la simbología de BPMN 2.0 (la versión actual de BPMN es la 2.0.2). Y si quieres profundizar en el tema esta es [la especificación formal](http://www.omg.org/spec/BPMN/2.0/PDF/) del estándar.

# Partes de una arquitectura software completa para BPM

¿Qué debe tener una solución software para poder ser considerada útil dentro del ámbito de BPM?

- **workflow**
Es el corazón de BPM, el motor de ejecución de procesos. Permite automatizar, sincronizar e integrar procesos. Es el que se encarga de entender y ejecutar el flujo de tareas, eventos y operaciones que tienen definidos los procesos.

- **Integrador de aplicaciones**
Permite compartir datos y procesos entre diferentes programas de la organización, independientemente de la plataforma y tecnología. Se refiere a la arquitectura tecnológica que permite comunicar los procesos con otros servicios y/o componentes, por ejemplo con el servicio de gestión de usuarios, con el servicio de permisos, facturas, comunicaciones de red o con cualquier otro servicio que de sentido al negocio de la organización (CRM, ERP, ITIL, gestión documental, eCommerce, ...).

- **Integrador de negocios**
Permite a los actores externos de la organización interactuar con sus procesos de una forma ágil, flexible y dinámica. Digamos que es la aplicación software con la que operan los usuarios: una aplicación Web, una aplicación de escritorio, una destinada a dispositivos móviles, o todo a la vez. Este integrador será el encargado de organizar la información y mostrarla al usuario, de presentar los formularios, de permitir ejecutar acciones, etc.

- **Herramientas de monitorización**
Permiten recopilar, procesar y distribuir información sobre el funcionamiento de los procesos. Estas herramientas nos permiten obtener informes útiles para la toma de decisiones y/o proveer datos para aplicar técnicas de gestión del rendimiento corporativo (CPM - Corporate Performance Management) dentro del área de Business Intelligence (BI), cumplimiento de los KPI (Key Performance Indicator), retorno de la inversión (ROI), etc.

El concepto de BPM es muy amplio y abarca multitud de conceptos como BPA (análisis), BRE (reglas), EAI (integración), BPMS (orquestación), BAM (monitorización), BI (reporting), etc.

# BPM y Java Open Source
A continuación enumeraré y describiré los principales frameworks Java open source que tenemos a nuestra disposición para afrontar una arquitectura software BPM como la que he mencionado anteriormente:

## jBPM

Potente paquete de proyectos integrados. Es una suite completa para BPM con multitud de herramientas y utilidades. Es uno de los frameworks que más tiempo lleva en el mercado. Es bueno para incluir en un proyecto iniciado desde cero, pero para integrarlo en una aplicación ya existente (por ejemplo un ERP) es un poco complejo de configurar y tiene muchas dependencias. Está muy orientado a la gestión del flujo de trabajo mediante reglas Java Drools.
Es un proyecto de JBoss.
Tiene un plugin de modelado de diagramas BPMN para Eclipse.

- [Página del proyecto](http://www.jbpm.org)
- [Descarga](http://www.jbpm.org/download/download.html)
- [Código fuente en Github](https://github.com/droolsjbpm/jbpm)
- [Documentación](https://docs.jboss.org/jbpm/v6.0/userguide)
- [Comunidad](http://www.jbpm.org/community/forum.html)

## BonitaSoft

Su eslogan es "Bonita BPM: La plataforma más poderosa de aplicaciones basada en BPM. Todo lo que necesita para construir aplicaciones de negocio atractivas y personalizadas que se adapten a su negocio en tiempo real."
Es un set de herramientas más un entorno de desarrollo integrado.
Tiene un workflow Open Source y una solución software completa para BPM. La aplicación puede adquirirse en dos versiones: la "Community Edition", que es open source; y la "Subscription Edition" que ofrece soporte, personalización de interfaces de usuario, etc.

La suite incluye:

- Bonita BPM Studio - Diseñador gráfico de procesos BPMN 2.0
- Bonita BPM Portal - Interfaz de usuario para interactuar con las aplicaciones de procesos
- Bonita BPM Engine - Motor de workflow Java
- Web Forms Designer - Editor de formularios para las tareas de usuario BPM
- Multiple API and Connection Points - APIs para extender su funcionalidad

Está poco orientado a desarrolladores, su enfoque está más destinado a analistas, gestores y administradores de procesos. Está pensado para ofrecer todas las herramientas necesarias para el manejo de procesos, pero no tanto para usar su "core" e integrarlo en otros proyectos. Toda la configuración e implementación de procesos se realiza a través de sus herramientas, sin necesidad de escribir una sola línea de código, lo cual, de cara a un desarrollador, se traduce en pérdida de control y dificultad para la personalización.
Su finalidad última es comercial (dar soporte, mantenimiento, formación, etc.) por lo que no es fácil encontrar mucha documentación técnica sobre su motor de procesos.

- [Página del proyecto](http://es.bonitasoft.com/)
- Descarga: La aplicación puede obtenerse desde su [página oficial de descarga](http://www.bonitasoft.com/downloads-v2), pero requiere crear una cuenta. También está disponible en [sourceforge](https://sourceforge.net/projects/bonita/).
- [Código fuente en Github](https://github.com/bonitasoft)
- [Documentación](http://documentation.bonitasoft.com/)
- [Comunidad](http://community.bonitasoft.com/)

## Activiti BPM Platform

Activiti es principalmente un motor de procesos BPMN 2 ligero, rápido y robusto para proyectos Java. Es de código abierto y distribuido bajo licencia Apache. Se ejecuta en cualquier aplicación Java, en un servidor, en un clúster o en la nube.
Activiti es un proyecto de [Alfresco](https://www.alfresco.com/es) (es el motor de procesos que usa Alfresco), pero es un proyecto independiente.
El core es similar a jBPM (Activiti está creado por los fundadores de jBPM).
Está muy orientado al desarrollador. El motor (el 'process Engine') es ofrecido a través de un API Java independiente y de fácil manejo. Y luego ofrece diferentes jars para ampliar sus características (comunicación por REST, integración con Spring, CDI, Camel, OSGI, etc.). 
Incluye una aplicación Web para la gestión de procesos, pero está más pensada como ejemplo que como solución final. 
Tiene un plugin de modelado de diagramas BPM para Eclipse.
Es muy fácil integrarlo en un proyecto existente.
No está orientado a Java Drools, aunque puede usarse y tiene buena documentación.

- [Página del proyecto](http://activiti.org/)
- [Descarga](http://activiti.org/download.html)
- [Documentación](http://www.activiti.org/userguide/) (hay un libro: "[Activiti in Action](https://www.manning.com/books/activiti-in-action)" que incluye el código fuente de los ejemplos usados, y también un [blog](http://bpmn20inaction.blogspot.com.es/) donde el autor publica actualizaciones).
- [Código fuente en Github](https://github.com/Activiti)
- [Comunidad](https://forums.activiti.org/)

## Camunda

Camunda es una plataforma de código abierto para la gestión de flujos de trabajo y procesos de negocio. El proyecto está gestionado por una empresa de unos 30 empleados con sede en Berlín fundada en Abril de 2013 y con una filosofía prácticamente idéntica a la de Activiti (Open Source BPM Platform). El equipo de Camunda era parte importante de la comunidad de Activiti y se separaron para seguir su propio camino y crear su propia solución BPM comercial, una suite similar a BonitaSoft. La decisión de Camunda de separarse supuso una gran [sorpresa y decepción](http://www.jorambarrez.be/blog/2013/03/18/a-sad-day-for-open-source-camunda-decides-to-fork-activiti/) a la comunidad de Activiti, quienes se referían a Camunda como "un 'fork' de Activiti, con el mismo 'core' y distinto nombre". 

Camunda no está asociado a ningún vendedor (Activiti es parte de Alfresco) y quizás por ahí estén los motivos de la separación.
Su código es Open Source y muy modularizado.
Tienen abundante documentación, incluida una extensa guía de cómo migrar de Activiti a Camunda.
A día de hoy, la principal diferencia con Activiti es que Camunda, además de BPMN, se integra también con CMMN (Case Management Model & Notation) y con DMN (Decision Model & Notation), que son técnicas de modelado operacional bastante recientes (la versión 1.0 de CMMN fue lanzada por OMG en mayo de 2014).

- [Página del proyecto](https://camunda.org/)
- [Descarga](https://camunda.org/download/)
- [Documentación](https://docs.camunda.org/manual/7.4/)
- [Código fuente en Github](https://github.com/camunda)
- [Comunidad](https://network.camunda.org/)
- [Página de la solución comercial](https://camunda.com/)

## Otras alternativas
En los puntos anteriores únicamente menciono los que considero "principales" desde mi punto de vista, que es puramente técnico. Estos proyectos son los que mayor actividad tienen dentro de sus comunidades de desarrollo y también son los que más documentación técnica ofrecen para poder integrar sus workflow de procesos en proyectos Java EE. Pero existen muchos otros proyectos BPM open source como [Intalio](http://www.intalio.com/) o [ProcessMaker](https://www.processmaker.com/es). 
Aquí te dejo una [lista de Java-Source.net](http://java-source.net/open-source/workflow-engines) donde podrás encontrar multitud de proyectos "Open Source Workflow Engine",  algunos de ellos basados en BPEL (Business Process Execution Language).
