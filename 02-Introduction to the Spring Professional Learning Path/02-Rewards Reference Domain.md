## Purpose

Los laboratorios de este curso enseñan conceptos clave en el contexto de una problemática de dominio. La aplicación **Reward Dining** proporciona un contexto del mundo real para aplicar las técnicas que ha aprendido para desarrollar aplicaciones comerciales útiles.
Esta sección proporciona una descripción general del **dominio** y las **aplicaciones** en las que trabajará dentro de él.

## Domain Overview

El dominio se llama **Reward Dining**. La idea es que los clientes puedan ahorrar dinero cada vez que coman en uno de los restaurantes que participan en la red, un programa de **"viajero frecuente"** para restaurantes.
Por ejemplo, a **Keith** le gustaría **ahorrar dinero** para la educación de sus hijos. Cada vez que cena en un restaurante miembro de la red, se realiza una aportación a su cuenta.

<img src="https://github.com/palmerovicdev/spring-certified-professional-course-es/blob/1d9566c064143b674f5049674ce5fff2a5ac802b/99-Assets/keith-dining.png" alt="Reward Dining Domain" width="800"/>

**Figure 1**: Papa **Keith** cena en un restaurante de **Reward Network**.

La contribución de la cuenta (recompensa) será compartida por sus dos hijos **Annabelle** y **Corgan**. Así, Annabelle obtiene un fondo para ayudarla con los gastos de la universidad:

<img src="https://raw.githubusercontent.com/vmware-tanzu-learning/spring-academy-assets/main/courses/course-spring-professional/annabelle.png" 
alt="Annabelle" width="400"/>

**Figure 2**: La hija **Annabelle** recibe contribuciones a su fondo universitario

## Reward Dining Domain ApplicationsReward Dining Domain Applications

La siguiente sección proporciona una descripción general de las aplicaciones en el dominio de comidas premiadas en las que trabajará en este curso.

### The Rewards ApplicationThe Rewards Application

La aplicación **"rewards"** recompensa una cuenta por cenar en un restaurante que participa en la red de recompensas. Una recompensa toma la forma de una 
contribución monetaria a una cuenta que se distribuye entre los beneficiarios de la cuenta. Así es como se utiliza esta aplicación:
1. Cuando tienen hambre, los miembros cenan en los restaurantes participantes utilizando sus tarjetas de crédito habituales.
2. Cada dos semanas se genera un archivo que contiene las transacciones con tarjetas de crédito de comedor realizadas por los socios durante ese 
   período. A continuación se muestra un ejemplo de uno de estos archivos: