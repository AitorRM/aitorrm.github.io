---
title: "Java Microbenchmarking con JMH"
header:
  teaser: "post_teaser_testing.jpg"
categories: 
  - Código Efectivo
tags:
  - Java
  - JMH
  - Benchmark
---

¿Cómo puedo medir el rendimiento de mi código Java? ¿Qué implementación de mi funcionalidad es más óptima? ¿Qué API de terceros ejecuta lo que necesito con mayor velocidad? ¿Cuál es la mejor manera de concatenar texto, con String.concat o con StringBuffer? preguntemos a JMH.

En ocasiones, cuando desarrollamos una funcionalidad, nos surgen dudas de si realmente nuestro código es lo más óptimo posible o hay otra forma mejor de hacerlo. Para resolver nuestra duda lo mejor será comparar ambos códigos y analizarlos. Podemos hacer la típica main, crear un bucle, usar System.nanoTime() para obtener los tiempos, calcular el tiempo total y dividirlo por el número de iteraciones. Parece simple, pero ¿y si no me vale con conocer cuánto tarda en ejecutarse mi método? Digamos que quiero saber el tiempo medio de ejecución, o la cantidad de operaciones que puedo realizar en una unidad de tiempo, o quiero probarlo en varios hilos de ejecución. La cosa se complica. Además, para qué reinventar la rueda si existe un proyecto creado específicamente para este fin.

JMH (Java Microbenchmarking Harness) es un proyecto de OpenJDK creado para hacer benchmarking en Java (y en otros lenguajes de la Java Virtual Machine como Groovy, Scala o Kotlin). El benchmarking, en informática, es una técnica utilizada para medir el rendimiento de un sistema o componente, normalmente comparándolo con otro sistema o componente.

Aquí veremos cómo usarlo con un ejemplo práctico sencillo: quiero saber quién gana al concatenar cadenas de texto en Java. Los cuatro aspirantes son "String.concat", "StringBuffer", "StringBuilder" y la concatenación simple con el operador '+'.

## Crear el proyecto

