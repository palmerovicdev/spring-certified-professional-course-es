# Changes from the Previous Lab

Hemos hecho los siguientes cambios en el laboratorio anterior.

- Se ha añadido un par de accesorios de datos de `CashCard` a `src/test/resources/data.sql`
```sql
INSERT INTO CASH_CARD(ID, AMOUNT) VALUES (99, 123.45);

INSERT INTO CASH_CARD(ID, AMOUNT) VALUES (100, 1.00);

INSERT INTO CASH_CARD(ID, AMOUNT) VALUES (101, 150.00);
```

- Refactorizado `CashCardJsonTest.java` para incorporar los nuevos ajustes de datos.
```java
package example.cashcard;


import org.assertj.core.util.Arrays;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.json.JsonTest;
import org.springframework.boot.test.json.JacksonTester;

import java.io.IOException;

import static org.assertj.core.api.Assertions.assertThat;

@JsonTest
class CashCardJsonTest {

    @Autowired
    private JacksonTester<CashCard> json;

    @Autowired
    private JacksonTester<CashCard[]> jsonList;

    private CashCard[] cashCards;

    @BeforeEach
    void setUp() {
        cashCards = Arrays.array(
                new CashCard(99L, 123.45),
                new CashCard(100L, 100.00),
                new CashCard(101L, 150.00));
    }

    @Test
    void cashCardSerializationTest() throws IOException {
        CashCard cashCard = cashCards[0];
        assertThat(json.write(cashCard)).isStrictlyEqualToJson("single.json");
        assertThat(json.write(cashCard)).hasJsonPathNumberValue("@.id");
        assertThat(json.write(cashCard)).extractingJsonPathNumberValue("@.id")
                .isEqualTo(99);
        assertThat(json.write(cashCard)).hasJsonPathNumberValue("@.amount");
        assertThat(json.write(cashCard)).extractingJsonPathNumberValue("@.amount")
                .isEqualTo(123.45);
    }

    @Test
    void cashCardDeserializationTest() throws IOException {
        String expected = """
                {
                    "id": 99,
                    "amount": 123.45
                }
                """;
        assertThat(json.parse(expected))
                .isEqualTo(new CashCard(99L, 123.45));
        assertThat(json.parseObject(expected).id()).isEqualTo(99);
        assertThat(json.parseObject(expected).amount()).isEqualTo(123.45);
    }
}
```

- Se cambió el nombre de `expected.json` a `single.json` y se agregó otro archivo `JSON` de contrato de datos: `list.json`.

```json
[

{"id": 99, "amount": 123.45 },

{"id": 100, "amount": 1.00 },

{"id": 101, "amount": 150.00 }

]
```

- Se han añadido algunas importaciones a las clases de prueba, ¡para que no tengas que hacerlo!
```java
package example.cashcard;

import com.jayway.jsonpath.DocumentContext;
import com.jayway.jsonpath.JsonPath;
import net.minidev.json.JSONArray;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.test.annotation.DirtiesContext;

import java.net.URI;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.test.annotation.DirtiesContext.*;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@DirtiesContext(classMode = ClassMode.AFTER_EACH_TEST_METHOD)
class CashCardApplicationTests {
    @Autowired
    TestRestTemplate restTemplate;

    @Test
    void shouldReturnACashCardWhenDataIsSaved() {
        ResponseEntity<String> response = restTemplate.getForEntity("/cashcards/99", String.class);
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);

        DocumentContext documentContext = JsonPath.parse(response.getBody());
        Number id = documentContext.read("$.id");
        assertThat(id).isEqualTo(99);

        Double amount = documentContext.read("$.amount");
        assertThat(amount).isEqualTo(123.45);
    }

    @Test
    void shouldNotReturnACashCardWithAnUnknownId() {
        ResponseEntity<String> response = restTemplate.getForEntity("/cashcards/1000", String.class);

        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);
        assertThat(response.getBody()).isBlank();
    }

    @Test
    //@DirtiesContext
    void shouldCreateANewCashCard() {
        CashCard newCashCard = new CashCard(null, 250.00);
        ResponseEntity<Void> createResponse = restTemplate.postForEntity("/cashcards", newCashCard, Void.class);
        assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);

        URI locationOfNewCashCard = createResponse.getHeaders().getLocation();
        ResponseEntity<String> getResponse = restTemplate.getForEntity(locationOfNewCashCard, String.class);
        assertThat(getResponse.getStatusCode()).isEqualTo(HttpStatus.OK);

        DocumentContext documentContext = JsonPath.parse(getResponse.getBody());
        Number id = documentContext.read("$.id");
        Double amount = documentContext.read("$.amount");

        assertThat(id).isNotNull();
        assertThat(amount).isEqualTo(250.00);
    }
}
```

