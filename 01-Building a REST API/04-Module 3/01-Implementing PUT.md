# Implementing `PUT`

Hasta ahora podemos crear y recuperar `CashCard`. 
¡El siguiente paso lógico es la posibilidad de ajustar el saldo de una Card!
Cómo implementar la operación de actualización en una `API RESTful` es un tema candente,
que es lo que abordaremos en esta lección.

Cuando decimos "actualizar el saldo" en una `CashCard`, lo que realmente queremos decir es 
actualizar el monto en un registro de base de datos existente. Hacerlo implicará:

- Crear un nuevo endpoint para recibir una solicitud `HTTP` con un verbo, `URI` y cuerpo

- Devolver una respuesta adecuada desde el endpoint para condiciones de éxito y error

Ya estamos familiarizados con el verbo `HTTP` `POST`, que utilizamos para el endpoint `CREATE`.
Ahora hablemos de `HTTP` `PUT` y `PATCH`, y de cómo se relacionan los tres.

## PUT and PATCH

Tanto `PUT` como `PATCH` se pueden utilizar para actualizar, pero funcionan de diferentes maneras. Básicamente, `PUT` significa "crear o reemplazar el registro completo", mientras que `PATCH` significa "actualizar sólo algunos campos del registro existente"; en otras palabras, una actualización parcial.

¿Por qué querrías hacer una actualización parcial? Las actualizaciones parciales liberan al cliente de tener que cargar el registro completo y luego transmitirlo nuevamente al servidor. Si el registro es lo suficientemente grande, esto puede tener un impacto no trivial en el rendimiento.

Para nuestra aplicación, elegiremos no implementar una actualización parcial.

## PUT and POST

Hay algo que no le dijimos cuando escribimos el endpoint `CREATE`: ¡el estándar `HTTP` no especifica si se prefiere el verbo `POST` o `PUT` para una 
operación `CREATE`! Esto es relevante porque usaremos el verbo `PUT` para nuestro endpoint de `UPDATE`, por lo que debemos decidir si nuestra `API` 
admitirá el uso de `PUT` para `CREATE` o `UPDATE` un recurso.

Hay diferentes maneras de ver la relación entre las operaciones `CREATE` y `UPDATE` y cómo se implementan en `REST` usando verbos `HTTP`. En las siguientes
secciones, lo analizamos desde tres puntos de vista diferentes y luego resumimos lo que hemos cubierto hasta ahora en una tabla.

Lo importante de las siguientes secciones no es memorizar todos los detalles, sino simplemente darse cuenta de que hay muchas opciones diferentes y que nosotros, como autores de `Cash Card API`, estamos tomando decisiones conscientes sobre cómo implementar `REST`.

### Surrogate and Natural Keys

¿Por qué querríamos utilizar una operación `PUT` para crear un recurso? Esto tiene que ver con la definición `HTTP` de los dos verbos. La diferencia es sutil. Vamos a explicarlo comparando dos sistemas diferentes: nuestra `API` de `CashCard` y otra `API` que presentaremos con fines explicativos, llamada `API` de factura. La `API` de factura acepta el número de factura como identificador único. Este es un ejemplo del uso de una clave natural (proporcionada por el cliente a la `API`) en lugar de una clave sustituta (normalmente generada por el servidor, que es lo que estamos haciendo en nuestra API de `CashCard`).

La diferencia importante es si el servidor debe generar el `URI` (que incluye el `ID` del recurso) o no. Así es como lo piensan `PUT` y `POST`:

### 1. Si necesita que el servidor devuelva el `URI` del recurso creado (o los datos que usa para construir el `URI`), entonces debe usar `POST`.

Este es el caso de nuestra API de `CashCard`: para crear una `CashCard`, proporcionamos el endpoint `POST` `/cashcards`. El `URI` real para la 
`CashCard` creada depende del `ID` generado y lo proporciona el servidor, por ejemplo, `/cashcards/101` si el `ID` de la tarjeta creada es `101`.

### 2. Alternativamente, cuando se conoce el `URI` del recurso en el momento de la creación (como es el caso en nuestro ejemplo de `API` de factura), puede 
utilizar `PUT`.

Para la `API` de factura, podríamos escribir un endpoint de creación que acepte solicitudes como `PUT` `/invoice/1234-567`. La llamada de lectura correspondiente utilizaría exactamente el mismo `URI`: `GET /invoice/1234-567`.

### Resources and Sub-Resources

Otra forma de ver la diferencia es en términos de `URI` y colecciones de subrecursos. Este es el lenguaje utilizado por la documentación `HTTP`, por lo que es bueno estar familiarizado con él. Siguiendo los ejemplos anteriores, encontraríamos:
 
