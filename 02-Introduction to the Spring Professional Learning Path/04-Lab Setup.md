## Download Lab Zip File

1. Descargue el **[archivo zip del código base de laboratorio](https://github.com/vmware-tanzu-learning/spring-pro-code/releases/download/core-spring-release-1.18.1/core-spring-labfiles.zip)**.
2. Una vez que haya descargado el archivo, descomprímalo en el directorio de su elección. El directorio descomprimido `core-spring-labfiles/lab` contiene los proyectos de laboratorio.

## Install Required Software
Instale el siguiente software si aún no está instalado en su máquina.

1. Java 11:
    - [Oracle Java 11](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html)
    - [Open JDK Installations](https://openjdk.java.net/install/)

2. Cliente REST
    - [curl](https://curl.haxx.se/download.html) o
    - [Postman](https://www.getpostman.com/apps)

## Build the Projects

1. Cree los proyectos en el directorio `core-spring-labfiles/lab`. Esto descargará e instalará las dependencias necesarias en el repositorio local.

**Windows**:
```shell
mvnw clean install
```

**MacOSX or Linux**:
```shell
chmod 755 mvnw
./mvnw clean install
```

2. Si experimenta algún error de compilación, verifique la configuración de firewalls o proxy en el archivo `<Home-Directory>/.m2/settings.xml` .

## Choose an IDE

Si bien los proyectos de laboratorio se pueden crear y ejecutar sin un `IDE`, se recomienda encarecidamente el uso de un `IDE` de `Java` convencional.

El curso asume uno de los siguientes, aunque puede elegir uno alternativo con paridad de características:

- [Spring Tools 4 (STS)](https://spring.io/tools)
- [JetBrains IntelliJ](https://www.jetbrains.com/idea)

**Spring Tools 4 (STS)** es un **IDE** gratuito creado en la plataforma **Eclipse** con complementos adicionales para admitir **Spring**, 
**Programación Orientada a Aspectos (AOP)** y la plataforma de implementación del **Servicio de Aplicaciones Tanzu**.

## Import Projects Into Your IDE

1. Importe los proyectos en `core-spring-labfiles/lab` a su **IDE**. Puede importarlos como proyectos **Maven** o **Gradle**.
    
    - Para **STS**, puede **"importar"** desde el directorio `lab` , lo que importará los proyectos que se encuentran debajo.
        
    - Para **IntelliJ**, puede **"importar proyectos"** a través del archivo principal `pom.xml` o `build.gradle`, e **IntelliJ** los configurará 
      automáticamente como un proyecto de varios módulos.
        
2. Verifique que los proyectos se importen correctamente.
    
    - STS:
   
   <img src="https://raw.githubusercontent.com/vmware-tanzu-learning/spring-academy-assets/main/courses/course-spring-professional/sts-with-imported-projects.png" alt="STS with imported projects" width="600"/>
   
    - IntelliJ
   
   <img src="https://raw.githubusercontent.com/vmware-tanzu-learning/spring-academy-assets/main/courses/course-spring-professional/intellij-with-imported-projects.png" alt="IntelliJ with imported projects" width="600"/>
   
3. Revise el artículo [_Uso de tareas TODO_ ](https://spring.academy/spring-dev-tools-configuring-todos).
    

## Troubleshooting

Después de importar los proyectos de laboratorio, su **IDE** puede informar errores, generalmente relacionados con las dependencias del proyecto. Puede resolver estos problemas utilizando las siguientes técnicas:

- Para STS:
    - Seleccione todos los proyectos, haga clic derecho, **Maven**, **Update Projects**.
    - Limpiar todos los proyectos seleccionados
    - Cerrar y abrir todos los proyectos