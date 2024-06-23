Estamos desarrollando una **API**. Esto plantea muchas preguntas sobre cómo debería comportarse la **API**:

- ¿Cómo deberían interactuar los consumidores de la **API** con la **API**?
- ¿Qué datos necesitan enviar los consumidores en varios escenarios?
- ¿Qué datos debería devolver la **API** a los consumidores y cuándo?
- ¿Qué comunica la **API** cuando se usa incorrectamente (o algo sale mal)?

Siempre que sea posible, los proveedores de **API** y los consumidores deben discutir estos escenarios y llegar a acuerdos. Mejor aún, deberían documentar estos acuerdos no solo en un sistema de documentación compartida, sino también de una manera que apoye las pruebas automatizadas que pasan (o no) basadas en estas decisiones.

Aquí es donde entra en juego el concepto de contratos.
## API Contracts

La industria del software ha adoptado varios patrones para capturar el comportamiento acordado de la **API** en la documentación y el código. Estos acuerdos a menudo se denominan **"contracts"**. Dos ejemplos incluyen los contratos impulsados por el consumidor **(Consumer Driven Contracts)** y los contratos impulsados por el proveedor **(Provider Driven Contracts)**. Proporcionaremos recursos para estos patrones, pero no los discutiremos en detalle en este curso. En su lugar, discutiremos un concepto ligero llamado contratos **API**.

Definimos un contrato de **API** como un acuerdo formal entre un **proveedor** de software y un **consumidor** que comunica de forma abstracta cómo interactuar entre sí. Este contrato define cómo interactúan los **proveedores** de **API** y los **consumidores**, cómo son los intercambios de datos y cómo comunicar los casos de éxito y fracaso.

El **proveedor** y los **consumidores** <u>no tienen que compartir el mismo lenguaje de programación</u>, solo los mismos contratos de **API**. Para el dominio **Family Cash Card**, supongamos que actualmente hay un contrato entre el servicio **Cash Card** y todos los servicios que lo utilizan. A continuación se muestra un ejemplo de ese primer contrato de **API**. No te preocupes si no entiendes todo el contrato. Abordaremos todos los aspectos de la información a continuación a medida que complete este curso.
  
```yaml
Request
  URI: /cashcards/{id}
  HTTP Verb: GET
  Body: None

Response:
  HTTP Status:
    200 OK if the user is authorized and the Cash Card was successfully retrieved
    401 UNAUTHORIZED if the user is unauthenticated or unauthorized
    404 NOT FOUND if the user is authenticated and authorized but the Cash Card cannot be found
  Response Body Type: JSON
  Example Response Body:
    {
      "id": 99,
      "amount": 123.45
    }
```

### Why Are API Contracts Important?

Los contratos de la **API** son importantes porque comunican el comportamiento de una **API REST**. Proporcionan detalles específicos sobre los datos que se serializan (o deserializan) para cada comando y parámetro que se intercambian. Los contratos de la **API** están escritos de tal manera que se pueden traducir fácilmente a la funcionalidad del proveedor de **API** y al consumidor, y a las pruebas automatizadas correspondientes. Implementaremos tanto la funcionalidad del proveedor de **API** como las pruebas automatizadas en los laboratorios.

## What Is JSON?

**JSON** (Javascript Object Notation) proporciona un formato de intercambio de datos que representa la información particular de un objeto en un formato que se puede leer y entender fácilmente. Utilizaremos **JSON** como nuestro formato de intercambio de datos para la **API** de **Family Cash Card**.

Este es el ejemplo que usamos arriba:
```json
{
  "id": 99,
  "amount": 123.45
}
```

Agregue este archivo json en la ubicacion `src/test/resources/example/cashcard/expected.json`

Otros formatos de datos populares incluyen **YAML** (Yet Another Markup Language) y **XML** (Extensible Markup Language). En comparación con **XML**, **JSON** lee y escribe más rápido, es más fácil de usar y ocupa menos espacio. Puedes usar **JSON** con la mayoría de los lenguajes de programación modernos y en todas las plataformas principales. También funciona a la perfección con aplicaciones basadas en **Javascript**.

Por estas razones, **JSON** ha reemplazado en gran medida a **XML** como el formato más utilizado para las **API** utilizadas por las aplicaciones web, incluidas las **API REST**.

## Summary
Los contratos de **API** son un medio popular para que los proveedores de **API** y los consumidores se pongan de acuerdo sobre cómo se comportará una **API**. Los contratos de **API** pueden ser tan simples como la documentación compartida para sofisticados marcos de gestión y validación de contratos. Los contratos combinados con **JSON** (el formato de intercambio de datos más popular para aplicaciones modernas basadas en la web) pueden ser un medio poderoso para ayudar a los proveedores de **API** y a los consumidores a desarrollar y probar las **API**.

Implementarás pruebas automatizadas y verificación de los contratos de **API** más adelante en este curso.