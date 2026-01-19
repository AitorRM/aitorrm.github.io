---
title: "Interfaces en Java 8, ¿Traits? ¿Herencia múltiple?"
header:
  teaser: "/images/post_teaser_interface.jpg"
categories: 
  - Java 8
tags:
  - Java
  - Interface
  - Default Method
  - Static Method
---

Java 8 añade una nueva característica a las interfaces: la posibilidad de implementar código en el cuerpo de métodos. Son los llamados *métodos por defecto* y *métodos estáticos*.

¿Cómo funciona esto? ¿Qué implicaciones tiene? ¿Por qué añaden esta característica ahora? ¿Cuándo es conveniente usarlo? Recordemos que una clase puede implementar múltiples interfaces, ¿significa esto que ahora Java soporta herencia múltiple como C++?

En esta entrada explicaré esta nueva característica, comentaré algunos conceptos teóricos que conviene conocer, intentaré aclarar otros y daré mi opinión sobre el tema.

## Interfaces en Java 8

Hasta Java 7 una interface solo podía contener declaraciones de métodos, constantes y otras interfaces internas anidadas.

A partir de Java 8 además pueden contener métodos por defecto y métodos estáticos.

```java
public interface MiInterface {
	// una constante
	static final String MI_CONSTANTE = "miConstante";
	
	// una declaración de método (método abstracto)
	void operacion();
	
	// una interface interna
	interface MiInterfaceInterna {
		void operacionInterna();
	}
	
	// un método por defecto (con cuerpo)
	default String operacionDefecto(String texto) {
		return "operación por defecto";
	}
	
	// un método estático (con cuerpo)
	static String operacionEstatica(String texto) {
		return "operación estática";
	}
}
```

Todos los métodos declarados en una interface son implícitamente públicos, así que podemos omitir el modificador *public*.

### Métodos por defecto:

Veamos para qué pueden ser útiles estos métodos por defecto.

Supongamos que tenemos la siguiente interface que define una operación.

```java
public interface MiInterface {
	void operacion();	
}
```

Esta interface la metemos dentro de una librería y le ponemos la versión 1.0.

Ahora tenemos un proyecto que usa esa librería, creamos una clase que implementa *MiInterface* y le damos cuerpo a la operación.

```java
public class MiClase implements MiInterface {

	@Override
	public void operacion() {
		System.out.println("Realizar operación");
	}
}
```

Pasado un tiempo resulta que aparece una nueva necesidad y tenemos que añadir otra operación a *MiInterface*. Pues la añadimos a la interface y actualizamos la versión de nuestra librería a 1.1

```java
public interface MiInterface {
	void operacion();
	void otraOperacion();
}
```

¿Qué supone esto? Pues que ahora, con la nueva versión de la librería, tenemos que modificar la clase *MiClase* e implementar la nueva operación ya que de lo contrario el proyecto no compilará.

Este escenario está más o menos controlado: nosotros creamos la libraría y nosotros la usamos. No hay mayor problema.

Ahora digamos que la librería en cuestión la hemos ofrecido a terceros, o la hemos publicado como proyecto open source. El escenario cambia. No sabemos cuanta gente la ha usado ni cuantas clases existen por ahí implementando la interface *MiInterface* en su versión 1.0. Si publicamos la nueva versión 1.1 todo el mundo verá errores de compilación en sus proyectos, tendrán que investigar el motivo, nos llamarán cabreados, nos criticarán por lo foros... ¿qué hacemos?

La solución que plantea Java 8 es usar los métodos por defecto. La versión 1.1 de *MiInterface* quedaría así:

```java
public interface MiInterface {
	void operacion();
	
	default void otraOperacion() {
		// Hacer un trabajo por defecto
	}
}
```

Ahora la gente podrá actualizar sus proyectos con la versión 1.1, el código compilará perfectamente y, si quieren, pueden implementar un comportamiento concreto para esa nueva operación.

### Métodos estáticos:

Los métodos estáticos son métodos que se asocian con la clase en la que se implementa la interface. Cada instancia de la clase comparte esos métodos estáticos. Java 8 añade esta característica para facilitar la organización de los métodos de ayuda en las bibliotecas (tipo "utils", "helper", "manager", etc.). Ahora se pueden mantener los métodos estáticos específicos de una interfaz en la propia interfaz en lugar de en una clase separada.


