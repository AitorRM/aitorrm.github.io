---
title: "Especificaciones JSR para CDI"
header:
  teaser: "post_teaser_especificacion.jpg"
categories: 
  - Java EE
tags:
  - Java
  - CDI
  - JSR
---

CDI (Contexts and Dependency Injection) es un conjunto de servicios pensado para facilitar la interconexión de la capa de presentación web y la capa transaccional de una aplicación Java EE. Se posiciona entre la tecnología JavaServer Faces (JSF) y tecnologías de servicio como los Enterprise Java Beans (EJB), persistencia de datos (JPA) y transaccionalidad (JTA). De hecho a los Beans CDI originalmente se les denominaba Web Beans. Pero actualmente CDI tiene usos mucho más amplios y permite a los desarrolladores una gran flexibilidad para integrar diversos tipos de componentes de una manera desacoplada pero typesafe (segura ante tipos: un programa no puede realizar una operación en un objeto a menos que la operación sea válida para ese objeto). 

Principalmente proporciona los siguientes servicios:

* *Contextos*: Este servicio permite enlazar e interactuar con los componentes que gestionan el ciclo de vida de una aplicación Java EE.

* *Inyección de dependencias*: Este servicio permite inyectar componentes en una aplicación y elegir la implementación particular a instanciar en tiempo de despliegue.

Así pues **los servicios principales de CDI son la inversión de control (IoC) y la inyección de dependencias (DI)**. Si no tienes claros estos conceptos o no los diferencias [aquí te los explico]({% post_url 2016-05-24-dip_di_ioc %}) con más detalle.

CDI es una especificación y por lo tanto Java deja en manos de terceros su implementación. Es algo parecido a lo que ocurre con JPA (Java Persistence API), que tiene varias implementaciones: Hibernate, EclipseLink, etc. O con JSF y las implementaciones de Mojarra y MyFaces.

La implementación de referencia para CDI es [Weld](http://weld.cdi-spec.org/). Existen otras como Resin CanDI y Apache OWB (OpenWebBeans), pero Weld es la implementación que usan los principales servidores Java EE como GlassFish y Wildfly (JBoss). Weld también puede ser integrado en servidores servlet container como Tomcat o Jetty.

# Evolución de las especificaciones CDI

A continuación haré un breve repaso de la evolución que ha tenido la especificación de CDI desde que fue incorporado en Java EE 6.

## CDI 1.0

Java EE 6 incorpora por primera vez una especificación para aplicar inyección de dependencias (Dependency Injection, DI) en aplicaciones Java EE. Se crean dos especificaciones: JSR-330 y JSR-299.

* [JSR 330: Dependency Injection for Java](https://jcp.org/en/jsr/detail?id=330): Es una especificación muy simple y define únicamente la semántica básica de inyección de dependencias. Incluye el paquete javax.inject que contiene los siguientes elementos: Inject, Named, Provider, Qualifier, Scope y Singleton. Su propósito es maximizar la reutilización, la capacidad de prueba y mantenimiento de código Java mediante la estandarización de una API de inyección de dependencias extensible.

* [JSR 299: Contexts and Dependency Injection for the Java EE platform](https://jcp.org/en/jsr/detail?id=299): Esta especificación amplía y mejora considerablemente la JSR-330 añadiendo capacidades de inversión de control (IoC) con mecanismos como los interceptores, decoradores, ámbitos personalizados y un modelo de notificación de eventos. El propósito de esta especificación es unificar la gestión de beans JSF y EJB, lo que resulta en un modelo de programación simplificado de manera significativa para las aplicaciones basadas en Web. JSR-299 es una capa superior de JSR-330. JSR-299 usa internamente JSR-330, pero JSR-330 puede ser usado sin JSR-299. 

Las implementaciones que soportan CDI 1.0 son Weld 1.1 y Apache OWB 1.2

## CDI 1.1

En Java EE 7 se amplía CDI 1.0 con las actualizaciones y aclaraciones más solicitadas por la comunidad Java.

* [JSR 346: Contexts and Dependency Injection for Java EE 1.1](https://jcp.org/en/jsr/detail?id=346): Se añaden mejoras para considerar CDI como un mecanismo eficaz que permita unificar el modelado de todos los componentes Java EE y alinear las demás especificaciones alrededor de ella para hacer una plataforma más cohesionada. Esto incluye alinear los alcances (scopes) de JSF a CDI, desacoplar la transaccionalidad del modelo de componentes EJB a través de JTA 1.2 @Transactional CDI interceptor, la modernización de la API JMS 2 utilizando CDI, un mejor soporte para CDI en Bean Validation 1.1 y muchos otros.

La implementación que soporta CDI 1.1 es Weld 2.0

## CDI 1.2

En 2014 la especificación 1.1 de CDI es actualizada por una versión de mantenimiento. Esta versión de mantenimiento, conocida como CDI 1.2, trae bastantes aclaraciones, correcciones y pequeños cambios de comportamiento, pero no afectan lo suficiente a la especificación JSR como para necesitar una nueva versión, así que CDI 1.2 sigue vinculado a la JSR-346. [Aquí](http://www.cdi-spec.org/news/2014/04/14/CDI-1_2-released) puedes acceder al anuncio de esa nueva versión.

Las implementaciones que soportan CDI 1.2 son Weld 2.0 y Apache OWB 1.6

## CDI 2.0

Con Java 8 aparece la última actualización de la especificación CDI a la fecha de publicación de este post.

* [JSR 365: Contexts and Dependency Injection for Java 2.0](https://jcp.org/en/jsr/detail?id=365): La última versión de Java evoluciona CDI con el principal objetivo de añadir modularidad para facilitar su integración con otras especificaciones o frameworks y para dar soporte a Java SE. De hecho renombran el título de la especificación y donde ponía "Java EE" ahora ponen simplemente "Java". Este JSR se encuentra aún en fase "Early Draft Review".

La implementación que soporta CDI 2.0 es Weld 3.0.0.Alpha13

Desde [este enlace](http://www.cdi-spec.org/download/) puedes descargarte todas las especificaciones comentadas en este artículo, incluidos los JavaDoc.