- `POST` crea un subrecurso (recurso secundario) debajo (después) o dentro del `URI` de solicitud. Esto es lo que hace la `API Cash Card`: el cliente 
llama al endpoint `CREATE` en `POST /cashcards`, pero el `URI` real del recurso creado contiene una `ID` generada al final: `/cashcards/101`

- `PUT` crea o reemplaza (actualiza) un recurso en un `URI` de solicitud específico. Para el ejemplo de `/invoice` anterior, el endpoint `CREATE` sería 
`PUT /invoice/1234-567` y el `URI` del recurso creado sería el mismo que el `URI` enviado en la solicitud `PUT`.

### Response Body and Status Code

En relación con la decisión de permitir que `PUT` cree objetos, debe decidir cuál debe ser el código de estado y el cuerpo de la respuesta. Dos opciones diferentes son:

- Devuelve `201 CREATED` (si creó el objeto) o `200 OK` (si reemplazó un objeto existente). En este caso, se recomienda devolver el objeto en el cuerpo 
de la respuesta. Esto es útil si el servidor agregó datos al objeto (por ejemplo, si el servidor registra la fecha de creación).

o

- Devuelve `204 NO CONTENT` y un cuerpo de respuesta vacío. La razón en este caso es que dado que un `PUT` simplemente coloca un objeto en el `URI` de 
la solicitud, el cliente no necesita ninguna información: sabe que el objeto de la solicitud se ha guardado, palabra por palabra, en el servidor.

## POST, PUT, PATCH and CRUD Operations - Summary

Las secciones anteriores se pueden resumir utilizando la siguiente tabla:

| HTTP Verb   | CRUD Operation | Definition of Resource URI                 | What does it do?                                                                                   | Response Status Code | Response Body          |
|-------------|----------------|--------------------------------------------|----------------------------------------------------------------------------------------------------|----------------------|------------------------|
| **`POST`**	 | **`CREATE`**   | **El servidor genera y devuelve el `URI`** | 	**Crea un subrecurso ("debajo" o "dentro" del `URI` pasado)**                                     | **`201 CREATED`**	   | **El recurso creado**  |
| `PUT`	      | `CREATE`	      | El cliente proporciona el `URI`	           | Crea un recurso (en el `URI` de solicitud)	                                                        | `201 CREATED`	       | El recurso creado      |
| **`PUT`**	  | **`UPDATE`**   | **El cliente proporciona el `URI`**	       | **Reemplaza el recurso: todo el registro es reemplazado por el objeto en la Solicitud**	           | **`204 NO CONTENT`** | ** Vacio**             |
| `PATCH`     | `UPDATE`	      | El cliente proporciona el `URI`	           | Actualización parcial: modifica solo los campos incluidos en la solicitud en el registro existente | `200 OK`	            | El recurso actualizado |

En la `API` de `Cash Card`, no necesitamos permitir que `PUT` cree recursos. Tampoco necesitamos agregar datos en el lado del servidor para una operación de actualización, ni necesitamos permitir una actualización parcial. Por lo tanto, nuestro endpoint `PUT` se limita a la fila 3 de la tabla anterior.

Las filas en **negrita** de la tabla anterior se implementan mediante la `API Cash Card`. Los que no están en **negrita** no.

## Security

Una decisión más que tomaremos es cómo aplicar la lógica de seguridad a la nueva operación de `UPDATE`.

Recuerde de la lección de Seguridad simple que decidimos devolver `404 NOT FOUND` para solicitudes `GET` en dos casos: identificaciones inexistentes e intento de acceso a tarjetas para las cuales el usuario no está autorizado. Usaremos la misma estrategia para el endpoint de `UPDATE`, por razones similares (sobre las cuales entraremos en más detalles en el laboratorio).

# Our API Decisions

¡Vaya, fueron muchas decisiones! Para resumir, decidimos:
- `PUT` no admitirá la creación de una `CashCard`.
- Nuestro nuevo endpoint de actualización (que crearemos en el próximo laboratorio):
    - utilizará el verbo `PUT`.
    - acepta una `CashCard` y reemplaza la `CashCard` existente con ella.
    - en caso de éxito, devolverá `204 NO CONTENT` con un cuerpo vacío.
    - devolverá un `404 NOT FOUND` para una actualización no autorizada, así como intentos de actualizar `ID` inexistentes.

# Summary

Esta lección abordó el tema de las operaciones `CRUD` `UPDATE` y `CREATE` y cómo se pueden implementar usando `REST`.

Ahora que hemos tomado algunas decisiones sobre cómo implementar la Actualización en la `API` de `Cash Card`, escribamos algo de código. ¡Pasemos al laboratorio!


