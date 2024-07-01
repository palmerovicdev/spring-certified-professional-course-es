# Overview

En la lección asociada, aprendió a implementar la operación `DELETE` en una `API REST`. En esta práctica de laboratorio, implementaremos la eliminación 
definitiva en nuestra `API Cash Card`, utilizando las especificaciones de `API` que definimos en la lección asociada.

## Credentials for test user Kumar

Si ha realizado prácticas de laboratorio anteriores en este curso, notará los siguientes cambios en el código base, que hemos realizado en su nombre, para que esta práctica de laboratorio sea más fácil de entender y completar.

Hemos agregado credenciales para el usuario `kumar2` al bean `testOnlyUsers` en `src/main/java/example/cashcard/SecurityConfig.java`.

¡Implementemos el endpoint `DELETE`!

# Test the Happy Path

Comencemos con el camino feliz más simple: eliminar con éxito una `CashCard` que existe.

Necesitamos el endpoint `DELETE` para devolver el código de estado `204 NO CONTENT`.

### 1. Escribe la prueba.

Agregue el siguiente método de prueba a `src/test/java/example/cashcard/CashCardApplicationTests.java`:

```java
@Test
@DirtiesContext
void shouldDeleteAnExistingCashCard() {
    ResponseEntity<Void> response = restTemplate
            .withBasicAuth("sarah1", "abc123")
            .exchange("/cashcards/99", HttpMethod.DELETE, null, Void.class);
    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.NO_CONTENT);
}
```

Observe que hemos agregado la anotación `@DirtiesContext`. Agregaremos esta anotación a todas las pruebas que cambien los datos. Si no lo hacemos, estas pruebas podrían afectar el resultado de otras pruebas en el archivo.

#### Why not use RestTemplate.delete()?

Observe que estamos usando `RestTemplate.exchange()` aunque `RestTemplate` proporciona un método que parece que podríamos usar: `RestTemplate.delete()`. Sin embargo, veamos la firma:

```java
public class RestTemplate ... {
    public void delete(String url, Object... uriVariables)
```

Los otros métodos que hemos estado usando (como `getForEntity()` e `exchange()`) devuelven una `ResponseEntity`, pero `delete()` no. Más bien, es un 
método nulo. ¿Por qué es esto?

El framework **Spring Web** proporciona el método `delete()` por conveniencia, pero viene con algunas suposiciones:
- Una respuesta a una solicitud `DELETE` no tendrá cuerpo.
- Al cliente no debería importarle cuál sea el código de respuesta a menos que sea un error, en cuyo caso, generará una excepción.

Dadas esas suposiciones, no se necesita ningún valor de retorno de `delete()`.

Pero la segunda suposición hace que `delete()` no sea adecuado para nosotros: ¡necesitamos `ResponseEntity` para poder hacer valer el código de estado! 

Por lo tanto, no usaremos el método de conveniencia, sino más bien el método más general: `exchange()`.

### 2. Ejecute las pruebas.

Como siempre, usaremos `./gradlew test` para ejecutar las pruebas.

```shell
[~/exercises] $ ./gradlew test
...
CashCardApplicationTests > shouldDeleteAnExistingCashCard() FAILED
    org.opentest4j.AssertionFailedError:
    expected: 204 NO_CONTENT
    but was: 403 FORBIDDEN
```

La prueba falló porque la solicitud `DELETE /cashcards/99` devolvió un `403 FORBIDDEN`.

En este punto probablemente esperaba este resultado: **Spring Security** devuelve una respuesta `403` para cualquier endpoint que no esté asignado.

Necesitamos implementar el método Controlador. ¡Hagamoslo!

### 3. Implemente el endpoint `DELETE` en el `Controller`.

Agregue el siguiente método a la clase `CashCardController`:

```java
@DeleteMapping("/{id}")
private ResponseEntity<Void> deleteCashCard(@PathVariable Long id) {
    return ResponseEntity.noContent().build();
}
```

