# Test the HTTP POST Endpoint

Como hemos hecho en laboratorios anteriores, comenzaremos escribiendo una prueba de cómo esperamos que sea el éxito.

1. Añade una prueba para el endpoint de **POST**.

El ejemplo más simple de éxito es una solicitud **HTTP POST** no fallida a nuestra **API** de **Family Cash Card**. Probaremos una respuesta de `200 OK` en lugar de un `201 CREATED` por ahora. No te preocupes, cambiaremos esto pronto.

Edite `src/test/java/example/cashcard/CashCardApplicationTests.java` y agregue el siguiente método de prueba.

```java
    @Test
    void shouldCreateANewCashCard() {
       CashCard newCashCard = new CashCard(null, 250.00);
       ResponseEntity<Void> createResponse = restTemplate.postForEntity("/cashcards", newCashCard, Void.class);
       assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
    }
```

2. Entender la prueba.
```java
CashCard newCashCard = new CashCard(null, 250.00);
```

La base de datos creará y administrará todos los valores únicos de `CashCard.id` para nosotros. No deberíamos proporcionar uno.

```java
restTemplate.postForEntity("/cashcards", newCashCard, Void.class);
```

Esto es muy similar a `restTemplate.getForEntity`, pero también debemos proporcionar nuevos datos de `CashCard` para la nueva `CashCard`.

Además, y a diferencia de `restTemplate.getForEntity`, no esperamos que se nos devuelva una `CashCard`, por lo que esperamos un cuerpo de respuesta de `Void`.

3. Corre las pruebas.
Siempre usaremos  `./gradlew test` para ejecutar nuestras pruebas.

```shell
[~/exercises] $ ./gradlew test
```

¿Qué esperas que suceda?

```java
CashCardApplicationTests > shouldCreateANewCashCard() FAILED
   org.opentest4j.AssertionFailedError:
   expected: 200 OK
   but was: 404 NOT_FOUND
```

No deberíamos sorprendernos por el error `404 NOT_FOUND`. ¡Todavía no hemos añadido el endpoint **POST**!

Hagámoslo a continuación.

# Add the POST endpoint

El endpoint **POST** es similar al endpoint **GET** en nuestro `CashCardController`, pero utiliza la anotación `@PostMapping` de **Spring Web**.

El endpoint de **POST** debe aceptar los datos que estamos enviando para nuestra nueva `CashCard`, específicamente la cantidad.

Pero, ¿qué pasa si no aceptamos la `CashCard`?

1. Añade el endpoint **POST** sin aceptar datos de `CashCard`.

Edite `src/main/java/example/cashcard/CashCardController.java` y agregue el siguiente método.

No olvides añadir la importación para `PostMapping`.

```java
import org.springframework.web.bind.annotation.PostMapping;
...

@PostMapping
private ResponseEntity<Void> createCashCard() {
   return null;
}
```

Tenga en cuenta que al no devolver nada en absoluto, **Spring Web** generará automáticamente un código de estado de respuesta **HTTP** de `200 OK`.

2. Ejecuta las pruebas.

Cuando repetimos las pruebas, pasan.

```shell
BUILD SUCCESSFUL in 7s
```

Pero esto no es muy satisfactorio, ¡nuestro endpoint **POST** no hace nada!

Así que mejoremos nuestras pruebas.

# Testing based on semantic correctness

Queremos que nuestra **API** de `CashCard` se comporte de la manera más semántica posible. Es decir, los usuarios de nuestra **API** no deberían sorprenderse por cómo se comporta.

