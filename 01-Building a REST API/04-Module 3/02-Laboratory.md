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

#  Implement @PutMapping in the Controller

Siguiendo el patrón que hemos usado hasta ahora en nuestro `Controller`, implementemos el endpoint `PUT` en nuestro `CashCardController`.

### 1. Agregue un @PutMapping mínimo.

Edite `src/main/java/example/cashcard/CashCardController.java` y agregue el endpoint `PUT`.

```java
@PutMapping("/{requestedId}")
private ResponseEntity<Void> putCashCard(@PathVariable Long requestedId, @RequestBody CashCard cashCardUpdate) {
    // just return 204 NO CONTENT for now.
    return ResponseEntity.noContent().build();
}
```

Este endpoint del controlador se explica por sí mismo:

- @PutMapping admite el verbo `PUT` y proporciona el `ID` solicitado de destino.
- @RequestBody contiene los datos actualizados de `CashCard`.
- Devuelve un código de respuesta `HTTP 204 NO_CONTENT` por ahora, solo para comenzar.

### 2. Ejecute las pruebas.

¿Qué piensas tú que sucederá?

Ejecutémoslos ahora.

```shell
...
CashCardApplicationTests > shouldUpdateAnExistingCashCard() PASSED
...
BUILD SUCCESSFUL in 6s
```

¡Pasan!

Hemos implementado con éxito la actualización de una `CashCard`.

Ahora dirijamos nuestra atención a la seguridad. Específicamente, ¿qué sucede si intentamos actualizar una `CashCard` que no es de nuestra propiedad o que no existe?

Probemos esos escenarios a continuación.

# Additional Testing and Spring Security's Influence

Nos quedan dos escenarios para probar: 
- el caso en el que un usuario intenta actualizar una `CashCard` que no existe
- el caso en el que un usuario intenta actualizar una `CashCard` que no le pertenece.

## Case 1: Attempt to update a Cash Card which does not exist.

Empecemos por el primero de esos dos casos. Comience agregando una prueba.

### 1. Intente actualizar una `CashCard` que no existe.

Agregue la siguiente prueba a `CashCardApplicationTests`.

```java
@Test
void shouldNotUpdateACashCardThatDoesNotExist() {
    CashCard unknownCard = new CashCard(null, 19.99, null);
    HttpEntity<CashCard> request = new HttpEntity<>(unknownCard);
    ResponseEntity<Void> response = restTemplate
            .withBasicAuth("sarah1", "abc123")
            .exchange("/cashcards/99999", HttpMethod.PUT, request, Void.class);
    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);
}
```

Aquí intentaremos actualizar una `CashCard` con `ID 99999`, que no existe.
La prueba debería esperar un error genérico `404 NOT_FOUND`.

### 2. Ejecute las pruebas.

Antes de realizar la prueba, adivina cuál será el resultado.

Bien, ¿has adivinado? Ahora, ejecuta la prueba para verificar tu hipótesis.

```shell
...
CashCardApplicationTests > shouldNotUpdateACashCardThatDoesNotExist() FAILED
 org.opentest4j.AssertionFailedError:
 expected: 404 NOT_FOUND
  but was: 403 FORBIDDEN
```

Bueno, eso es...interesante. ¿Por qué obtuvimos un `403 FORBIDDEN`?
Antes de ejecutarlos nuevamente, editemos `build.gradle` para habilitar resultados de prueba adicionales.

```groovy
test {
 testLogging {
     ...
     // Change to `true` for more verbose test output
     showStandardStreams = true
 }
}
```

Después de volver a ejecutar las pruebas, busque en el resultado lo siguiente:

```shell
CashCardApplicationTests > shouldNotUpdateACashCardThatDoesNotExist() STANDARD_OUT
...
java.lang.NullPointerException: Cannot invoke "example.cashcard.CashCard.id()" because "cashCard" is null
```

¡Una excepción `NullPointerException`! ¿Por qué una excepción `NullPointerException`?

Al observar `CashCardController.putCashCard`, podemos ver que si no encontramos `cashCard`, las llamadas al método `cashCard` generarán una `NullPointerException`. Eso tiene sentido.

Pero, ¿por qué se lanza una excepción `NullPointerException` en nuestro controlador que genera un `403 FORBIDDEN` en lugar de un `500 INTERNAL_SERVER_ERROR`, dado que el servidor "falló"?

Recuerde: aprendimos en el módulo **Spring Security** que para evitar "filtrar" información sobre nuestra aplicación, **Spring Security** ha 
configurado **Spring Web** para devolver un `403 FORBIDDEN` genérico en la mayoría de las condiciones de error. ¡Gracias de nuevo, **Spring Security**!

Ahora que entendemos lo que está pasando, solucionémoslo.

### 3. No te estreses.
Aunque estamos agradecidos con **Spring Security**, nuestra aplicación no debería fallar; no deberíamos permitir que nuestro código arroje una `NullPointerException`. En su lugar, deberíamos manejar la condición cuando `cashCard == null` y devolver una respuesta `HTTP` genérica `404 NOT_FOUND`.