- Se ha añadido la anotación `@DirtiesContext` a la clase `CashCardApplicationTests`.

Esta lista es solo el resumen. Ampliaremos cada punto a lo largo de las instrucciones del laboratorio.

# Testing the New Data Contract

Como hemos hecho en talleres anteriores, comenzaremos escribiendo una prueba de cómo esperamos que sea el éxito.

Ya que estamos introduciendo un nuevo contrato de datos, ¡Empenzaremos por probarlo!

### 1. Mira los nuevos accesorios de datos.

Mira el archivo `src/test/resources/example/cashcard/list.json`. Contiene la siguiente matriz **JSON**:

```json
[
  { "id": 99, "amount": 123.45 },
  { "id": 100, "amount": 1.0 },
  { "id": 101, "amount": 150.0 }
]
```

Este es nuestro nuevo contrato de datos que contiene una lista de `CashCard`, que coinciden con los datos del nuevo archivo `data.sql`; adelante, mira el archivo `src/test/resources/data.sql` para verificar que los valores del archivo **JSON** coincidan.

Ahora abre el archivo `CashCardJsonTest.java`. Tenga en cuenta que la variable de nivel de clase `cashCards` está configurada para contener la siguiente matriz **Java**:

```java
cashCards = Arrays.array(
     new CashCard(99L, 123.45),
     new CashCard(100L, 100.00),
     new CashCard(101L, 150.00));
```

Si miras de cerca, verás que uno de los objetos de `CashCard` en nuestra prueba no coincide con los datos de la prueba en `data.sql`. ¡Esto es para prepararnos para escribir una prueba fallida!

### 2. Añade una prueba de serialización para la lista de `CashCard`.

Añade una nueva prueba a `CashCardJsonTest.java`:

```java
@Test
void cashCardListSerializationTest() throws IOException {
   assertThat(jsonList.write(cashCards)).isStrictlyEqualToJson("list.json");
}
```

El código de prueba se explica por sí mismo: serializa la variable `cashCards` en **JSON**, luego afirma que `list.json` debe contener los mismos datos que la variable serializada `cashCards`.

### 3. Ejecuta las pruebas.

¿Puedes predecir si la prueba fallará y, si lo hace, cuál será la causa del fallo? ¡Adelante, haz la llamada! ¿Qué crees que pasará?

Verifica tu predicción ejecutando las pruebas.

Tenga en cuenta que siempre ejecutaremos `./gradlew test` para ejecutar las pruebas.

```
[~/exercises] $ ./gradlew test
...
> Task :test FAILED
...
java.lang.AssertionError: JSON Comparison failure: [1].amount
Expected: 1.0
     got: 100.0
```

¡Tu predicción fue correcta (con suerte)! La prueba falló. Afortunadamente, el mensaje de error señala el lugar exacto donde se produce el fallo: el campo de cantidad de la segunda `CashCard` en la matriz (índice [1]) no es lo que se esperaba.

### 4. Arreglar y volver a ejecutar las pruebas.

Cambie `cashCards[1].amount` para esperar la cantidad correcta en la `CashCard`.

