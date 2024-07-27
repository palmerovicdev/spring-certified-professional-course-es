## Configuraciones en el application.properties
Para configurar Jackson en tu archivo `application.properties`, puedes utilizar varias propiedades específicas de Spring Boot que afectan el comportamiento de la serialización y deserialización JSON. Aquí te muestro algunas de las más comunes relacionadas con Jackson:

### 1. **Ignorar propiedades nulas**: 

- Para ignorar las propiedades nulas durante la serialización, ya has mencionado la propiedad correcta:
`spring.jackson.default-property-inclusion=non_null`

### 2. **Indicar si se deben incluir o excluir ciertas propiedades**:

-  Para especificar una lista de propiedades a incluir, usa:
`spring.jackson.serialization.inclusion=include spring.jackson.serialization.include=propiedad1,propiedad2`

- Para especificar una lista de propiedades a excluir, usa:
`spring.jackson.serialization.inclusion=exclude spring.jackson.serialization.exclude=propiedad1,propiedad2`

### 3. **Configurar el formato de fecha**:

- Para establecer un patrón de fecha personalizado, utiliza:
`spring.jackson.date-format=yyyy-MM-dd'T'HH:mm:ssZ`

### 4. **Habilitar o deshabilitar la escritura de fechas como timestamps**:

-  Para escribir fechas como timestamps (números largos), usa:
`spring.jackson.time-zone=UTC spring.jackson.write-dates-as-timestamps=true`

- Para escribir fechas en un formato legible por humanos, usa:
`spring.jackson.write-dates-as-timestamps=false`

### 5. **Permitir o no caracteres especiales en nombres de propiedades**:

- Para permitir caracteres especiales en nombres de propiedades, usa:
`spring.jackson.mapper.allow-non-standard-characters=true`

### 6. **Habilitar o deshabilitar la lectura de propiedades desconocidas**:

- Para permitir la lectura de propiedades desconocidas sin lanzar errores, usa:
`spring.jackson.deserialization.fail-on-unknown-properties=false`

## Configuracion del ObjectMapper

```java
@Configuration

public class JacksonConfiguration {
	@Bean
	ObjectMapper objectMapper(){
		return new ObjectMapper();
	}
}
```
### 1. `Si se hace esta config` se deben cambiar los `LocalDate` por `Date` de `java.util` debido a que por defecto `Jackson` no soporta conversion de este tipo.

## Annotations
La anotación `@JacksonNaming` en Java se utiliza para especificar la estrategia de nomenclatura que Jackson debe usar al serializar y deserializar objetos JSON. La estrategia `SnakeCaseStrategy` es una de las muchas disponibles y convierte los nombres de las propiedades del objeto a minúsculas con guiones bajos (snake case). Aquí te muestro cómo configurarla y algunas otras estrategias comunes:

### Configuración básica

Para aplicar la estrategia `SnakeCaseStrategy`, simplemente añade la anotación `@JacksonNaming` a tu clase o campo específico:

```java
import com.fasterxml.jackson.databind.PropertyNamingStrategies;
import com.fasterxml.jackson.databind.annotation.JsonNaming;

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
public class MiClase {
    private String miPropiedad; // Se convertirá a "mi_propiedad" en JSON
}

```
### Otras estrategias de nomenclatura

Jackson ofrece varias estrategias de nomenclatura predefinidas además de `SnakeCaseStrategy`. Algunas de ellas incluyen:

- **CamelCaseStrategy**: Convierte los nombres de las propiedades a camel case (por defecto).
- **PascalCaseStrategy**: Convierte los nombres de las propiedades a Pascal case.
- **KebabCaseStrategy**: Convierte los nombres de las propiedades a kebab-case.
- **UpperCamelCaseStrategy**: Convierte los nombres de las propiedades a Upper Camel Case.
- **LowerCamelCaseStrategy**: Convierte los nombres de las propiedades a lower camel case.

### Aplicación global

Si deseas aplicar una estrategia de nomenclatura a nivel global en lugar de solo a clases o campos individuales, puedes hacerlo configurando el `ObjectMapper`:

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.PropertyNamingStrategies;

ObjectMapper mapper = new ObjectMapper();
mapper.setPropertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE);

```
Con esta configuración, todos los objetos serializados y deserializados por este `ObjectMapper` seguirán la estrategia especificada, a menos que se especifique lo contrario mediante anotaciones en las clases o campos individuales.

Recuerda que estas estrategias son útiles para mantener la consistencia entre tus objetos Java y la representación JSON, especialmente cuando trabajas con APIs RESTful o almacenamiento de datos basado en JSON.

#learnspring