Empezaremos creando el proyecto siguiendo las recomendaciones que hace JMH en su [página](http://openjdk.java.net/projects/code-tools/jmh/): usar directamente Maven, sin ningún IDE. Se puede hacer perfectamente en cualquier IDE como Eclipse o Netbeans, pero tratándose de pruebas de rendimiento recomiendan no involucrar a terceros que puedan contaminar los resultados.

Así que nos situamos en nuestra carpeta de trabajo y creamos el proyecto con Maven:

```
$ mvn archetype:generate \
          -DinteractiveMode=false \
          -DarchetypeGroupId=org.openjdk.jmh \
          -DarchetypeArtifactId=jmh-java-benchmark-archetype \
          -DgroupId=com.aezin.workshop.benchmark.jmh.concatstring \
          -DartifactId=java-benchmarks-jmh \
          -Dversion=1.0
```

Veamos qué ha pasado. Se han creado únicamente dos archivos. Por una parte el pom.xml con las dependencias y configuración necesaria:

```xml
...
<dependencies>
	<dependency>
		<groupId>org.openjdk.jmh</groupId>
		<artifactId>jmh-core</artifactId>
		<version>${jmh.version}</version>
	</dependency>
	<dependency>
		<groupId>org.openjdk.jmh</groupId>
		<artifactId>jmh-generator-annprocess</artifactId>
		<version>${jmh.version}</version>
		<scope>provided</scope>
	</dependency>
</dependencies>
...
```
		  
Y por otra parte se ha creado la siguiente clase Java de ejemplo:

```java
package com.aezin.workshop.benchmark.jmh.concatstring;

import org.openjdk.jmh.annotations.Benchmark;

public class MyBenchmark {

    @Benchmark
    public void testMethod() {
        // This is a demo/sample template for building your JMH benchmarks. Edit as needed.
        // Put your benchmark code here.
    }

}
```

## Crear mi primer benchmark

Ahora vamos a importar este proyecto en nuestro IDE favorito. En mi caso Eclipse. JMH recomienda crear el proyecto y ejecutarlo sin IDE, pero no creo que le moleste que use Eclipse para editar mis clases.

Los pasos en Eclipse son:

```
File > Import > Maven - Existing Maven Project 
```

Podríamos empezar a trabajar con la clase de ejemplo MyBenchmark, pero mejor la borramos y creamos nuestra propia clase desde cero.
Como estamos haciendo benchmark de cadenas de texto llamaremos a la clase "BenchmarkConcatString". Crearemos un método para cada uno de los aspirantes y haremos que cada uno concatene, por ejemplo, cuatro asteriscos.

```java
package com.aezin.workshop.benchmark.jmh.concatstring;

import org.openjdk.jmh.annotations.Benchmark;

public class BenchmarkConcatString {

	@Benchmark
	public String testFourConcatStringsWithPlus() {
		String text = "";
        for (int i = 0; i < 4; i++) {
        	text += "*";
        }
        return text;
	}

	@Benchmark
	public String testFourConcatStringsWithConcat() {
		String text = "";
        for (int i = 0; i < 4; i++) {
        	text = text.concat("*");
        }
        return text;
	}

	@Benchmark
	public String testFourConcatStringsWithStringBuilder() {
		StringBuilder text = new StringBuilder();
        for (int i = 0; i < 4; i++) {
        	text.append("*");
        }
        return text.toString();
	}
	
	@Benchmark
	public String testFourConcatStringsWithStringBuffer() {
		StringBuffer text = new StringBuffer();
        for (int i = 0; i < 4; i++) {
        	text.append("*");
        }
        return text.toString();
	}

}
```

Nuestros métodos devuelven un valor. Esto no es un problema para JMH, de hecho es conveniente hacerlo así para ayudar al optimizador de JMH.

## Ejecutar el benchmark

Ahora, para ejecutar el benchmark, lo primero que haremos será irnos a la consola, compilar el proyecto con Maven y generar el jar.

```
$ cd java-benchmarks-jmh
$ mvn clean install
```

Maven ha creado dentro de la carpeta "target" el jar de nuestro proyecto (java-benchmarks-jmh-1.0.jar) y un archivo llamado "benchmarks.jar". Lo que hay que ejecutar es el archivo benchmarks.jar

Pero, antes de ejecutarlo, una advertencia: este proceso va a tardar bastante. Date cuenta de que en ningún momento le hemos dicho aún qué tipo de prueba queremos. No le hemos dicho cuantas pruebas queremos hacer ni cuantas iteraciones por prueba. Vamos a ejecutarlo con la configuración por defecto, que son 10 pruebas y por cada una de ellas 20 iteraciones y 20 preparaciones (warmup). Y esto se hará por cada benchmark. Nosotros tenemos cuatro!!! 

La configuración por defecto se ejecuta en modo AverageTime (calcula la media de tiempo de ejecución de cada benchmark) y muestra los resultados en milisegundos.

Te adelanto que esta ejecución a mí me ha tardado unos 25 minutos.

Bueno, quién dijo miedo, vamos allá:

```
$ java -jar target/benchmarks.jar
```

Y empezamos a ver la ejecución por consola:

```
# JMH 1.12 (released 22 days ago)
# VM version: JDK 1.8.0_91, VM 25.91-b14
# VM invoker: C:\Program Files (x86)\Java\jre1.8.0_91\bin\java.exe
# VM options: <none>
# Warmup: 20 iterations, 1 s each
# Measurement: 20 iterations, 1 s each
# Timeout: 10 min per iteration
# Threads: 1 thread, will synchronize iterations
# Benchmark mode: Throughput, ops/time
# Benchmark: com.aezin.workshop.benchmark.jmh.concatstring.BenchmarkConcatString.testFourConcatStringsWithConcat

# Run progress: 0,00% complete, ETA 00:24:40
# Fork: 1 of 10
# Warmup Iteration   1: 10625672,024 ops/s
# Warmup Iteration   2: 10578590,365 ops/s
...
```

Aquí podemos comprobar que se ha iniciado el fork 1 de los 10 que tiene previstos para el método "testFourConcatStringsWithConcat". Realizará 20 "Warmup Iteration" y acto seguido 20 "Iteration". Cuando todo esto termine hará la misma operación con los otros tres métodos que le quedan "testFourConcatStringsWithPlus", "testFourConcatStringsWithStringBuilder" y "testFourConcatStringsWithStringBuffer". Así que JMH hará trabajar duro a nuestros cuatro aspirantes.

Tras un "ratito" de espera ya tengo los resultados (hay que nerviossss):

```
Result "testFourConcatStringsWithStringBuilder":
  12368649,351 ±(99.9%) 61589,974 ops/s [Average]
  (min, avg, max) = (11419948,451, 12368649,351, 12744871,319), stdev = 260775,807
  CI (99.9%): [12307059,377, 12430239,326] (assumes normal distribution)


# Run complete. Total time: 00:26:50

Benchmark                                                      Mode  Cnt         Score       Error  Units
BenchmarkConcatString.testFourConcatStringsWithConcat         thrpt  200  10694766,998 ± 97108,416  ops/s
BenchmarkConcatString.testFourConcatStringsWithPlus           thrpt  200   4359771,709 ± 26595,529  ops/s
BenchmarkConcatString.testFourConcatStringsWithStringBuffer   thrpt  200   9977884,666 ± 48314,065  ops/s
BenchmarkConcatString.testFourConcatStringsWithStringBuilder  thrpt  200  12368649,351 ± 61589,974  ops/s
```

Vemos que el vencedor ha sido la concatenación con '+'. En segunda posición está StringBuffer, seguido muy de cerca por String.concat, y StringBuilder se queda a la cola de la competición.

## ¿Qué más puedo hacer con JMH?

Podemos modificar la configuración por defecto haciendo uso de algunas anotaciones.

Por ejemplo, con @BenchmarkMode podemos elegir el tipo de resultado que queremos:
- <code>@BenchmarkMode(Mode.AverageTime)</code>: Muestra la media de tiempo de ejecución. Es el valor por defecto.
- <code>@BenchmarkMode(Mode.SampleTime)</code>: Muestra el tiempo total de ejecución de un benchmark.
- <code>@BenchmarkMode(Mode.SingleShotTime)</code>: Hace la ejecución sin iteraciones.
- <code>@BenchmarkMode(Mode.Throughput)</code>: Muestra el número de operaciones por unidad de tiempo.
- <code>@BenchmarkMode(Mode.All)</code>: Todos los anteriores.

También podemos controlar los benchmark con las siguientes anotaciones:
- <code>@Fork(n)</code>: Ejecutará 'n' pruebas.
- <code>@Measurement(n)</code>: Por cada prueba ejecutará 'n' iteraciones.
- <code>@Threads(n)</code>: Usará 'n' hilos concurrentes para las pruebas.
- <code>@Warmup(n)</code>: Por cada prueba ejecutará 'n' preparaciones.

Con @OutputTimeUnit podemos determinar la unidad de tiempo calculada: días, horas, segundos, milisegundos, nanosegundos, etc. Por ejemplo <code>@OutputTimeUnit(TimeUnit.MICROSECONDS)</code>.

Estas anotaciones podrán ir a nivel de método o nivel de clase si se desea que apliquen a todos los benchmark.

```java
public class MyBenchmark {

	@Fork(4)
	@Measurement(iterations = 20)
	@Warmup(iterations = 20)
	@Benchmark
	public String benchmark01() {
		...
	}
	
	@Fork(10)
	@Measurement(iterations = 10)
	@Warmup(iterations = 10)
	@Benchmark
	public String benchmark02() {
		...
	}
}
```

```java
@Fork(4)
@Measurement(iterations = 20)
@Warmup(iterations = 20)
public class MyBenchmark {

	@Benchmark
	public String benchmark01() {
		...
	}
	
	@Benchmark
	public String benchmark02() {
		...
	}
}
```

JMH tiene opciones de configuración para multitud de casuísticas, permite gestionar parámetros de entrada para los benchmarks, provee de métodos before y after al estilo de JUnit, se pueden configurar timeouts, etc.

También permite ser ejecutado desde una main, desde la cual se puede hacer cualquier configuración sin necesidad de usar las anotaciones.

```java
public static void main(String[] args) throws Exception {
	Options options = new OptionsBuilder()
			.include(".*" + BenchmarkConcatString.class.getSimpleName() + ".*")
			.forks(4)
			.warmupIterations(20)
			.measurementIterations(20)
			.build();
	new Runner(options).run();
}
```

## Refinar nuestro benchmark

Volvamos a nuestra batalla particular, la concatenación de Strings, y recordemos una cosa importante: hasta ahora hemos probado el rendimiento de los aspirantes concatenando cuatro cadenas de texto. ¿Quién nos dice que al concatenar 50 pase lo mismo? ¿y al concatenar sólo dos?

Seamos un poco más ambiciosos y, ya que estamos, vamos a ver en qué casos compensa usar un método u otro según la cantidad de concatenaciones. Vamos a repetir todo el proceso pero con cuatro escenarios distintos:
- Concatenar únicamente dos cadenas de texto.
- Concatenar tres cadenas de texto.
- Concatenar cuatro cadenas de texto.
- y concatenar cincuenta cadenas de texto.

Los cambios que haremos serán:
- crear una clase independiente que concatene tantos asteriscos como se le indique por parámetro. La llamaremos "ConcatString".
- modificar nuestra clase de benchmark para que llame a la clase concatenadora.
- añadir tantos benchmarks como escenarios de prueba: en total tendremos 16 benchmarks (cuatros formas de concatenar por cuatro cantidades de concatenaciones).

Y para no esperar horas hasta ver los resultados modificaremos los parámetros de configuración del benchmark: Dejaremos en 20 las iteraciones de Warmup y Measurement pero ejecutaremos sólo cuatro forks. Además mostraremos los resultados en microsegundos.

```java
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
@Fork(4)
@Measurement(iterations = 20)
@Warmup(iterations = 20)
public class BenchmarkConcatString {

	private static final int TWO_CONCATS = 2;
	private static final int THREE_CONCATS = 3;
	private static final int FOUR_CONCATS = 4;
	private static final int LONG_CONCATS = 50;

	@Benchmark
	public String testTwoConcatStringsWithPlus() {
		return ConcatString.concatStringsWithPlus(TWO_CONCATS);
	}

	@Benchmark
	public String testTwoConcatStringsWithConcat() {
		return ConcatString.concatStringsWithConcat(TWO_CONCATS);
	}

	...

	@Benchmark
	public String testFourConcatStringsWithStringBuilder() {
		return ConcatString.concatStringsWithStringBuilder(FOUR_CONCATS);
	}
	
	...

	@Benchmark
	public String testLongConcatStringsWithStringBuilder() {
		return ConcatString.concatStringsWithStringBuilder(LONG_CONCATS);
	}
	
	@Benchmark
	public String testLongConcatStringsWithStringBuffer() {
		return ConcatString.concatStringsWithStringBuffer(LONG_CONCATS);
	}
```

No pongo aquí todo el código por no saturar, pero puedes descargarte el ejemplo completo desde mi Github: https://github.com/aitorrm/java-benchmarks-jmh

Mostraré directamente el resultado que he obtenido:

```
Result:		 
# Run complete. Total time: 00:43:11

Benchmark                                                      Mode  Cnt  Score    Error  Units
BenchmarkConcatString.testFourConcatStringsWithConcat          avgt   80  0,159 ±  0,006  us/op
BenchmarkConcatString.testFourConcatStringsWithPlus            avgt   80  0,060 ±  0,002  us/op
BenchmarkConcatString.testFourConcatStringsWithStringBuffer    avgt   80  0,040 ±  0,001  us/op
BenchmarkConcatString.testFourConcatStringsWithStringBuilder   avgt   80  0,040 ±  0,001  us/op
BenchmarkConcatString.testLongConcatStringsWithConcat          avgt   80  2,037 ±  0,067  us/op
BenchmarkConcatString.testLongConcatStringsWithPlus            avgt   80  0,860 ±  0,016  us/op
BenchmarkConcatString.testLongConcatStringsWithStringBuffer    avgt   80  0,621 ±  0,019  us/op
BenchmarkConcatString.testLongConcatStringsWithStringBuilder   avgt   80  0,600 ±  0,020  us/op
BenchmarkConcatString.testThreeConcatStringsWithConcat         avgt   80  0,047 ±  0,002  us/op
BenchmarkConcatString.testThreeConcatStringsWithPlus           avgt   80  0,035 ±  0,001  us/op
BenchmarkConcatString.testThreeConcatStringsWithStringBuffer   avgt   80  0,034 ±  0,002  us/op
BenchmarkConcatString.testThreeConcatStringsWithStringBuilder  avgt   80  0,039 ±  0,003  us/op
BenchmarkConcatString.testTwoConcatStringsWithConcat           avgt   80  0,030 ±  0,001  us/op
BenchmarkConcatString.testTwoConcatStringsWithPlus             avgt   80  0,022 ±  0,001  us/op
BenchmarkConcatString.testTwoConcatStringsWithStringBuffer     avgt   80  0,027 ±  0,001  us/op
BenchmarkConcatString.testTwoConcatStringsWithStringBuilder    avgt   80  0,026 ±  0,001  us/op
```

Lo cual me lleva a la siguiente conclusión:
- Usar el operador '+' para concatenaciones pequeñas, de no más de 4 cadenas de texto.
- Usar StringBuilder o StringBuffer para concatenar grandes cantidades de texto (los resultados son prácticamente iguales).
- No usar nunca String.concat.

## Más Información

Decidí hacer este artículo a raíz de [este](http://www.adam-bien.com/roller/abien/entry/what_is_faster_ejbs_or) post del blog de Adam Biem, donde usa JMH para comparar la velocidad de ejecución de CDI y EJB.

Luego me fue de gran ayuda [este](http://java-performance.info/jmh/) otro artículo.

Y otros artículos que encontré en español como [este](http://www.apuntesdejava.com/2015/12/midiendo-el-rendimiento-con-jmh-java.html) de "Apuntes de Java" y [este](http://www.adictosaltrabajo.com/tutoriales/introduccion-a-jmh/) de "Adistosaltrabajo".

Si quieres trastear más aquí puedes ver multitud de [ejemplos de JMH](http://hg.openjdk.java.net/code-tools/jmh/file/tip/jmh-samples/src/main/java/org/openjdk/jmh/samples/).