Para verificar la cantidad que debe esperar la prueba, vuelva a mirar el archivo `list.json`. Verás que la cantidad esperada debería ser `1,00` en lugar de `100,0`. Así que cambiemos el código de prueba:

```java
...
new CashCard(100L, 1.00),
...
```

Después de corregir el código de prueba para esperar `1.0` en lugar de `100.0` para el valor del campo de cantidad, vuelva a ejecutar las pruebas. Pasarán:

```java
    [~/exercises] $ ./gradlew test
    ...
    BUILD SUCCESSFUL in 7s
```

### 5. Añade una prueba de deserialización.

Ahora vamos a probar la deserialización. Añade la siguiente prueba:

```java
@Test
void cashCardListDeserializationTest() throws IOException {
   String expected="""
         [
            { "id": 99, "amount": 123.45 },
            { "id": 100, "amount": 100.00 },
            { "id": 101, "amount": 150.00 }
         ]
         """;
   assertThat(jsonList.parse(expected)).isEqualTo(cashCards);
}
```

Una vez más, hemos afirmado intencionalmente un valor incorrecto para que sea obvio lo que la prueba está probando.

### 6. Ejecuta las pruebas.

Cuando ejecutes las pruebas, verás que se ha capturado el valor incorrecto.

```shell
[~/exercises] $ ./gradlew test
...
> Task :test FAILED
...
 expected:
   [CashCard[id=99, amount=123.45],
       CashCard[id=100, amount=1.0],
       CashCard[id=101, amount=150.0]]
  but was:
   [CashCard[id=99, amount=123.45],
       CashCard[id=100, amount=100.0],
       CashCard[id=101, amount=150.0]]
```

Esta vez, la prueba falló porque deserializamos la cadena **JSON** esperada y la comparamos con la variable `cashCards`. Una vez más, esa molesta `CashCard` de `100,00 $` no coincide con las expectativas.

Cambia la expectativa:

```java
String expected="""
    [
        { "id": 99, "amount": 123.45 },
        { "id": 100, "amount": 1.00 },
        { "id": 101, "amount": 150.00 }
    ]
    """;
```

A continuación, vuelva a ejecutar las pruebas y vea cómo se pasa la prueba:

```shell
[~/exercises] $ ./gradlew test
...
CashCardJsonTest > cashCardListDeserializationTest() PASSED
```

Ahora que hemos probado el contrato de datos, pasemos al endpoint del controlador.

# Test for an Additional GET Endpoint

### 1. Escriba una prueba fallida para un nuevo endpoint **GET**.

Agreguemos un nuevo método de prueba que espera un endpoint **GET** que devuelva varios objetos de `CashCard`.

En `CashCardApplicationTests.java`, añade una nueva prueba:

```java
 @Test
 void shouldReturnAllCashCardsWhenListIsRequested() {
     ResponseEntity<String> response = restTemplate.getForEntity("/cashcards", String.class);
     assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
 }
```

Aquí estamos haciendo una solicitud al endpoint `/cashcards`. Dado que estamos recibiendo la lista completa de tarjetas, no necesitamos especificar ninguna información adicional en la solicitud.

### 2. Ejecuta las pruebas y observa el fallo.

La prueba debería fallar porque no hemos implementado un endpoint del controlador para manejar esta solicitud **GET**.

¿Cómo crees que va a fracasar? ¿Tal vez un `404 NOT FOUND`?

En una lección anterior escribimos una prueba que falló porque aún no existía un endpoint que coincidiera con la ruta solicitada. El resultado fue un error `404 NOT FOUND`. Podríamos esperar que suceda lo mismo cuando ejecutemos la nueva prueba, ya que no hemos añadido ningún código al controlador.

A ver qué pasa. Ejecute la prueba y busque el siguiente fallo:

```shell
expected: 200 OK
 but was: 405 METHOD_NOT_ALLOWED
```

