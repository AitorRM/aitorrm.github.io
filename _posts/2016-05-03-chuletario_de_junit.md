---
title: "Chuletario de JUnit"
header:
  teaser: "post_teaser_test.jpg"
categories: 
  - Testing
tags:
  - Java
  - Testing
  - JUnit
---

Con poco que hayas trabajado en Java seguro que ya has oído hablar de JUnit. De hecho el arquetipo más básico de Maven, el "maven-archetype-quickstart" ya viene configurado por defecto con él. JUnit es un framework ampliamente utilizado para hacer pruebas unitarias de aplicaciones Java. Existen otros proyectos como TestNG, Jtest o SpryTest, pero JUnit es el más popular.

Este artículo no pretende ser otro post más de introducción a JUnit. Mi intención es mostrar un ejemplo de test, a modo de resumen (o lo que yo llamo "chuletario"), que reúna muchas de las características básicas de este framework y que pueda servirte como referencia, para refrescar conceptos o, si no lo conoces, para que tengas algunas "palabras clave" con las que iniciar una búsqueda más extensa de información. También es probable que a pesar de llevar tiempo usándolo descubras cosas que te sorprendan.

Échale un vistazo a esta clase:

```java
// Import estático para poder usar los asserts directamente: podremos usar "assertTrue(...)" en vez de "Assert.assertTrue(...)"
import static org.junit.Assert.*;

import org.junit.After;
import org.junit.AfterClass;
import org.junit.Before;
import org.junit.BeforeClass;
import org.junit.FixMethodOrder;
import org.junit.Ignore;
import org.junit.Test;
import org.junit.runners.MethodSorters;

// Se ejecutan los test en orden alfabético.
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class MyTest {
	
	// Se ejecuta una vez al cargar la clase. 
	// Debe ser estático y solo tiene acceso a métodos y variables estáticas.
	@BeforeClass
	public static void beforeClass() {
		System.out.println("MyTest.beforeClass");
	}
	
	// Se ejecuta antes de cada test (tantas veces como métodos @Test).
	// Usado para inicializaciones.
	@Before
	public void before() {
		System.out.println("MyTest.before");
	}
	
	// Un test con asserts.
	@Test
	public void test03() {
		System.out.println("MyTest.test03");
		
		// Pasa si la condición es verdadera
		assertTrue(true);
		
		// Pasa si el objeto es nulo
		Object obj = null;
		assertNull(obj);
		
		// Pasa si los objetos son iguales (según sus métodos equals)
		Integer intEq1 = new Integer("1");
		Integer intEq2 = new Integer("1");
		assertEquals(intEq1, intEq2);
		
		// Pasa si los objetos son el mismo objeto (misma referencia en memoria)
		Integer intSame1 = new Integer("1");
		Integer intSame2 = intSame1;
		assertSame(intSame1, intSame2);
		
		// Pasa si los arrays son iguales (según sus métodos equals)
		Integer[] array1 = new Integer[]{1,2,3};
		Integer[] array2 = new Integer[]{1,2,3};
		assertArrayEquals(array1, array2);
	}
	
	// Un test con asserts negativos.
	@Test
	public void test01() {
		System.out.println("MyTest.test01");
		
		// Pasa si la condición es falsa
		assertFalse(false);
		
		// Pasa si el objeto no es nulo
		Object obj = new String();
		assertNotNull(obj);
		
		// Pasa si los objetos no son el mismo objeto, aunque tengan el mismo valor
		Integer int1 = new Integer("1");
		Integer int2 = new Integer("1");
		assertNotSame(int1, int2);
	}
	
	// Un test que evalúa una excepción
	@Test(expected = RuntimeException.class)
	public void test02() {
		System.out.println("MyTest.test02");
		throw new RuntimeException();
	}
	
	// Un test que será ignorado
	@Ignore
	@Test
	public void test04() {
		System.out.println("MyTest.test04");
	}
	
	// Un test que dará 'fail' si no finaliza antes de un determinado tiempo
	@Test(timeout=100)
	public void test05() throws Exception {
		System.out.println("MyTest.test05");
		Thread.sleep(90);
	}
	
	// Se ejecuta después de cada método @Test.
	@After
	public void after() {
		System.out.println("MyTest.after");
	}
	
	// Se ejecuta una vez al finalizar la clase, 
	// después del último @Test.
	// Debe ser estático y solo tiene acceso a métodos y variables estáticas.
	@AfterClass
	public static void afterClass() {
		System.out.println("MyTest.afterClass");
	}
}
```

La ejecución de esta clase provoca la siguiente salida por consola:

```
MyTest.beforeClass
MyTest.before
MyTest.test01
MyTest.after
MyTest.before
MyTest.test02
MyTest.after
MyTest.before
MyTest.test03
MyTest.after
MyTest.before
MyTest.test05
MyTest.after
MyTest.afterClass
```

Analicemos un poco su contenido:

La clase contiene cinco métodos de test (marcados con la anotación <code>@Test</code>) y escritos en este orden: test03, test01, test02, test04 y test05, pero se ejecutan en orden alfabético gracias a la anotación <code>@FixMethodOrder(MethodSorters.NAME_ASCENDING)</code>. 

Los métodos anotados con <code>@BeforeClass</code> y <code>@AfterClass</code> se ejecutan antes y después de todos los test. Son métodos a nivel de clase y por eso deben ser estáticos. Por su parte los métodos anotados con <code>@Before</code> y <code>@After</code> se ejecutan antes y después de cada uno de los métodos de test (más abajo hablo de estas anotaciones con más detalle).

Se han ejecutado únicamente los métodos test01, test02, test03 y test05 ya que el test04 está anotado con <code>@Ignore</code>. Fijémonos en que tampoco se han ejecutado los métodos 'before' ni 'after' correspondientes al test04.

