# Spring Initializr

Complete los siguientes pasos para usar **Spring Initializr** para configurar la aplicación **Family Cash Card REST API**.

1. Abra la pestaña del panel con la etiqueta **Spring Initializr**:

<img src="https://github.com/palmerovicdev/spring-certified-professional-course-es/blob/main/99-Assets%2Finitializr-metadata.png">

> Nota: Puede notar que el panel de control de **Initializr** tiene versiones diferentes a las que le mostramos aquí. El equipo de **Spring** actualiza continuamente el **Initializr** con las últimas versiones disponibles de **Spring** y **Spring Boot**.


2. Seleccione las siguientes opciones:
	- **Proyecto**: Maven
	- **Idioma**: Java
	- **SpringBoot**: Elige el último 3.3. Versión X

3. Introduzca los siguientes valores junto a los campos de metadatos del proyecto correspondientes:
	- **Grupo**: ejemplo
	- **Artefacto**: `CashCard`
	- **Nombre**: CashCard
	- **Descripción**: Servicio de CashCard para `CashCard` familiares
	- **Embalaje**: Tarro
	- **Java**: 17

   > Nota: No tienes que introducir el campo **"Package name"** - ¡**Spring Initializr** rellenará esto por ti!

4. Seleccione el botón **ADD DEPENDENCIES**... en el panel **DEPENDENCIES**.
5. Seleccione la siguiente opción, ya que sabemos que vamos a crear una aplicación web:
	- **Opciones web**: Spring Web

   > Más adelante en el curso, agregarás dependencias adicionales sin usar Spring Initializr.

<img src="https://github.com/palmerovicdev/spring-certified-professional-course-es/blob/main/99-Assets%2Finitializr-dependencies.png">

6. Haga clic en el botón **CREAR**. **Spring Initializr** genera un archivo zip de código y lo descomprime en su directorio de inicio.
8. Desde la línea de comandos en la pestaña **Terminal**, introduzca los siguientes comandos para usar la envoltura de gradle para crear y probar la aplicación generada.
Vaya al directorio de `cashcard` en la pestaña del panel de control del `Terminal`.

`Dashboard: Open dashboard "Terminal"`
```bash
[~] $ cd cashcard 
[~/cashcard] $
```

A continuación, ejecute el comando de compilación `./gradlew`:
```bash
[~/cashcard] $ ./gradlew build
```

El resultado muestra que la aplicación pasó las pruebas y se construyó con éxito.
```
[~/cashcard] $ ./gradlew build
Downloading https://services.gradle.org/distributions/gradle-bin.zip
............10%............20%............30%.............40%............50%............60%............70%.............80%............90%............100%

Welcome to Gradle!
...
Starting a Gradle Daemon (subsequent builds will be faster)

> Task :test
...
BUILD SUCCESSFUL in 39s
7 actionable tasks: 7 executed
```

## Summary

¡Enhorabuena! Acabas de aprender a iniciar rápida y fácilmente una base de código de Spring Boot usando Spring Initializr.