Los mensajes de error no dejan claro por qué estamos recibiendo un error `405 METHOD_NOT_ALLOWED`. La razón es un poco difícil de descubrir, así que la resumiremos rápidamente: ya hemos implementado un endpoint `/cashcards`, pero no para un verbo **GET**.

Este es el proceso de **Spring**:

- **Spring** recibe una solicitud al endpoint `/cashcards`.

- No hay una asignación para el verbo `HTTP GET` en ese endpoint.

- Sin embargo, hay una asignación a ese endpoint para el verbo `HTTP POST`. ¡Es el endpoint para la operación de creación que implementamos en una lección anterior!

- Por lo tanto, **Spring** informa de un error de `405 METHOD_NOT_ALLOWED` en lugar de `404 NOT FOUND` - la ruta se encontró de hecho, pero no es compatible con el verbo **GET**.

### 3. Implemente el endpoint **GET** en el controlador.

Para superar el error `405`, necesitamos implementar el endpoint `/cashcards` en el controlador usando una anotación `@GetMapping`:

```java
    @GetMapping()
    private ResponseEntity<Iterable<CashCard>> findAll() {
       return ResponseEntity.ok(cashCardRepository.findAll());
    }
```

### 4. Comprende el método del controlador.

Una vez más, estamos utilizando una de las implementaciones integradas de **Spring Data**: `CrudRepository.findAll()`. Nuestro repositorio de implementación, `CashCardRepository`, devolverá automáticamente todos los registros de `CashCard` de la base de datos cuando se invoque `findAll()`.

### 5. Vuelve a ejecutar las pruebas.

Cuando volvemos a ejecutar las pruebas, vemos que todas pasan, incluida la prueba para el endpoint **GET** para una lista de `CashCard`.

```java
[~/exercises] $ ./gradlew test 
... 
BUILD SUCCESSFUL in 7s
```

# Enhance the List Test

Como hemos hecho en lecciones anteriores, hemos probado que nuestro controlador de **API** de `CashCard` está "escuchando" nuestras llamadas **HTTP** y no se bloquea cuando se invoca, esta vez para un **GET** sin más parámetros.

Mejoremos nuestras pruebas y asegurémonos de que se devuelvan los datos correctos de nuestra solicitud **HTTP**.

### 1. Mejora la prueba.

En primer lugar, completemos la prueba para afirmar los valores de datos esperados:

```java
     @Test
     void shouldReturnAllCashCardsWhenListIsRequested() {
         ResponseEntity<String> response = restTemplate.getForEntity("/cashcards", String.class);
         assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
    
         DocumentContext documentContext = JsonPath.parse(response.getBody());
         int cashCardCount = documentContext.read("$.content.length()");
         assertThat(cashCardCount).isEqualTo(3);
    
         JSONArray ids = documentContext.read("$..id");
         assertThat(ids).containsExactlyInAnyOrder(99, 100, 101);
    
         JSONArray amounts = documentContext.read("$..amount");
         assertThat(amounts).containsExactlyInAnyOrder(123.45, 100.0, 150.00);
     }
```

### 2. Entender la prueba.

```java
documentContext.read("$.length()");
...
documentContext.read("$..id");
...
documentContext.read("$..amount");
```

¡Mira estas nuevas expresiones de `JsonPath`!

`documentContext.read("$.length()")` calcula la longitud de la matriz.

`.read("$..id")` recupera la lista de todos los valores de identificación devueltos, mientras que `.read("$..amount")` recopila todas las cantidades devueltas.

