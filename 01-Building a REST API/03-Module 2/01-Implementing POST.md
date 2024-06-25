Nuestra **API REST** ahora puede obtener `CashCard` con una identificación específica. En esta lección, añadirás el endpoint **CREATE** a la **API**.

Cuatro preguntas que tendremos que responder mientras hacemos esto son:

1. ¿Quién especifica el **ID**: el cliente o el servidor?

2. En la solicitud de **API**, ¿cómo representamos el objeto que se va a crear?

3. ¿Qué método **HTTP** deberíamos usar en la solicitud?

4. ¿Qué envía la **API** como respuesta?

Comencemos respondiendo a la primera pregunta: "¿Quién especifica la identificación?" En realidad, ¡esto depende del creador de la **API**! **REST** no es exactamente un estándar; es simplemente una forma de usar **HTTP** para realizar operaciones de datos. **REST** contiene una serie de directrices, muchas de las cuales estamos siguiendo en este curso.

Aquí elegiremos dejar que el servidor cree el **ID**. ¿Por qué? Porque es la solución más simple, y las bases de datos son eficientes en la gestión de identificaciones únicas. Sin embargo, para completar, discutamos nuestras alternativas:

- Podríamos solicitar al cliente que proporcione la identificación. Esto podría tener sentido si hubiera una identificación única preexistente, pero ese no es el caso.

- Podríamos permitir que el cliente proporcione el **ID** opcionalmente (y crearlo en el servidor si el cliente no lo proporciona). Sin embargo, no tenemos un requisito para hacer esto, y complicaría nuestra solicitud. Si crees que podrías querer hacer esto "por si acaso", el artículo de [**Yagni**](https://martinfowler.com/bliki/Yagni.html) podría disuadirte.

Antes de responder a la tercera pregunta, "¿Qué método HTTP se debe usar en la solicitud?", hablemos sobre el concepto relevante de **idempotence**.

## Idempotence and HTTP

Una operación **idempotente** se define como una que, si se realiza más de una vez, da como resultado el mismo resultado. En una **API REST**, una operación **idempotente** es una que, incluso si se realizara varias veces, los datos resultantes en el servidor serían los mismos que si se hubieran realizado solo una vez.

Para cada método, el estándar **HTTP** especifica si es **idempotente** o no. **GET**, **PUT** y **DELETE** son **idempotentes**, mientras que **POST** y **PATCH** no lo son.

Dado que hemos decidido que el servidor creará **ID** para cada operación de creación, la operación de creación en nuestra **API** "**NO**" es idempotente. Dado que el servidor creará un nuevo **ID** (en cada solicitud de creación), si llamas a **CREATE** dos veces, incluso con el mismo contenido, terminarás con dos objetos diferentes con el mismo contenido, pero con **ID** diferentes. Eso fue un bocado, así que para resumir: cada solicitud de creación generará una nueva identificación, por lo que no habrá **idempotencia**.

<img src="https://github.com/palmerovicdev/spring-certified-professional-course-es/blob/main/99-Assets/idempotency.jpg">

Esto nos deja con las opciones **POST** y **PATCH**. Resulta que **REST** permite **POST** como uno de los métodos adecuados para crear operaciones, por lo que lo usaremos. Revisaremos **PATCH** en una lección posterior.

## The POST Request and Response

Ahora hablemos sobre el contenido de la solicitud **POST** y la respuesta.

### The Request

El método **POST** permite un cuerpo, por lo que usaremos el cuerpo para enviar una representación **JSON** del objeto:

Request:

- Method: `POST`
- URI: `/cashcards/`
- Body:
```json
    {
        amount: 123.45
    }
```

Por el contrario, si recuerda de una lección anterior, la operación **GET** incluye la identificación de la `CashCard` en el **URI**, pero no en el cuerpo de la solicitud.

Entonces, ¿por qué no hay identificación en la solicitud? Porque decidimos permitir que el servidor creara el **ID**. Por lo tanto, el contrato de datos para la operación de lectura es diferente al de la operación **CREATE**.

### The Response

Vamos a la respuesta. En una creación exitosa, ¿qué código de estado de respuesta **HTTP** se debe enviar? Podríamos usar `200 OK` (la respuesta que devuelve **READ**), pero hay un código más específico y preciso para las `API REST: 201 CREATED`.

El hecho de que **CREATED** sea el nombre del código hace que parezca intuitivamente apropiado, pero hay otra razón más técnica para usarlo: un código de respuesta de **200 OK** no responde a la pregunta "¿Hubo algún cambio en los datos del servidor?". Al devolver el estado `201 CREATED`, la **API** está comunicando específicamente que los datos se agregaron al almacén de datos en el servidor.

En una lección anterior aprendiste que una respuesta **HTTP** contiene dos cosas: un código de estado y un cuerpo. ¡Pero eso no es todo! Una respuesta también contiene encabezados. Los encabezados tienen un nombre y un valor. El estándar **HTTP** especifica que el encabezado de ubicación en una respuesta `201 CREATED` debe contener el URI del recurso creado. Esto es útil porque permite a la persona que llama obtener fácilmente el nuevo recurso utilizando el endpoint **GET** (el que implementamos anteriormente).

Aquí está la respuesta completa:

Response:
- Status Code: `201 CREATED`
- Header: `Location=/cashcards/42`

### Spring Web Convenience Methods

En el laboratorio que lo acompaña, verá que **Spring Web** proporciona métodos orientados al uso recomendado de **HTTP** y **REST**.

Por ejemplo, utilizaremos el método `ResponseEntity.created(uriOfCashCard)` para crear la respuesta anterior. Este método requiere que especifique la ubicación, se asegura de que el **URI** de ubicación esté bien formado (utilizando la clase **URI**), agregue el encabezado de ubicación y establezca el código de estado para usted. Y al hacerlo, esto nos evita usar métodos más detallados. Por ejemplo, los siguientes dos fragmentos de código son equivalentes (siempre y cuando `uriOfCashCard` no sea nulo):
```java
return  ResponseEntity
        .created(uriOfCashCard)
        .build();
```

Versus:
```java
return ResponseEntity
        .status(HttpStatus.CREATED)
        .header(HttpHeaders.LOCATION, uriOfCashCard.toASCIIString())
        .build();
```

¿No te alegras de que Spring Web proporcione el método de conveniencia `.created()`?

## Summary
Hemos tomado algunas decisiones sobre cómo nuestra **API** apoyará la creación de `CashCard` familiares:

- La **API** aceptará solicitudes **POST** para crear una `CashCard`.

- El servidor creará identificaciones para todas las `CashCard`.

- Si tiene éxito, la **API** devolverá una respuesta con el código de estado: `201 CREATED`, que contiene el **URI** (ubicación) del nuevo recurso de `CashCard` en los encabezados de respuesta.

¡En el laboratorio, implementaremos estas decisiones!
