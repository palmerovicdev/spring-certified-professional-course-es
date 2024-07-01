Ahora que nuestra **API** puede crear `CashCard`, es razonable aprender a obtener todas (¡o algunas!) de las `CashCard`. En esta lección, implementaremos el endpoint "Read many" y entenderemos cómo esta operación difiere sustancialmente del endpoint de lectura que creamos anteriormente.

## Requesting a List of Cash Cards

Podemos esperar que cada uno de nuestros usuarios de **Family Cash Card** tenga algunas tarjetas: tal vez una para cada dependiente, y tal vez algunas que se regalaron. La **API** debería poder devolver varias `CashCard` en respuesta a una sola solicitud de **REST**.

Cuando haga una solicitud de **API** para varias `CashCard`, lo ideal sería hacer una sola solicitud, que devuelva una lista de `CashCard`. Por lo tanto, necesitaremos un nuevo contrato de datos. En lugar de una sola `CashCard`, el nuevo contrato debe especificar que la respuesta es una matriz **JSON** de objetos de `CashCard`:

```json
[
  {
    "id": 1,
    "amount": 123.45
  },
  {
    "id": 2,
    "amount": 50.0
  }
]
```

Resulta que nuestro viejo amigo, `CrudRepository`, tiene un método `findAll` que podemos usar para obtener fácilmente todas las `CashCard` de la base de datos. Sigamos adelante y usemos ese método. A primera vista, parece bastante simple:

```java
@GetMapping()
private ResponseEntity<Iterable<CashCard>> findAll() {
   return ResponseEntity.ok(cashCardRepository.findAll());
}
```

Sin embargo, resulta que hay mucho más en esta operación que simplemente devolver todas las `CashCard` en la base de datos. Algunas preguntas me vienen a la mente:

### 1. ¿Cómo devuelvo solo las `CashCard` que posee el usuario? (¡Gran pregunta! Discutiremos esto en la próxima lección de Spring Security).

### 2. ¿Y si hay cientos (o miles) de `CashCard`? ¿Debería la **API** devolver un número ilimitado de resultados o devolverlos en "partes"? Esto se conoce como **Paginación**.

### 3. ¿Deberían devolverse las `CashCard` en un orden en particular (es decir, deberían clasificarse)?

Dejaremos la primera pregunta para más tarde, pero responderemos a las preguntas de paginación y clasificación de esta lección.

## Pagination and Sorting

Para comenzar nuestro trabajo de paginación y clasificación, utilizaremos una versión especializada del `CrudRepository`, llamada `PagingAndSortingRepository`. Como puedes adivinar, esto hace exactamente lo que su nombre sugiere. Pero primero, hablemos de la funcionalidad "**Paging**".

Aunque es poco probable que tengamos usuarios con miles de `CashCard`, nunca sabemos cómo los usuarios podrían usar el producto. Idealmente, una **API** no debería ser capaz de producir una respuesta con un tamaño ilimitado, porque esto podría abrumar la memoria del cliente o del servidor, ¡por no hablar de llevar mucho tiempo!

Con el fin de garantizar que una respuesta de la **API** no incluya un número astronómicamente grande de `CashCard`, utilicemos la funcionalidad de paginación de **Spring Data**. La paginación en **Spring** (y muchos otros marcos) es especificar la longitud de la página (por ejemplo, 10 elementos) y el índice de la página (a partir de 0). Por ejemplo, si un usuario tiene 25 `CashCard` y usted elige solicitar la segunda página donde cada página tiene 10 elementos, solicitaría una página de tamaño 10 y un índice de página de 1.

¡Bingo! ¿Verdad? Pero espera, esto plantea otro obstáculo. Para que la paginación produzca el contenido correcto de la página, los elementos deben ordenarse en un orden específico. ¿Por qué? Bueno, digamos que tenemos un montón de `CashCard` con las siguientes cantidades:

- `0,19` $ (este casi se ha ido, ¡oh, bueno!)

- `1.000,00 $` (este es para compras de emergencia para un estudiante universitario)

- `50,00 $`

- `20,00 $`

- `10,00 $` (alguien le regaló este a tu sobrina por su cumpleaños)

Ahora vamos a ver un ejemplo usando un tamaño de página de 3. Dado que hay 5 `CashCard`, haríamos dos solicitudes para devolverlas todas. La página 1 (índice 0) contiene tres elementos, y la página 2 (índice 1, la última página) contiene 2 elementos. Pero, ¿qué artículos van a dónde? Si especifica que los artículos deben ordenarse por cantidad en orden descendente, entonces así es como se paginan los datos:

Página 1:

- `1,000.00 $`

- `50,00 $`

- `20,00 $`

Página 2:

- `10,00 $`

- `0,19 $`

### Regarding Unordered Queries

Aunque **Spring** proporciona una estrategia de clasificación "no ordenada", seamos explícitos cuando seleccionemos qué campos para ordenar. ¿Por qué hacer esto? Bueno, imagina que eliges usar una paginación "no ordenada". En realidad, el pedido no es aleatorio, sino predecible; nunca cambia en las solicitudes posteriores. Digamos que haces una solicitud, y **Spring** devuelve los siguientes resultados "no ordenados":

Página 1:

- `0,19 $`

- `1,000.00 $`

- `50,00 $`

Página 2:

- `20,00 $`

- `10,00 $`

Aunque se ven al azar, cada vez que hagas la solicitud, las tarjetas volverán exactamente en este orden, de modo que cada artículo se devuelva exactamente en una página.

