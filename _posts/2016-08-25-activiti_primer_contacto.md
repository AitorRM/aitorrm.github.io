---
title: "Activiti BPM: Primera toma de contacto"
excerpt: "Ejemplo práctico de implementación de proceso BPM con Activiti."
header:
  teaser: "post_teaser_activiti_practica_01.jpg"
categories: 
  - BPM
tags:
  - BPM
  - Activiti
  - Procesos
  - Java
---

Hoy tendremos nuestra primera toma de contacto práctica con la implementación de procesos BPM usando el API de Activiti.

Si aún no lo has hecho te recomiendo que empieces leyendo los dos post teóricos que preceden a éste: primero el de [introducción a BPM]({% post_url 2016-05-13-bpm_y_java_open_source %}), donde repaso qué es, para qué sirve y qué beneficios aporta la disciplina BPM; y luego el post de [introducción a Activiti]({% post_url 2016-06-16-activiti_bpm_platform_implantacion %}), donde explico qué ofrece este API y qué experiencia tengo con él.

Empezaremos, como no podía ser de otra manera, con un test JUnit. Veremos un ejemplo rápido de diseño, despliegue, ejecución y testeo de un proceso muy básico.

Y sin más dilación... al turrón!!

## Requisitos

* Java 1.5 o superior
* Eclipse IDE con Plugin "Activiti Eclipse BPMN 2.0 Designer"
* Maven
* Conexión a internet

## Preparar el entorno de desarrollo

### Instalar Eclipse
En principio puedes trabajar con NetBeans, IntelliJ o cualquier otro IDE Java, pero Activiti ha creado un plugin para Eclipse que permite diseñar procesos de forma gráfica. Este plugin es bastante útil y sólo está disponible para Eclipse así que usaremos Eclipse para los ejemplos.

