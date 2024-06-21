En esta lección, aprenderás qué es **REST** y cómo usar **Spring Boot** para implementar un solo endpoint **RESTful**.

## REST, CRUD, and HTTP

Comenciemos con una definición concisa de **REST**: Transferencia de Estado Representacional (Representational State Transfer). En un sistema **RESTful**, los objetos de datos se llaman Representaciones de Recursos (Resource Representations). El propósito de una **API RESTful** (Interfaz de Programación de Aplicaciones) es gestionar el estado de estos recursos.

Dicho de otra manera, se puede pensar que el "estado" es "valor" y la "representación de recursos" es un "objeto" o "cosa". Por lo tanto, REST es solo una forma de gestionar los valores de las cosas. Se puede acceder a esas cosas a través de una API, y a menudo se almacenan en un almacén de datos persistente, como una base de datos.
![[rest-http-flow.png]]
Un concepto mencionado con frecuencia cuando se habla de REST es **CRUD**. CRUD significa "create, read, update y delete". Estas son las cuatro operaciones básicas que se pueden realizar en objetos de una base de datos. Aprenderemos que REST tiene directrices específicas para implementar cada una.

Otro concepto común asociado con REST es el Protocolo de Transferencia de Hipertexto (HTTP). En **HTTP**, una persona que llama envía una solicitud a un URI. Un servidor web recibe la solicitud y la dirige a un controlador de solicitudes. El controlador crea una respuesta, que luego se envía de vuelta a la persona que llama.

Los componentes de la solicitud y respuesta son:

Request
- Método (también llamado Verb)
- URI (también llamado endpoint)
- Body

Response

- Código de estado
- Body

Si quieres profundizar más en los métodos de solicitud y respuesta, echa un vistazo al **estándar HTTP**.

El poder de REST radica en la forma en que hace referencia a un recurso y en cómo se ven la solicitud y la respuesta para cada operación de CRUD. Echemos un vistazo a cómo será nuestra API cuando terminemos con este curso:

- Para CREATE: utilice el método HTTP POST.
- Para READ: utilice el método HTTP GET.
- Para UPDATE: utilice el método HTTP PUT.
- Para DELETE: utilice el método HTTP DELETE.

El **URI** del punto final para los objetos de CashCard comienza con la palabra clave `/cashcards`. Las operaciones de READ, UPDATE y DELETE requieren que proporcionemos el identificador único del recurso de destino. La aplicación necesita este identificador único para realizar la acción correcta exactamente en el recurso correcto. Por ejemplo, para hacer READ, UPDATE o DELETE a una tarjeta de efectivo con el identificador de "42", el punto final sería `/cashcards/42`.

Tenga en cuenta que no proporcionamos un identificador único para la operación CREATE. Como aprenderemos con más detalle en futuras lecciones, CREATE tendrá el efecto secundario de crear una nueva tarjeta de efectivo con una nueva identificación única. No se debe proporcionar un identificador al crear una nueva tarjeta de efectivo porque la aplicación creará un nuevo identificador único para nosotros.

La siguiente tabla tiene más detalles sobre las operaciones de RESTful CRUD.

| Operation | API Endpoint      | HTTP Method | Response Status  |
| --------- | ----------------- | ----------- | ---------------- |
| Create    | `/cashcards`      | `POST`      | 201 (CREATED)    |
| Read      | `/cashcards/{id}` | `GET`       | 200 (OK)         |
| Update    | `/cashcards/{id}` | `PUT`       | 204 (NO CONTENT) |
| Delete    | `/cashcards/{id}` | `DELETE`    | 204 (NO CONTENT) |
### The Request Body
Cuando seguimos las convenciones REST para crear o actualizar un recurso, necesitamos enviar datos a la API. Esto se conoce a menudo como el cuerpo de solicitud. Las operaciones CREATE y UPDATE requieren que un cuerpo de solicitud contenga los datos necesarios para crear o actualizar correctamente el recurso. Por ejemplo, una nueva CashCard podría tener una cantidad de valor en efectivo inicial, y una operación de ACTUALIZACIÓN podría cambiar esa cantidad.

### Cash Card Example

Usemos el ejemplo de un endpoint de lectura. Para la operación de lectura, la ruta URI (endpoint) es `/cashcards/{id}`, donde `{id}` se reemplaza por un identificador de tarjeta de efectivo real, sin las llaves, y el método **HTTP** es **GET**.

