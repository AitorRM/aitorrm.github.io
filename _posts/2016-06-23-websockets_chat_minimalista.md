---
title: "Java Websockets: Un Chat con el mínimo código"
header:
  teaser: "post_teaser_websockets.jpg"
categories: 
  - Java EE
tags:
  - Websockets
  - Java
  - Chat
---

WebSocket es una tecnología que proporciona un **canal de comunicación bidireccional** entre un navegador y un servidor web, aunque puede ser utilizado también por cualquier aplicación cliente/servidor. Gracias a esta tecnología un servidor puede enviar mensajes a un navegador Web sin que éste se lo haya solicitado previamente. Es la incorporación de sockets TCP al mundo Web.

Hasta ahora para crear, por ejemplo un chat en una aplicación Web, había que utilizar técnicas como [Push](https://es.wikipedia.org/wiki/Tecnolog%C3%ADa_Push), [Comet](https://es.wikipedia.org/wiki/Comet) o Long Polling (usado por el chat de Gmail). En este artículo veremos cómo hacer un Chat de forma fácil y sencilla con los WebSockets.

En Java, los WebSockets aparecen a partir de Java EE 7 y son especificados en la [JSR-356](https://jcp.org/en/jsr/detail?id=356). 

En cuanto a la parte cliente, la W3C se está encargando de estandarizarlo. Al ser una tecnología relativamente novedosa no todos los navegadores Web lo soportan (en 2009 Google Chrome fue el primero en hacerlo). Aquí tienes una lista de los [navegadores que soportan Websockets](http://caniuse.com/#search=websockets).

Y después de esta breve introducción vayamos al grano: creemos un chat!!

## Creación del servidor de Chat

Solo necesitamos Java 7, Maven 3 y un servidor que soporte Websockets (por ejemplo [Wildfly](http://www.wildfly.org/) o [GlassFish](https://glassfish.java.net/)). Yo he usado Wildfly 9.

Creamos el proyecto con Maven desde la consola (posicionándonos en la carpeta donde queramos crear el proyecto):

```
$>mvn archetype:generate -DgroupId=com.aezin.workshop.websockets -DartifactId=java-websockets-minimal-chatroom -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false
```
Esto nos genera un proyecto con nombre "java-websockets-minimal-chatroom".

Modificamos el pom.xml que nos ha generado por el siguiente:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.aezin.workshop.websockets</groupId>
	<artifactId>java-websockets-minimal-chatroom</artifactId>
	<packaging>war</packaging>
	<version>1.0</version>
	<name>Chat Room Minimalista</name>
	
	<dependencies>
		<dependency>
			<groupId>javax</groupId>
			<artifactId>javaee-api</artifactId>
			<version>7.0</version>
			<scope>provided</scope>
		</dependency>
	</dependencies>
	
	<build>
		<finalName>minimal-chat</finalName>
		<plugins>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.2</version>
				<configuration>
					<source>1.7</source>
					<target>1.7</target>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>
```

Dentro de src\main creamos un nuevo paquete: java\com\aezin\workshop\websockets\server.
Y dentro de él añadimos la clase Java que gestiona el Websocket servidor: ChatEndpoint.java.

```java
package com.aezin.workshop.websockets.server;

import java.io.IOException;
import javax.websocket.EncodeException;
import javax.websocket.OnMessage;
import javax.websocket.Session;
import javax.websocket.server.ServerEndpoint;

@ServerEndpoint("/chatroom")
public class ChatEndpoint {
	@OnMessage
	public void message(String message, Session client) throws IOException, EncodeException {
		for (Session openSession : client.getOpenSessions()) {
			openSession.getBasicRemote().sendText(message);
		}
	}
}
```

Cuando el servidor reciba un mensaje de un cliente lo reenviará al resto de clientes conectados.

Y esto es todo.

Ahora generamos el .war con Maven. Nos metemos en la carpeta raíz de nuestro proyecto java-websockets-minimal-chatroom y ejecutamos el install de Maven:

```
$>cd java-websockets-minimal-chatroom
$>mvn clean install
```

Copiamos el "minimal-chat.war" que se ha generado en la carpeta "target" y lo desplegamos en nuestro servidor.

Para comprobar si se ha desplegado bien escribimos esta url en el navegador: http://localhost:8080/minimal-chat. Tiene que aparecer el texto "Hello World!", que es el que añade por defecto el archetype "maven-archetype-webapp" de Maven.

Ahora nuestro servidor de Chat escucha mensajes desde http://localhost:8080/minimal-chat/chatroom

## Creación del cliente

Solo necesitamos HTML, Javascript y un navegador que soporte Websockets.

Nos creamos un fichero HTML, por ejemplo "index.html", donde pondremos lo siguiente:

```html
<html>
<head>
	<title>WebSockets: ChatRoom</title>
</head>
<body>
	<h2>WebSockets: ChatRoom</h2>
	<form action="">
		<input id="newUserField" name="user" value="" type="text"> 
		<input id="newUserButton" onclick="join();" value="Entrar" type="button">
		
		<br/><br/>
		<textarea id="chatRoomField" rows="10" cols="30" readonly disabled></textarea>
		
		<br/>
		<input id="sendField" name="message" value="" type="text" disabled>
		<input id="sendButton" onclick="send_message();" value="Enviar" type="button" disabled>
	</form>

	<script type="text/javascript" src="websocket.js">
	</script>
</body>
</html>
```

No requiere mucha explicación ;) Unos inputs para registrar al usuario, un área de texto donde aparecerán los mensajes de conversación y más inputs para enviar mensajes al servidor.

En la misma carpeta creamos un fichero javascript llamado "websocket.js" que gestione la comunicación con el servidor:

```javascript
var username;
var websocket = new WebSocket("ws://localhost:8080/minimal-chat/chatroom");

websocket.onmessage = function(evt) { 
	document.getElementById("chatRoomField").innerHTML += evt.data + "\n";
};

function join() {
    username = document.getElementById("newUserField").value;
    websocket.send("*** " + username + " se ha unido!!");
    document.getElementById("newUserField").disabled = true;
    document.getElementById("newUserButton").disabled = true;
    document.getElementById("chatRoomField").disabled = false;
    document.getElementById("sendField").disabled = false;
    document.getElementById("sendButton").disabled = false;
}

function send_message() {
    websocket.send(username + ": " + document.getElementById("sendField").value);
}
```

Crea una conexión Websocket al cargar la página (new WebSocket). Declara un callback (websocket.onmessage) que será llamado cuando reciba un mensaje del servidor (esta es la parte más interesante). Y luego tiene dos funciones para enviar mensajes (websocket.send) al servidor: uno para el registro del usuario (join) y otro para chatear (send_message).

Y ya está, no necesitamos nada más.

## A chatear!!

Abrimos el "index.html" en dos navegadores distintos y... empezamos a chatear!!

![no-alignment]({{ site.url }}{{ site.baseurl }}/images/websockets_chat/wbsockets_chat_01.jpg)
![no-alignment]({{ site.url }}{{ site.baseurl }}/images/websockets_chat/wbsockets_chat_02.jpg)
![no-alignment]({{ site.url }}{{ site.baseurl }}/images/websockets_chat/wbsockets_chat_03.jpg)
![no-alignment]({{ site.url }}{{ site.baseurl }}/images/websockets_chat/wbsockets_chat_04.jpg)
![no-alignment]({{ site.url }}{{ site.baseurl }}/images/websockets_chat/wbsockets_chat_05.jpg)

## Código fuente

Puedes obtener todo el código fuente en [mi github](https://github.com/AitorRM/java-websockets-minimal-chatroom). 
Yo he añadido tanto la parte servidora como la parte cliente en el mismo proyecto .war y he adaptado ligeramente la parte cliente para incluirlo en el index.jsp típico de un proyecto Web Java.

## Conclusiones
Hemos usado la potencia de los Websockets para crear en pocos pasos un chat que puede servirnos como base para añadir todas las florituras que deseemos.

***Mejoras que podemos hacer en la parte servidora***: control de apertura de conexiones (@OnOpen), control de errores (@OnError), control de cierres de conexión (@OnClose), comunicación entre cliente y servidor con objetos complejos (Data-Transfer-Object) que podríamos codificar y decodificar en JSON. Podemos almacenar usuarios y conversaciones en base de datos. Podemos integrarlo en algún framework MVC Java como JSF o Spring. Y podemos añadir al proyecto los tests unitarios oportunos.

***Mejoras que podemos hacer en la parte cliente***: al igual que en el servidor aquí también deberíamos controlar las aperturas (websocket.onopen) y los posibles errores (websocket.onerror). La conexión no se realiza igual si el servidor usa HTTP ("**ws**://localhost:8080/minimal-chat/chatroom") que si usa HTTPS ("**wss**://localhost:8080/minimal-chat/chatroom"). Y se puede enriquecer la interfaz gráfica con jQuery, CSS3, Bootstrap, etc.

También podemos crear otro tipo de clientes que se conecten a nuestro servidor de chat (no tienen por qué ser solo navegadores web): podemos crear clases java cliente (ver el proyecto [Tyrus](https://tyrus.java.net/)), una aplicación Android, una aplicación javaFX, etc.

Y, por supuesto, los websockets pueden dar mucho más de sí que para un "simple" chat. Si quieres ver algunos ejemplos de su potencial te animo a que te des un paseo por la web de [websocket.org](https://www.websocket.org/demos.html). También te animo a visitar [Plink](http://dinahmoelabs.com/plink), un juego multijugador online, basado en Websockets, en el que cualquiera puede entrar y hacer música en tiempo real.
