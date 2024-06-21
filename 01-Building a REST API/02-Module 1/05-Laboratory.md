# Writing a Failing Test
Aquí cubriremos una breve introducción a la biblioteca de pruebas de **JUnit** y la herramienta de construcción de **Gradle**. También utilizaremos el enfoque **Test-First** para crear software.

Las clases de prueba en un proyecto **Java** estándar están en el directorio `src/test`, no en `src/main`. En nuestro caso, hemos decidido poner nuestro código en el paquete `example.cashcard`, por lo que nuestros archivos de prueba deben estar en el directorio `src/test/java/example/cashcard`.

1. Crea la clase `CashCardJsonTest`.
Lo primero que tenemos que hacer es crear nuestra nueva clase de prueba en el directorio `src/test/java/example/cashcard`.
Puede hacer esto utilizando el Editor de la siguiente manera:
- Abra la sección **JAVA PROJECTS** en la parte inferior izquierda de la página.
- Abra el directorio `src/test/java`.
- Dentro de `src/test/java`, seleccione el paquete `example.cashcard` y haga clic en el signo `+` a la derecha.
- Aparecerá un cuadro de diálogo preguntando por el nuevo nombre de la clase. Introduzca `CashCardJsonTest.java`.

Aquí hay una captura de pantalla que muestra los pasos anteriores:
![[create-test-class-in-package.jpg.png]]

2. Edite el nuevo archivo resultante para que contenga el siguiente contenido:
```java
package example.cashcard;

import org.junit.jupiter.api.Test;
import static org.assertj.core.api.Assertions.assertThat;

class CashCardJsonTest {

   @Test
   void myFirstTest() {
      assertThat(1).isEqualTo(42);
   }
}
```

Tómese un momento para entender el código de prueba: la anotación **@Test** es parte de la biblioteca **JUnit**, y el método **assertThat** es parte de la biblioteca **AssertJ**. Ambas bibliotecas se importan después de la declaración del paquete.

Una convención común (pero no un requisito) es usar siempre el sufijo de prueba para las clases de prueba. Lo hemos hecho aquí. El nombre completo de la clase **CashCardJsonTest** te da una pista sobre la naturaleza de la prueba que estamos a punto de escribir.

De la verdadera manera de **Test-First**, hemos escrito primero una prueba fallida. Es importante tener una prueba fallida primero para que pueda tener una gran confianza en que todo lo que hizo para arreglar la prueba realmente funcionó.

No te preocupes por que la prueba (assert that 1 isEqualsTo 42), así como el nombre del método de prueba, parezca extraño. Estamos a punto de cambiarlos.

3. Ejecute la prueba desde la línea de comandos de su terminal (asegure de estar primero en el directorio de `exercices`):
```bash
[~/exercises] $ ./gradlew test
```
Deberías recibir una salida como esta (hemos omitido algunas de las salidas menos importantes):
```shell
> Task :test

CashCardApplicationTests > contextLoads() PASSED

CashCardJsonTest > myFirstTest() FAILED
    org.opentest4j.AssertionFailedError:
    expected: 42
     but was: 1
...
        at app//example.cashcard.CashCardJsonTest.myFirstTest(CashCardJsonTest.java:11)

2 tests completed, 1 failed
```
Este es el resultado esperado de la herramienta de compilación de **Gradle** cuando tienes una prueba fallida. En este caso, su nueva prueba falló, mientras que la prueba de aplicación de **CashCards** existente de la lección anterior tuvo éxito.

La información de fallo pertinente está en la parte superior de la salida:
```shell
expected: 42
 but was: 1
```

Podrías haber esperado esto, ya que el número 1 no es igual al número 42.

4. Para "arreglar" la prueba, puedes crear una afirmación que sabes que es cierta:
```java
assertThat(42).isEqualTo(42
```

5. Ahora ejecuta la prueba de nuevo. ¡Pasa!
```shell
[~/exercises] $ ./gradlew test

> Task :test

CashCardJsTest > myFirstTest() PASSED

CashCardApplicationTests > contextLoads() PASSED

BUILD SUCCESSFUL in 4s
```

¡Enhorabuena! Ha completado con éxito una iteración de desarrollo de prueba primero: escriba una prueba fallida y luego corrija el código para que la prueba pase. Ahora está listo para continuar con el uso de la metodología **Test-First** para escribir la **API REST** de **CashCard**.