Para obtener más información sobre `JsonPath`, un buen lugar para comenzar está aquí en la [documentación de JsonPath](https://github.com/json-path/JsonPath).

```java
assertThat(...).containsExactlyInAnyOrder(...)
```

No hemos garantizado el orden de la lista de `CashCard`: salen en cualquier orden que la base de datos elija para devolverlas. Dado que no especificamos el orden, `containsExactlyInAnyOrder(...)` afirma que, si bien la lista debe contener todo lo que afirmamos, el orden no importa.

### 3. Ejecuta las pruebas.

¿Cuál crees que será el resultado de la prueba?

```shell
 Expecting actual:
   [123.45, 1.0, 150.0]
 to contain exactly in any order:
   [123.45, 100.0, 150.0]
 elements not found:
   [100.0]
 and elements not expected:
   [1.0]
```

El mensaje de fallo señala exactamente la causa del fallo. Hemos escrito a hurtadillas una prueba que espera que la segunda `CashCard` tenga una cantidad de `100,00 $`, mientras que en `src/test/resources/data.sql` el valor real es de `1,00 $`.

### 4. Corrija las pruebas y vuelva a ejecutarlas.

Cambie la expectativa de la `CashCard` de `1 $`:

```java
assertThat(amounts).containsExactlyInAnyOrder(123.45, 1.00, 150.00);
```

¡Y mira como pasa la prueba!

```java
[~/exercises] $ ./gradlew test 
... 
CashCardApplicationTests > shouldReturnAllCashCardsWhenListIsRequested() PASSED 
... 
BUILD SUCCESSFUL in 6s
```

# Test Interaction and @DirtiesContext

Tomemos un momento para hablar sobre la anotación `@DirtiesContext`.

Verá dos usos de esta anotación en la clase `CashCardApplicationTests`: uno en la definición de la clase, uno en la definición del método de prueba `shouldCreateANewCashCard`, que está comentada por ahora.

Expliquemos.

En primer lugar, comente la anotación a nivel de clase:

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
//@DirtiesContext(classMode = ClassMode.AFTER_EACH_TEST_METHOD)
class CashCardApplicationTests {
```

Ejecuta todas las pruebas:

```shell
[~/exercises] $ ./gradlew test
...
 org.opentest4j.AssertionFailedError:
 expected: 3
  but was: 4
     ...
     at app//example.cashcard.CashCardApplicationTests.shouldReturnAllCashCardsWhenListIsRequested(CashCardApplicationTests.java:70)
```

¡Nuestra nueva prueba `shouldReturnAllCashCardsWhenListIsRequested` no pasó esta vez! ¿Por qué?

La razón es que una de las otras pruebas está interfiriendo con nuestra nueva prueba al crear una nueva `CashCard`. `@DirtiesContext` soluciona este problema haciendo que **Spring** comience con un borrón y cuenta nueva, como si esas otras pruebas no se hubieran ejecutado. Eliminarlo (comentándolo) de la clase hizo que nuestra nueva prueba fallara.

#### Learning Moment

Aunque puedes usar `@DirtiesContext` para evitar la interacción entre pruebas, no deberías usarlo indiscriminadamente; deberías tener una buena razón. Nuestra razón aquí es limpiar después de crear una nueva `CashCard`.

Deja `@DirtiesContext` comentado a nivel de clase, y descomentalo en el método que crea una nueva `CashCard`:

```java
//@DirtiesContext(classMode = ClassMode.AFTER_EACH_TEST_METHOD)
class CashCardApplicationTests {
   ...

    @Test
    @DirtiesContext
    void shouldCreateANewCashCard() {
   ...
```

¡Haz las pruebas y pasan!

# Pagination

¡Ahora implementemos la paginación, comenzando con una prueba!

Tenemos 3 `CashCard` en nuestra base de datos. Hagamos una prueba para buscarlos de uno en uno (tamaño de página de 1), y luego ordenar sus cantidades de más alta a menor (descendente).

### 1. Escribe la prueba de paginación.

Agregue la siguiente prueba a `CashCardApplicationTests`, y tenga en cuenta que estamos agregando parámetros a la solicitud **HTTP** de `?page=0&size=1`. Nos encargaremos de esto en nuestro **Controller** más tarde.

```java
    @Test
    void shouldReturnAPageOfCashCards() {
        ResponseEntity<String> response = restTemplate.getForEntity("/cashcards?page=0&size=1", String.class);
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
    
        DocumentContext documentContext = JsonPath.parse(response.getBody());
        JSONArray page = documentContext.read("$.length()");
        assertThat(page.size()).isEqualTo(1);
    }
```

### 2. Ejecuta las pruebas.

Cuando ejecutamos las pruebas, no deberíamos sorprendernos de que se devuelvan todas las `CashCard`.

```shell
    expected: 1
    but was: 3
```

### 3. Implementar la paginación en el `CashCardController`.

¡Así que vamos a añadir nuestro nuevo endpoint al controlador! Agregue el siguiente método al `CashCardController` (no elimine el método `findAll()` existente):

```java
    @GetMapping
    private ResponseEntity<List<CashCard>> findAll(Pageable pageable) {
        Page<CashCard> page = cashCardRepository.findAll(
                PageRequest.of(
                        pageable.getPageNumber(),
                        pageable.getPageSize()
        ));
        return ResponseEntity.ok(page.getContent());
    }
```

### 4. Comprende el código de paginación.

```java
findAll(Pageable pageable)
```

`Pageable` es otro objeto que **Spring Web** nos proporciona. Dado que especificamos los parámetros **URI** de `page=0&size=1`, `pageable` contendrá los valores que necesitamos.

```java
    PageRequest.of(
      pageable.getPageNumber(),
      pageable.getPageSize()
    ));
```

`PageRequest` es una implementación básica de **Java Bean** de `Pageable`. Las cosas que quieren la implementación de la paginación y la clasificación a menudo admiten esto, como algunos tipos de repositorios de datos de **Spring**.

¿Nuestra `CashCardRepository` ya es compatible con la paginación y la clasificación? Vamos a averiguarlo.

### 5. Intenta compilar.

¡Cuando ejecutamos las pruebas descubrimos que nuestro código ni siquiera se compila!

```shell
[~/exercises] $ ./gradlew test
...
> Task :compileJava FAILED
exercises/src/main/java/example/cashcard/CashCardController.java:50: error: method findAll in interface CrudRepository<T,ID> cannot be applied to given types;
        Page<CashCard> page = cashCardRepository.findAll(
                                                ^
  required: no arguments
  found:    PageRequest
```

¡Pero por supuesto! No hemos cambiado el repositorio para ampliar la interfaz adicional. Así que hazlo. En `CashCardRepository.java`, extienda también `PagingAndSortingRepository`:

### 6. Extienda `PagingAndSortingRepository` y vuelva a ejecutar las pruebas.

Actualice `CashCardRepository` para ampliar también `PagingAndSortingRepository`.

¡No olvides añadir la nueva importación!

```java
import org.springframework.data.repository.PagingAndSortingRepository;
...

interface CashCardRepository extends CrudRepository<CashCard, Long>, PagingAndSortingRepository<CashCard, Long> { ... }
```

Ahora nuestro repositorio es compatible con `pagination` y `sorting`.

¡Pero nuestras pruebas todavía fracasan! Busque el siguiente fallo:

```shell
[~/exercises] $ ./gradlew test
...
Failed to load ApplicationContext
java.lang.IllegalStateException: Failed to load ApplicationContext
...
Caused by: java.lang.IllegalStateException: Ambiguous mapping. Cannot map 'cashCardController' method
example.cashcard.CashCardController#findAll(Pageable)
to {GET [/cashcards]}: There is already 'cashCardController' bean method
example.cashcard.CashCardController#findAll() mapped.
```

(El output real es inmensamente larga. Hemos incluido el mensaje de error más útil en la salida anterior.)

### 7. Comprender y resolver el fallo.

Entonces, ¿qué pasó?

¡No eliminamos el método `findAll()` existente en el `Controller`!

¿Por qué es un problema? ¿No tenemos nombres de métodos únicos y todo se compila?

El problema es que tenemos dos métodos asignados al mismo endpoint. **Spring** detecta este error en tiempo de ejecución, durante el proceso de inicio de **Spring**.

Así que eliminemos el antiguo método `findAll()` sobrante:

```java
// Delete this one:
@GetMapping()
private ResponseEntity<Iterable<CashCard>> findAll() {
    return ResponseEntity.ok(cashCardRepository.findAll());
}
```

### 8. Ejecute las pruebas y asegúrese de que pasen.

```shell
    BUILD SUCCESSFUL in 7s
```

A continuación, implementemos el **Sorting**.
# Sorting

Nos gustaría que las `CashCard` volvieran en un orden que tenga sentido para los humanos. Así que ordenémoslos por cantidad en un orden descendente con las cantidades más altas primero.

### 1. Escribe una prueba (que esperamos que falle).

Añade la siguiente prueba a `CashCardApplicationTests`:

```java
    @Test
    void shouldReturnASortedPageOfCashCards() {
        ResponseEntity<String> response = restTemplate.getForEntity("/cashcards?page=0&size=1&sort=amount,desc", String.class);
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
    
        DocumentContext documentContext = JsonPath.parse(response.getBody());
        JSONArray read = documentContext.read("$[*]");
        assertThat(read.size()).isEqualTo(1);
    
        double amount = documentContext.read("$[0].amount");
        assertThat(amount).isEqualTo(150.00);
    }
```

### 2. Entender la prueba.

El **URI** que estamos solicitando contiene información tanto de `pagination` como de `sorting`: `/cashcards?page=0&size=1&sort=amount,desc`

- `page=0`: Consigue la primera página. Los índices de página comienzan en 0.

- `size=1`: Cada página tiene un tamaño de 1.

- `sort=amount,desc`

La extracción de datos (¡usando más `JSONPath`!) Y las afirmaciones que lo acompañan esperan que la `CashCard` devuelta sea la de `150,00` dólares.

¿Crees que la prueba pasará? Antes de ejecutarlo, trata de averiguar si lo hará o no. Si crees que no pasará, ¿dónde crees que estará el fracaso?

### 3. Ejecuta la prueba.

```shell
[~/exercises] $ ./gradlew test
...
org.opentest4j.AssertionFailedError:
 expected: 150.0
  but was: 123.45
```

La prueba esperaba obtener la `CashCard` de `150,00 $`, pero recibió la de `123,45 $`. ¿Por qué?

La razón es que, dado que no especificamos un orden de clasificación, las tarjetas se devuelven en el orden en que se devuelven de la base de datos. Y resulta que esto es el mismo que el orden en el que se insertaron.

Una observación importante: no todas las bases de datos actuarán de la misma manera. Ahora, debería tener aún más sentido por qué especificamos un orden de `sorting` (en lugar de confiar en el orden predeterminado de la base de datos).

### 4. Implementar `sorting` en el controlador.

Añadir `sorting` al código del controlador es una adición súper simple de una sola línea. En la clase `CashCardController`, agregue un parámetro adicional a la llamada `PageRequest.of()`:

```java
PageRequest.of(
     pageable.getPageNumber(),
     pageable.getPageSize(),
     pageable.getSort()
));
```

El método **getSort()** extrae el parámetro de consulta de clasificación del **URI** de la solicitud.

Ejecuta las pruebas de nuevo. ¡Ellas pasan!

```shell
    CashCardApplicationTests > shouldReturnAllCashCardsWhenListIsRequested() PASSED
```

### 5. Aprende rompiendo cosas.

Para tener un poco más de confianza en la prueba, hagamos un experimento.

En la prueba, cambie el orden de clasificación de descendente a ascendente:

```java
ResponseEntity<String> response = restTemplate.getForEntity("/cashcards?page=0&size=1&sort=amount,asc", String.class);
```

Esto debería hacer que la prueba falle porque la primera `CashCard` en orden ascendente debería ser la tarjeta de `1,00 $`. Ejecute las pruebas y observe el fallo:

```shell
    CashCardApplicationTests > shouldReturnASortedPageOfCashCards() FAILED
    org.opentest4j.AssertionFailedError:
     expected: 150.0
      but was: 1.0
```

¡Correcto! Este resultado refuerza nuestra confianza en la prueba. En lugar de escribir una prueba completamente nueva, usamos una existente para realizar un pequeño experimento.

Ahora volvamos a cambiar la prueba para solicitar el orden de clasificación descendente para que vuelva a pasar.

# Paging and Sorting defaults

Ahora tenemos un endpoint que requiere que el cliente envíe cuatro piezas de información: el índice y el tamaño de la página, el orden de clasificación y la dirección. Esto es mucho pedir, así que hagámoslo más fácil para ellos.

### 1. Escribe una nueva prueba que no envíe ningún parámetro de paginación o clasificación.

Añadiremos una nueva prueba que espera valores predeterminados razonables para los parámetros.

Los valores predeterminados serán:

- Ordenar por cantidad ascendente.

- Un tamaño de página de algo más grande que 3, para que se devuelvan todos nuestros accesorios.

```java
    @Test
    void shouldReturnASortedPageOfCashCardsWithNoParametersAndUseDefaultValues() {
        ResponseEntity<String> response = restTemplate.getForEntity("/cashcards", String.class);
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
    
        DocumentContext documentContext = JsonPath.parse(response.getBody());
        JSONArray page = documentContext.read("$[*]");
        assertThat(page.size()).isEqualTo(3);
    
        JSONArray amounts = documentContext.read("$..amount");
        assertThat(amounts).containsExactly(1.00, 123.45, 150.00);
    }
```

### 2. Ejecuta las pruebas.

```shell
[~/exercises] $ ./gradlew test
...
Actual and expected have the same elements but not in the same order, at index 0 actual element was:
  123.45
whereas expected element was:
  1.0
```

El fallo de la prueba muestra:

- Todas las `CashCard` están siendo devueltas, ya que la afirmación `(page.size()).isEqualTo(3)` tuvo éxito.

- PERO: No están ordenados ya que la afirmación `(amounts).containsExactly(1.00, 123.45, 150.00)` falla:

### 3. Haga que la prueba pase.

Cambie la implementación añadiendo una sola línea al método del `Controller`:

```java
    ...
    PageRequest.of(
            pageable.getPageNumber(),
            pageable.getPageSize(),
            pageable.getSortOr(Sort.by(Sort.Direction.ASC, "amount"))
    ));
    ...
```

### 4. Comprender la implementación.

Entonces, ¿qué acaba de pasar?

La respuesta es que el método `getSortOr()` proporciona valores predeterminados para la página, el tamaño y los parámetros de clasificación. Los valores predeterminados provienen de dos fuentes diferentes:

- **Spring** proporciona los valores predeterminados de la página y el tamaño (son `0` y `20`, respectivamente). Un valor predeterminado de `20` para el tamaño de la página explica por qué se devolvieron nuestras tres `CashCard`. Una vez más: no necesitábamos definir explícitamente estos valores predeterminados. **Spring** los proporciona.

- Definimos el parámetro de `sorting` predeterminado en nuestro propio código, pasando un objeto de `Sorting` a `getSortOr()`:

```java
    Sort.by(Sort.Direction.ASC, "amount")
```

El resultado neto es que si alguno de los tres parámetros requeridos no se pasa a la aplicación, se proporcionarán valores predeterminados razonables.

### 5. Ejecuta las pruebas... ¡otra vez!

¡Enhorabuena!

Todo está pasando ahora.

```shell
[~/exercises] $ ./gradlew test 
... 
BUILD SUCCESSFUL in 7s
```

# Summary

En este laboratorio, implementamos un endpoint "**GET many**" y añadimos `sorting` y `pagination`. Estos lograron dos cosas:

- Se aseguró de que los datos recibidos del servidor están en un orden predecible y comprensible.

- Protegió al cliente y al servidor de sentirse abrumados por una gran cantidad de datos (el tamaño de la página pone un límite a la cantidad de datos que se pueden devolver en una sola respuesta).