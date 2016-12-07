---
title: "El camino hacia una arquitectura software limpia - Hexagonal, Onion y Clean Architecture"
header:
  teaser: "post_teaser_architecture.jpg"
categories: 
  - Técnicas y Metodologías
tags:
  - Arquitectura Software
  - Principios
  - Buenas prácticas
---

¿Te interesa la arquitectura software? Espero que sí porque éste es uno de esos aspectos que deberían interesarte sea cual sea tu nivel de experiencia dentro del mundo del desarrollo software. Si eres programador junior deberías interesarte por ello lo antes posible si quieres evolucionar como profesional. No puedes considerarte senior o analista sin conocimientos del tema. Un jefe de equipo, un Scrum Master, un QA, un CTO o cualquier rol de negocio necesitará conocer en mayor o menor medida aspectos de la arquitectura de los sistemas si quiere comprender su alcance, asegurar su mantenibilidad, la capacidad de reutilización de componentes, calidad del código, escalabilidad, coste de futuras migraciones, valorar dependencias tecnológicas, ampliaciones, etc. 

Vale, está claro que la arquitectura es importante, pero cuando lees "Arquitectura Software" ¿qué te viene a la cabeza? Quizás pienses *"me tengo que poner las pilas con..."* y te vengan palabras del tipo REST, SOAP, frameworks como Spring, Vaadin o JSF, almacenamiento de datos con Hibernate, MongoDB, Elasticsearch, o quizás otras de tipo NodeJS, Websockets, Angular e incluso es posible que pienses en Docker, Amazon AWS... pues en mi opinión creo que vas mal encaminado. Todo esto no son más que tecnologías, herramientas. Quién te dice dónde estarán dentro de 5 años, o incluso dentro de 5 meses. Tu arquitectura debe estar preparada para adaptarse a los cambios. "Modularidad" y "lógica de negocio" serán las palabras clave.

> Lo primero que necesitamos es una forma de estructurar el código de aplicaciones empresariales con alto nivel de complejidad, de forma que se maximice la modularidad y la separación de responsabilidades. Luego vendrán las tecnologías.

En los últimos años se está hablando mucho sobre los conceptos de Hexagonal Architecture, Onion Architecture y Clean Architecture. Se confunden y se asumen como una única cosa, pero no son exactamente lo mismo. Tienen un orden cronológico de aparición y cada uno se basa en el anterior e intenta mejorarlo.