¿Qué pasará si ejecutamos las pruebas?

### 4. Ejecute las pruebas.

```shell
[~/exercises] $ ./gradlew test
...
BUILD SUCCESSFUL in 8s
```

¡Pasan!

¿Eso significa que hemos terminado? ¡Aún no!

No hemos escrito el código para eliminar el elemento. Hagamos eso a continuación.

Por supuesto, primero escribiremos la prueba.

### 5. Pruebe que en realidad estamos eliminando la `CashCard`.

Agregue las siguientes afirmaciones a la prueba `shouldDeleteAnExistingCashCard()`:

```java
@Test
@DirtiesContext
void shouldDeleteAnExistingCashCard() {
    ResponseEntity<Void> response = restTemplate
            .withBasicAuth("sarah1", "abc123")
            .exchange("/cashcards/99", HttpMethod.DELETE, null, Void.class);
    assertThat(response.getStatusCode()).isEqualTo(HttpStatus.NO_CONTENT);

    // Add the following code:
    ResponseEntity<String> getResponse = restTemplate
            .withBasicAuth("sarah1", "abc123")
            .getForEntity("/cashcards/99", String.class);
    assertThat(getResponse.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);
}
```

### 6. Comprenda el código de prueba.

Queremos probar que la `CashCard` eliminada realmente se elimina, por lo que intentamos hacer `GET` y afirmamos que el código de resultado es `404 
NOT FOUND`.

Ejecute la prueba. ¿Pasa? ¡Por supuesto que no!

```shell
CashCardApplicationTests > shouldDeleteAnExistingCashCard() FAILED
    org.opentest4j.AssertionFailedError:
    expected: 404 NOT_FOUND
    but was: 200 OK
```
¿Qué debemos hacer para pasar la prueba? ¡Escribe un código para eliminar el registro!

Vamos.

# Implement the DELETE Endpoint

Ahora necesitamos escribir un método de controlador que se llamará cuando enviemos una solicitud `DELETE` con el `URI` adecuado.

Agregue código al `Controller` para eliminar el registro.

Cambie el método `CashCardController.deleteCashCard()`:

```java
@DeleteMapping("/{id}")
private ResponseEntity<Void> deleteCashCard(@PathVariable Long id) {
    cashCardRepository.deleteById(id); // Add this line
    return ResponseEntity.noContent().build();
}
```

El cambio es sencillo:
- Usamos `@DeleteMapping` con el parámetro "`{id}`", que **Spring Web** compara con el parámetro del método id.
- `CashCardRepository` ya tiene el método que necesitamos: `deleteById()` (se hereda de `CrudRepository`).

¡Realiza las pruebas y observa cómo pasan!

```shell
[~/exercises] $ ./gradlew test
...
BUILD SUCCESSFUL in 8s
```

Genial, ¿qué hacer a continuación?

# Test case: The Cash Card doesn't exist

Nuestro contrato establece que debemos devolver `404 NOT FOUND` si intentamos eliminar una `CashCard` que no existe.

### 1. Escribe la prueba.

Agregue el siguiente método de prueba a `CashCardApplicationTests`:

```java
@Test
void shouldNotDeleteACashCardThatDoesNotExist() {
    ResponseEntity<Void> deleteResponse = restTemplate
            .withBasicAuth("sarah1", "abc123")
            .exchange("/cashcards/99999", HttpMethod.DELETE, null, Void.class);
    assertThat(deleteResponse.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);
}
```

### 2. Ejecute las pruebas.

```shell
CashCardApplicationTests > shouldNotDeleteACashCardThatDoesNotExist() FAILED
    org.opentest4j.AssertionFailedError:
    expected: 404 NOT_FOUND
    but was: 204 NO_CONTENT
```

¡No hay sorpresa aquí!

Necesitamos reforzar la seguridad de nuestra aplicación verificando que el usuario que intenta eliminar una `CashCard` sea el propietario.

