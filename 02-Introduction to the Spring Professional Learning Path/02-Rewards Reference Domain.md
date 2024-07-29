## Purpose

Los laboratorios de este curso enseñan conceptos clave en el contexto de una problemática de dominio. La aplicación **Reward Dining** proporciona un contexto del mundo real para aplicar las técnicas que ha aprendido para desarrollar aplicaciones comerciales útiles.
Esta sección proporciona una descripción general del **dominio** y las **aplicaciones** en las que trabajará dentro de él.

## Domain Overview

El dominio se llama **Reward Dining**. La idea es que los clientes puedan ahorrar dinero cada vez que coman en uno de los restaurantes que participan en la red, un programa de **"viajero frecuente"** para restaurantes.
Por ejemplo, a **Keith** le gustaría **ahorrar dinero** para la educación de sus hijos. Cada vez que cena en un restaurante miembro de la red, se realiza una aportación a su cuenta.

<img src="https://raw.githubusercontent.com/vmware-tanzu-learning/spring-academy-assets/main/courses/course-spring-professional/keith-dining.png" alt="Reward Dining Domain" width="800"/>

**Figure 1**: Papa **Keith** cena en un restaurante de **Reward Network**.

La contribución de la cuenta (recompensa) será compartida por sus dos hijos **Annabelle** y **Corgan**. Así, Annabelle obtiene un fondo para ayudarla con los gastos de la universidad:

<img src="https://raw.githubusercontent.com/vmware-tanzu-learning/spring-academy-assets/main/courses/course-spring-professional/annabelle.png" 
alt="Annabelle" width="400"/>

**Figure 2**: La hija **Annabelle** recibe contribuciones a su fondo universitario

## Reward Dining Domain ApplicationsReward Dining Domain Applications

La siguiente secc ión proporciona una descripción general de las aplicaciones en el dominio de comidas premiadas en las que trabajará en este curso.

### The Rewards ApplicationThe Rewards Application

La aplicación **"rewards"** recompensa una cuenta por cenar en un restaurante que participa en la red de recompensas. Una recompensa toma la forma de una 
contribución monetaria a una cuenta que se distribuye entre los beneficiarios de la cuenta. Así es como se utiliza esta aplicación:
1. Cuando tienen hambre, los miembros cenan en los restaurantes participantes utilizando sus tarjetas de crédito habituales.
2. Cada dos semanas se genera un archivo que contiene las transacciones con tarjetas de crédito de comedor realizadas por los socios durante ese 
   período. A continuación se muestra un ejemplo de uno de estos archivos:

| AMOUNT | CREDIT_CARD_NUMBER | MERCHANT_NUMBER | DATE       |
|--------|--------------------|-----------------|------------|
| 100.00 | 1234123412341234   | 1234567890      | 12/29/2010 |
| 49.67  | 1234123412341234   | 0234567891      | 12/31/2010 |
| 100.00 | 1234123412341234   | 1234567890      | 01/01/2010 |
| 27.60  | 2345234523452345   | 3456789012      | 01/02/2010 |


**Figure 3**: Ejemplos de registros de comidas
3. Una aplicación `DiningBatchProcessor` independiente lee este archivo y envía cada registro de **Comida** a la solicitud de recompensas para su procesamiento.

## Public Application InterfacePublic Application Interface

`RewardNetwork` es la interfaz central que utilizan los clientes, como `DiningBatchProcessor`, para invocar la aplicación:

```java
    public interface RewardNetwork {
        RewardConfirmation rewardAccountFor(Dining dining);
    }
```

**Figure 4**: El servicio de `RewardNetwork`

Es el único componente de la capa de servicio de la aplicación.
`RewardNetwork` recompensa una cuenta por cenar haciendo una contribución monetaria a la cuenta que a su vez se distribuye entre los beneficiarios 
de la cuenta. El siguiente diagrama de secuencia muestra la interacción de un cliente con la aplicación que ilustra este proceso:

<img src="https://raw.githubusercontent.com/vmware-tanzu-learning/spring-academy-assets/main/courses/course-spring-professional/rewards
-application.png" alt="Rewards Application" width="800"/>

**Figure 5**: Un cliente llama a `RewardNetwork` para recompensar una cuenta por cenar


En este ejemplo, la cuenta con tarjeta de crédito `1234123412341234` recibe una recompensa por gastar `$100,00` en un restaurante con `ID` de comerciante `1234567890` el **29 de diciembre de 2010** (la fecha está en formato de fecha norteamericana).

