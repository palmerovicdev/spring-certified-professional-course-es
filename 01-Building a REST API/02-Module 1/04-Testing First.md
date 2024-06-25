## What Is Test Driven Development?

Es común que los equipos de desarrollo de software creen conjuntos de pruebas automatizadas para protegerse contra las regresiones. A menudo, estas pruebas se escriben después de crear el código de las **features** de la aplicación. Adoptaremos un enfoque alternativo: escribiremos pruebas antes de implementar el código de la aplicación. Esto se llama desarrollo impulsado por pruebas (**TDD**).

¿Por qué aplicar **TDD**? Al afirmar el comportamiento esperado antes de implementar la funcionalidad deseada, estamos diseñando el sistema en función de lo que queremos que haga, en lugar de lo que el sistema ya hace.

Otro beneficio de **"test-driving"** es que las pruebas te guían para escribir el código mínimo necesario para satisfacer la implementación. Cuando las pruebas pasan, tienes una implementación que funciona (el código de la aplicación) y una protección contra la introducción de errores en el futuro (las pruebas).

¿No estás seguro de cómo implementar realmente **TDD**? No te preocupes, obtendrás mucha práctica de prueba de conducción de la aplicación **Family Cash Card** a lo largo de este curso.

## The Testing Pyramid

Se pueden escribir diferentes pruebas en diferentes niveles del sistema. En cada nivel, hay un equilibrio entre la velocidad de ejecución, el "costo" de mantener la prueba y la confianza que aporta a la corrección del sistema. Esta jerarquía a menudo se representa como una "pirámide de pruebas".

<img src="https://github.com/palmerovicdev/spring-certified-professional-course-es/blob/main/99-Assets/test-pyramid.jpg">

**Pruebas unitarias (Unit Tests)**: Una prueba unitaria ejerce una pequeña **"unidad"** del sistema que está aislada del resto del sistema. Deberían ser simples y rápidos. Quieres una alta proporción de pruebas unitarias en tu pirámide de pruebas, ya que son clave para diseñar un software altamente cohesivo y poco acoplado.

**Pruebas de integración (Integration Tests)**: Las pruebas de integración ejercen un subconjunto del sistema y pueden ejercer grupos de unidades en una sola prueba. Son más complicados de escribir y mantener, y se ejecutan más lentamente que las pruebas unitarias.

**Pruebas de extremo a extremo (End-to-End Tests)**: Una prueba de extremo a extremo ejerce el sistema utilizando la misma interfaz que un usuario, como un navegador web. Aunque son extremadamente exhaustivas, las pruebas de extremo a extremo pueden ser muy lentas y frágiles porque utilizan interacciones de usuario simuladas en interfaces de usuario potencialmente complicadas. Implemente el menor número de estas pruebas.

## The Red, Green, Refactor Loop

A los equipos de desarrollo de software les encanta moverse rápido. Entonces, ¿cómo vas rápido para siempre? Al mejorar y simplificar continuamente su refactorización de código. Una de las únicas formas en que puedes refactorizar de forma segura es cuando tienes un conjunto de pruebas de confianza. Por lo tanto, el mejor momento para refactorizar el código en el que te estás enfocando actualmente es durante el ciclo **TDD**. Esto se llama el bucle de desarrollo **red, green & refactor**:

1. **Rojo (Red)**: Escribe una prueba fallida para la funcionalidad deseada.

2. **Verde (Green)**: Implementa lo más simple que puede funcionar para aprobar la prueba.

3. **Refactorización (Refactoring)**: Busque oportunidades para simplificar, reducir la duplicación o mejorar el código sin cambiar ningún comportamiento, para refactorizar.

4. **¡Repite! (Repeat)**

A lo largo de los laboratorios de este curso, practicarás el bucle **Red, Green, Refactor** para desarrollar la **API REST** de **Family Cash Card**.

## Summary
El desarrollo impulsado por pruebas (**TDD**) es una técnica probada para ayudar a los desarrolladores de aplicaciones a diseñar un software simple pero robusto y protegerse contra las regresiones de la funcionalidad y los errores.

A continuación, implementará pruebas unitarias utilizando **TDD** para los contratos **JSON** que utilizaremos mientras desarrollamos nuestra **API** de **Family Cash Card**. ¿Ves? ¡Te dijimos que lo haríamos!
