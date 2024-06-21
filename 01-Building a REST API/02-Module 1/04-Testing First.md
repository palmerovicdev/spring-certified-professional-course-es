## What Is Test Driven Development?

Es común que los equipos de desarrollo de software creen conjuntos de pruebas automatizadas para protegerse contra las regresiones. A menudo, estas pruebas se escriben después de crear el código de las **features** de la aplicación. Adoptaremos un enfoque alternativo: escribiremos pruebas antes de implementar el código de la aplicación. Esto se llama desarrollo impulsado por pruebas (**TDD**).

¿Por qué aplicar **TDD**? Al afirmar el comportamiento esperado antes de implementar la funcionalidad deseada, estamos diseñando el sistema en función de lo que queremos que haga, en lugar de lo que el sistema ya hace.

Otro beneficio de **"test-driving"** es que las pruebas te guían para escribir el código mínimo necesario para satisfacer la implementación. Cuando las pruebas pasan, tienes una implementación que funciona (el código de la aplicación) y una protección contra la introducción de errores en el futuro (las pruebas).

¿No estás seguro de cómo implementar realmente **TDD**? No te preocupes, obtendrás mucha práctica de prueba de conducción de la aplicación **Family Cash Card** a lo largo de este curso.

## The Testing Pyramid

Se pueden escribir diferentes pruebas en diferentes niveles del sistema. En cada nivel, hay un equilibrio entre la velocidad de ejecución, el "costo" de mantener la prueba y la confianza que aporta a la corrección del sistema. Esta jerarquía a menudo se representa como una "pirámide de pruebas".
![[test-pyramid.jpg]]