Actualice `CashCardController.putCashCard` para devolver `404 NOT_FOUND` si no se encuentra ninguna `CashCard` existente.

```java
@PutMapping("/{requestedId}")
private ResponseEntity<Void> putCashCard(@PathVariable Long requestedId, @RequestBody CashCard cashCardUpdate, Principal principal) {
    CashCard cashCard = cashCardRepository.findByIdAndOwner(requestedId, principal.getName());
    if (cashCard != null) {
        CashCard updatedCashCard = new CashCard(cashCard.id(), cashCardUpdate.amount(), principal.getName());
        cashCardRepository.save(updatedCashCard);
        return ResponseEntity.noContent().build();
    }
    return ResponseEntity.notFound().build();
}
```

### 4. Ejecute las pruebas.

Nuestras pruebas pasan ahora que devolvemos un `404` cuando no se encuentra ninguna `CashCard`.

```shell
[~/exercises] $ ./gradlew test
...
BUILD SUCCESSFUL in 6s
```

## Case 2: Attempt to update a Cash Card owned by someone else.

### 1. Escribe una prueba para el caso restante.

Escribamos una prueba que afirme que `sarah1` no puede actualizar una de las `CashCard` de `kumar2`.

```java
@Test
void shouldNotUpdateACashCardThatIsOwnedBySomeoneElse() {
    CashCard kumarsCard = new CashCard(null, 333.33, null);
    HttpEntity<CashCard> request = new HttpEntity<>(kumarsCard);
    ResponseEntity<Void> response = restTemplate
            .withBasicAuth("sarah1", "abc123")
            .exchange("/cashcards/102", HttpMethod.PUT, request, Void.class);
    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);
}
```

### 2. Formule una hipótesis sobre cuál será el resultado de la prueba y luego verifique la hipótesis ejecutando la prueba.

```shell
[~/exercises] $ ./gradlew test
...
CashCardApplicationTests > shouldNotUpdateACashCardThatIsOwnedBySomeoneElse() PASSED
```

¿Fue correcta su hipótesis?

La prueba pasó porque la modificación al `Controller` que hicimos en el paso anterior (es decir, verificar si hay un resultado nulo al recuperar la 
`CashCard` existente) también hace que esta prueba pase. Pero para estar seguros de este diagnóstico, hagamos un experimento.

### 3. Verifique el motivo del éxito de la prueba.

¡Adelante! Comente los cambios en el `Controller` y ejecute las pruebas nuevamente:

```java
@PutMapping("/{requestedId}")
private ResponseEntity<Void> putCashCard(@PathVariable Long requestedId, @RequestBody CashCard cashCardUpdate, Principal principal) {
    CashCard cashCard = cashCardRepository.findByIdAndOwner(requestedId, principal.getName());
    // if (null != cashCard) {
        CashCard updatedCashCard = new CashCard(cashCard.id(), cashCardUpdate.amount(), principal.getName());
        cashCardRepository.save(updatedCashCard);
        return ResponseEntity.noContent().build();
    // }
    // return ResponseEntity.notFound().build();
}
```

```shell
[~/exercises] $ ./gradlew test
...
CashCardApplicationTests > shouldNotUpdateACashCardThatIsOwnedBySomeoneElse() FAILED
org.opentest4j.AssertionFailedError:
expected: 404 NOT_FOUND
 but was: 403 FORBIDDEN
...
CashCardApplicationTests > shouldNotUpdateACashCardThatDoesNotExist() FAILED
org.opentest4j.AssertionFailedError:
expected: 404 NOT_FOUND
 but was: 403 FORBIDDEN
```

Volvemos al estado de "fallo": ambas pruebas dan como resultado `403 FORBIDDEN` debido a la excepción `NullPointerException` subyacente.

### 4. Deshaga sus cambios experimentales.

Recuerde descomentar las tres líneas en `CashCardController.putCashCard`

```java
@PutMapping("/{requestedId}")
private ResponseEntity<Void> putCashCard(@PathVariable Long requestedId, @RequestBody CashCard cashCardUpdate, Principal principal) {
    CashCard cashCard = cashCardRepository.findByIdAndOwner(requestedId, principal.getName());
    if (null != cashCard) {
        CashCard updatedCashCard = new CashCard(cashCard.id(), cashCardUpdate.amount(), principal.getName());
        cashCardRepository.save(updatedCashCard);
        return ResponseEntity.noContent().build();
    }
    return ResponseEntity.notFound().build();
}
```

#### Learning Moment: Controversy Exists

En este punto usted puede preguntarse: ¿Debería escribir dos pruebas diferentes que actúen exactamente igual con respecto a un único cambio en el código de la aplicación? 

Y si debería, **¿por qué?**

Una razón por la que es posible que desee conservar las dos pruebas es que en algún momento futuro, alguien podría cambiar la implementación del controlador de alguna manera que haga que las dos pruebas actúen de manera diferente.

