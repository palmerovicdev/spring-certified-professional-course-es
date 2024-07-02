**Spring** y **Spring Boot** son frameworks de **Java** que se utilizan para crear aplicaciones. Piensa en ellos como kits de herramientas que ayudan a los desarrolladores a construir y estructurar su código de una manera eficiente y escalable.

### Spring

**Spring** es un framework integral que proporciona varios módulos para construir diferentes tipos de aplicaciones. Es como tener una caja de herramientas gigante con todas las herramientas que puedas necesitar para construir lo que quieras. A medida que construyamos nuestra **API** de **Family Cash Card**, utilizaremos **Spring MVC** para la aplicación web, **Spring Data** para el acceso a los datos y **Spring Security** para la autenticación y autorización.

Esta versatilidad tiene un coste. La realizacion de una aplicación **Spring** requiere mucha configuración, y los desarrolladores necesitan 
hacer manualmente varios componentes del framework para ponerla en marcha.

### Spring Boot

Afortunadamente, **Spring Boot** hace que trabajar con **Spring** sea mucho más sencillo. **Spring Boot** es como una versión más obstinada de **Spring**. Viene con muchas configuraciones y dependencias que se utilizan comúnmente en las aplicaciones de **Spring**. Esto hace que sea muy fácil empezar rápidamente, sin tener que preocuparse por configurar todo desde cero. Además, **Spring Boot** viene con un servidor web integrado, por lo que puede crear e implementar fácilmente aplicaciones web sin necesidad de un servidor externo.

En resumen, **Spring** es un framework poderoso e integral que te da mucha flexibilidad, pero que puede ser un poco abrumador de configurar. **Spring 
Boot** es una versión más funcional y simplificada de **Spring** que viene con muchas características integradas para ayudarte a empezar de forma rápida y sencilla.

## Spring's Inversion of Control Container

**Spring Boot** aprovecha el **contenedor de inversión de control (IoC)** de **Spring Core**. Utilizarás esta función de **Spring** ampliamente 
mientras creas la aplicación **Family Cash Card**. Hay una amplia documentación para este concepto, pero aquí lo mantendremos simple.

**Spring Boot** le permite configurar cómo y cuándo se proporcionan las dependencias a su aplicación en tiempo de ejecución. Esto le da el control 
de cómo funciona la misma en diferentes escenarios.

Por ejemplo, es posible que desee utilizar una base de datos diferente para el desarrollo local que para su aplicación en vivo y orientada al público. A su código no le debería importar esta distinción; si lo hiciera, tendría que codificar todos los escenarios posibles en la lógica. En su lugar, **Spring Boot** le permite proporcionar una configuración externa que especifica cómo y cuándo se utilizan dichas dependencias.

La **inversión del control** a menudo se llama **inyección de dependencia (DI)**, aunque esto no es estrictamente correcto. La inyección de 
dependencias y los frameworks que las implementan son una forma de lograr la **inversión de control**, y los desarrolladores de **Spring** a menudo 
afirmarán 
que las dependencias se "inyectan" en sus aplicaciones en tiempo de ejecución. Estamos dejando clara esta distinción porque muchos lenguajes y frameworks de software implementan **IoC**, pero no necesariamente lo llaman "inyección de dependencia". Sin embargo, dentro de la comunidad de **Spring**, a menudo escucharás los términos utilizados indistintamente.

Obtenga más información sobre el contenedor **IoC** de **Spring** en la [documentación oficial de Spring](https://docs.spring.io/spring-framework/reference/core/beans.html).

## Spring Initializr

Al iniciar una nueva aplicación de **Spring Boot**, **Spring Initializr** es el primer paso recomendado. Puedes pensar en **Spring Initializr** como 
un carrito de compras para todas las dependencias que tu aplicación pueda necesitar. Generará rápida y fácilmente una aplicación de **Spring Boot** completa y lista para ejecutar.

En el siguiente laboratorio, usaremos **Spring Initializr**. El flujo general para **Spring Initializr** es rellenar los metadatos, añadir dependencias relevantes y generar su nuevo proyecto. Para la aplicación **Family Cash Card**, proporcionaremos todas estas instrucciones.

Para el resto de este curso, trabajaremos con esta aplicación de **Spring Boot** generada por **Initializr**.