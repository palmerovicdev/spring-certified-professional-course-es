# Implementing DELETE

En esta lección, implementaremos la última de las cuatro operaciones `CRUD`: ¡`DELETE`! A estas alturas, ya debería estar familiarizado con la pregunta que debemos hacernos primero: ¿Cuál es la especificación de datos de la `API` para el endpoint `DELETE`? La especificación incluye los detalles de la Solicitud y Respuesta.

A riesgo de estropear el resultado, esta es la especificación que definiremos:

**Request**:
- **Verb**: `DELETE`
- **URI**: `/cashcards/{id}`
- **Body**: (vacío)

**Response**:
- **Status Code**: `204 NO CONTENT`
- **Body**: (vacío)

Devolveremos el código de estado `204 NO CONTENT` para una eliminación exitosa, pero hay casos adicionales:

| Response Code	   | Use Case                                                                                                     |
|------------------|--------------------------------------------------------------------------------------------------------------|
| `204 NO CONTENT` | - The record exists, and <br> - The Principal is authorized, and <br> - The record was successfully deleted. |
| `404 NOT FOUND`  | - The record does not exist (a non-existent `ID` was sent).                                                  |
| `404 NOT FOUND`  | - The record does exist but the Principal is not the owner.                                                  |

¿Por qué devolvemos `404` para los casos "`ID` no existe" y "no autorizado para acceder a este `ID`"? 
* **Para no "filtrar" información!!**: si la `API` arrojó resultados diferentes para los dos casos, entonces un usuario no autorizado podría descubrir `ID`s específicas a las que no está autorizado a acceder.

# Additional Options

Profundicemos en algunas opciones más que consideraremos al implementar una operación de eliminación.

## Hard and Soft Delete

Entonces, ¿qué significa eliminar una `CashCard` desde el punto de vista de una base de datos?
* De manera similar a cómo decidimos que nuestra operación de `UPDATE` significa "reemplazar todo el registro existente" (en lugar de admitir una actualización parcial), debemos decidir qué sucede con los recursos cuando se eliminan.

Una opción sencilla, denominada eliminación definitiva, es eliminar el registro de la base de datos. Con una eliminación completa, desaparece para siempre. 
Entonces, ¿qué podemos hacer si necesitamos datos que existían antes de su eliminación?

* Una alternativa es la **eliminación temporal** que funciona marcando registros como "eliminados" en la base de datos (para que se conserven, pero se marquen como eliminados). Por ejemplo, podemos introducir una columna de marca de tipo **boolean** `IS_DELETED` o de tipo **timestamp** `DELETED_DATE` y luego establecer ese valor, en lugar de eliminar completamente el registro eliminando las filas de la base de datos. 
* Con una **eliminación temporal**, también necesitamos cambiar la forma en que los repositorios interactúan con la base de datos. Por ejemplo, un repositorio debe respetar la columna **"eliminado"** y excluir los registros marcados como eliminados de las solicitudes de lectura.

# Audit Trail and Archiving

Cuando trabaje con bases de datos, encontrará que a menudo es necesario mantener un registro de las modificaciones de los registros de datos. Por ejemplo:
- Es posible que un representante de servicio al cliente necesite saber cuándo un cliente eliminó su `CashCard`.
- Es posible que existan regulaciones de cumplimiento de retención de datos que requieran que los datos eliminados se conserven durante un cierto 
período de tiempo.

Si la `CashCard` se elimina por completo, necesitaremos almacenar datos adicionales para poder responder esta pregunta. Analicemos algunas formas de registrar información histórica:
1. Archive (mueva) los datos eliminados a una ubicación diferente.
2. Agregue campos de auditoría al propio registro. Por ejemplo, la columna `DELETED_DATE` que ya mencionamos. Se pueden agregar campos de auditoría adicionales, por ejemplo `DELETED_BY_USER`. Nuevamente, esto no se limita a las operaciones de eliminación, sino también a las de creación y actualización.
	- Las `API` que implementan campos de auditoría y eliminación temporal pueden devolver el estado del objeto en la respuesta y el código de estado `200 OK`. Entonces, ¿por qué elegimos utilizar `204` en lugar de `200`? Porque el estado `204 NO CONTENT` implica que no hay cuerpo en la respuesta.
3. Mantener un registro de auditoría. La pista de auditoría es un registro de todas las operaciones importantes realizadas en un registro. Puede contener no sólo operaciones de eliminación, sino también creación y actualización.
	- La ventaja de un seguimiento de auditoría sobre los campos de auditoría es que un seguimiento registra todos los eventos, mientras que los campos de auditoría del registro capturan sólo la operación más reciente. Una pista de auditoría se puede almacenar en una ubicación de base de datos diferente o incluso en archivos de registro.

Cabe mencionar que es posible una combinación de varias de las estrategias anteriores. Aquí hay unos ejemplos:

- Podríamos implementar la **eliminación temporal** y luego tener un proceso separado que **elimine por completo** o **archive** los registros eliminados temporalmente **después de un cierto período de tiempo**, como una vez al año.

- Podríamos implementar una **eliminación completa** y **archivar** los registros eliminados.

En cualquiera de los casos anteriores, podríamos mantener un registro de auditoría de qué operaciones ocurrieron y cuándo.

Finalmente, observe que incluso la especificación simple que hemos elegido no determina si implementamos la eliminación definitiva o parcial. 
Tampoco determina si agregamos campos de auditoría o mantenemos un registro de auditoría. Sin embargo, el hecho de que hayamos elegido devolver `204 NO CONTENT` implica que no se está produciendo una eliminación temporal, ya que si así fuera, probablemente elegiríamos devolver `200 OK` con el registro en el cuerpo.

# Summary

Esta lección presentó el concepto de eliminar una `CashCard` y los códigos de respuesta `HTTP` semánticos involucrados. Describimos en detalle los conceptos de eliminación temporal y temporal, junto con las ventajas y desventajas de ambos. Además, describimos varios conceptos de auditoría de datos relacionados con el seguimiento de las fechas y horas en que se elimina una entidad.

En el laboratorio, haremos lo más simple: implementar la **eliminación definitiva** y no implementar la **auditoría** ni el **archivado**.