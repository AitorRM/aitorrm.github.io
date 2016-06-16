---
title: "Activiti BPM: por qué y para qué (caso de implantación)"
header:
  teaser: "post_teaser_activiti.jpg"
categories: 
  - BPM
tags:
  - BPM
  - Activiti
  - Procesos
  - Java
---

A la hora de elegir una solución BPM lo primero que hay que tener claro es cuál es nuestra necesidad: ¿necesito un software BPM y no preocuparme por detalles técnicos ni soporte?, ¿ya tengo una aplicación y lo que necesito es usar un sistema BPM con el que comunicarme?, ¿quiero construir una nueva solución BPM y comercializarla?, ¿o lo que necesito es un fremework o un motor de procesos para gestionar algún flujo de trabajo (workflow) de mi aplicación?

Yo me encuentro dentro del último grupo: necesito un buen motor de flujos de trabajo basado en BPMN 2.0 para integrarlo en una aplicación Java EE ya existente. Bueno, realmente me encontraba en esta situación hace algunos años, me decanté por [Activiti BPM Platform](http://activiti.org/) y ahora, en este artículo, quiero comentar cómo llegué a él, citar algunas características que ofrece y hacer algunas reflexiones basadas en mi experiencia tras implantarlo en un entorno real de producción.

Antes de comenzar quiero aclarar que el objetivo principal de este artículo es servir como introducción a otros que iré publicando más adelante donde me centraré en detalles de implementación de procesos BPMN con Activiti. En ningún caso pretendo entrar en comparativas ni debates sobre si Activiti es mejor o peor que otros frameworks como jBPM o Camunda, principalmente porque no los conozco lo suficiente, de hecho Activiti es el único que he probado en un entorno real de producción. El resto sólo los valoré en la fase de estudio.

También asumo que conoces BPM. Si no es así [aquí te cuento algunos conceptos]({% post_url 2016-05-13-bpm_y_java_open_source %}) generales sobre esta disciplina, la simbología BPMN y algunas soluciones software dentro del mundo Java Open Sopurce.

## ¿Por qué Activiti?

![no-alignment]({{ site.url }}{{ site.baseurl }}/images/activiti/activiti-logo.jpg)

Si lo que necesitas es un framework para gestionar un flujo de trabajo en una aplicación Java, Activiti puede ser una buena opción. Activiti se crea con el objetivo de ser un workflow de ejecución de procesos BPMN 2.0 potente, robusto, ligero, de código abierto y fácil de implantar en cualquier aplicación Java, ya sea Web, de escritorio o directamente por línea de comandos. Su objetivo no es el de luchar por ser la mejor suite completa BPM. **Activiti está más orientado a desarrolladores que a gestores y analistas de procesos**.

Te cuento cómo llegué hasta Activiti y por qué me decanté por él:

Lo primero de todo nos tenemos que remontar algunos años atrás, a finales de 2012 y principios de 2013. Por aquél entonces lo que yo buscaba era un framework para automatizar y gestionar los flujos de acción y navegación de una aplicación Web. Algo que me permitiese manejar los formularios y las acciones que provocaban esos formularios, controlar bifurcaciones condicionales, realizar bucles, controlar quién, cómo y cuándo podía acceder a esos formularios, etc. Hasta ese momento estas tareas las realizábamos de forma "tradicional", es decir con nuestra propia lógica gestionada por clases Java. Pero los flujos eran cada vez más complejos, empezaban a depender unos de otros, a necesitar varios caminos simultáneos, a implicar a más tipos de usuarios, a ser más exigentes, más cambiantes. En definitiva se hacían muy difíciles de mantener y extender. 

La primera alternativa que valoré fue usar Spring Web Flow, pero descubrí que realmente no era eso lo que necesitaba, yo necesitaba persistir el estado de esos flujos de navegación. No me servían los típicos ámbitos (scopes) Web. Es decir **realmente no necesitaba gestionar un flujo de navegación Web, sino un flujo de trabajo**.

Anteriormente había trabajado con máquinas de estados multithread y con XMI (XML Metadata Interchange) así que empecé a investigar para incluir algo similar en nuestro proyecto.

Descubro XPDL (XML Process Definition Language), un lenguaje que permitía definir diagramas de procesos y que estaba estandarizado por el WfMC (Workflow Management Coalition). La cosa tenía buena pinta así que empecé a buscar proyectos que soportasen esa notación a ver qué tal se comportaban. Probé [jXPDL](https://sourceforge.net/projects/jxpdl/) (Java XPDL model) y [Enhydra Shark](http://shark.ow2.org/doc/overview.html). Pero decidí seguir buscando más alternativas. 

La búsqueda de estos proyectos me lleva a conocer BPEL (Business Process Execution Language), un lenguaje estandarizado por OASIS (Organization for the Advancement of Structured Information Standards) y con el que se podían especificar acciones dentro de procesos de negocio, pero todo estaba muy orientado a los servicios web. Desde este momento se me abre un abanico de conceptos y términos que me empiezan a abrumar: BPML (Business Process Modeling Language), BPDM (Business Process Definition Metamodel), BPMS (Business Process Management System), ... y por supuesto BPM y la notación BPMN.

La notación BPMN se ajustaba a todas mis necesidades, parecía encontrarse en un estado muy maduro (versión 2.0) y estaba estandarizada por OMG (Object Management Group). Así que centré mi atención ahí.

Me encuentro con bastantes soluciones software, pero **me focalizo en jBPM, Bonita y Activiti ya que eran las más mencionadas en las comunidades de desarrollo** (Camunda aún era parte de Activiti y no existía como proyecto independiente). 

Comienzo probando **jBPM**, que tenía una importante trayectoria y estaba apoyado por JBoss. Pruebo la versión 5.4.0 (la última estable del momento). Los ejemplos básicos no me suponen problemas importantes, pero la verdad es que me resultó un poco tedioso de configurar y no me quedaba claro cómo podía aprovechar su motor de ejecución para mi proyecto. Me pareció muy completo pero a la vez muy complejo. Tenía bastantes dependencias. Estaba todo muy pensado para iniciar un proyecto desde cero y completamente basado en procesos de negocio y Java Drools, cosa que por el momento no necesitaba. Ofrecía multitud de herramientas, sí, pero yo sólo necesitaba el motor de workflow. La curva de aprendizaje parecía alta debido a su complejidad. Así que continúo buscando y dejo a jBPM "en la recamara" porque seguro que dedicándole más tiempo podría ser una buena alternativa.

**Bonita** lo descarté pronto ya que tenía un enfoque de cero-codificación. Proporcionaba muchas facilidades para construir sus procesos sin necesidad de ningún tipo de codificación, lo cual puede ser útil para analistas de procesos pero malo para mi ya que me dificultaba el control sobre su motor de workflow. No se ajustaba a mis necesidades. Además me costó encontrar documentación técnica y encima tenía licencia GPL/LGPL, lo que podría causar problemas cuando quisiera integrarlo en nuestro proyecto.

Y finalmente analizo **Activiti BPM Platform**. La última versión estable del momento era la 5.11, pero comienzo mi estudio con la versión 5.9 ya que, buscando documentación, me topé con el libro ["Activiti in Action"](https://www.manning.com/books/activiti-in-action), cuya publicación era bastante reciente (2012) y venía con ejemplos de código sobre esa versión. En poco tiempo conseguí grandes resultados. Su motor de procesos (Activiti Engine) era muy fácil de manejar y muy fácil de integrar en mi aplicación. Era un proyecto reciente (fundado en 2010), pero estaba financiado por Alfresco y creado por uno de los fundadores de jBPM, Tom Baeyens, y por Joram Barrez, unos de los "Core Enginer" de jBPM. Su comunidad era muy activa y tenía licencia [Apache 2.0](http://www.apache.org/licenses/LICENSE-2.0.html), que no es una licencia copyleft, lo que significa que era completamente libre en términos de uso y extensibilidad, cosa muy importante en un entorno de producción.

La verdad es que podía haber seguido investigando durante semanas, pero Activiti tenía todo lo que necesitaba y su curva de aprendizaje era rápida, así que me decanté por él.

Comencé a usar Activiti en producción a partir de su versión 5.13.

## ¿Qué ofrece Activiti?

A continuación citaré algunas características que ofrece Activiti a día de hoy, más allá de la mera implementación del estándar BPMN, y que pueden determinar si este framework encaja dentro de tus necesidades. Son muchos conceptos y cada uno podría dar para varios artículos. Tómate las siguientes líneas como el resumen de un estudio preliminar. Me he basado en la versión 5.21 lanzada recientemente. 

### Bases de datos soportadas

Activiti soporta DB2, H2, MySql, MS SQL, PostgreSql y Oracle, y usa internamente MyBatis como framework de acceso a su base de datos. Realiza sus operaciones de forma transaccional por lo que si una operación implica la ejecución de varias tareas, o se ejecutan todas o no se ejecuta ninguna. Además Activiti se integra con JTA (Java Transaction API), por lo que podemos gestionar todo el nivel de transaccionalidad desde nuestra aplicación.

### Tareas del estándar BPMN 2.0

#### User Task

Las tareas de usuario requieren de la intervención del usuario, generalmente a través de un formulario. Activiti añade tags XML al estándar BPMN que permiten especificar inputs de datos de tipo string, long, enum, date y boolean. Pero además, para formularios más complejos, permite vincular la tarea con una clase externa de renderización gráfica y así tener total libertad en cuanto a la implementación de los formularios: podemos usar JSF, Spring, Struts, Vaadin, Swing, AWT, etc.

También es posible usar [XForms](https://es.wikipedia.org/wiki/XForms). Aunque la integración no es directa, en [este artículo](https://saeidmirzaeitech.wordpress.com/2012/10/31/using-activiti-with-xform/) explican cómo hacerlo.

#### Service Task

Con las Service Task podemos asociar lógica Java a una tarea del proceso, lo cual nos abre las puertas a un sinfín de alternativas. Muchas de las extensiones y paquetes de integración que ofrece Activiti se basan en este tipo de tareas.

#### Script Task

Las tareas de script permiten añadir lógica a los procesos sin necesidad de escribir código Java. Los lenguajes de scripting que están disponibles por defecto en Activiti son JavaScript, Groovy y JUEL (Java Unified Expression Language), pero se puede utilizar cualquier motor de script compatible con la especificación JSR-223.

#### Business Rule Task

Permiten ejecutar de forma síncrona una o más reglas de negocio. Activiti utiliza [Java Drools](http://www.drools.org/) como motor de reglas. Los archivos .drl que contienen las reglas tienen que ser desplegados junto con la definición del proceso. También se puede utilizar cualquier otro motor de reglas, pero entonces no estaría representado como una "Business Rule Task", sino como una "Service Task" donde podremos gestionar las reglas como queramos.

#### Otras tareas

* Manual Task: se utiliza para representar un trabajo realizado por alguna entidad externa que el motor no necesita conocer. El workflow BPM continúa su ejecución automáticamente (según se inicia se completa sin causar ninguna interrupción).

* Receive Task: posibilita que el proceso se detenga a la espera de un mensaje determinado.

### Extensiones al estándar BPMN 2.0

#### Email Task

Permiten enviar correos electrónicos a uno o más destinatarios. Se implementa como una Service Task añadiendo tags XML específicos de Activiti que permiten indicar las propiedades típicas de correo (from, to, subject, cc, cco, contenido HTML, etc.).

El Activiti Engine se encargará de enviar estos emails a un servidor externo con capacidades SMTP previamente configurado.

#### Web Service Task

Permite invocar de forma síncrona un servicio Web externo. Es representada como una Service Task, pero activiti incluye una serie de tags XML que permiten especificar las entradas y salidas de los mensajes. Activiti usa el framework Apache CXF como cliente de servicios Web. Las tareas de servicios Web se encuentran en fase experimental.

#### Camel Task

[Apache Camel](http://camel.apache.org/) es un framework Java open source que facilita la integración de sistemas software. Con él podemos especificar reglas de enrutamiento para determinar las fuentes desde las que queremos recibir mensajes, cómo procesarlos y dónde enviarlos. Provee implementaciones de los principales patrones de integración empresarial (Enterprise Integration Patterns - EIP), conectividad a una gran variedad de APIs de transporte y facilidades para el uso de Lenguajes Específicos de Dominio (DSLs).

Al no ser parte del estándar BPMN 2.0 las incluye dentro de una Service Task, añadiendo ciertos tags XML específicos para Camel.

#### Mule Task

Mule es un framework de mensajería ESB (Enterprise Service Bus). Es un gestor de objetos escalable y distribuible que puede manejar interacciones con servicios y aplicaciones que usan distintas tecnologías de transporte y mensajería. Activiti añade tags XML para especificar propiedades de Mule (endpointUrl, language, payloadExpression, resultVariable).

#### Shell Task

Permite ejecutar instrucciones de línea de comandos. Los tags XML añadidos por Activiti permiten definir argumentos de entrada, la salida estándar de error, etc. Se implementa como una Service Task.

### Interfaz REST

Activiti provee una gran cantidad de operaciones de comunicación vía REST para poder usar todo el motor de procesos como servicio externo independiente. Pero además ofrece mecanismos para extenderlo e implementar nuestras propias operaciones.

### Integración de test JUnit

Está totalmente integrado con JUnit para la realización de los test. Simplemente tenemos que usar ActivitiRule (su propia implementación de TestRule) y Activiti se encargará de arrancar una base de datos en memoria (H2), ejecutar las sentencias SQL de creación de todo su modelo de datos, de poblar dicha BD con datos de prueba, de desplegar nuestro proceso y de ofrecernos todo su Activiti Engine para que sólo nos tengamos que preocupar de lo realmente importante: probar nuestro proceso. Cuando termine el test se cerrará la BD y se liberarán los recursos utilizados.

También ofrece mecanismos para hacer "dobles de test" (Mock) de sus tareas de servicio (Service Task).

### Integración con Spring framework

Activiti provee muchas características de integración con el framework de Spring, de hecho toda su configuración se suele realizar con el típico XML de Spring, el Bean Configuration File, procesado con spring-beans. El Activiti Engine puede ser configurado como un Spring Bean más y ser accesible desde el Spring Container.

Actualmente tienen en fase experimental una integración entre Activiti y Spring Boot.

### Integración con OSGi

OSGI (Open Services Gateway Initiative) es un conjunto de especificaciones que permite crear aplicaciones Java de forma modular. Indica cómo se deben crear esos módulos y cómo deben interactuar entre sí en tiempo de ejecución.

Cada módulo (bundle) no es más que un conjunto de clases empaquetadas en un .jar, que ofrece un servicio y que expone al exterior sus características a través de un fichero (MANIFEST.MF). Esos módulos serán desplegados en un contenedor que permita esa comunicación.

Por ejemplo el IDE Eclipse se basa en OSGi para permitir ampliaciones y personalizaciones mediante el desarrollo de plugins.

Activiti ofrece mecanismos para integrar sus procesos en este tipo de arquitecturas modulares. Está testeado para Apache Karaf OSGi container.

### Integración con JPA

Se pueden usar Entities JPA como variables del proceso (Activiti interpretará sus anotaciones), lo cual permite:

* actualizar esas Entities con variables del proceso que hayan sido rellenadas por el usuario a través de un formulario de una User Task.
* reutilizar un modelo existente de nuestra aplicación sin tener que escribir objetos de datos específicos para los procesos.
* poder tomar decisiones del flujo del proceso (bifurcaciones con Gateways) en base al contenido de nuestras Entities JPA.

### Integración con CDI

Podemos aprovechar y extender las capacidades de CDI añadiendo un simple .jar a nuestro proyecto: "activiti-cdi.jar". Con este módulo Activiti nos ofrece:

* Un ámbito (scope) llamado @BusinessProcessScoped donde los beans CDI serán gestionados a nivel de instancias de proceso. Esto nos permite asociar una conversación con una instancia de proceso.
* Podremos injectar en nuestra aplicación variables de procesos y cualquier servicio del motor (ProcessEngine, TaskService, HistoryService, ...). Podremos hacer cualquier operación sobre el motor sin tener que depender de su implementación con una simple inyección CDI.
* Integración con el bus de eventos de CDI. Podremos recibir eventos provocados por el proceso.
* Un EL-Resolver que permite utilizar Beans CDI dentro de los procesos, incluyendo EJBs. Podremos condicionar nuestro flujo de navegación en función del contenido de esos beans.
* Soporte para pruebas unitarias.

### Integración con LDAP

LDAP (Lightweight Directory Access) es un protocolo que provee servicios de directorio, organizando la información de forma similar a como lo hace un sistema de archivos, o el servicio de DNS en Internet. Funciona como una base de datos optimizada para las operaciones de lectura y búsqueda y permite, entre otras cosas, definir políticas de seguridad por cada nodo.

Activiti ofrece una extensión para configurar fácilmente una conexión con un sistema LDAP. 

### Integración con Alfresco

Por supuesto no podían faltar mecanismos para integrar Activiti con Alfresco, que es quien financia el proyecto. Por ejemplo las actividades AlfrescoStartEvent y AlfrescoUserTask facilitan el uso de formularios de Alfresco; y AlfrescoScriptTask y AlfrescoMailTask por su parte se integran con los parámetros de configuración de Alfresco.

### Simulación con CrystalBall

Activiti Crystalball es una extensión que permite realizar simulaciones, modelos y análisis estadísticos. Las simulaciones con CrystalBall nos pueden ser útiles para:

* La toma de decisiones: podemos simular cómo se comportarían nuestros procesos en un entorno de producción concreto y determinar si debemos añadir más recursos al sistema para cumplir los objetivos.
* Optimización: para probar los cambios y determinar su posible impacto.
* Formación: puede ser utilizado para formar al personal antes del despliegue.
* Mantenimiento: puede ser muy útil para reproducir errores que han ocurrido en producción y son difíciles de reproducir. Con esta extensión podemos usar la información historificada por Activiti y simular de nuevo lo que ocurrió. Y esto lo podemos reproducir una y otra vez hasta dar con la solución.

Esta extensión se encuentra actualmente en fase experimental.

### Conexión con JMX

JMX (Java Management Extensions) es una tecnología que suministra herramientas para la monitorización y administración de aplicaciones basadas en Java.

Es posible conectarse al motor de Activiti utilizando cualquier cliente que se ajuste a esta tecnología. De esta manera podemos obtener información, activar y desactivar el ejecutor de procesos, desplegar nuevas definiciones de procesos, eliminarlas, etc. y todo ello utilizando JMX y sin escribir una sola línea de código.

### Business Activity Monitoring (BAM)

Activiti no tiene una solución específica para Business Activity Monitoring (BAM), pero el libro "Activiti in Action" dedica una sección en torno a este aspecto y explica una forma de abordarlo integrando Activiti con Esper framework, un software Java open source que permite analizar los eventos provocados por el workflow de procesos. 

## Caso de implantación 

Todo nuestro entorno es Java EE y tenemos configurado Activiti como servicio EJB independiente, utilizando el patrón BCE (Boundary Control Entity), empaquetado en su propio .jar y desplegado en un servidor JBoss. 

Tenemos Oracle como sistema gestor de base de datos. Las tablas propias de Activiti están integradas junto a las del resto del sistema. Toda la persistencia de nuestra aplicación está gestionada con JPA e Hibernate, mientras que Activiti usa MyBatis. Esto me supuso dudas a la hora de valorar la integración de Activiti pero, gracias a la gestión de la transaccionalidad de JTA, se realizan de forma conjunta tanto las operaciones de Activiti como las de nuestro sistema sin ningún problema (si el proceso falla se hace Rollback de todas las operaciones realizadas por todos los servicios implicados).

Modelamos los procesos con el plugin de Eclipse Activiti Designer, que tenemos integrado en Eclipse Mars. 

A nivel front-end, los formularios presentados para las User Task los tenemos implementados con JSF y están enriquecidos con Richfaces, HTML5, jQuery, CSS3, etc.

Empezamos la implantación desplegando un proceso piloto para gestionar un flujo de trabajo interno, posteriormente fuimos adaptando a BPM otros procesos más críticos y actualmente tenemos en producción 30 definiciones (modelos) de proceso desplegadas (algunas ya van por la séptima versión). El sistema mantiene aproximadamente 600 instancias de procesos activas simultáneamente (procesos ejecutados que aún no han finalizado) y tiene historificados más de 24000 procesos finalizados.

### Mi momento de dudas con Activiti

A mediados de 2013, con la implantación de Activiti recién salida del horno, me entero de dos cosas:

1. Camunda, uno de los principales contribuyentes al proyecto desde los inicios, sale del proyecto para seguir su propio camino a partir de un fork de Activiti.
2. Y por otro lado Tom Baeyens, uno de los fundadores de Activiti, había dejado el proyecto para comenzar con la solución BPM basada en la nube Effektif.com (lo dejó en diciembre de 2012 pero me entero más tarde a raíz de conocer la noticia de Camunda).

Esto hace que me surjan dudas sobre la continuidad de Activiti y de si la decisión de implantarlo en nuestro sistema había sido correcta. Pero decido dejar pasar un tiempo y darle un voto de confianza ya que la alternativa de migrar a Camunda era aún más arriesgada debido a que el proyecto todavía se encontraban en fase Alpha (la primera versión estable fue publicada en agosto de 2013). 

Desde entonces hasta hoy no he notado ninguna degradación en la continuidad de Activiti. Joram Barrez sigue al frente del proyecto. La comunidad sigue siendo muy activa y contestan a tus preguntas con bastante rapidez. Siguen publicando actualización periódicamente (aproximadamente cada 4 meses, mientras que jBPM y Camunda lo hacen cada 6 meses). Y tienen previsto un cambio de versión importante: Activiti 6, que actualmente se encuentra en fase Beta2.

Lo que si echo en falta es una nueva edición de su libro "Activiti in Action", ya que sigue siendo la misma que leí en su día y me consta que hay muchos cambios y mejoras que merece la pena mencionar y más teniendo en cuenta que su autor, Tijs Rademakers, es "project lead" de Activiti. Supongo que estarán esperando al lanzamiento de la versión 6.

## Conclusiones

Ahora, tres años después de implantar Activiti, la gestión de procesos ha cobrado un gran protagonismo en nuestro proyecto. Ha pasado de ser una solución puntual a convertirse en uno de los pilares del sistema. Las posibilidades que ofrece BPM nos han permitido dotar a nuestros procesos de funcionalidades que de otra manera habrían sido excesivamente costosas: procesos que ejecutan y gestionan subprocesos, tareas que son interrumpidas cuando reciben un determinado mensaje de otro proceso, ejecución de tareas de forma asíncrona, tratamiento controlado de errores o historificación detallada de todo lo que va sucediendo son sólo algunos ejemplos de lo que BPMN y Activiti nos están aportando. Y todo esto se traduce en un mejor servicio a los usuarios.

Como técnico tengo muchas cosas que agradecer a Activiti, pero destacaría: 

* El amplio abanico de posibilidades de integración y comunicación que ofrece.
* Lo fácil que es usar y comprender su motor de workfkw. El Process Engine está implementado con el patrón "Method Chainig", que facilita enormemente la búsqueda de operaciones.
* Su buen rendimiento.
* La calidad de su comunidad. La mayoría de las dudas que me han surgido ya estaban planteadas en su foro. Las preguntas que he formulado me las han resuelto en un tiempo razonable. Y también puedes abrir incidencias o proponer mejoras a través de su Jira (activiti.atlassian.net). Todo esto da cierta seguridad en cuanto a su continuidad de cara al futuro.

Como dato de interés aquí te muestro la popularidad que tienen hoy (16/06/2016) en Github los proyectos mencionados en este artículo:

| Proyecto | Watch | Star | Fork | Enlace                                                                                             |
|----------|-------|------|------|----------------------------------------------------------------------------------------------------|
| Activiti | 322   | 1353 | 1495 | [https://github.com/Activiti/Activiti](https://github.com/Activiti/Activiti)                       |
| jBPM     | 152   | 480  | 673  | [https://github.com/droolsjbpm/jbpm](https://github.com/droolsjbpm/jbpm)                           |
| Camunda  | 67    | 225  | 233  | [https://github.com/camunda/camunda-bpm-platform](https://github.com/camunda/camunda-bpm-platform) |
| Bonita   | 41    | 45   | 51   | [https://github.com/bonitasoft/bonita-engine](https://github.com/bonitasoft/bonita-engine )        |

Como habrás observado, en este post menciono una gran variedad de conceptos técnicos que le dan mucha potencia y versatilidad al framework de Activiti. No he hablado de lo buena o mala que es su aplicación de modelado de procesos (Activiti Modeler), ni lo buena o mala que pueda ser su aplicación para la gestión y seguimiento de los procesos (Activiti Explorer) ya que no considero que estas sean sus principales armas y, en ese sentido, hay otras alternativas BPM mucho más completas. Aunque parece ser que Activiti 6 traerá consigo muchos [cambios y mejoras de interfaz gráfica](http://bpmn20inaction.blogspot.com.es/2015/09/getting-started-with-new-activiti-6-ui.html).

En definitiva, la experiencia con Activiti está siendo muy positiva.

En el próximo artículo ya por fin entraré en materia práctica: diseñaremos, implementaremos y ejecutaremos nuestro primer proceso con Activiti.