En las solicitudes **GET**, el cuerpo está vacío. Por lo tanto, la solicitud de leer la CashCard con una identificación de 123 sería:
  
```yaml
Request:
  Method: GET
  URL: http://cashcard.example.com/cashcards/123
  Body: (empty)
```

La respuesta a una solicitud de lectura exitosa tiene un cuerpo que contiene la representación JSON del recurso solicitado, con un código de estado de respuesta de 200 (OK). Por lo tanto, la respuesta a la solicitud de lectura anterior se vería así:
```yaml
Response:
  Status Code: 200
  Body:
  {
    "id": 123,
    "amount": 25.00
  }
```

A medida que avancemos en este curso, también aprenderás a implementar todas las operaciones restantes de CRUD.
## REST in Spring Boot
Ahora que hemos hablado de REST en general, echemos un vistazo a las partes de Spring Boot que usaremos para implementar REST. Comencemos discutiendo el contenedor IoC de Spring.

### Spring Annotations and Component Scan

Una de las principales cosas que hace Spring es configurar e instanciar objetos. Estos objetos se llaman Spring Beans, y generalmente son creados por Spring (en lugar de usar la palabra clave de Java "new"). Puedes dirigir a Spring para que cree Beans de varias maneras.

En esta lección, anotarás una clase con una anotación de Spring, que indica a Spring que cree una instancia de la clase durante la fase de escaneo de componentes de primavera. Esto sucede al iniciar la aplicación. El Bean se almacena en el contenedor de IoC de Spring. Desde aquí, el Bean se puede inyectar en cualquier código que lo solicite.

### Spring Web Controllers

En `Spring Web`, las solicitudes son manejadas por los controladores. En esta lección, utilizarás el `RestController` más específico:
  
```java
@RestController
class CashCardController {
}
```

Eso es todo lo que se necesita para decirle a Spring: "crear un controlador REST". El Controlador se inyecta en Spring Web, que enruta las solicitudes de API (manejadas por el Controlador) al método correcto.

![[webcontroller-implementingGET.jpg]]
Un método de controlador puede ser designado como un método de controlador, que se llamará cuando se recibe una solicitud que el método sabe cómo manejar (llamada "solicitud de coincidencia"). ¡Vamos a escribir un método de controlador de solicitudes de lectura! Aquí hay un comienzo:
  
```java
private CashCard findById(Long requestedId) {
}
```

Dado que REST dice que los endpints de lectura deben usar el método HTTP GET, debe decirle a Spring que enrute las solicitudes al método solo en las solicitudes GET. Puedes usar la anotación @GetMapping, que necesita la ruta URI:

```java
@GetMapping("/cashcards/{requestedId}")
private CashCard findById(Long requestedId) {
}
```

Spring necesita saber cómo obtener el valor del parámetro `requestedId`. Esto se hace usando la anotación `@PathVariable`. El hecho de que el nombre del parámetro coincida con el texto `{requestedId}` dentro del parámetro `@GetMapping` permite a Spring asignar (inyectar) el valor correcto a la variable `requestedId`:

```java
@GetMapping("/cashcards/{requestedId}")
private CashCard findById(@PathVariable Long requestedId) {
}
```

REST dice que la Respuesta debe contener una CashCard en su cuerpo y un código de respuesta de 200 (OK). Spring Web proporciona la clase `ResponseEntity` para este propósito. También proporciona varios métodos de utilidad para producir `ResponseEntities`. Aquí, puede usar `ResponseEntity` para crear una `ResponseEntity` con el código 200 (OK) y un cuerpo que contenga una `CashCard`. La implementación final se ve así:

```java
@RestController
class CashCardController {
  @GetMapping("/cashcards/{requestedId}")
  private ResponseEntity<CashCard> findById(@PathVariable Long requestedId) {
     CashCard cashCard = /* Here would be the code to retrieve the CashCard */;
     return ResponseEntity.ok(cashCard);
  }
}
```

## Summary
En esta lección, aprendiste cómo REST utiliza HTTP para definir las operaciones CRUD en una API. Luego, aprendiste a definir un controlador de primavera para implementar el endpoint de lectura REST, y cómo usar la funcionalidad adicional de Spring para procesar fácilmente la solicitud y la respuesta en un controlador.

¡Ahora, vamos a implementar nuestro primer punto final REST usando Spring y Spring Boot!