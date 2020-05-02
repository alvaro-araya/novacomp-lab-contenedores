# NOVACOMP
## LABORATORIO CONTENEDORES Y MICROSERVICIOS
#### Creado por Álvaro Araya O.<br/>aarayao [at]  novacompcr.com<br/>2020-05-20

## DESCRIPCIÓN

Este laboratorio realiza la implementación de una aplicación web basada en Spring Boot / Java, con seguridad provista por un servidor de autenticación por medio de un microservicio, incorpora un servidor Postgres, una aplicación de administración de Postgres pgAdmin, y el servidor Keycloak por medio de contenedores.

![Arquitectura de la aplicación](https://github.com/alvaro-araya/novacomp-lab-contenedores/blob/master/arquitectura.png?raw=true)

## PREPARACIÓN DEL AMBIENTE

#### Requisitos:

1. Debe instalar [Docker Desktop](https://www.docker.com/products/docker-desktop)
2. Debe instalar **docker-compose** siguiendo las instrucciones correspondientes a su sistema operativo [Install Docker Compose](https://docs.docker.com/compose/install/)
3. Debe contar con Java versión 11 [Instalar Java](https://adoptopenjdk.net)
4. Debe contar con Maven [Instalar Maven](https://maven.apache.org)

** Preferible desarrollar el laboratorio en un ambiente Linux o macOS

## ARCHIVO DOCKER COMPOSE

Crear el archivo de configuración de **docker-compose.yml** en un folder llamado **nova-containers**.

Código:
	
```yaml
# Creado por Alvaro Araya 2020-05-02
# Novacomp 2020
# Revisión 1.0

version: '3.7'

services:
	
  ## POSTGRESQL ##
  postgres:
    container_name: postgres_container_nova
    image: postgres
    environment:
      POSTGRES_USER: nova_root
      POSTGRES_PASSWORD: pwd_Werar7Oic4
      POSTGRES_DB: novadb
    ports:
      - "5432:5432"
    volumes:
      - 'postgres-data:/var/lib/postgresql/data/'
    networks:
      - nova-net
    restart: always

  ## PGADMIN ##
  pgadmin:
    container_name: pgadmin4_container_nova
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_DEFAULT_EMAIL:-usuario@novacompcr.com}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_DEFAULT_PASSWORD:-Werar7Oic4}
    volumes:
      - 'pgadmin-data:/root/.pgadmin'
    ports:
      - "5454:80"
    networks:
      - nova-net
    restart: always
    depends_on:
      - postgres

  ## KEYCLOAK ##
  keycloak:
    container_name: keycloak_container_nova
    image: quay.io/keycloak/keycloak:latest
    environment:
      DB_VENDOR: postgres
      DB_ADDR: postgres_container_nova
      DB_PORT: 5432
      DB_DATABASE: novadb
      DB_SCHEMA: public
      DB_USER: nova_root
      DB_PASSWORD: pwd_Werar7Oic4
      KEYCLOAK_USER: kc_admin
      KEYCLOAK_PASSWORD: kc_Werar7Oic4
      # JDBC_PARAMS: "ssl=true"
    ports:
      - '9090:8080'
    networks:
      - nova-net
    restart: always
    depends_on:
      - pgadmin

networks:
  nova-net:
    driver: bridge

volumes:
  postgres-data:
    driver: local
  pgadmin-data:
    driver: local
```   

Para iniciar los contenedores debe ejecutar el comando desde terminal en el directorio **nova-containers**. 

```
$ docker-compose up
```

### ACCEDER AL PGADMIN


Ingresar con los siguientes credenciales:

```
URL: 			http://localhost:5454
Usuario:		usuario@novacompcr.com
Clave:			Werar7Oic4
```

Registrar un nuevo servidor con estos datos:

```
Name:			postgres_container_nova
Host:			postgres_container_nova
Username:		nova_root
Password:		pwd_Werar7Oic4
Save:			Yes
```

### ACCEDER AL SERVIDOR DE KEYCLOAK

URL: **http://localhost:9090/auth/admin/master/console/#/realms/master**

Ingresar con los siguientes credenciales:

```
Usuario:		kc_admin
Clave:			kc_Werar7Oic4
```

Agregar un nuevo realm en keycloak:

```
Nuevo realm: 		novacomp
Display name:		Novacomp Acceso
```

Agregar un cliente:

```
Client ID: 		novaweb
Root URL:		http://localhost:8080
```

Agregar en los roles del realm:

```
admin
user
```

Agregar en los roles del cliente:

```
admin
user
```

Agregar un usuario:

```
Username:		usuario@novacompcr.com
Email:			usuario@novacompcr.com
First Name:		Usuario
Last Name:		Acceso
```

Cambiar los credenciales:

```
Password:	temporal123
Temporal: 	Sí
```

Agregar en Role Mappigns:

```
user
```

**Nota**: Luego se debe agregar el rol de admin para poder dar acceso al usuario al recurso de admin en la aplicación.

## APLICACION EN SPRING-BOOT

Iniciar con **https://start.spring.io** y descargar el proyecto:

```
Group:				cr.aao.keycloak
Artifact:			client
Java Version:			11
Description:			Lab Keycloak y Contenedores Novacomp
Tipo Proyecto:			Maven
```

Agregar al proyecto

1. Spring Web
1. Spring Security

Agregar en una sección nueva de **dependencyManagement** en el archivo **pom.xml**

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.keycloak.bom</groupId>
            <artifactId>keycloak-adapter-bom</artifactId>
            <version>10.0.0</version>
            <type>pom</type>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Agregar en la sección de **dependencies** en el archivo **pom.xml**

```xml
<!-- DEPENDENCIAS KEYCLOAK -->
<dependency>
    <groupId>org.keycloak</groupId>
    <artifactId>keycloak-spring-boot-2-adapter</artifactId>
    <version>10.0.0</version>
</dependency>
<dependency>
    <groupId>org.keycloak</groupId>
    <artifactId>keycloak-tomcat7-adapter</artifactId>
    <version>10.0.0</version>
</dependency>
```

Agregar en **/src/main/resources/static** los siguientes archivos:

Archivo: **index.html**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Lab Keycloack - Web Pública</title>
</head>
<body>
	<h1>INICIO</h1>
	<a href="home.html">Ingresar</a><br/>
</body>
</html>
```

Archivo: **home.html**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Lab Keycloack - Web Privada</title>
</head>
<body>
<h1>APLICACION</h1>
	<a href="admin/index.html">Administración del Sistema</a><br/>
	<a href="/logout">Salir</a>
</body>
</html>
```

Archivo **/admin/index.html**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Lab Keycloack - Web Privada / Rol Admin</title>
</head>
<body>
<h1>ADMINISTRACIÓN (ROL ADMIN)</h1>
	<a href="../home.html">Volver...</a>
</body>
</html>
```

Crear los siguientes clases de Java:

Clase: **KeycloakConfig**

```java
package cr.aao.keycloak.client;

import org.keycloak.adapters.springboot.KeycloakSpringBootConfigResolver;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class KeycloakConfig {

	@Bean
	public KeycloakSpringBootConfigResolver keycloakConfigResolver() {
		return new KeycloakSpringBootConfigResolver();
	}
}
```

Clase: **SecurityConfig**

```java
package cr.aao.keycloak.client;

import org.keycloak.adapters.springsecurity.KeycloakConfiguration;
import org.keycloak.adapters.springsecurity.authentication.KeycloakAuthenticationProvider;
import org.keycloak.adapters.springsecurity.config.KeycloakWebSecurityConfigurerAdapter;
import org.keycloak.adapters.springsecurity.management.HttpSessionManager;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.core.authority.mapping.SimpleAuthorityMapper;
import org.springframework.security.core.session.SessionRegistryImpl;
import org.springframework.security.web.authentication.session.RegisterSessionAuthenticationStrategy;
import org.springframework.security.web.authentication.session.SessionAuthenticationStrategy;

@KeycloakConfiguration
public class SecurityConfig extends KeycloakWebSecurityConfigurerAdapter {

	@Autowired
	public void configureGlobal(AuthenticationManagerBuilder auth) {
		SimpleAuthorityMapper grantedAuthorityMapper = new SimpleAuthorityMapper();
		grantedAuthorityMapper.setPrefix("ROLE_");
		KeycloakAuthenticationProvider keycloakAuthenticationProvider = keycloakAuthenticationProvider();
		keycloakAuthenticationProvider.setGrantedAuthoritiesMapper(grantedAuthorityMapper);
		auth.authenticationProvider(keycloakAuthenticationProvider);
	}

	@Bean
	@Override
	@ConditionalOnMissingBean(HttpSessionManager.class)
	protected HttpSessionManager httpSessionManager() {
		return new HttpSessionManager();
	}

	@Bean
	@Override
	protected SessionAuthenticationStrategy sessionAuthenticationStrategy() {
		return new RegisterSessionAuthenticationStrategy(new SessionRegistryImpl());
	}

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		super.configure(http);
		http
				.authorizeRequests()
				.antMatchers("/admin/*").hasAnyRole("admin")
				.antMatchers("/home.html").hasAnyRole("admin", "user")
				.anyRequest().permitAll();
	}
}
```

Clase: **IndexController**

```java
package cr.aao.keycloak.client.controller;

import org.keycloak.KeycloakSecurityContext;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;

@Controller
public class IndexController {

	private final HttpServletRequest request;

	@Autowired
	public IndexController(HttpServletRequest request) {
		this.request = request;
	}

	@GetMapping(value = "/logout")
	public String logout() throws ServletException {
		request.logout();
		return "redirect:/index.html";
	}

	private KeycloakSecurityContext getKeycloakSecurityContext() {
		return (KeycloakSecurityContext) request.getAttribute(KeycloakSecurityContext.class.getName());
	}
}
```

Agregar los datos al **application.properties** en **/src/main/resources**

```properties
keycloak.realm=novacomp
keycloak.resource=novaweb
keycloak.auth-server-url=http://localhost:9090/auth
keycloak.ssl-required=external
keycloak.public-client=true
```

## PRUEBAS:

1. Ingresar con los siguientes credenciales en la aplicación:

	```text
	URL:				http://localhost:8080
	Usuario:			usuario@novacompcr.com
	Clave:				temporal123  **debe cambiar la clave**
	```
2.	Trata de ingresar a la sección de Administración.
3. Agregar el Rol de admin para el usuario: **usuario@novacompcr.com**.

