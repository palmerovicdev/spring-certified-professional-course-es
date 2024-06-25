Volvamos a centrar nuestra atención en nuestro hilo, concentrándonos en otro componente de la arquitectura: Security.

## What Is Security?

La seguridad del software puede significar muchas cosas. El campo es un tema enorme que merece su propio curso. En esta lección, hablaremos sobre la seguridad web. Más específicamente, cubriremos cómo funciona la **autenticación** y **autorización** **HTTP**, las formas comunes en las que el ecosistema web es vulnerable a los ataques y cómo podemos usar **Spring Security** para evitar el acceso no autorizado a nuestro servicio **Family Cash Card**.

## Authentication

La autenticación es el acto de un usuario que demuestra su identidad al sistema. Una forma de hacer esto es proporcionar credenciales (por ejemplo, un nombre de usuario y una contraseña utilizando la autenticación básica). Decimos que una vez que se han presentado las credenciales adecuadas, se autentica el usuario o, en otras palabras, el usuario ha iniciado sesión correctamente.

**HTTP** es un protocolo sin estado, por lo que cada solicitud debe contener datos que demuestren que es de un usuario autenticado. Aunque es posible presentar las credenciales en cada solicitud, hacerlo es ineficiente porque requiere más procesamiento en el servidor. En su lugar, se crea una sesión de autenticación (Authentication Session, Auth Session, o simplemente Session) cuando un usuario se autentica. Las sesiones se pueden implementar de muchas maneras. Utilizaremos un mecanismo común: un Session Token (una cadena de caracteres aleatorios) que se genera y se coloca en una cookie. Una cookie es un conjunto de datos almacenados en un cliente web (como un navegador) y asociados con un **URI** específico.

Un par de cosas bonitas sobre las **Cookies**:

- Las cookies se envían automáticamente al servidor con cada solicitud (no es necesario escribir ningún código adicional para que esto suceda). Siempre que el servidor compruebe que el token de la cookie es válido, las solicitudes no autenticadas pueden ser rechazadas.

- Las cookies pueden persistir durante un cierto período de tiempo, incluso si la página web se cierra y luego se vuelve a visitar. Esta capacidad suele mejorar la experiencia del usuario del sitio web.

## Spring Security and Authentication

**Spring Security** implementa la autenticación en **Filter Chain**. **Filter Chain** es un componente de la arquitectura web de **Java** que permite a los programadores definir una secuencia de métodos que se llaman antes del Controller. Cada filtro de la cadena decide si se permite que el procesamiento de solicitudes continúe o no. **Spring Security** inserta un filtro que comprueba la autenticación del usuario y devuelve con una respuesta `401 UNAUTHORIZED` si la solicitud no está autenticada.

## Authorization

Hasta ahora hemos hablado de la autenticación. Pero en realidad, la autenticación es solo el primer paso. La autorización ocurre después de la autenticación y permite que diferentes usuarios del mismo sistema tengan diferentes permisos.

**Spring Security** proporciona autorización a través del control de **Role-Based Access Control** (RBAC). Esto significa que un usuario tiene una serie de funciones. Cada recurso (u operación) especifica qué roles debe tener un usuario para realizar acciones con la autorización adecuada. Por ejemplo, es probable que un usuario con un rol de administrador esté autorizado a realizar más acciones que un usuario con un rol de propietario de la tarjeta. Puede configurar la autorización basada en roles tanto a nivel global como por método.

## Same Origin Policy

La web es un lugar peligroso, donde los malos actores intentan constantemente explotar las vulnerabilidades de seguridad. El mecanismo de protección más básico se basa en que los clientes y servidores **HTTP** implementen **Single Origin Policy** (SOP). Esta política establece que solo los scripts que están contenidos en una página web pueden enviar solicitudes al origen (**URI**) de la página web.

El procedimiento operativo estándar es fundamental para la seguridad de los sitios web porque sin la política, cualquiera podría escribir una página web que contenga un script que envíe solicitudes a cualquier otro sitio. Por ejemplo, echemos un vistazo a un sitio web bancario típico. Si un usuario ha iniciado sesión en su cuenta bancaria y visita una página web maliciosa (en una pestaña o ventana del navegador diferente), las solicitudes maliciosas podrían enviarse (con las cookies de autenticación) al sitio bancario. Esto podría resultar en acciones no deseadas, ¡como un retiro de la cuenta bancaria del usuario!

