# Overview

Como se analizó en la lección asociada, permitiremos a los usuarios de la `API REST` de `Family Cash Card` actualizar una `CashCard` mediante un `HTTP PUT`.

¡Implementemos esta funcionalidad ahora!

# Write the Test First

Como lo hemos hecho en casi todos los laboratorios, comencemos con una prueba.

¿Cuál es la funcionalidad que queremos que tenga nuestra aplicación? ¿Cómo queremos que se comporte nuestra aplicación?

Definamos esto ahora y luego avancemos hacia la satisfacción de nuestras aspiraciones.

### 1. Escribe la prueba de actualización.

Usaremos `PUT` para actualizar `CashCards`.

Tenga en cuenta que esperaremos una respuesta `204 NO_CONTENT` en lugar de `200 OK`. El `204` indica que la acción se realizó con éxito y que la persona que llama no necesita realizar ninguna otra acción.

Edite `src/test/java/example/cashcard/CashCardApplicationTests.java` y agregue la siguiente prueba, que actualiza `CashCard 99` y establece su monto en `19,99`.

No olvides agregar las dos nuevas importaciones.

```java
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpMethod;
...
@Test
@DirtiesContext
void shouldUpdateAnExistingCashCard() {
    CashCard cashCardUpdate = new CashCard(null, 19.99, null);
    HttpEntity<CashCard> request = new HttpEntity<>(cashCardUpdate);
    ResponseEntity<Void> response = restTemplate
            .withBasicAuth("sarah1", "abc123")
            .exchange("/cashcards/99", HttpMethod.PUT, request, Void.class);
    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.NO_CONTENT);
}
```

#### Learning Moment: what's up with `restTemplate.exchange()`?

¿Notaste que no estamos usando `RestTemplate` de la misma manera que lo hicimos en nuestras pruebas anteriores?

Todas las demás pruebas utilizan métodos `RestTemplate.xyzForEntity()` como `getForEntity()` y `postForEntity()`.

Entonces, ¿por qué no seguimos el mismo patrón de utilización de `putForEntity()`?

- Respuesta: ¡`putForEntity()` no existe! Lea más sobre esto aquí en una [issue de `GitHub`](https://github.com/spring-projects/spring-framework/issues/15256)
  sobre el tema.

Afortunadamente, `RestTemplate` admite múltiples formas de interactuar con las `API REST`, como `RestTemplate.exchange()`.

Aprendamos ahora sobre `RestTemplate.exchange()`.

### 2. Comprenda `RestTemplate.exchange()`.
Entendamos más sobre lo que está pasando.

El método `exchange()` es una versión más general de los métodos `xyzForEntity()` que hemos usado en otras pruebas: `exchange()` requiere que el verbo y la entidad de solicitud (el cuerpo de la solicitud) se proporcionen como parámetros.

Usando `getForEntity()` como ejemplo, puedes imaginar que las siguientes dos líneas de código logran el mismo objetivo:
`.exchange("/cashcards/99", HttpMethod.GET, nueva HttpEntity(null), String.class);`

La línea anterior es funcionalmente equivalente a la siguiente línea:
`.getForEntity("/cashcards/99", String.class);`

Ahora expliquemos el código de prueba.

- Primero creamos la `HttpEntity` que necesita el método `exchange()`:
```java
HttpEntity<CashCard> request = new HttpEntity<>(existingCashCard);
```

Luego llamamos a `exchange()`, que envía una solicitud `PUT` para el `ID` de destino `99` y datos actualizados de la `CashCard`:
```java
.exchange("/cashcards/99", HttpMethod.PUT, request, Void.class);
```

### 3. Run the tests.
Es hora de ver cómo nuestras pruebas fallan por las razones correctas.

¿Por qué fallará nuestra nueva prueba?

Tenga en cuenta que siempre usaremos `./gradlew test` para ejecutar nuestras pruebas.

```shell
[~/exercises] $ ./gradlew test
...
CashCardApplicationTests > shouldUpdateAnExistingCashCard() FAILED
 org.opentest4j.AssertionFailedError:
 expected: 204 NO_CONTENT
  but was: 403 FORBIDDEN
...
BUILD FAILED in 6s

```

- Respuesta: ¡Aún no hemos implementado un controlador de método de solicitud `PUT`!

Como puede ver, la prueba falló con un código de respuesta `403 FORBIDDEN`.

¡Sin un endpoint del controlador, esta llamada `PUT` está prohibida! `Spring Security` manejó automáticamente este escenario por nosotros. ¡Lindo!

A continuación, implementemos el endpoint del controlador.