La recompensa confirmada, número `9831`, toma la forma de una contribución de cuenta de `$8,00` distribuida equitativamente entre los beneficiarios **Annabelle** y su hermano **Corgan**.
## Implementación de aplicaciones internas

Internamente, la implementación de `RewardNetwork` delega a objetos de dominio la realización de una transacción de `rewardAccountFor(Dining)`. Existen clases para los dos conceptos de dominio centrales de la aplicación: **Cuenta** y **Restaurante**.

- Un **Restaurante** es responsable de calcular el beneficio elegible para una cuenta para una cena.
- Una **Cuenta** es responsable de distribuir el beneficio entre sus beneficiarios como una **"contribución"**.
Este flujo se muestra a continuación:

<img src="https://raw.githubusercontent.com/vmware-tanzu-learning/spring-academy-assets/main/courses/course-spring-professional/rewardnetwork
-domainobject-interaction.png" alt="Reward Network Domain Object Interaction" width="800"/>

**Figure 6**: Objetos trabajando juntos para llevar a cabo el caso de uso de `rewardAccountFor(Dining)`

`RewardNetwork` le pide al **Restaurante** que calcule cuánto beneficio otorgar y luego aporta esa cantidad a la **Cuenta**.
Para mayor comodidad, el proyecto común de recompensas contiene varias clases auxiliares:

- MonetaryAmount envuelve un `BigDecimal` y mantiene una cantidad con 2 decimales.
- El porcentaje envuelve un `BigDecimal` y mantiene un porcentaje con 2 decimales.

## Supporting Reward Network Components

La información de la cuenta y del restaurante se almacena de forma persistente dentro de una base de datos relacional. La implementación de `RewardNetwork` delega los componentes de soporte de acceso a datos llamados **'Repositorios'** para cargar objetos de **Cuenta** y **Restaurante** desde sus representaciones relacionales.
- Un `AccountRepository` se utiliza para encontrar una cuenta por su número de tarjeta de crédito.
- Un `RestaurantRepository` se utiliza para encontrar un restaurante por su número de comerciante.
- Un `RewardRepository` se utiliza para realizar un seguimiento de las transacciones de recompensas confirmadas con fines contables. Contiene los 
  mismos datos que la **Confirmación** de recompensa en el diagrama (Figure 5) anterior.

Un **registro de comidas** contiene:
- Número de tarjeta de crédito del cliente
- Identificación del comerciante del restaurante.
- Cantidad pagada por la comida
- Fecha en que cenó el cliente (no utilizado en nuestra implementación simple)

La secuencia completa de `rewardAccountFor(Dining)` que incorpora estos repositorios es:
- Obtener la cuenta del `AccountRepository`
- Obtener el restaurante desde `RestaurantRepository`
- Determine la contribución de recompensa (una instancia de `MonetaryAmount`) usando `Restaurant.calculateBenefitFor(Account, Dining)`
- Actualice los beneficiarios de la cuenta usando `Account.makeContribution(MonetaryAmount)`
- Guarde la información de la cuenta modificada usando `AccountRepository.updateBeneficiaries(Account)`
- Cree una `RewardConfirmation` utilizando `RewardRepository`

## Reward Dining Database Schema

Las aplicaciones de `Reward` `Dining` utilizan una base de datos con este esquema:

<img src="https://raw.githubusercontent.com/vmware-tanzu-learning/spring-academy-assets/main/courses/course-spring-professional/rewardDining-databaseSchema.png" alt="Reward Dining Database Schema" width="800"/>

**Figure 7**: El esquema de la base de datos de Reward Dining


En la mayoría de los laboratorios se le proporciona una base de datos de pruebas. Se completa con datos de prueba ejecutando scripts en `00-rewards-common/src/main/resources/rewards/testdb`. Están disponibles como recursos de classpath.

Hay dos guiones:
- `schema.sql`: crea las tablas necesarias y
- `data.sql`: agrega datos de prueba (varias cuentas y un solo restaurante)

## Summary

En la lección, aprendió sobre un dominio problemático de `Rewards` `Network` existente que se usará en la ruta de aprendizaje de **Spring Professinal**. En la lección, aprendió sobre un dominio problemático de `Rewards` `Network` existente que se usará en la ruta de aprendizaje de **Spring Professinal**.