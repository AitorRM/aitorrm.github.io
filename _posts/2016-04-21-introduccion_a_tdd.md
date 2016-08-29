---
title: "Introducción a TDD"
excerpt: "Introducción a Test Driven Development"
header:
  teaser: "post_teaser_tecnicas.jpg"
categories: 
  - Técnicas y Metodologías
tags:
  - TDD
  - Testing
---

Empezaremos por el principio: **¿Qué es eso del TDD?** Bueno, TDD es Test Driven Development, que en la lengua de Cervantes viene a decir "desarrollo dirigido por pruebas". Esto nos puede dar una idea inicial del asunto, por una parte tenemos "desarrollo" y por la otra "pruebas", así que lo primero que podríamos pensar es que TDD es una técnica para probar nuestro código. Vale, pues no. TDD va más allá y se podría definir como **una técnica para diseñar software de calidad apoyándose en el uso de ejemplos. Esos ejemplos son las pruebas.** Por lo tanto no se trata de una técnica de testing, ni de refactorización, se trata de una técnica de diseño.

> **Nota:** TDD no es una técnica de testing, ni de refactorización, es una técnica de diseño.


## Ciclo de desarrollo

La técnica de TDD se fundamenta en **tres conceptos: test, desarrollo y refactorización**. 
La idea a grandes rasgos es la siguiente:

1. Test: Hago el ejemplo (el test) que usa la funcionalidad que quiero implementar. Evidentemente este test fallará porque aún no tenemos creado el código.
2. Desarrollo: Escribo el código mínimo para que el ejemplo funcione. Ahora el test anterior debe funcionar.
3. Refactorización: Reviso y mejoro el código, pero siempre asegurándome de que el test sigue funcionando.

Y voy repitiendo este ciclo de forma iterativa por cada una de las funcionalidades que necesito desarrollar:

4. Hago un nuevo ejemplo para la nueva funcionalidad.
5. Escribo el código mínimo para pasar el test.
6. Refactorizo el código. Es posible que esta segunda funcionalidad sea muy similar a la primera y me encuentre con mucho código repetido. Es el momento de pensar en temas como la herencia, la delegación, división de responsabilidades, etc. Haremos estos cambios y nos aseguraremos de que siguen funcionando todos los test. Si para hacer funcionar los test nos vemos forzados a modificarlos puede ser un indicativo de que nuestro diseño no es del todo correcto.

> **Nota:** La secuencia es escribir un ejemplo de uso (test), desarrollar código que lo cumple y, por último, refactorizar.

Poco a poco iremos evolucionando nuestro código con cada nueva funcionalidad y, si nuestros test están bien construidos, siempre tendremos la certeza de no estropear nada. 


## La filosofía detrás de TDD

En TDD **cada test es un requisito**, o un caso de uso, y lo que haremos será crear código que cumpla esos requisitos. En cada iteración implementamos pequeños ejemplos hasta llegar a la arquitectura que necesitamos usar (conceptos muy relacionados con el desarrollo ágil). El diseño evoluciona en función de las necesidades y se mantiene simple y limpio. Además evitamos la típica tentación de escribir código "por si acaso", es decir código que hoy no es necesario, pero que creemos que podrá serlo en un futuro. Esas predicciones de futuro tienen muchas desventajas (aconsejo leer sobre los principios de desarrollo YAGNI, KISS y la navaja de Ockham).

Y si esto te sabe a poco aquí te resumo otras de las **ventajas** más importantes que aporta esta técnica:

- Mejora la calidad del software
- El código es altamente reutilizable.
- Mejora el trabajo en equipo y la confianza en los desarrollos de los compañeros.
- Ayuda a los departamentos de calidad.
- Reduce la necesidad de documentación.

La filosofía de TDD está muy relacionada con el mundo del desarrollo software, pero poco a poco está calando también a nivel de negocio y, a día de hoy, existen técnicas como el BDD (Behavior Driven Development o "Desarrollo Guiado por el Comportamiento") que amplía las ideas de TDD y las combina con otras ideas de diseño; y ATDD (Desarrollo Dirigido por Tests de Aceptación), también conocido como Story Test-Driven Development (STDD), que es igual que TDD pero a nivel de negocio: en este caso los test de aceptación aseguran que el software cumple los requisitos de negocio que el cliente demanda.


## Más información

Te animo a que busques más información sobre esto del TDD por internet y si realmente te interesa te recomiendo las siguientes lecturas:

- *"Diseño ágil con TDD" de Carlos Ble*. Es el primer libro publicado en castellano sobre TDD (de 2010). Es ideal para una lectura rápida y amena con la que afianzar conceptos. Su autor lo distribuye de forma gratuita en su versión online.
- *"Test-driven Development By Example" de Kent Beck*. Es el libro de TDD por excelencia. Kent Beck es uno de los creadores de TDD y de la metodología eXtreme Programming (XP). El libro es de 2002 pero es idóneo para entender los orígenes de esta técnica.
- *“Growing Object Oriented Software Guide By Test” de Stephen Freeman y Nat Pryce*. Este libro amplía el concepto de TDD con un doble bucle y enfatiza en los roles, responsabilidades e interacciones.