Ahora para la frase: Imagina que ahora creas una nueva `CashCard` con una cantidad de `42,00 $`. ¿En qué página crees que estará? Como puedes adivinar, no hay otra forma de saberlo que hacer la solicitud y ver dónde aterriza la nueva `CashCard`.

Entonces, ¿cómo podemos hacer que esto sea un poco más útil? Optemos por ordenar por un campo específico. Hay algunas buenas razones para hacerlo, incluyendo:

- Minimizar la sobrecarga cognitiva: Otros desarrolladores (sin mencionar a los usuarios) probablemente apreciarán un pedido reflexivo al desarrollarlo.

- Minimizar los errores futuros: ¿Qué sucede cuando una nueva versión de **Spring**, o **Java**, o la base de datos, de repente hace que el orden "aleatorio" cambie de la noche a la mañana?

### Spring Data Pagination API

Afortunadamente, **Spring Data** proporciona las clases `PageRequest` y `Sort` para la paginación. Echemos un vistazo a una consulta para obtener la página 2 con tamaño de página 10, ordenando por cantidad en orden descendente (las cantidades más grandes primero):

```java
Page<CashCard> page2 = cashCardRepository.findAll(
    PageRequest.of(
        1,  // page index for the second page - indexing starts at 0
        10, // page size (the last page might have fewer items)
        Sort.by(new Sort.Order(Sort.Direction.DESC, "amount"))));
```

### The Request and Response

Ahora usemos **Spring Web** para extraer los datos para alimentar la funcionalidad de paginación:

- **Paginación**: **Spring** puede analizar los parámetros de la página y el tamaño si pasa un objeto `Pageable` a un método de búsqueda de `PagingAndSortingRepository...()`.

- **Clasificación**: **Spring** puede analizar el parámetro de clasificación, que consiste en el nombre del campo y la dirección separada por una coma, pero tenga cuidado, ¡no se permite ningún espacio antes o después de la coma! Una vez más, estos datos forman parte del objeto `Pageable`.

### The URI

Ahora vamos a aprender cómo podemos componer un **URI** para el nuevo endpoint, paso a paso (hemos omitido el prefijo `https://dominio` a continuación):

### 1. Pedir la segunda pagina

```java
/cashcards**?page=1**
```

### 2. …donde una página tiene una longitud de 3

```java
/cashcards?page=1**&size=3**
```

### 3. …ordenado por el saldo actual de la `CashCard`

```java
/cashcards?page=1&size=3**&sort=amount**
```

### 4. …en orden descendente (el saldo más alto primero)

```java
/cashcards?page=1&size=3&sort=amount**,desc**
```

### The Java Code

Vamos a repasar la implementación completa del método del `Controller` para nuestro nuevo endpoint "pedir una página de `CashCard`":
```java
@GetMapping
private ResponseEntity<List<CashCard>> findAll(Pageable pageable) {
   Page<CashCard> page = cashCardRepository.findAll(
           PageRequest.of(
                   pageable.getPageNumber(),
                   pageable.getPageSize(),
                   pageable.getSortOr(Sort.by(Sort.Direction.DESC, "amount"))));
   return ResponseEntity.ok(page.getContent());
}
```

Vamos a sumergirnos en un poco más de detalle:

- Primero vamos a analizar los valores necesarios de la cadena de consulta:
	- Utilizamos `Pageable`, que permite a **Spring** analizar los parámetros de la cadena de consulta de número de página y tamaño.
		- Nota: Si la persona que llama no proporciona los parámetros, **Spring** proporciona valores predeterminados: `página=0`, `tamaño=20`.

- Utilizamos `getSortOr()` para que incluso si la persona que llama no proporciona el parámetro de clasificación, haya un valor predeterminado. A diferencia de los parámetros de página y tamaño, para los que tiene sentido que **Spring** proporcione un valor predeterminado, no tendría sentido que **Spring** elija arbitrariamente un campo de clasificación y una dirección.

- Utilizamos el método `page.getContent()` para devolver las `CashCard` contenidas en el objeto `Page` a la persona que llama.

Entonces, ¿qué contiene el objeto `Page` además de las `CashCard`? Aquí está el objeto `Page` en formato **JSON**. Las `CashCard` están contenidas en el contenido. El resto de los campos contienen información sobre cómo esta página está relacionada con otras páginas de la consulta.

```json
{
  "content": [
    {
      "id": 1,
      "amount": 10.0
    },
    {
      "id": 2,
      "amount": 0.19
    }
  ],
  "pageable": {
    "sort": {
      "empty": false,
      "sorted": true,
      "unsorted": false
    },
    "offset": 3,
    "pageNumber": 1,
    "pageSize": 3,
    "paged": true,
    "unpaged": false
  },
  "last": true,
  "totalElements": 5,
  "totalPages": 2,
  "first": false,
  "size": 3,
  "number": 1,
  "sort": {
    "empty": false,
    "sorted": true,
    "unsorted": false
  },
  "numberOfElements": 2,
  "empty": false
}
```

Aunque podríamos devolver todo el objeto de la página al cliente, no necesitamos toda esa información. Definiremos nuestro contrato de datos para devolver solo las `CashCard`, no el resto de los datos de la página.

## Summary

**Spring Boot** y **Spring Data** hacen que sea tan fácil recuperar varios elementos de una aplicación como recuperar solo uno. Además, las potentes pero fáciles de usar capacidades de paginación, clasificación y filtrado de **Spring Data** facilitan a los desarrolladores la incorporación de estas características en una aplicación.

Hablando de eso, ¡vamos a implementar listas, paginación y clasificación en nuestra propia **API** de **Family Cash Card**!