El último de ellos, Clean Architecture, propuesto por Robert C. Martin (Uncle Bob) volverá a sonar muy fuerte en el próximo año 2017 debido a la publicación de su [próximo libro](https://www.amazon.es/Clean-Architecture-Robert-C-Martin/dp/0134494164) para finales de este mes de diciembre.

En este post te haré un resumen introductorio de estos tres conceptos que inician el camino de baldosas amarillas hacia una arquitectura software limpia.

## Hexagonal Architecture

En 2005 Alistair Cockburn propone el concepto de "[Hexagonal Architecture](http://alistair.cockburn.us/Hexagonal+architecture)", también denominado arquitectura de "Puertos y Adaptadores" (Ports & Adapters).

La idea, a groso modo, se fundamenta en la construcción de sistemas software basados en una serie de puertos (interfaces públicas de acceso) y unos adaptadores (implementaciones de esas interfaces para un contexto concreto) que se comunican con el núcleo de la aplicación, que es donde está toda la lógica de negocio.

![Hexagonal Architecture](http://4.bp.blogspot.com/-ugWyu8WPbVM/TdZQyfFqHHI/AAAAAAAAAb4/8T7ehzOcIN8/s1600/figure2.png)

Fuente: [http://blog.kennardconsulting.com/2011_05_01_archive.html](http://blog.kennardconsulting.com/2011_05_01_archive.html)

Con esto se persigue implementar la lógica de negocio (el núcleo) de forma aislada e independiente de cualquier agente externo. Los puertos y adaptadores son los puntos de entrada, salida y conversión de datos. Existirá un adaptador para cada agente externo: un adaptador para la interfaz de usuario (UI), otro para la persistencia, otro para la ejecución de tests, otro para la comunicación con otros servicios, etc. De esta manera se consiguen sistemas más reutilizables, más modulares y más mantenibles. Y la forma de conseguir este desacoplamiento es a través del **principio de inversión de dependencias** (del cual ya hablé con mayor profundidad en [este artículo]({% post_url 2016-05-24-dip_di_ioc %}).

> Crea tu aplicación para que funcione sin interfaz de usuario ni base de datos, únicamente para que pueda ser ejecutada por pruebas automáticas de regresión.

Sin embargo, este concepto por sí solo, se me antoja demasiado abstracto. La arquitectura hexagonal no establece ningún tipo de regla estructural de código. Tampoco plantea restricciones de interacción entre componentes. ¿Un componente de UI puede comunicarse directamente con un componente de persistencia? ¿o debe hacerlo a través del núcleo? Quizás deberíamos asumir todo esto como un marco conceptual o conjunto de buenas prácticas más que como una arquitectura propiamente dicha. Te invito a leer [este interesante y crítico artículo](http://www.javiervelezreyes.com/ni-nueva-ni-arquitectura-ni-hexagonal/) al respecto.

## Onion Architecture

En 2008 Jeffrey Palermo introduce un nuevo concepto al que llama "[Onion Architecture](http://jeffreypalermo.com/blog/the-onion-architecture-part-1/)".

El propio autor lo define como "un patrón arquitectónico" que pretende evitar uno de los mayores inconvenientes del uso de las tradicionales arquitecturas de tres capas: el acoplamiento entre las capas. Según Jeffrey el enfoque tradicional crea sistemas donde la interfaz de usuario no puede funcionar sin la lógica de negocio, y la lógica de negocio no puede funcionar sin el acceso de datos. Y como consecuencia la interfaz de usuario está acoplada al acceso a datos.

Para evitar éstos acoplamientos propone una arquitectura basada en capas circulares a modo de cebolla. Todo este sistema de capas estará restringido por una regla fundamental: todo el código puede depender de las capas más centrales, pero no puede depender de las capas más alejadas del núcleo. Es decir, todo el acoplamiento es hacia el centro.

![Onion Architecture](http://jeffreypalermo.com/files/media/image/WindowsLiveWriter/TheOnionArchitecturepart1_70A9/image%7B0%7D%5B59%5D.png)

Fuente: [http://jeffreypalermo.com/blog/the-onion-architecture-part-1/](http://jeffreypalermo.com/blog/the-onion-architecture-part-1/)

Cada capa tendrá una responsabilidad concreta:

* **Núcleo de aplicación**:

  * **Modelo de Dominio**: En el centro estará el modelo de dominio, que representa los objetos y estados de la organización.

  * **Servicios de Dominio**: Interfaces que definen comportamientos y operaciones sobre el modelo de dominio.

  * **Servicios de Aplicación**: Interfaces que definen comportamientos específicos de la aplicación.

* **Elementos Externos**: En la parte más externa se encuentra la **interfaz de usuario**, la **infraestructura** y el sistema de **pruebas**. La capa exterior está reservada para las cosas que pueden cambiar más a menudo. Estas cosas deben estar aisladas del núcleo de la aplicación. Por ejemplo, aquí se implementarían las interfaces definidas en las capas de servicio. Estas implementaciones sí tendrán dependencias tecnológicas.

La arquitectura de cebolla se basa en estos cuatro principios:

* La aplicación está construida en torno a un modelo de objetos independiente.
* Las capas interiores definen las interfaces. Las capas exteriores implementan esas interfaces.
* La dirección del acoplamiento es siempre hacia el centro.
* Todo el núcleo de código de la aplicación se puede compilar y ejecutar de forma aislada a la infraestructura.

Tanto la arquitectura hexagonal como la arquitectura de cebolla comparten la siguiente premisa: externalizar aspectos de infraestructura y la existencia de código de adaptación que permita independizar esa infraestructura.

## Clean Architecture

Y por último, en 2012, Uncle Bob, en su ya famoso [artículo](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html) introduce el concepto de "Clean Architecture".

> "La primera preocupación de un arquitecto es asegurarse de que la casa es utilizable, no asegurarse de que la casa está hecha de ladrillo."
> (Uncle Bob)

Uncle Bob se da cuenta de que, en definitiva, todos estos conceptos y arquitecturas se fundamentan en una misma idea: la separación de capas. Y focaliza su atención en que todas coinciden en dos cosas: en considerar a la capa de lógica de negocio como el eje principal del sistema y en definir una serie de interfaces para comunicarse con el resto del sistema. Puedes llamarlo Dominio, Business Logic, Core, Application o como quieras, pero en cualquier caso esta capa es la más importante, la que maneja los hilos, y todo debe estar diseñado y modularizado en torno a ella. Así que Uncle Bob decide englobar estas propuestas en un único concepto al que denomina "Clean Arquitecture" y resume las **características** que debe tener un sistema construido con este tipo de arquitectura:

* **Independiente de frameworks**. La arquitectura no debe depender de ningún framework ni librería cuyas características condicionen nuestro sistema a sus requisitos y restricciones. Los frameworks deben tomarse como herramientas de apoyo.

* **Testeable**. La lógica de negocio se debe poder testear sin necesidad de ningún ajente externo (interfaz de usuario, base de datos, servidor web, etc.).

* **Independiente de la UI**. La interfaz gráfica de usuario debe poder ser reemplazable con facilidad sin afectar al resto del sistema (por ejemplo, cambiar una UI web por una UI de escritorio, e incluso por una interfaz de consola, no debe suponer ningún cambio para la lógica de negocio).

* **Independiente de la base de datos**. De forma similar a lo que sucede con la UI, la base de datos también debe ser fácilmente reemplazable. Las reglas del negocio deben permanecer ajenas al sistema de persistencia, le da igual que usemos Oracle, SQL Server, MongoDB o un simple sistema de ficheros.

* **Independiente de factores externos**. En definitiva, la lógica de negocio no debe tener conocimiento directo de nada que provenga del mundo exterior (el modo de comunicación con otros sistemas -REST, SOAP, RMI, etc.-, el servidor donde se ejecuta -Wildfly, Glassfish, Tomcat, etc.-).

Siguiendo con su intención de unificar criterios, Uncle Bob integra todas estas arquitecturas en una sola idea representada por el siguiente esquema:

![Clean Architecture](https://8thlight.com/blog/assets/posts/2012-08-13-the-clean-architecture/CleanArchitecture-8b00a9d7e2543fa9ca76b81b05066629.jpg)

Fuente: [https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html)

Cada círculo representa un **área** del software:

* **Enterprise Business Rules**: Área que encapsula la lógica de negocio a nivel empresarial. Aquí estarán los objetos, estructuras de datos y/o funciones que permiten modelar toda la lógica empresarial. *Entidades* del tipo Persona, Grupo, Pedido, Línea de Pedido, Factura, etc. formarán parte de esta capa. Ojo, no confundir con las Entities de JPA!!

* **Application Business Rules**: Área que contiene las reglas de negocio específicas de la aplicación. Aquí se implementan los *casos de uso* que hace nuestra aplicación sobre las entidades empresariales. Mientras la capa anterior se ocupaba de los actores, ésta se ocupa de las interacciones (asociar una Persona como miembro de un Grupo, crear un Pedido con Líneas de Pedido...). Los cambios en esta capa no deberían afectar a las entidades, en cambio una modificación en las entidades si puede afectar a los casos de uso.

* **Interface Adapters**: Área que contiene un conjunto de adaptadores donde se convierten datos entendibles por los casos de uso a datos aceptados por elementos externos, y viceversa. Por ejemplo, para comunicarse con la interfaz gráfica necesitaremos tener *Presentadores* que se encarguen de convertir objetos de nuestra lógica de negocio a objetos específicos del framework que estemos usando para la UI (JSF, Spring, Swing, Vaadin...). Con respecto al sistema de persistencia tendremos *Repositorios* que realicen esa adaptación en función del sistema de persistencia utilizado (ORMs, JPA, Hibernate, MyBatis...). Otro ejemplo serían los convertidores de servicios SOAP, o estructuras JSON, proporcionadas por algún servicio externo.

* **Frameworks y Drivers**: Área más externa que se compone de frameworks y herramientas como la base de datos, el framework web, sistemas de testing, etc. Aquí van todos los detalles. La web es un detalle. La base de datos es un detalle. Mantenemos estas cosas fuera donde no pueden hacer mucho daño.

Y por último aquí indico algunas **consideraciones, reglas y restricciones** que plantea esta arquitectura:

* Las dependencias solo deben apuntar hacia dentro, a capas o círculos interiores. Una capa interior no debe saber nada de una capa de nivel superior.
* El control de flujo entre capas se consigue aplicando el principio de inversión de dependencias. Gracias a este principio (del que también es autor Uncle Bob) podemos conseguir que las dependencias de código fuente no afecten al flujo de control
* Los datos que cruzan los límites entre capas son estructuras de datos simples o DTO (Data Transfer Objects). Por ejemplo, entidades o filas de una base de datos no podrían atravesar estas fronteras ya que transmitiríamos dependencias, aunque sean simples anotaciones (como ocurre con las Entities de JPA dentro del mundo Java EE).
* No tienen porqué ser estrictamente 4 niveles, pueden ser más siempre que se cumplan las reglas anteriores.

Como podrás comprobar tampoco dice nada nuevo, pero si pone un poco de orden en toda esta idea del desacoplamiento a partir del principio de inversión de dependencias.

## Conclusiones

De momento, el concepto de "Clean architecture", parece básicamente una recopilación de mejores prácticas basada en otros conceptos e ideas previas (y en la propia experiencia del autor). La principal mejora que parece haber aportado a la comunidad es una definición más detallada y precisa de las capas, reglas, restricciones, consideraciones y características que debe tener una arquitectura software para poder ser considerada "clean".

Pero la palabra "clean" va mucho más allá, tanto como para escribir un libro. El señor Uncle Bob ya tiene en su bagaje éxitos como ["Clean Code"](https://www.amazon.es/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882/ref=sr_1_1?s=foreign-books&ie=UTF8&qid=1481121550&sr=1-1&keywords=clean+code) o ["Clean Coder"](https://www.amazon.es/Clean-Coder-Conduct-Professional-Programmers/dp/0137081073/ref=sr_1_1?s=foreign-books&ie=UTF8&qid=1481121577&sr=1-1&keywords=clean+coder). Y ahora cierra la trilogía con el libro ["Clean Architecture"](https://www.amazon.es/Clean-Architecture-Robert-C-Martin/dp/0134494164) donde, a parte de explicar con mayor profundidad lo visto en este post, dará respuesta a preguntas del tipo ¿cuáles son los principios básicos de la arquitectura de software?, ¿cuál es el papel de un arquitecto software?, ¿qué hace que una arquitectura vaya mal y qué podemos hacer al respecto? y un largo etcetera.

Bajo mi punto de vista la principal ventaja de este tipo de planteamientos es la de **poder posponer decisiones tecnológicas**, lo cual es una consecuencia de la modularización. Podremos comenzar a implementar el núcleo tan pronto como sepamos de qué va la aplicación. Imagina que comienzas un proyecto. Aún no está claro si será una aplicación Web o una APP para Smartphone. Tampoco se sabe si se quieren persistir los datos en un modelo relacional o NoSQL. Lo que sí se sabe es que tendrá que manejar usuarios, pedidos, facturas, un carro de la compra... ¡pues para qué esperar! empecemos a implementar el núcleo y vayamos ganando tiempo hasta que negocio se aclare.

Otra de las principales ventajas es **poder minimizar el impacto de los cambios**. Esto también es una consecuencia de la modularización, de tener todo bien separadito y las responsabilidades bien repartidas.

Por otro lado, hay que tener presente que una arquitectura de este tipo **tiene sentido en aplicaciones grandes**. En una aplicación pequeña y sin perspectivas de ampliación puede ser incluso contraproducente. No tiene sentido complicar las cosas más de la cuenta.