Aquí puedes ver la [documentación oficial de Oracle](http://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html).

No tiene mucho más misterio, ¿no?... ¿o sí?

## Un poco de teoría de POO

Hagamos un repaso rápido a algunos conceptos de la programación orientada a objetos (POO):

**Una Interface** es un conjunto de declaraciones de funciones que sirve para definir un comportamiento. Es un protocolo, un contrato, un compromiso, donde se indican las operaciones que soporta la clase que lo implementa. Las interfaces dicen qué hace una clase, pero no cómo lo hace. **Sirven para tipificar una clase** (las clases que implementan una interfaz son compatibles con este tipo). No permiten implementación de funciones ni atributos, solo declaraciones de funciones (y constantes).

**Un [Trait](https://en.wikipedia.org/wiki/Trait_(computer_programming))** es un concepto que representa un conjunto de métodos que se pueden utilizar para **ampliar la funcionalidad** de una clase. Su objetivo es agrupar funcionalidades muy específicas de manera coherente. No permiten extender el comportamiento del estado de una clase, es decir no permiten atributos (que son los que definen ese estado). Solo permite implementaciones de métodos. Lenguajes como Scala los soportan.

**Un [Mixin](https://en.wikipedia.org/wiki/Mixin)**, es una clase que **ofrece cierto comportamiento para ser heredado** por una subclase, pero no está ideada para ser autónoma (no se puede instanciar). Heredar de un mixin no es una forma de especialización sino más bien un medio de obtener funcionalidad que puede afectar al propio estado de la clase. Permite atributos e implementaciones de métodos pero evita muchos de los problemas de la herencia múltiple convencional.

Y por último, **la herencia múltiple** es aquella en la que **cada clase puede heredar métodos y atributos de cualquier número de superclases**. Permiten relaciones de extensión, especialización y combinación entre clases y superclases. Lenguajes como C++, ObjetiveC, Smalltalk, Eiffel, etc. lo soportan.

## La teoría aplicada a Java

**En Java (incluido Java 8) la herencia múltiple de clases no está permitida**: una clase solo puede heredar de una superclase. Hay maneras de simularlo (más o menos) haciendo combinaciones entre herencia simple, clases abstractas e interfaces (yo personalmente nunca he tenido esta necesidad), pero no deja de ser eso: una simulación.

La herencia múltiple de interfaces en cambio sí ha estado siempre permitida. Una clase puede implementar cualquier número de interfaces. Entonces, dado que una interface tipifica una clase, podríamos decir que **en Java (incluso antes de Java 8) está permitida la herencia múltiple en cuanto a jerarquía de tipos**. Este tipo de herencia la tenemos totalmente controlada ya que el hecho de que una clase implemente varias interfaces lo único que significa es que podemos tener varios comportamientos, pero no hay posibilidad de ambigüedad ya que es la propia clase la que finalmente implementa esos comportamientos.

Ahora, con los cambios que incorpora Java 8, las interfaces pueden ser utilizadas como traits. Y, como tenemos herencia múltiple en interfaces, podemos decir que: **desde Java 8 está permitida también la herencia múltiple en cuanto a jerarquía de funcionalidades**. Y este tipo de herencia sí que **puede provocar cierta ambigüedad**: ¿qué hacemos si el método está implementado en más de una superclase? La solución adoptada por Java es que la semántica del lenguaje considera una colisión de nombres como ilegal y rechaza la compilación de la clase. El programador debe eliminar esa ambigüedad explícitamente implementando su propia versión del método en la propia clase.

Pero recordemos que **seguimos sin tener herencia múltiple de atributos** y que **las interfaces no permiten la extensión, especialización ni combinación de estos métodos por defecto**, siempre hay una única implementación para un método. No se puede usar el típico "super" de la herencia convencional, lo único que podemos hacer es sobreescribir toda la funcionalidad. Por lo tanto no puede surgir el problema de la herencia repetida, también conocido como "[problema del diamante](https://es.wikipedia.org/wiki/Problema_del_diamante)" (si una clase hereda de 2 ó más superclases padre que a su vez heredan de la misma superclase abuelo ¿qué camino sigo cuando se invoca un método del abuelo?). 

## Y un poco de opinión (conclusiones)

* *El mito de la herencia múltiple*

Desde que se anunció la incorporación de esta característica a Java 8 he leído comentarios de todo tipo y muchos de ellos poniendo el grito en el cielo porque Oracle ha incorporado la herencia múltiple al lenguaje Java: *como Java permite "heredar" de varias interfaces y las interfaces pueden contener código en el cuerpo de los métodos pues, por una simple regla de tres, Java permite herencia múltiple como C++. Así que ¡¡se están cargando Java!!.*

Como hemos visto anteriormente esto no es para nada real. Aunque las interfaces pueden contener implementaciones por defecto de métodos, sigue sin ser posible la herencia múltiple de atributos, solo se permite una única implementación para un método dentro de toda la jerarquía de interfaces, etc. y no se puede dar el conocido problema del diamante, que es el principal inconveniente de la herencia múltiple de clases. Cada lenguaje que soporta herencia múltiple (C++, Python, Eiffel, ...) aborda el problema del diamante a su manera. En Java no existe este problema.

Aunque, por otro lado, sí que podemos encontrarnos ante un problema que antes no teníamos: la ambigüedad ante métodos repetidos en distintas interfaces. De ahí la confusión y el enfado de muchos.

* *¿Por qué han añadido esta característica?*

Según Oracle: *"Los métodos por defecto le permiten añadir nuevas funcionalidades a las interfaces de sus librerías asegurando la compatibilidad binaria con el código escrito para versiones anteriores de dichas interfaces"*. Es decir **se añaden como solución a un problema concreto: la compatibilidad hacia atrás**.

Gracias a estas implementaciones por defecto, al añadir un nuevo método a una interfaz, podemos hacer que las clases que ya usaban esa interfaz no dejen de compilar. Esto tiene sentido si estoy desarrollando una librería que da servicio a terceros, donde quiero asegurar la compatibilidad con versiones anteriores de mi código (como en el ejemplo que puse al principio).

En ningún momento hablan de traits, ni de reutilización de código, ni de herencia de funcionalidades. Únicamente hablan sobre la compatibilidad hacia atrás.

* *¿Y por qué añaden esto ahora si el problema de compatibilidad se ha tenido siempre?*

Pues digamos que les soluciona la vida a ellos mismos.

Esta nueva característica **está estrechamente ligada a la inclusión de las expresiones Lambda y el API Stream**.

Java 8 ahora nos permite hacer cosas tan útiles como esto:

```java
List<?> miLista = ...
miLista.sort (...);
miLista.forEach (...);
```

Pero la interfaz java.util.List no tenía declarados los métodos *sort* ni *forEach* en Java 7. Añadir esos métodos a la interfaz supondría perder compatibilidad hacia atrás (todo el que haya hecho alguna vez una clase implementando java.util.List vería errores de compilación en su código con Java 8). Y, por otra parte, sería muy frustrante tener Lambdas y Streams en Java 8 pero no poder usar las múltiples interfaces estándar que ya existen en Java (List, Collection, Iterable, etc.) por no sacrificar la compatibilidad hacia atrás. Las interfaces por defecto solucionan este problema: ahora el método *sort* está definido como método por defecto en java.util.List y *forEach* es otro método por defecto de java.lang.Iterable.

* *¿Qué nos aportan?*

Pues, en mi opinión, creo que no se ha añadido esta funcionalidad para aportar nada más allá que el hecho de evitar esos problemas de compatibilidad hacía atrás. El poder usar ahora las interfaces como traits creo que es "un mal necesario" que realmente no pretendían añadir a Java dentro de su ámbito imperativo. Digamos que no les ha quedado más remedio si querían incorporar características de programación funcional (Lambdas) y programación declarativa (Streams) al veterano lenguaje Java. Quién algo quiere algo le cuesta.

**Si usamos Java como programación imperativa de toda la vida no creo que surja una necesidad real de utilizar estas implementaciones de métodos en las interfaces**. De hecho me atrevería a decir que deberíamos intentar "no sucumbir" a ellos ya que puede parecer una alternativa fácil para ciertos casos y luego provocarnos serios dolores de cabeza en el futuro.

Tampoco digo que haya que huir de ellos como de la peste, solo que **si se usan que se haga con cabeza**, por ejemplo no mezclando los conceptos originales de Interface y Trait. Si lo que necesitamos es un Trait en toda regla pues perfecto (una buena práctica podría ser que la palabra "Trait" formara parte del nombre de la interface, para dejar claro su cometido y evitar confusiones). Pero recordemos que están ahí para solucionar problemas de compatibilidad de librerías.

Si nos salimos del mundo imperativo y/o estamos desarrollando un API, pues eso ya es otro cantar.

Una de las principales virtudes de Java, la compatibilidad hacía atrás, es también su mayor lacra, por eso es conveniente saber el porqué de las cosas y saber cuándo conviene usarlas. Como decía un antiguo anuncio de una conocida marca de neumáticos: **"la potencia sin control no sirve de nada"**.