Consultemos la solicitud oficial de comentarios para la semántica y el contenido **HTTP** ([RFC 9110](https://www.rfc-editor.org/rfc/rfc9110)) para obtener orientación sobre cómo debe comportarse nuestra **API**.

Para nuestro endpoint de **POST**, revise esta sección sobre [HTTP POST](https://www.rfc-editor.org/rfc/rfc9110#name-post); tenga en cuenta que hemos añadido énfasis:

> Si se han creado uno o más recursos en el servidor de origen como resultado del procesamiento exitoso de una solicitud **POST**, el servidor de origen **DEBE** enviar una respuesta `201` (creada) que contenga un campo de encabezado de ubicación que proporcione un identificador para el recurso principal creado...

Explicaremos más sobre esta especificación mientras escribimos nuestra prueba.

1. Comenzemos actualizando la prueba **POST**.

Actualice la prueba `shouldCreateANewCashCard`.

Así es como codificaremos la especificación **HTTP** como expectativas en nuestra prueba. Asegúrese de añadir la importación adicional.

```java
    import java.net.URI;
    ...
    
    @Test
    void shouldCreateANewCashCard() {
       CashCard newCashCard = new CashCard(null, 250.00);
       ResponseEntity<Void> createResponse = restTemplate.postForEntity("/cashcards", newCashCard, Void.class);
       assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);
    
       URI locationOfNewCashCard = createResponse.getHeaders().getLocation();
       ResponseEntity<String> getResponse = restTemplate.getForEntity(locationOfNewCashCard, String.class);
       assertThat(getResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
    }
```

2. Comprende las actualizaciones de la prueba.

Hemos hecho bastantes cambios. Vamos a revisar.

```java
assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);
```

Según la especificación oficial:

> El servidor de origen **DEBERÍA** enviar una respuesta **201** (creada)...

Ahora esperamos que el código de estado de la respuesta **HTTP** sea `201 CREATED`, que es semánticamente correcto si nuestra **API** crea una nueva `CashCard` a partir de nuestra solicitud.

```java
URI locationOfNewCashCard = createResponse.getHeaders().getLocation();
```

La especificación oficial sigue afirmando lo siguiente:

> enviar una respuesta `201` (creada) que contenga un campo de encabezado de ubicación que proporcione un identificador para el recurso principal creado...

En otras palabras, cuando una solicitud **POST** da como resultado la creación exitosa de un recurso, como una nueva `CashCard`, la respuesta debe incluir información sobre cómo recuperar ese recurso. Lo haremos proporcionando un **URI** en un [encabezado de respuesta (Response Header)](https://www.rfc-editor.org/rfc/rfc9110#section-10.2.2) llamado **"Location"**.

Tenga en cuenta que **URI** es de hecho la entidad correcta aquí y no una **URL**; una **URL** es un tipo de **URI**, mientras que un **URI** es más genérico.

```java
ResponseEntity<String> getResponse = restTemplate.getForEntity(locationOfNewCashCard, String.class);
assertThat(getResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
```
Por último, utilizaremos la información del encabezado de la ubicación para buscar la `CashCard` recién creada.

3. Ejecuta las pruebas.

No es de esperar que fallen en la primera **assertion** cambiada.

```java
    expected: 201 CREATED
      but was: 200 OK
```

¡Empecemos a arreglar las cosas!

# Implement the POST Endpoint

Nuestro endpoint **POST** en `CashCardController` está actualmente vacío. Implementemos la lógica correcta.

1. Devuelve un estado `201 CREATED`.

A medida que aprobamos gradualmente nuestra prueba, podemos comenzar devolviendo `201 CREATED`.

Como aprendimos anteriormente, debemos proporcionar un encabezado de ubicación con el **URI** para saber dónde encontrar la `CashCard` recién creada. Todavía no hemos llegado, así que usaremos un **URI** de marcador de posición por ahora.

Asegúrese de añadir las dos nuevas declaraciones de importación.

```java
    import java.net.URI;
    import org.springframework.web.bind.annotation.RequestBody;
    ...
    
     @PostMapping
     private ResponseEntity<Void> createCashCard(@RequestBody CashCard newCashCardRequest) {
         return ResponseEntity.created(URI.create("/what/should/go/here?")).build();
     }
```

2. Corra las pruebas.
```shell
expected: 200 OK
 but was: 404 NOT_FOUND
```

Sorprendentemente, nuestra nueva prueba pasa hasta la última línea de la nueva prueba.

```java
...
assertThat(getResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
```

Aquí esperamos haber recuperado nuestra `CashCard` recién creada, que no hemos creado o devuelto de nuestro `CashCardController`. Por lo tanto, nuestra expectativa falla con un resultado de `NOT_FOUND`.

3. Guarda la nueva `CashCard` y devuelve su ubicación.

Añadamos el resto de la implementación de **POST**, que describiremos en detalle.

Asegúrate de añadir la nueva importación.

```java
    import org.springframework.web.util.UriComponentsBuilder;
    ...
    
    @PostMapping
    private ResponseEntity<Void> createCashCard(@RequestBody CashCard newCashCardRequest, UriComponentsBuilder ucb) {
       CashCard savedCashCard = cashCardRepository.save(newCashCardRequest);
       URI locationOfNewCashCard = ucb
                .path("cashcards/{id}")
                .buildAndExpand(savedCashCard.id())
                .toUri();
       return ResponseEntity.created(locationOfNewCashCard).build();
    }
```

A continuación, repasaremos estos cambios en detalle.

# Understand CrudRepository.save

Esta línea en `CashCardController.createCashCard` es engañosamente simple:

```java
CashCard savedCashCard = cashCardRepository.save(newCashCardRequest);
```

Como se aprendió en lecciones y laboratorios anteriores, `CrudRepository` de **Spring Data** proporciona métodos que admiten la creación, lectura, actualización y eliminación de datos de un almacén de datos. `cashCardRepository.save(newCashCardRequest)` hace justo lo que dice: guarda una nueva `CashCard` para nosotros y devuelve el objeto guardado con una identificación única proporcionada por la base de datos. ¡Increíble!

# Understand the other changes to CashCardController

Nuestro `CashCardController` ahora implementa la entrada y los resultados esperados de un `HTTP POST`.
```java
createCashCard(@RequestBody CashCard newCashCardRequest, ...)
```

A diferencia del **GET** que añadimos anteriormente, el **POST** espera un "cuerpo" de solicitud. Esto contiene los datos enviados a la **API**. **Spring Web** deserializará los datos en una `CashCard` para nosotros.

```java
URI locationOfNewCashCard = ucb
   .path("cashcards/{id}")
   .buildAndExpand(savedCashCard.id())
   .toUri();
```

Esto es construir un **URI** para la `CashCard` recién creada. Este es el **URI** que la persona que llama puede usar para obtener por medio de un **GET** la `CashCard` recién creada.

Tenga en cuenta que `savedCashCard.id` se utiliza como identificador, que coincide con la especificación del endpoint **GET** de `CashCard` `/<CashCard.id>`.

- ¿De dónde viene `UriComponentsBuilder`?

Pudimos agregar `UriComponentsBuilder ucb` como un argumento de método a este método de controlador **POST** y se pasó automáticamente. ¿Cómo es así? Fue inyectado por nuestro amigo ahora familiar, el contenedor **IoC** de **Spring**. ¡Gracias, **Spring Web**!

```
return ResponseEntity.created(locationOfNewCashCard).build();
```

Por último, devolvemos `201 CREATED` con el encabezado de `Location` correcto.

# Final Testing and Learning Moment

1. Ejecuta las pruebas.

¡Ellas pasan!

```shell
BUILD SUCCESSFUL in 7s
```

Se creó la nueva `CashCard`, y utilizamos el **URI** proporcionado en el encabezado de respuesta de `Location` para recuperar el recurso recién creado.

2. Añade más **assertions** de prueba.

Si lo desea, agregue más **assertions** de prueba para la nueva identificación y cantidad para solidificar su aprendizaje.

```java
...
assertThat(getResponse.getStatusCode()).isEqualTo(HttpStatus.OK);

// Add assertions such as these
DocumentContext documentContext = JsonPath.parse(getResponse.getBody());
Number id = documentContext.read("$.id");
Double amount = documentContext.read("$.amount");

assertThat(id).isNotNull();
assertThat(amount).isEqualTo(250.00);
```

Las adiciones verifican que el nuevo `CashCard.id` no es nulo, y que la cantidad de `CashCard`. recién creada es de `250,00`, tal y como especificamos en el momento de la creación.

### Learning Moment

Anteriormente dijemos que la base de datos (a través del **Repository**) gestionaría la creación de todos los valores de identificación de la base de datos para nosotros.

¿Qué pasaría si proporcionáramos una identificación para nuestra nueva `CashCard` sin guardar?

Vamos a averiguarlo.

1. Actualice la prueba para enviar un `CashCard.id`

Cambie la identificación enviada de nula a una que no exista, como `44L`.

```java
@Test
void shouldCreateANewCashCard() {
   CashCard newCashCard = new CashCard(44L, 250.00);
   ...
```

Además, edite `build.gradle` para permitir una salida de prueba más detallada, lo que nos ayudará a identificar el próximo fallo de la prueba.

```groovy
    test {
      testLogging {
        ...
        // Set to `true` for more detailed logging.
        showStandardStreams = true
      }
    }
```

2. Ejecuta las pruebas.

Cuando ejecutamos la prueba, vemos que la **API** se bloquea con un código de estado de `500`.

```shell
[~/exercises] $ ./gradlew test
...
expected: 201 CREATED
 but was: 500 INTERNAL_SERVER_ERROR
```

Averigüemos por qué la prueba está fallando.

3. Encuentre y comprenda el fallo de la base de datos.

Busca el siguiente mensaje en la salida de la prueba:

```shell
Failed to update entity [CashCard[id=44, amount=250.0]]. Id [44] not found in database.
```

El Repositorio está tratando de encontrar `CashCard` con una identificación de `44` y lanza un error cuando no puede encontrarla. ¡Interesante! ¿Puedes adivinar por qué?

El suministro de una identificación a `cashCardRepository.save` se admite cuando se realiza una actualización en un recurso existente.

Cubriremos este escenario en un laboratorio posterior centrado en la actualización de una `CashCard` existente.

En este momento de aprendizaje aprendiste que la `API` requiere que no proporciones una `CashCard.id` al crear una nueva `CashCard`.

¿Deberíamos validar ese requisito en la **API**? ¡Puedes apostar! Una vez más, estad atentos a cómo hacerlo en una lección futura.

#  Summary

En este laboratorio aprendiste lo sencillo que es añadir otro endpoint a nuestra **API**: el endpoint **POST**. También aprendiste a usar ese endpoint para crear y guardar una nueva `CashCard` en nuestra base de datos usando **Spring Data**. No solo eso, sino que el endpoint implementa con precisión la especificación `HTTP POST`, que verificamos utilizando el desarrollo basado en pruebas. ¡La **API** está empezando a ser útil!