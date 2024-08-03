En este tutorial, desarrollaremos una aplicación Spring que se integre con OAuth2 para la autenticación mediante GitHub y Google.

## Dependencies of the project
Primeramente, debemos agregar las siguientes dependencias en el proyecto.
Agregado al archivo `pom.xml`:

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-oauth2-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <scope>annotationProcessor</scope>
        </dependency>
    </dependencies>
```

Breve explicación del uso de cada dependencia:

- `spring-boot-starter-oauth2-client`: Facilita la integración de clientes **OAuth2** en aplicaciones **Spring Boot**, permitiendo la autenticación y 
autorización mediante proveedores **OAuth2** como **Google**, **Facebook**, etc.

- `spring-boot-starter-security`: Proporciona soporte básico para seguridad en aplicaciones **Spring Boot**, incluyendo autenticación, protección contra 
ataques **CSRF**, y configuración de seguridad personalizada.

- `spring-boot-starter-web`: Esencial para desarrollar aplicaciones web, incluye **Spring MVC** y **Tomcat** como servidor embebido, facilitando la creación 
de servicios **RESTful**.

- `spring-boot-devtools`: Mejora el desarrollo proporcionando recarga automática de código, acceso a herramientas de desarrollo live, y otras 
características útiles durante el desarrollo.

- `lombok`: Reduce la boilerplate en el código **Java** generando automáticamente getters, setters, constructores, y más, mejorando la legibilidad y 
mantenibilidad del código.

- `spring-boot-starter-test`: Contiene todas las dependencias necesarias para realizar pruebas unitarias y de integración en aplicaciones **Spring Boot**,
incluyendo **JUnit**, **Mockito**, y **Spring Test**.

- `spring-security-test`: Extiende las capacidades de prueba de **Spring Boot** para incluir soporte específico para **Spring Security**, facilitando la 
escritura de pruebas de seguridad.

- `spring-boot-configuration-processor`: Genera metadatos para la configuración de **Spring Boot**, ayudando a **IDEs** y editores de texto a ofrecer 
autocompletado y validación para propiedades de configuración.

## Creating endpoints to test

Debemos crear dos endpoints que nos sirvan de guia para saber si hemos implementado correctamente la autenticación por **OAuth2**.

Crea un archivo llamado HomeController, que contenga el siguiente código:

```java
package com.palmerodev.springbootoauth2client.controller;  
  
import lombok.RequiredArgsConstructor;  
import org.springframework.web.bind.annotation.GetMapping;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RestController;  
  
@RestController  
@RequestMapping  
@RequiredArgsConstructor  
public class HomeController {  
  
    @GetMapping("/hello")  
    private String hello(){  
        return "Hello World!";  
    }  
    @GetMapping("/helloSecured")  
    private String helloSecured(){  
        return "Hello World! (Secured)";  
    }  
}
```

## Application.properties configuration

```properties
spring.application.name=spring-boot-oauth2-client  
server.port=8080  
server.address=localhost  
  
# OAuth test users  
spring.security.user.name=victor  
spring.security.user.password=victor  
spring.security.user.roles=USER  
  
# Github OAuth2 Client  
spring.security.oauth2.client.registration.github.client-id=${GITHUB_CLIENT_ID}  
spring.security.oauth2.client.registration.github.client-secret=${GITHUB_CLIENT_SECRET}  
  
# Google OAuth2 Client  
spring.security.oauth2.client.registration.google.client-id=${GOOGLE_CLIENT_ID}  
spring.security.oauth2.client.registration.google.client-secret=${GOOGLE_CLIENT_SECRET}
```

## SecurityConfig class

```java
package com.palmerodev.springbootoauth2client.config;  
  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
import org.springframework.http.HttpMethod;  
import org.springframework.security.config.Customizer;  
import org.springframework.security.config.annotation.web.builders.HttpSecurity;  
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;  
import org.springframework.security.web.SecurityFilterChain;  
  
@Configuration  
@EnableWebSecurity  
public class SecurityConfig {  
  
    @Bean  
    public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity) throws Exception {  
        return httpSecurity  
                .authorizeHttpRequests(requests -> {  
                    requests.requestMatchers(HttpMethod.GET, "/", "/hello").permitAll();  
                    requests.anyRequest().authenticated();  
                })                .formLogin(Customizer.withDefaults())  
                .oauth2Login(Customizer.withDefaults())  
                .build();  
    }}
```

## TaskScheduler class

```java
package com.palmerodev.springbootoauth2client.config;  
  
import lombok.RequiredArgsConstructor;  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.boot.context.event.ApplicationReadyEvent;  
import org.springframework.context.event.EventListener;  
import org.springframework.core.env.Environment;  
import org.springframework.stereotype.Component;  
  
@Slf4j  
@Component  
@RequiredArgsConstructor  
public class TaskScheduleConfig {  
    private final Environment environment;  
  
    @EventListener(ApplicationReadyEvent.class)  
    private void showUrl() {  
        var baseUrl = "http://" + environment.getProperty("server.address") + ":" + environment.getProperty("server.port") + "/";  
        log.info("Application is ready. Your application will be available at the following URL: {}", baseUrl);  
    }  
}
```