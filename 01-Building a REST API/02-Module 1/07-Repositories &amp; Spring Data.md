En este punto de nuestro viaje de desarrollo, tenemos un sistema que devuelve un registro de tarjeta de efectivo codificado de nuestro controlador. Sin embargo, lo que realmente queremos es devolver datos reales, de una base de datos. ¡Así que continuemos con nuestro hilo de acero cambiando nuestra atención a la base de datos!

**Spring Data** funciona con **Spring Boot** para simplificar la integración de la base de datos. Antes de entrar, hablemos brevemente sobre la arquitectura de **Spring Data**.

## Controller-Repository Architecture

El principio **Separation of Concerns** establece que el software bien diseñado debe ser modular, y cada módulo tiene preocupaciones distintas y separadas de cualquier otro módulo.

Hasta ahora, nuestra base de código solo devuelve una respuesta codificada del Controlador. Esta configuración viola el principio de separación de preocupaciones al mezclar las preocupaciones de un controlador, que es una abstracción de una interfaz web, con las preocupaciones de leer y escribir datos en un almacén de datos, como una base de datos. Para resolver esto, utilizaremos un patrón de arquitectura de software común para hacer cumplir la separación de la gestión de datos a través del patrón de repositorio.

Un marco arquitectónico común que divide estas capas, normalmente por función o valor, como las capas de negocio, datos y presentación, se llama arquitectura en capas. En este sentido, podemos pensar en nuestro Repositorio y Controlador como dos capas en una arquitectura en capas. El Controlador está en una capa cerca del Cliente (a medida que recibe y responde a las solicitudes web), mientras que el Repositorio está en una capa cerca del almacén de datos (a medida que lee y escribe en el almacén de datos). También puede haber capas intermedias, dictadas por las necesidades del negocio. No necesitamos ninguna capa adicional, ¡al menos todavía no!

El repositorio es la interfaz entre la aplicación y la base de datos, y proporciona una abstracción común para cualquier base de datos, lo que facilita el cambio a una base de datos diferente cuando sea necesario.

![[layers.png]]

Como buena noticia, **Spring Data** proporciona una colección de herramientas sólidas de gestión de datos, incluidas las implementaciones del patrón del repositorio.

## Choosing a Database
Para la selección de nuestra base de datos, utilizaremos una base de datos incrustada (**embedded**) en memoria. "Incrustado (**Embedded**)" simplemente significa que es una biblioteca Java, por lo que se puede agregar al proyecto como cualquier otra dependencia. "En memoria" significa que almacena datos solo en la memoria, en lugar de los datos persistentes en un almacenamiento permanente y duradero. Al mismo tiempo, nuestra base de datos en memoria es en gran medida compatible con los sistemas de gestión de bases de datos relacionales (RDBMS) de grado de producción como MySQL, SQL Server y muchos otros. Específicamente, utiliza JDBC (la biblioteca Java estándar para la conectividad de bases de datos) y SQL (el lenguaje de consulta de base de datos estándar).

![[db-types.png]]
Hay ventajas para usar una base de datos en memoria en lugar de una base de datos persistente. Por un lado, la memoria le permite desarrollar sin instalar un **RDBMS** separado, y garantiza que la base de datos esté en el mismo estado (es decir, vacía) en cada ejecución de prueba. Sin embargo, necesita una base de datos persistente para la aplicación de "producción" en vivo. Esto conduce a un desajuste de paridad **Dev-Prod**: su aplicación puede comportarse de manera diferente cuando se ejecuta la base de datos en memoria que cuando se ejecuta en producción.

La base de datos específica en memoria que usaremos es **H2**. Afortunadamente, **H2** es altamente compatible con otras bases de datos relacionales, por lo que la paridad de dev-prod no será un gran problema. Utilizaremos **H2** por conveniencia para el desarrollo local, pero queremos reconocer las compensaciones.

## Auto Configuration

En el laboratorio, todo lo que necesitamos para la funcionalidad completa de la base de datos es añadir dos dependencias. Esto muestra maravillosamente una de las características más poderosas de **Spring Boot**: Configuración automática. Sin **Spring Boot**, tendríamos que configurar **Spring Data** para hablar con **H2**. Sin embargo, debido a que hemos incluido la dependencia de **Spring Data** (y un proveedor de datos específico, **H2**), **Spring Boot** configurará automáticamente su aplicación para comunicarse con **H2**.

## Spring Data’s CrudRepository

Para nuestra selección de repositorio, utilizaremos un tipo específico de repositorio: Spring Data's `CrudRepository`. A primera vista, es un poco mágico, pero vamos a desempaquetar esa magia.

La siguiente es una implementación completa de todas las operaciones de **CRUD** mediante la extensión de `CrudRepository`:
```java
interface CashCardRepository extends CrudRepository<CashCard, Long> {
}
```

Con solo el código anterior, una persona que llama puede llamar a cualquier número de métodos predefinidos de `CrudRepository`, como `findById`:

```java
cashCard = cashCardRepository.findById(99);
```

Puede que te preguntes de inmediato: ¿Dónde está la implementación del método `CashCardRepository.findById()`? ¡`CrudRepository` y todo de lo que hereda es una interfaz sin código real! Bueno, basado en el marco específico de **Spring Data** utilizado (que para nosotros será **Spring Data JDBC**), **Spring Data** se encarga de esta implementación para nosotros durante el tiempo de inicio del contenedor **IoC**. El tiempo de ejecución de **Spring** expondrá el repositorio como otro **bean** más al que puede hacer referencia donde sea necesario en su aplicación.

Como hemos aprendido, normalmente hay compensaciones. Por ejemplo, el `CrudRepository` genera declaraciones **SQL** para leer y escribir sus datos, lo que es útil para muchos casos, pero a veces necesita escribir sus propias declaraciones **SQL** personalizadas para casos de uso específicos. Por ahora, nos complace aprovechar sus métodos convenientes y listos para usar, así que vamos al laboratorio.

## Summary

La mayoría de las aplicaciones son inútiles sin los datos que los usuarios necesitan gestionar. Por suerte, **Spring Data** se integra a la perfección con **Spring** y **Spring Boot** para hacer que el almacenamiento y la gestión de datos sean muy fáciles. Los repositorios plug-and-play minimizan la necesidad de escribir franjas de código de gestión de datos personalizado. Además, el potente motor de configuración automática de **Spring** funciona junto con los diversos componentes de **Spring** para hacer que la implementación de una arquitectura en capas sea una obviedad.

Ahora, vamos al laboratorio, donde agregará conectividad de base de datos a su **API REST** utilizando **Spring Data Repositories**.