En [la página oficial de Activiti](http://www.activiti.org/userguide/#eclipseDesignerInstallation) aseguran la compatibilidad de su plugin con Eclipse Kepler e Indigo. Yo lo he probado con Juno, Luna, Mars y Neon y en todos ha funcionado sin problemas.

La última versión a día de hoy es Eclipse Neon y puedes descargarlo desde [este enlace](https://eclipse.org/).

### Plugin Activiti Eclipse BPMN 2.0 Designer

El plugin en cuestión se llama "Activiti Eclipse BPMN 2.0 Designer" y lo usaremos para diseñar los diagramas de procesos. Para Instalarlo hay que ir al menú superior "Help > Install New Software" (no aparece en el Marketplace) y luego pinchar en el botón "Add", donde pondremos:

* Name: Activiti BPMN 2.0 designer
* Location: http://activiti.org/designer/update/

La versión que a mí me aparece es la 5.18.0.201508100929, pensada a priori para generar procesos BPMN compatibles con la versión 5.18 de Activiti, pero que también son compatibles con la versión 5.21 que usaremos en este post.

## Crear y ejecutar el proyecto

Vamos a crear un proyecto Maven usando el archetype que nos ofrece Activiti en su [guía de usuario](http://www.activiti.org/userguide/#mavenArchetypes).

```
mvn archetype:generate -DarchetypeGroupId=org.activiti -DarchetypeArtifactId=activiti-archetype-unittest -DarchetypeVersion=5.21.0 -DgroupId=com.aezin.workshop.activiti.simpleprocess -DartifactId=activiti-bpm-simpleprocess -DinteractiveMode=false
```

Importamos el proyecto generado en nuestro Eclipse: File > import... > Existing Maven Projects (seleccionamos el proyecto "activiti-bpm-simpleprocess" que acabamos de crear).

Este proyecto contiene un pequeño ejemplo completo y totalmente funcional. No necesitamos hacer nada más para ver los primeros resultados, solo crear el proyecto y ejecutar el test, que nos permite probar el siguiente proceso:

![no-alignment]({{ site.url }}{{ site.baseurl }}/images/activiti/activiti_test01_process.png)

Este archetype fue creado por la comunidad de Activiti para facilitar la comunicación en su foro y para el reporte de incidencias. Cuando alguien tiene un problema de código y lo plantea en el foro lo primero que hace es crear este proyecto, implementar el test que demuestra su problema y adjuntarlo al foro junto con el comentario. De esta forma cualquiera puede descargarlo y ejecutarlo con facilidad para buscar la solución.

Veamos qué contiene.

## Análisis del código

El proyecto ha creado la siguiente estructura:

![no-alignment]({{ site.url }}{{ site.baseurl }}/images/activiti/activiti_test01_estructura.png)

Nos ha generado estos archivos: pom.xml, activiti.cfg.xml, log4j.properties, my-process.bpmn20.xml y MyUnitTest.java.

### Dependencias

El *pom.xml* de Maven tiene el siguiente aspecto:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
		http://maven.apache.org/xsd/maven-4.0.0.xsd">
		
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.aezin.workshop.activiti.simpleprocess</groupId>
	<artifactId>activiti-bpm-simpleprocess</artifactId>
	<version>1.0-SNAPSHOT</version>

	<dependencies>
		<dependency>
			<groupId>org.activiti</groupId>
			<artifactId>activiti-engine</artifactId>
			<version>5.21.0</version>
		</dependency>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.11</version>
		</dependency>
		<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
			<version>1.3.168</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>1.7.5</version>
		</dependency>
	</dependencies>

</project>

```

Nos incluye las siguientes dependencias al proyecto:

* *junit*:Para la realización de test unitarios.
* *slf4j*: Fachada de logging que permite elegir el framework concreto en tiempo de despliegue. Usaremos Log4j.
* *h2*: Base de datos relacional Java ejecutada en memoria. Es usada en los test para el despliegue y ejecución de los procesos.
* *activiti-engine*: Motor de ejecución de procesos (worflow) de Activiti.

### Configuración del motor de Activiti

Toda la configuración de Activiti se realiza a partir del fichero *activiti.cfg.xml*. A través de este archivo se especifican cosas como la configuración de la base de datos a usar, el nivel de historificación de información, las posibles capacidades de concurrencia, escuchadores de eventos, etc.

En nuestro caso únicamente se configuran aspectos de la base de datos. En concreto H2 Database.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" 
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans   
       	http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneInMemProcessEngineConfiguration">
    <property name="jdbcUrl" value="jdbc:h2:mem:activiti;DB_CLOSE_DELAY=1000" />
    <property name="jdbcDriver" value="org.h2.Driver" />
    <property name="jdbcUsername" value="sa" />
    <property name="jdbcPassword" value="" />
  </bean>

</beans>
```

### Configuración de Logging

No entraré en mucho detalle aquí. Solo comentar que usaremos Log4j para las trazas de log y que el fichero *log4j.properties* está configurado a nivel DEBUG.

```
log4j.rootLogger=DEBUG, CA

# ConsoleAppender
log4j.appender.CA=org.apache.log4j.ConsoleAppender
log4j.appender.CA.layout=org.apache.log4j.PatternLayout
log4j.appender.CA.layout.ConversionPattern= %d{hh:mm:ss,SSS} [%t] %-5p %c %x - %m%n


log4j.logger.org.apache.ibatis.level=INFO
log4j.logger.javax.activation.level=INFO
```

### Proceso BPMN

El fichero *my-process.bpmn20.xml* contiene el diseño del proceso en notación BPMN.

```xml
<?xml version="1.0" encoding="UTF-8"?>

<definitions xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:activiti="http://activiti.org/bpmn"
	xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI" 
	xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC"
	xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI" 
	typeLanguage="http://www.w3.org/2001/XMLSchema"
	expressionLanguage="http://www.w3.org/1999/XPath" 
	targetNamespace="http://www.activiti.org/test">

	<process id="my-process">

		<startEvent id="start" />
		<sequenceFlow id="flow1" sourceRef="start" targetRef="someTask" />
		
		<userTask id="someTask" name="Activiti is awesome!" />
		<sequenceFlow id="flow2" sourceRef="someTask" targetRef="end" />

		<endEvent id="end" />

	</process>

</definitions>
```

No es muy complicado de entender: tiene un proceso ```<process id="my-process">``` con un evento de inicio ```<startEvent id="start" />```, una tarea ```<userTask id="someTask"```, un evento de finalización ```<endEvent id="end" />``` y dos flujos de secuencia ```<sequenceFlow``` que enlazan los eventos a la tarea.

Sin embargo en procesos más elaborados este fichero XML puede resultar muy complejo de manejar. Aquí es donde entra en escena el plugin Activiti Designer para Eclipse. 

Prueba a pinchar con el botón derecho del ratón en el fichero y seleccionar "Open With > Other... > Activiti Diagram Editor". Te debería aparecer una ventana similar a esta:

![no-alignment]({{ site.url }}{{ site.baseurl }}/images/activiti/activiti_test01_editor.png)

Desde aquí puedes diseñar el proceso fácilmente: en la zona central aparece el proceso representado gráficamente, a la derecha tenemos la paleta de componentes que podemos usar y en la zona inferior las propiedades del componente seleccionado (en este caso, al no haber nada seleccionado, muestra las propiedades generales del proceso). Si no ves la pestaña de propiedades puedes incluirla desde "Window > Show View > Other... > Properties" (o también puedes abrir una perspectiva especial para Activiti desde "Window > Perspective > Open Perspective > Other... > Activiti").

Ahora prueba a realizar cualquier cambio en el diagrama, como por ejemplo desplazar los componentes. Guarda los cambios y vuelve a abrir el fichero *my-process.bpmn20.xml* sin el editor. Verás que el XML ahora contiene algunos tags "extraños" del tipo ```<bpmndi:...>```. 

```xml
<definitions ...>
  <process id="my-process" isExecutable="true">
    ...
  </process>
  
  <bpmndi:BPMNDiagram id="BPMNDiagram_my-process">
    <bpmndi:BPMNPlane bpmnElement="my-process" id="BPMNPlane_my-process">
      <bpmndi:BPMNShape bpmnElement="start" id="BPMNShape_start">
        <omgdc:Bounds height="35.0" width="35.0" x="20.0" y="47.0"></omgdc:Bounds>
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape bpmnElement="someTask" id="BPMNShape_someTask">
        <omgdc:Bounds height="72.0" width="105.0" x="110.0" y="29.0"></omgdc:Bounds>

      ...
      
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

Se trata de tags BPMN Diagram Interchange que sirven para definir aspectos de representación gráfica. Estos tags no afectan a la ejecución del proceso.

### Clase de test

Y por último tenemos la clase JUnit *MyUnitTest.java*. 

```java
package com.aezin.workshop.activiti.simpleprocess;
import org.activiti.engine.runtime.ProcessInstance;
import org.activiti.engine.task.Task;
import org.activiti.engine.test.ActivitiRule;
import org.activiti.engine.test.Deployment;
import org.junit.Rule;
import org.junit.Test;

import static org.junit.Assert.*;

public class MyUnitTest {

	@Rule
	public ActivitiRule activitiRule = new ActivitiRule();

	@Test
	@Deployment(resources = {"com/aezin/workshop/activiti/simpleprocess/my-process.bpmn20.xml"})
	public void test() {
		ProcessInstance processInstance = activitiRule.getRuntimeService().startProcessInstanceByKey("my-process");
		assertNotNull(processInstance);

		Task task = activitiRule.getTaskService().createTaskQuery().singleResult();
		assertEquals("Activiti is awesome!", task.getName());
	}

}
```

Aquí si hay algunas cosas bastante interesantes para comentar. 

Lo primero de todo destaca el uso de un ```@Rule``` específico de Activiti.

```java
@Rule
public ActivitiRule activitiRule = new ActivitiRule();
```

Gracias a la clase *ActivitiRule* podemos acceder a todo el motor de procesos de Activiti dentro de nuestros test. Pero no se queda sólo en eso, sino que también se encarga de configurar el motor al iniciar los test. Lee el fichero *activiti.cfg.xml*, configura la base de datos H2, etc. y deja todo listo para que nos centremos en nuestra prueba.

También llama la atención la anotación ```@Deployment```.

```java
@Deployment(resources = {"com/aezin/workshop/activiti/simpleprocess/my-process.bpmn20.xml"})
```

Esta anotación también es específica de Activiti y facilita la labor de despliegue de procesos BPMN. Únicamente hay que decirle donde está el recurso y Activiti se encargará de su despliegue a la base de datos.

Ya entrando en el cuerpo del test tenemos básicamente dos instrucciones. Una que nos permite ejecutar el proceso y crear una instancia a partir del diseño. Nótese el uso de *ActivitiRule* para acceder al servicio de ejecución o "Runtime".

```java
ProcessInstance processInstance = activitiRule.getRuntimeService().startProcessInstanceByKey("my-process");
```

Y por último otra instrucción que consulta las tareas activas. Aquí se hace uso del servicio *Task* para el tratamiento de tareas de usuario.

```java
Task task = activitiRule.getTaskService().createTaskQuery().singleResult();
```

En este caso solo existe una tarea activa y por eso podemos hacer un *singleResult*. Si tuviésemos varias tareas tendríamos que obtenerlas de la siguiente forma:

```java
List<Task> taskList = activitiRule.getTaskService().createTaskQuery().list();
```

## Ejecutar el proyecto

Ya tenemos todo preparado, lo único que nos queda es ejecutar la clase de test "MyUnitTest.java" y ver nuestra barrita JUnit en verde.

Recordemos que el log4j está configurado a nivel DEBUG, por lo que si observamos las trazas que aparecen por la consola veremos una importante cantidad de información sobre lo que ha ido ocurriendo. 

* Primero vemos que se construye el Process Engine.
* Luego se carga el XML de configuración (activiti.cfg.xml) usando JAXP.
* Se inicializa la base de datos H2 y se ejecutan una serie de consultas SQL de creación y población de datos para los test.
* Se parsea el fichero my-process.bpmn20.xml donde se define el proceso BPMN.
* Se despliega el proceso en la base de datos.
* Se ejecuta el código de nuestro test.
* Cuando termina el test se elimina el proceso de la BD y toda la información generada durante la prueba.
* Se cierra la BD y se finaliza la ejecución.

## Añadir un listener

Hemos visto todo lo que hace Activiti internamente con el test, pero... ¿qué ha pasado con el flujo de ejecución de nuestro proceso?. Lo único que sabemos gracias al test es que el proceso se ha ejecutado correctamente y que el nombre de la tarea actual tras esa ejecución es "Activiti is awesome!" pero, ¿ha finalizado esa tarea?, ¿ha finalizado el proceso?

La respuesta es no. Ha finalizado el test, pero con el proceso activo.

Una buena manera de saber todo lo que está pasando con nuestro proceso es a través de sus eventos. Vamos a registrar un listener que escuche todos los eventos que nos ofrece Activiti.

Primero ponemos el Log4j a nivel WARN (log4j.properties) para que no salgan tantas trazas.

```
log4j.rootLogger=WARN, CA
```

Luego creamos nuestra clase listener para que escuche los eventos de Activiti. Debe implementar la interface *ActivitiEventListener*. La colocaremos en el mismo paquete que la clase *MyUnitTest*.

```java
package com.aezin.workshop.activiti.simpleprocess;

import org.activiti.engine.delegate.event.ActivitiActivityEvent;
import org.activiti.engine.delegate.event.ActivitiEvent;
import org.activiti.engine.delegate.event.ActivitiEventListener;

public class MyEventListener implements ActivitiEventListener {

	public void onEvent(ActivitiEvent event) {
		switch (event.getType()) {
			case ACTIVITY_STARTED:
			case ACTIVITY_COMPLETED:
				System.out.println(event.getType() 
						+ ": " 
						+ ((ActivitiActivityEvent) event).getActivityType()
						+ " --> "
						+ ((ActivitiActivityEvent) event).getActivityId());
				break;
	
			default:
				System.out.println(event.getType());
		}
	}

	public boolean isFailOnException() {
		return false;
	}
}
```

Y por último cambiamos la configuración de Activiti (activiti.cfg.xml) para que sea consciente de la existencia de nuestro listener.

```xml
<beans ...>

  <bean id="processEngineConfiguration" ...>
    ...
    
    <property name="eventListeners">
      <list>
         <bean class="com.aezin.workshop.activiti.simpleprocess.MyEventListener" />
      </list>
    </property>
  </bean>

</beans>
```

Si ejecutamos ahora el test veremos lo siguiente por consola:

```
ENGINE_CREATED
ENTITY_CREATED
ENTITY_CREATED
ENTITY_INITIALIZED
ENTITY_INITIALIZED
HISTORIC_PROCESS_INSTANCE_CREATED
HISTORIC_ACTIVITY_INSTANCE_CREATED
ENTITY_CREATED
ENTITY_INITIALIZED
PROCESS_STARTED
ACTIVITY_STARTED: startEvent --> start
ACTIVITY_COMPLETED: startEvent --> start
HISTORIC_ACTIVITY_INSTANCE_ENDED
SEQUENCEFLOW_TAKEN
HISTORIC_ACTIVITY_INSTANCE_CREATED
ACTIVITY_STARTED: userTask --> someTask
ENTITY_CREATED
ENTITY_INITIALIZED
TASK_CREATED
ENTITY_DELETED
ACTIVITY_CANCELLED
ENTITY_DELETED
PROCESS_CANCELLED
HISTORIC_ACTIVITY_INSTANCE_ENDED
HISTORIC_PROCESS_INSTANCE_ENDED
ENTITY_DELETED
ENTITY_DELETED
```

Tenemos los eventos de todo lo que ha ido sucediendo. Pero nos interesan especialmente los eventos ```ACTIVITY_STARTED``` y ```ACTIVITY_COMPLETED```, que indican las actividades que han sido iniciadas y completadas. Las tareas, eventos de proceso, compuertas, etc. son actividades. Si nos centramos en esas actividades:

```
ACTIVITY_STARTED: startEvent --> start
ACTIVITY_COMPLETED: startEvent --> start
ACTIVITY_STARTED: userTask --> someTask
```

El evento *start* se inicia y se completa. La tarea *someTask* se inicia pero no se completa. Y el evento 
*end* ni si quiera se inicia (evidentemente).

Hagamos el último cambio al proyecto!

Vamos a ir a nuestra clase de test *MyUnitTest* y vamos a añadir la siguiente instrucción al final de método *test*:

```java
activitiRule.getTaskService().complete(task.getId());
```

Esta instrucción completará la tarea de nuestro proceso (por abreviar no nos vamos a entretener en poner *asserts* de JUnit). Ahora ejecutamos el test y...

```
ACTIVITY_STARTED: startEvent --> start
ACTIVITY_COMPLETED: startEvent --> start
ACTIVITY_STARTED: userTask --> someTask
ACTIVITY_COMPLETED: userTask --> someTask
ACTIVITY_STARTED: endEvent --> end
ACTIVITY_COMPLETED: endEvent --> end
```

Ahora si hemos iniciado y finalizado todo el proceso.

## Conclusiones

En este primer post práctico de Activiti BPM hemos visto cómo crear en pocos pasos un proyecto de test que nos permita desplegar y ejecutar un proceso BPMN. 

Hemos analizado cada uno de los ficheros y clases implicados en el proyecto, abordando conceptos de configuración de Activiti, diseño de procesos BPMN, ejecución y pruebas.

Hemos presentado el plugin "Activiti Eclipse BPMN 2.0 Designer", por lo que ya podemos empezar a jugar con el diseño de procesos BPMN.

Y hemos realizado pequeños cambios de código que nos permiten entender un poco mejor las entrañas de este API. 

En definitiva con este pequeño proyecto podemos empezar a trastear con el motor de ejecución de procesos (workflow) de Activiti e ir descubriendo todo lo que ofrece.