**Spring Security** hace muchas cosas por nosotros, pero esta no es una de ellas.

# Enforce Ownership

Necesitamos verificar si el registro existe. Si no, no debemos eliminar la `CashCard` y devolver `404 NOT FOUND`.

### 1. Realice los siguientes cambios en `CashCardController.deleteCashCard`:

```java
private ResponseEntity<Void> deleteCashCard(
        @PathVariable Long id,
        Principal principal // Add Principal to the parameter list
    ) {
    // Add the following 3 lines:
    if (!cashCardRepository.existsByIdAndOwner(id, principal.getName())) {
        return ResponseEntity.notFound().build();
    }
...
}
```

¡Asegurémonos de agregar el parámetro Principal!

Estamos utilizando el Principal para verificar si la `CashCard` existe y, al mismo tiempo, hacer cumplir la propiedad.

### 2. Agregue el método `existeByIdAndOwner()` al `Repository`.

Además, agreguemos el nuevo método `existeByIdAndOwner()` a `CashCardRepository`:

```java
interface CashCardRepository extends CrudRepository<CashCard, Long>, PagingAndSortingRepository<CashCard, Long> {
    ...
    boolean existsByIdAndOwner(Long id, String owner);
    ...
}
```

### 3. Comprenda el código del repositorio.
Agregamos lógica al método del `Controller` para verificar si la `ID` de la `CashCard` en la solicitud realmente existe en la base de datos. El 
método que usaremos es `CashCardRepository.existsByIdAndOwner(id, userName)`.

Este es otro caso en el que **Spring Data** generará la implementación de este método siempre que lo agreguemos al `Repository`.

Entonces, ¿por qué no utilizar el método `findByIdAndOwner()` y comprobar si devuelve nulo? ¡Absolutamente podríamos hacer eso! Sin embargo, dicha llamada devolvería información adicional (el contenido de la `CashCard` recuperada), por lo que nos gustaría evitarla para no introducir complejidad adicional.

Si prefieres no utilizar el método `existeByIdAndOwner()`, ¡está bien! Puede optar por utilizar `findByIdAndOwner()`. ¡El resultado de la prueba será el mismo!

### 4. Mira el pase de la prueba.

Hagamos la prueba y, como era de esperar, ¡la prueba pasa!

```shell
[~/exercises] $ ./gradlew test
...
BUILD SUCCESSFUL in 8s
```

# Refactor

En este punto, tenemos la oportunidad de practicar el proceso de Red, Green, Refactor. Ya hicimos el Red (la prueba fallida) y el Green (la prueba 
aprobada). 

Ahora podemos preguntarnos, ¿deberíamos Refactorizar algo?

Aquí está el cuerpo de nuestro método `CashCardController.deleteCashCard`:

```java
...
if (!cashCardRepository.existsByIdAndOwner(id, principal.getName())) {
    return ResponseEntity.notFound().build();
}
cashCardRepository.deleteById(id);
return ResponseEntity.noContent().build();
```

Es posible que la siguiente versión, que es lógicamente equivalente pero un poco más simple, le resulte más fácil de leer:

```java
if (cashCardRepository.existsByIdAndOwner(id, principal.getName())) {
    cashCardRepository.deleteById(id);
    return ResponseEntity.noContent().build();
}
return ResponseEntity.notFound().build();
```

Las diferencias son leves, pero eliminar un `not-operator` (!) de una declaración if a menudo hace que el código sea más fácil de leer, ¡y la legibilidad es importante!

Si encuentra que la segunda versión es más fácil de leer y comprender, reemplace el código existente con la nueva versión.

Ejecute las pruebas nuevamente. ¡Todavía pasan!

```shell
[~/exercises] $ ./gradlew test
...
BUILD SUCCESSFUL in 8s
```

# Hide Unauthorized Records

