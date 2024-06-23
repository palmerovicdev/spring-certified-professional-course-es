# Write a Spring Boot Test for the GET endpoint

Como si estuviéramos en un proyecto real, usemos el desarrollo impulsado por pruebas para implementar nuestro primer endpoint de la **API**.

1. Escribe la prueba.

Comencemos implementando una prueba usando el `@SpringBootTest` de **Spring**.

Actualice `src/test/java/example/cashcard/CashCardApplicationTests.java` con lo siguiente:

```java
    package example.cashcard;
    
    import com.jayway.jsonpath.DocumentContext;
    import com.jayway.jsonpath.JsonPath;
    import org.junit.jupiter.api.Test;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.boot.test.web.client.TestRestTemplate;
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    
    import static org.assertj.core.api.Assertions.assertThat;
    
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
    class CashCardApplicationTests {
        @Autowired
        TestRestTemplate restTemplate;
    
        @Test
        void shouldReturnACashCardWhenDataIsSaved() {
            ResponseEntity<String> response = restTemplate.getForEntity("/cashcards/99", String.class);
    
            assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        }
    }
```

2. Entender la prueba.

Vamos a entender varios elementos importantes en esta prueba.

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
```

Esto iniciará nuestra aplicación **Spring Boot** y la pondrá a disposición para que nuestra prueba realice solicitudes.

```java
@Autowired
TestRestTemplate restTemplate;
```

Le hemos pedido a **Spring** que inyecte un ayudante de prueba que nos permita hacer solicitudes **HTTP** a la aplicación que se ejecuta localmente.

Nota: A pesar de que `@Autowired` es una forma de inyección de dependencia de **Spring**, es mejor usarla solo en pruebas. No te preocupes, discutiremos esto con más detalle más tarde.

```java
ResponseEntity<String> response = restTemplate.getForEntity("/cashcards/99", String.class);
```

Aquí usamos `restTemplate` para hacer una solicitud `HTTP GET` a nuestro endpoint de aplicación `/cashcards/99`.

`restTemplate` devolverá una `ResponseEntity`, que hemos capturado en una variable a la que hemos llamado **response**. `ResponseEntity` es otro objeto útil de **Spring** que proporciona información valiosa sobre lo que sucedió con nuestra solicitud. Utilizaremos esta información a lo largo de nuestras pruebas en este curso.

```java
assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
```

Podemos inspeccionar muchos aspectos de la respuesta, incluido el código de estado de la respuesta **HTTP**, que esperamos que esté `200 OK`.

3. Ahora ejecuta la prueba.

¿Qué crees que pasará cuando hagamos la prueba?

Fallará, como se esperaba. ¿Por qué? Como hemos aprendido en la práctica de prueba primero, describimos nuestras expectativas antes de implementar el código que satisface esas expectativas.

Ahora, vamos a hacer la prueba. Tenga en cuenta que ejecutaremos `./gradlew test` para cada ejecución de la prueba.

```shell
[~/exercises] $ ./gradlew test
```

¡Es un fracaso! Busque en la salida lo siguiente:

```shell
CashCardApplicationTests > shouldReturnACashCardWhenDataIsSaved() FAILED
  org.opentest4j.AssertionFailedError:
  expected: 200 OK
   but was: 404 NOT_FOUND
```

Pero, ¿por qué estamos teniendo este fracaso específico?

4. Comprender el fallo de la prueba.

Como explicamos, esperábamos que nuestra prueba fallara actualmente.

¿Por qué está fallando debido a un inesperado código de respuesta `HTTP 404 NOT_FOUND`?

Respuesta: Dado que no hemos instruido a **Spring Web** sobre cómo manejar `GET cashcards/99`, **Spring Web** responde automáticamente que el endpoint es `NOT_FOUND`.

¡Gracias por encargarte de eso por nosotros, **Spring Web**!

A continuación, hagamos que nuestra aplicación funcione correctamente.

# Create a REST Controller

Los controladores web de **Spring** están diseñados para manejar y responder a las solicitudes **HTTP**.

1. Crea el controlador.

Crea la clase **Controller** en `src/main/java/example/cashcard/CashCardController.java`.

```java
package example.cashcard;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

class CashCardController {
}
```

2. Añade el método del controlador.

Implemente un método `findById()` para manejar las solicitudes **HTTP** entrantes.

```java
class CashCardController {
   private ResponseEntity<String> findById() {
      return ResponseEntity.ok("{}");
   }
}
```

3. Ahora vuelve a ejecutar la prueba.

¿Qué esperamos que suceda cuando volvemos a realizar las pruebas?

```shell
expected: 200 OK
 but was: 404 NOT_FOUND