El test03 contiene los asserts más comunes para hacer validaciones afirmativas: assertTrue, assertNull, assertEquals, assertSame y assertArrayEquals. Nótese que se está usando directamente <code>assertTrue(..)</code> en vez de <code>Assert.assertTrue(..)</code>, esto es gracias al import estático que hay en la primera línea de la clase: <code>import **static** org.junit.Assert.*;</code>

El test01 contiene los asserts más comunes para hacer validaciones negativas: assertFalse, assertNotNull y assertNotSame.

El test02 evalúa una excepción. Está anotado con <code>@Test(expected = RuntimeException.class)</code> y dará "pass" si dentro del test ocurre la excepción esperada (RuntimeException). Esta forma de tratar excepciones está bien para casos sencillos. Para evaluaciones más complejas hay otros métodos más efectivos, como usar <code>@Rule</code> junto con <code>org.junit.rules.ExpectedException</code>, o incluso con la expresión <code>when(...).method(...).thenThrow(...)</code> que incluye el framework Mockito. Pero esto queda fuera del alcance de este artículo.

Y el test05 considera un tiempo máximo de ejecución. Gracias a la anotación <code>@Test(timeout=100)</code> JUnit asignará un tiempo máximo de ejecución para el test y en caso de que se exceda dará "fail". De nuevo aquí tenemos otras alternativas para controles más exhaustivos de tiempos como la regla <code>@Rule</code> junto con <code>org.junit.rules.Timeout</code>.

Si ya conoces JUnit probablemente habrás notado que me falta un tipo de assert, quizás el más importante, el 'assertThat'. No lo he incluido a propósito ya que 'assertThat' merece una explicación a parte. Puede dar mucho juego y me lo guardo para otro post ;)


## Ámbitos de ejecución

Ahora centremos la atención en los métodos anotados con <code>@BeforeClass</code>, <code>@AfterClass</code>, <code>@Before</code> y <code>@After</code>.

Como hemos visto anteriormente @BeforeClass se ejecuta el primero y @AfterClass el último de todos, y luego @Before y @After se ejecutan antes y después de cada test. Pero hay que tener claro el ámbito de ejecución de cada uno: @BeforeClass y @AfterClass son ejecutados a nivel de clase, aún no existe ninguna instancia de MyTest. Luego el motor de JUnit creará tantas instancias como métodos @Test existan. Y por cada una de esas instancias se ejecutarán los métodos @Before, @Test y @After.

¿Se entiende? Bueno, mejor con un ejemplo, ¿no? Mira la siguiente clase y piensa qué valor tendrán las variables "staticCount" y "testCount" cuando JUnit finalice su ejecución.

```java
public class MyTest {
	
	private static int staticCount = 0;
	private int testCount = 0;

	@BeforeClass
	public static void beforeClass() {
		staticCount++;
		System.out.format("MyTest.beforeClass \t staticCount=%s\n", staticCount);
	}
	
	@Before
	public void before() {
		staticCount++;
		testCount++;
		System.out.format("MyTest.before \t\t staticCount=%s  testCount=%s\n", staticCount, testCount);
	}
	
	@Test
	public void test01() {
		staticCount++;
		testCount++;
		System.out.format("MyTest.test01 \t\t staticCount=%s  testCount=%s\n", staticCount, testCount);
	}
	
	@Test
	public void test02() {
		staticCount++;
		testCount++;
		System.out.format("MyTest.test02 \t\t staticCount=%s  testCount=%s\n", staticCount, testCount);
	}
	
	@After
	public void after() {
		staticCount++;
		testCount++;
		System.out.format("MyTest.after \t\t staticCount=%s  testCount=%s\n", staticCount, testCount);
	}
	
	@AfterClass
	public static void afterClass() {
		staticCount++;
		System.out.format("MyTest.afterClass \t staticCount=%s\n", staticCount);
	}
}
```

¿Cómo lo ves? La variable "staticCount" terminará siendo 8 y la variable "testCount" será 3.
Esto es lo que saldrá por consola:

```
MyTest.beforeClass 	 staticCount=1
MyTest.before 		 staticCount=2  testCount=1
MyTest.test01 		 staticCount=3  testCount=2
MyTest.after 		 staticCount=4  testCount=3
MyTest.before 		 staticCount=5  testCount=1
MyTest.test02 		 staticCount=6  testCount=2
MyTest.after 		 staticCount=7  testCount=3
MyTest.afterClass 	 staticCount=8
```

Para que te hagas una idea más clara, a efectos prácticos, lo que hace JUnit es similar a ejecutar lo siguiente desde una clase main:

```java
public static void main(String[] args) {
	MyTest.beforeClass();
	MyTest test1 = new MyTest();
	test1.before();
	test1.test01();
	test1.after();
	MyTest test2 = new MyTest();
	test2.before();
	test2.test02();
	test2.after();
	MyTest.afterClass();
}
```

Te recomiendo ejecutar esta clase con JUnit en modo debug, poner un punto de interrupción en cada método y comprobar en cada caso cual es el identificador de memoria de MyTest.

A lo que quiero llegar es a que las cosas inicializadas en un before no son directamente visibles a otro before. Por ejemplo, si en el test01 hago una prueba de base de datos y en el before abro la conexión, cuando se ejecute el test02 no podré reutilizar esa conexión aunque la haya guardado como variable global. Cada test es independiente. Aunque, por supuesto, me las puedo ingeniar de otra manera y usar una clase externa con un Singleton, un Factory, etc., pero es importante tener en cuenta estos ámbitos de ejecución cuando se escriben pruebas con JUnit.