En este punto, es posible que se pregunte: "¿Hemos terminado?". ¡Eres la mejor persona para responder esa pregunta! Si lo desea, tómese un par de minutos para refrescarse con la lección adjunta para ver si hemos probado e implementado todos los aspectos del contrato `API` para `DELETE`.

Vale, fue un tiempo bien empleado, ¿no? Así es, hay un caso más que aún tenemos que probar: ¿Qué pasa si el usuario intenta eliminar una 
`CashCard` propiedad de otra persona? Decidimos en la lección asociada que la respuesta debería ser `404 NOT FOUND` en este caso. Esa es suficiente 
información para que podamos escribir una prueba para ese caso:

### 1. En `CashCardApplicationTests.java`, agregue el siguiente método de prueba al final de la clase:

```java
@Test
void shouldNotAllowDeletionOfCashCardsTheyDoNotOwn() {
    ResponseEntity<Void> deleteResponse = restTemplate
            .withBasicAuth("sarah1", "abc123")
            .exchange("/cashcards/102", HttpMethod.DELETE, null, Void.class);
    assertThat(deleteResponse.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);
}
```

### 2. Ejecute la prueba.

¿Crees que pasará la prueba? Tómese un momento para predecir el resultado y luego ejecute la prueba.

```shell
[~/exercises] $ ./gradlew test
...
BUILD SUCCESSFUL in 8s
```

¡Todos pasaron! Esas son buenas noticias.

Hemos escrito una prueba para un caso específico en nuestra `API`.

¡La prueba pasó sin ningún cambio de código!

Ahora, consideremos una pregunta que se le puede haber ocurrido: ¿Por qué necesito esa prueba, ya que pasa sin tener que realizar ningún cambio en el código? ¿No es el propósito de `TDD` utilizar pruebas para guiar la implementación de la aplicación? Si ese es el caso, ¿por qué me molesté en escribir esa prueba?

Respuestas:
- Sí, ese es uno de los muchos beneficios que ofrece `TDD`: un proceso para guiar la creación de código para llegar al resultado deseado.
- Sin embargo, las pruebas en sí mismas tienen otro propósito, aparte del `TDD`: las pruebas son una poderosa red de seguridad para hacer cumplir la 
  corrección. Dado que la prueba que acaba de escribir prueba un caso diferente a los ya escritos, proporciona valor. Si alguien hiciera un cambio en el código que provocara que esta nueva prueba fallara, entonces habrá detectado el error antes de que pueda convertirse en un problema. Bien por las pruebas.

### 3. Una prueba más.

¡Pero espera, dices!

¿No deberíamos comprobar que el registro que intentamos eliminar todavía existe en la base de datos y que no se eliminó?

Sí, esa es una prueba válida. ¡Gracias por mencionarlo!

Agregue el siguiente código al método de prueba para verificar que el registro que intentó eliminar sin éxito aún esté allí:

```shell
void shouldNotAllowDeletionOfCashCardsTheyDoNotOwn() {
 ...
    // Add this code at the end of the test method:
    ResponseEntity<String> getResponse = restTemplate
            .withBasicAuth("kumar2", "xyz789")
            .getForEntity("/cashcards/102", String.class);
    assertThat(getResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
}
```

¿Crees que pasará la prueba? ¡Por supuesto que sí! (¿bien?)

### 4. Ejecute las pruebas.

¡Una última prueba!

Ejecute todas las pruebas y asegúrese de que pasen.

```shell
[~/exercises] $ ./gradlew test
...
BUILD SUCCESSFUL in 6s
```

¡Somos los mejores!

# Summary

En esta práctica de laboratorio, implementó un endpoint `RESTful` `DELETE` que no "filtra" información de seguridad a posibles atacantes. También 
tuvo la oportunidad de hacer una refactorización pequeña pero útil para practicar el proceso de Red, Green, Refactor.
Esta es la última de las operaciones CRUD que se implementará en la API, lo que nos lleva a una conclusión exitosa de nuestro trabajo técnico. ¡Felicidades!