```

¡Mismo resultado! ¿Por qué?

A pesar del nombre, `CashCardController` no es realmente un **Spring Web Controller**; es solo una clase con el controlador en el nombre. Por lo tanto, no "escucha" nuestras solicitudes **HTTP**. Por lo tanto, tenemos que decirle a **Spring** que haga que el **Controller** esté disponible como **Web Controller** para manejar las solicitudes con **URL**: `cashcard/*`.

# Add the GET endpoint

1. Actualice el controlador.

Actualicemos nuestro `CashCardController` para que esté configurado para escuchar y manejar las solicitudes **HTTP** a `/cashcards`.

```java
    @RestController
    @RequestMapping("/cashcards")
    class CashCardController {
    
       @GetMapping("/{requestedId}")
       private ResponseEntity<String> findById() {
             return ResponseEntity.ok("{}");
       }
    }
```

2. Comprende las anotaciones de **Spring Web**.

Revisemos nuestras adiciones.

```java
@RestController
```

Esto le dice a **Spring** que esta clase es un componente de tipo `RestController` y capaz de manejar solicitudes **HTTP**.

```java
@RequestMapping("/cashcards")
```

Este es un complemento de `@RestController` que indica qué solicitudes de dirección deben tener para acceder a este controlador.

```java
@GetMapping("/{requestedId}")
Private ResponseEntity<String> findById() {...}
```

`@GetMapping` marca un método como método de controlador. Las solicitudes **GET** que coincidan con `cashcard/{requestedID}` se manejarán con este método.

3. Ejecuta las pruebas.

¡Ellas pasan!

```shell
[~/ejercicios] $ ./gradlew test

...

BUILD SUCCESSFUL in 6s
```

Finalmente tenemos un **Controller** y un endpoint que coincide con la solicitud realizada en nuestra prueba. ¡Genial!

# Complete the GET endpoint

A partir de ahora, nuestra prueba solo afirma que la solicitud tuvo éxito comprobando un estado de respuesta de `200 OK`. A continuación, vamos a probar que la respuesta contiene los valores correctos.

1. Actualiza la prueba.

```java
assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);

DocumentContext documentContext = JsonPath.parse(response.getBody());
Number id = documentContext.read("$.id");
assertThat(id).isNotNull();
```

2. Comprende las adiciones.

```java
DocumentContext documentContext = JsonPath.parse(response.getBody());
```

Esto convierte la cadena de respuesta en un objeto con uso de **JSON** con muchos métodos de ayuda.

```java
Number id = documentContext.read("$.id");
assertThat(id).isNotNull();
```

Esperamos que cuando solicitemos una `CashCard` con una identificación de `99`, se devuelva un objeto **JSON** con algo en el campo de identificación. Por ahora, afirma que la identificación no es nula.

3. Ejecute la prueba y vea el fallo.

Dado que devolvemos un objeto **JSON** vacío `{}`, no deberíamos sorprendernos de que el campo id esté vacío.

```java
    CashCardApplicationTests > shouldReturnACashCardWhenDataIsSaved() FAILED
        com.jayway.jsonpath.PathNotFoundException: No results for path: $['id']
    ```

4. Devuelve una `CashCard` del controlador.

Hagamos la prueba, pero devolvamos algo intencionalmente incorrecto, como `1000L`. Verás por qué más tarde.

Además, utilicemos la clase de modelo de datos de `CashCard` que creamos en una lección anterior. Por favor, revíselo en `src/main/java/example/cashcard/CashCard.java`, si es necesario.

```java
    @GetMapping("/{requestedId}")
    private ResponseEntity<CashCard> findById() {
       CashCard cashCard = new CashCard(1000L, 0.0);
       return ResponseEntity.ok(cashCard);
    }
```

5. Ejecuta la prueba.

¡Pasa! Sin embargo, ¿se siente correcto? En realidad no. Hacer que la prueba pase con datos incorrectos parece incorrecto.

Le pedimos que devolviera intencionalmente una identificación incorrecta de `1000L` para ilustrar un punto: es importante que las pruebas pasen o no por la razón correcta.

6. Actualiza la prueba.

Actualice la prueba para afirmar que la identificación es correcta.

```java
    DocumentContext documentContext = JsonPath.parse(response.getBody());
    Number id = documentContext.read("$.id");
    assertThat(id).isEqualTo(99);
```

7. Vuelva a ejecutar las pruebas y note el nuevo mensaje de fallo.

```shell
expected: 99
 but was: 1000
```

Ahora, la prueba está fallando por la razón correcta: no devolvimos la identificación correcta.

8. Arreglar `CashCardController`.

Actualice `CashCardController` para devolver la identificación correcta.

```java
    @GetMapping("/{requestedId}")
    private ResponseEntity<CashCard> findById() {
       CashCard cashCard = new CashCard(99L, 0.0);
       return ResponseEntity.ok(cashCard);
    }
```

9. Ejecuta la prueba.

¡Woo hoo, pasa!

10. Prueba la cantidad.

A continuación, agreguemos una afirmación para la cantidad indicada por el contrato **JSON**.

```java
    DocumentContext documentContext = JsonPath.parse(response.getBody());
    Number id = documentContext.read("$.id");
    assertThat(id).isEqualTo(99);
    
    Double amount = documentContext.read("$.amount");
    assertThat(amount).isEqualTo(123.45);
```

11. Ejecuta las pruebas y observa el fallo.

Por supuesto, no devolvemos la cantidad correcta en la respuesta.

```java
    expected: 123.45
    but was: 0.0
```

12. Devuelve la cantidad correcta.

Actualicemos el `CashCardController` para devolver la cantidad indicada por el contrato **JSON**.

```java
    @GetMapping("/{requestedId}")
    private ResponseEntity<CashCard> findById() {
       CashCard cashCard = new CashCard(99L, 123.45);
       return ResponseEntity.ok(cashCard);
    }
```

13. Vuelve a ejecutar las pruebas.

¡Ellas pasan! Excelente.

```shell
BUILD SUCCESSFUL in 6s
```

# Using the @PathVariable

Hasta ahora, hemos ignorado el **ID** solicitado en el método de controlador del controlador. Usemos esta variable de ruta en nuestro controlador para asegurarnos de devolver la `CashCard` correcta.

1. Añade un nuevo método de prueba.

Escribamos una nueva prueba que espera ignorar las `CashCard` que no tienen una identificación de `99`. Usa `1000`, como lo hemos hecho en pruebas anteriores.

```java
@Test
void shouldNotReturnACashCardWithAnUnknownId() {
  ResponseEntity<String> response = restTemplate.getForEntity("/cashcards/1000", String.class);

  assertThat(response.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);
  assertThat(response.getBody()).isBlank();
}
```

Tenga en cuenta que estamos esperando un código de estado de respuesta **HTTP** semántica de `404 NOT_FOUND`. Si solicitamos una `CashCard` que no existe, entonces esa `CashCard` de hecho "no se encuentra".

2. Ejecute la prueba y anote el resultado.

```shell
    expected: 404 NOT_FOUND
    but was: 200 OK
```

3. Añade `@PathVariable`.

Hagamos que la prueba pase haciendo que el **Controller** devuelva la `CashCard` específica solo si enviamos el identificador correcto.

Para hacer esto, primero haga que el controlador sea consciente de la variable de ruta que estamos enviando, agregando la anotación `@PathVariable` al argumento del método del controlador.

```java
@GetMapping("/{requestedId}")
private ResponseEntity<CashCard> findById(@PathVariable Long requestedId) {
    CashCard cashCard = new CashCard(99L, 123.45);
    return ResponseEntity.ok(cashCard);
}
```

`@PathVariable` hace que **Spring Web** sea consciente de la identificación solicitada proporcionada en la solicitud **HTTP**. Ahora está disponible para que lo usemos en nuestro método de manejo.

4. Utilice `@PathVariable`.

Actualice el método del controlador para devolver una respuesta vacía con el estado `NOT_FOUND` a menos que el **ID** solicitado sea `99`.

```java
    @GetMapping("/{requestedId}")
    private ResponseEntity<CashCard> findById(@PathVariable Long requestedId) {
        if (requestedId.equals(99L)) {
            CashCard cashCard = new CashCard(99L, 123.45);
            return ResponseEntity.ok(cashCard);
        } else {
            return ResponseEntity.notFound().build();
        }
    }
```

5. Vuelve a ejecutar las pruebas.

¡Increíble! ¡Ellas pasan!

```shell
[~/exercises] $ ./gradlew test 
... 
BUILD SUCCESSFUL in 6s
```

# Summary

¡Enhorabuena! En esta lección aprendiste a usar el desarrollo basado en pruebas para crear tu primer endpoint **REST** de **Family Cash Card**: un **GET** que devuelve una `CashCard` de una determinada identificación.