Después de todo, estamos probando dos escenarios diferentes:
- Existe un registro `CashCard` con `id=99999` en la base de datos.
- No existe un registro `CashCard` `con id=99999` en la base de datos.

La verdad es que si mencionas esto como tema de conversación, es posible que obtengas respuestas diferentes de diferentes personas en diferentes circunstancias. Si bien puede ser interesante hablar de eso, como desarrolladores de la `API Family Cash Card`, debemos seguir adelante.

Reconociendo que las opiniones pueden diferir, tomaremos la decisión de dejar ambas pruebas en su lugar.

# Refactor the Controller Code

Reforcemos su uso del ciclo de desarrollo **Rojo, Verde y Refactor**.

Acabamos de completar varias pruebas enfocadas en actualizar una `CashCard` existente.

¿Tenemos alguna oportunidad de simplificar, reducir la duplicación o refactorizar nuestro código sin cambiar el comportamiento?

Continúe para abordar varias oportunidades de refactorización.

## Simplify the Code

### 1. Retire el opcional.

Esto podría ser controvertido: dado que no estamos aprovechando las características de `Opcional` en `CashCardController.findById`, y ningún otro 
método de `Controller` usa un `Opcional`, simplifiquemos nuestro código eliminándolo.

Edite `CashCardController.findById` para eliminar el uso de `Opcional`:

```java
// remove the unused Optional import if present
// import java.util.Optional;

@GetMapping("/{requestedId}")
private ResponseEntity<CashCard> findById(@PathVariable Long requestedId, Principal principal) {
    CashCard cashCard = cashCardRepository.findByIdAndOwner(requestedId, principal.getName());
    if (cashCard != null) {
        return ResponseEntity.ok(cashCard);
    } else {
        return ResponseEntity.notFound().build();
    }
}
```

### 2. Ejecute las pruebas.

Como no hemos cambiado ninguna funcionalidad, las pruebas seguirán superándose.

```shell
./gradlew test
...
BUILD SUCCESSFUL in 6s
```

## Reduce Duplication

Tenga en cuenta que tanto `CashCardController.findById` como `CashCardController.putCashCard` tienen un código casi idéntico que recupera una `CashCard` de destino del `CashCardRepository` utilizando información de `CashCard` y `Principal`.

Reduzcamos la duplicación de código extrayendo un método auxiliar llamado `findCashCard` y utilizándolo tanto en `.findById` como en `.putCashCard`.

Esto nos permitirá actualizar la forma en que recuperamos una `CashCard` en un solo lugar a medida que el `Controller` cambia con el tiempo.

### 1. Cree un método findCashCard compartido.

Cree un nuevo método en `CashCardController` llamado `findCashCard`, usando la funcionalidad que hemos escrito antes:

```java
private CashCard findCashCard(Long requestedId, Principal principal) {
    return cashCardRepository.findByIdAndOwner(requestedId, principal.getName());
}
```

### 2. Actualice `CashCardController.findById` y vuelva a ejecutar las pruebas.

A continuación, utilice el nuevo método `findCashCard` en `CashCardController.findById`.

```java
@GetMapping("/{requestedId}")
private ResponseEntity<CashCard> findById(@PathVariable Long requestedId, Principal principal) {
    CashCard cashCard = findCashCard(requestedId, principal);
    ...

```

Ninguna funcionalidad ha cambiado, por lo que ninguna prueba debería fallar cuando las volvamos a ejecutar.

```shell
./gradlew test
...
BUILD SUCCESSFUL in 6s
```

### 3. Actualice `CashCardController.putCashCard` y vuelva a ejecutar las pruebas.

De manera similar al paso anterior, utilice el nuevo método `findCashCard` en `CashCardController.putCashCard`.

```java
@PutMapping("/{requestedId}")
private ResponseEntity<Void> putCashCard(@PathVariable Long requestedId, @RequestBody CashCard cashCardUpdate, Principal principal) {
    CashCard cashCard = findCashCard(requestedId, principal);
    ...
```

Al igual que con las otras refactorizaciones que hemos realizado, ninguna funcionalidad ha cambiado, por lo que ninguna prueba debería fallar cuando las volvamos a ejecutar.

```shell
./gradlew test
...
BUILD SUCCESSFUL in 6s
```

¡Mira eso! Hemos refactorizado nuestro código simplificándolo y también hemos reducido la duplicación de código. ¡Excelente!

# Summary

En esta práctica de laboratorio, aprendió cómo implementar un punto final `HTTP PUT` que permite a un propietario autorizado y autenticado actualizar su `CashCard`.

También aprendió cómo **Spring Security** administra automáticamente el manejo de errores por parte de **Spring Web** para garantizar que la información sensible a la seguridad no se revele accidentalmente cuando un controlador encuentra un error.

Además, reforzó su comprensión y utilización del ciclo de desarrollo Red, Green, Refactor al refactorizar nuestro código de controlador sin cambiar su funcionalidad.