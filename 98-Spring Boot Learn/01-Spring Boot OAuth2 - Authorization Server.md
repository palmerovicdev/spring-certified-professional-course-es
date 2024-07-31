# Getting Started

Si recién está comenzando con **Spring Authorization Server**, las siguientes secciones lo guiarán en la creación de su primera aplicación.

## System Requirements

**Spring Authorization Server** requiere un entorno de ejecución **Java 17** o superior.

## Installing Spring Authorization Server

**Spring Authorization Server** se puede utilizar en cualquier lugar donde ya se utilice [Spring Security](https://docs.spring.io/spring-security/reference/prequires.html).

La forma más sencilla de empezar a utilizar **Spring Authorization Server** es crear una aplicación basada en [Spring Boot](https://spring.io/projects/spring-boot). Puede utilizar [start.spring.io](https://start.spring.io/) para generar un proyecto básico o utilizar la [muestra de servidor de autorización predeterminado](https://github.com/spring-projects/spring-authorization-server/tree/main/samples/default-authorizationserver) como guía. Luego agregue el **starter** de **Spring Boot** para **Spring Authorization Server** como dependencia:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-authorization-server</artifactId>
</dependency>
```


> [!TIP]
> Consulte [Instalación de Spring Boot](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started.installing) para obtener más información sobre el uso de Spring Boot con Maven. o Gradle.

Alternativamente, puede agregar **Spring Authorization Server** sin **Spring Boot** usando el siguiente ejemplo:

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-oauth2-authorization-server</artifactId>
    <version>1.3.1</version>
</dependency>
```

## Developing Your First Application

Para comenzar, necesita los componentes mínimos requeridos definidos como `@Bean`. Cuando se utiliza la dependencia `spring-boot-starter-oauth2-authorization-server` , defina las siguientes propiedades y **Spring Boot** le proporcionará las definiciones `@Bean` necesarias:

**application.yml**
```yml
server:
  port: 9000

logging:
  level:
    org.springframework.security: trace

spring:
  security:
    user:
      name: user
      password: password
    oauth2:
      authorizationserver:
        client:
          oidc-client:
            registration:
              client-id: "oidc-client"
              client-secret: "{noop}secret"
              client-authentication-methods:
                - "client_secret_basic"
              authorization-grant-types:
                - "authorization_code"
                - "refresh_token"
              redirect-uris:
                - "http://127.0.0.1:8080/login/oauth2/code/oidc-client"
              post-logout-redirect-uris:
                - "http://127.0.0.1:8080/"
              scopes:
                - "openid"
                - "profile"
            require-authorization-consent: true
```


> [!TIP]
> Más allá de la introducción, la mayoría de los usuarios querrán personalizar la configuración predeterminada. La [siguiente sección](https://docs.spring.io/spring-authorization-server/reference/getting-started.html#defining-required-components) demuestra cómo proporcionar todos los beans necesarios usted mismo.

## Defining Required Components

Si desea personalizar la configuración predeterminada (independientemente de si está utilizando **Spring Boot**), puede definir los componentes mínimos requeridos como `@Bean` en una **Spring** `@Configuration`.

Estos componentes se pueden definir de la siguiente manera:

**SecurityConfig.java**
```java
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.util.UUID;

import com.nimbusds.jose.jwk.JWKSet;
import com.nimbusds.jose.jwk.RSAKey;
import com.nimbusds.jose.jwk.source.ImmutableJWKSet;
import com.nimbusds.jose.jwk.source.JWKSource;
import com.nimbusds.jose.proc.SecurityContext;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.Order;
import org.springframework.http.MediaType;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.oauth2.core.AuthorizationGrantType;
import org.springframework.security.oauth2.core.ClientAuthenticationMethod;
import org.springframework.security.oauth2.core.oidc.OidcScopes;
import org.springframework.security.oauth2.jwt.JwtDecoder;
import org.springframework.security.oauth2.server.authorization.client.InMemoryRegisteredClientRepository;
import org.springframework.security.oauth2.server.authorization.client.RegisteredClient;
import org.springframework.security.oauth2.server.authorization.client.RegisteredClientRepository;
import org.springframework.security.oauth2.server.authorization.config.annotation.web.configuration.OAuth2AuthorizationServerConfiguration;
import org.springframework.security.oauth2.server.authorization.config.annotation.web.configurers.OAuth2AuthorizationServerConfigurer;
import org.springframework.security.oauth2.server.authorization.settings.AuthorizationServerSettings;
import org.springframework.security.oauth2.server.authorization.settings.ClientSettings;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.LoginUrlAuthenticationEntryPoint;
import org.springframework.security.web.util.matcher.MediaTypeRequestMatcher;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

	@Bean (1)
	@Order(1)
	public SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http)
			throws Exception {
		OAuth2AuthorizationServerConfiguration.applyDefaultSecurity(http);
		http.getConfigurer(OAuth2AuthorizationServerConfigurer.class)
			.oidc(Customizer.withDefaults());	// Enable OpenID Connect 1.0
		http
			// Redirect to the login page when not authenticated from the
			// authorization endpoint
			.exceptionHandling((exceptions) -> exceptions
				.defaultAuthenticationEntryPointFor(
					new LoginUrlAuthenticationEntryPoint("/login"),
					new MediaTypeRequestMatcher(MediaType.TEXT_HTML)
				)
			)
			// Accept access tokens for User Info and/or Client Registration
			.oauth2ResourceServer((resourceServer) -> resourceServer
				.jwt(Customizer.withDefaults()));

		return http.build();
	}

	@Bean (2)
	@Order(2)
	public SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http)
			throws Exception {
		http
			.authorizeHttpRequests((authorize) -> authorize
				.anyRequest().authenticated()
			)
			// Form login handles the redirect to the login page from the
			// authorization server filter chain
			.formLogin(Customizer.withDefaults());

		return http.build();
	}

	@Bean (3)
	public UserDetailsService userDetailsService() {
		UserDetails userDetails = User.withDefaultPasswordEncoder()
				.username("user")
				.password("password")
				.roles("USER")
				.build();

		return new InMemoryUserDetailsManager(userDetails);
	}

	@Bean (4)
	public RegisteredClientRepository registeredClientRepository() {
		RegisteredClient oidcClient = RegisteredClient.withId(UUID.randomUUID().toString())
				.clientId("oidc-client")
				.clientSecret("{noop}secret")
				.clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
				.authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
				.authorizationGrantType(AuthorizationGrantType.REFRESH_TOKEN)
				.redirectUri("http://127.0.0.1:8080/login/oauth2/code/oidc-client")
				.postLogoutRedirectUri("http://127.0.0.1:8080/")
				.scope(OidcScopes.OPENID)
				.scope(OidcScopes.PROFILE)
				.clientSettings(ClientSettings.builder().requireAuthorizationConsent(true).build())
				.build();

		return new InMemoryRegisteredClientRepository(oidcClient);
	}

	@Bean (5)
	public JWKSource<SecurityContext> jwkSource() {
		KeyPair keyPair = generateRsaKey();
		RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
		RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();
		RSAKey rsaKey = new RSAKey.Builder(publicKey)
				.privateKey(privateKey)
				.keyID(UUID.randomUUID().toString())
				.build();
		JWKSet jwkSet = new JWKSet(rsaKey);
		return new ImmutableJWKSet<>(jwkSet);
	}

	private static KeyPair generateRsaKey() { (6)
		KeyPair keyPair;
		try {
			KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
			keyPairGenerator.initialize(2048);
			keyPair = keyPairGenerator.generateKeyPair();
		}
		catch (Exception ex) {
			throw new IllegalStateException(ex);
		}
		return keyPair;
	}

	@Bean (7)
	public JwtDecoder jwtDecoder(JWKSource<SecurityContext> jwkSource) {
		return OAuth2AuthorizationServerConfiguration.jwtDecoder(jwkSource);
	}

	@Bean (8)
	public AuthorizationServerSettings authorizationServerSettings() {
		return AuthorizationServerSettings.builder().build();
	}

}
```

Esta es una configuración mínima para comenzar rápidamente. Para comprender para qué se utiliza cada componente, consulte las siguientes descripciones:
1. Una cadena de filtros de Spring Security para los [endpoints de protocolo](https://docs.spring.io/spring-authorization-server/reference/protocol-endpoints.html).
2. Una cadena de filtros de Spring Security para [autenticación](https://docs.spring.io/spring-security/reference/servlet/authentication/index.html).
3. Una instancia de [`UserDetailsService`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/core/userdetails/UserDetailsService.html) para recuperar usuarios para autenticar.
4. Una instancia de [`RegisteredClientRepository`](https://docs.spring.io/spring-authorization-server/reference/core-model-components.html#registered-client-repository) para administrar clientes.
5. Una instancia de `com.nimbusds.jose.jwk.source.JWKSource` para firmar tokens de acceso.
7. Una instancia de `java.security.KeyPair` con claves generadas en el inicio utilizadas para crear el `JWKSource` de arriba.
8. Una instancia de [`JwtDecoder`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/oauth2/jwt/JwtDecoder.html) para decodificar firmado fichas de acceso.
9. Una instancia de [`AuthorizationServerSettings`](https://docs.spring.io/spring-authorization-server/reference/configuration-model.html#configuring-authorization-server-settings) para configurar **Spring Authorization Server**.