### Cross-Origin Resource Sharing

A veces, un sistema consiste en servicios que se ejecutan en varias máquinas con diferentes **URI** (es decir, **Microservicios**). **Cross-Origin Resource Sharing** (CORS) es una forma en que los navegadores y servidores pueden cooperar para relajar el **SOP**. Un servidor puede permitir explícitamente una lista de "orígenes permitidos" de solicitudes procedentes de un origen fuera del servidor.

**Spring Security** proporciona la anotación **@CrossOrigin**, lo que le permite especificar una lista de sitios permitidos. ¡Ten cuidado! Si usas la anotación sin ningún argumento, permitirá todos los orígenes, ¡así que ten esto en cuenta!

## Common Web Exploits

Además de explotar las vulnerabilidades de seguridad conocidas, los actores maliciosos en la web también están descubriendo constantemente nuevas vulnerabilidades. Afortunadamente, **Spring Security** proporciona un poderoso conjunto de herramientas para protegerse contra los exploits de seguridad comunes. Discutamos dos hazañas comunes, cómo funcionan y cómo **Spring Security** ayuda a mitigarlas.

### Cross-Site Request Forgery (falsificación de solicitudes entre sitios)

Un tipo de vulnerabilidad es **Cross-Site Request Forgery** (CSRF) que a menudo se pronuncia "**Sea-Surf**", y también se conoce como **Session Riding**. En realidad, **Session Riding** está habilitado por las cookies. Los ataques **CSRF** ocurren cuando una pieza de código malicioso envía una solicitud a un servidor donde un usuario está autenticado. Cuando el servidor recibe la cookie de autenticación, no tiene forma de saber si la víctima envió la solicitud dañina sin querer.

Para protegerte contra los ataques **CSRF**, puedes usar un token **CSRF**. Un token **CSRF** es diferente de un token **Auth** porque se genera un token único en cada solicitud. Esto hace que sea más difícil para un actor externo insertarse en la "conversación" entre el cliente y el servidor.

Afortunadamente, **Spring Security** tiene soporte integrado para tokens **CSRF** que está habilitado de forma predeterminada. Aprenderás más sobre esto en el próximo laboratorio.

### Cross-Site Scripting (scripting entre sitios)

Tal vez incluso más peligroso que la vulnerabilidad **CSRF** es **Cross-Site Scripting** (XSS). Esto ocurre cuando un atacante es de alguna manera capaz de "engañar" a la aplicación de la víctima para que ejecute código arbitrario. Hay muchas maneras de hacer esto. Un ejemplo simple es guardar una cadena en una base de datos que contiene una etiqueta `<script>`, y luego esperar hasta que la cadena se muestre en una página web, lo que resulta en la ejecución del script.

**XSS** es potencialmente más peligroso que **CSRF**. En **CSRF**, solo se pueden ejecutar las acciones que un usuario está autorizado a realizar. Sin embargo, en **XSS**, el código malicioso arbitrario se ejecuta en el cliente o en el servidor. Además, los ataques **XSS** no dependen de la autenticación. Más bien, los ataques **XSS** dependen de los "huecos" de seguridad causados por las malas prácticas de programación.

La forma principal de protegerse contra los ataques **XSS** es procesar adecuadamente todos los datos de fuentes externas (como formularios web y cadenas de consulta **URI**). En el caso de nuestro ejemplo de etiqueta `<script>`, los ataques se pueden mitigar escapando correctamente de los caracteres **HTML** especiales cuando se renderiza la cadena.

Eso es todo por nuestra breve introducción a la seguridad web. La seguridad web es un tema grande y diverso, ¡pero ahora tienes un amplio sentido de cómo puedes usar **Spring Security** para proteger a tus usuarios y aplicaciones!

# Summary

La autenticación y la autorización son los dos pilares más importantes de la seguridad del software. En las aplicaciones web modernas, estas dos preocupaciones están bajo un ataque constante a través de innumerables exploits de seguridad incluidos Cross-Site Request Forgery, Cross-Site Scripting, y el mal uso de Cross-Origin Resource Sharing.

Por suerte para nosotros, **Spring Security** es la herramienta de seguridad más poderosa de la comunidad de **Spring** para gestionar las preocupaciones de seguridad de una aplicación moderna de **Spring Boot**. Además, es extremadamente fácil de incorporar y configurar.

A continuación, protejamos nuestra creciente **API** de **Family Cash Card** usando **Spring Security**.