# Ejemplo de aplicación con microservicios que implementa distintas arquitecturas de bases de datos

En este repo se encuentra el jdl para generar una aplicación orientada a microservicios con Jhipster.
La aplicación consiste en un gateway, dos microservicios llamados pozos y acciones, un registry y un servidor de base de datos.

El microservicio pozos cuenta con tres entidades principales
* Pozo
* Nota
* CambioestadoExplotacion

El microservicio acciones cuenta con dos entidades principales
* Accion
* cambioEstadoGestion

El siguiente diagrama muestra las relaciones entre las entidades


![Diagrama de relaciones entre las entidades](https://raw.githubusercontent.com/mari967/jhipster-microservicio-bd-imp/main/application.png)

## 0. Requisitos
Se debe tener instalado lo siguiente:
1. Docker
2. Docker compose
3. Jhipster

## 1. Generar el proyecto
1. Luego de clonar el repo, ejecutar en el directorio raíz\
`jhipster jdl application.jdl`
2. Una vez generado el proyecto, editar los archivos de configuración application-prod.yml de cada servicio.

Para el gateway
```
  liquibase:
    contexts: prod
    url: jdbc:postgresql://localhost:5432/gateway #Base de datos por servicio
    #url: jdbc:postgresql://localhost:5432/shared-database-example #Base de datos compartida
    #url: jdbc:postgresql://localhost:5432/db-per-service-example?currentSchema=gateway #Base de datos por servicio. Esquemas diferentes
  mail:
    host: localhost
    port: 25
    username:
    password:
  r2dbc:
    url: r2dbc:postgresql://localhost:5432/gateway #Base de datos por servicio
    #url: r2dbc:postgresql://localhost:5432/shared-database-example #Base de datos compartida
    #url: r2dbc:postgresql://localhost:5432/db-per-service-example?currentSchema=gateway #Base de datos por servicio. Esquemas diferentes
    username: admin
    password: admin
```

Para el microservicio pozos
```
  datasource:
    type: com.zaxxer.hikari.HikariDataSource
    url: jdbc:postgresql://localhost:5432/pozos #Base de datos por servicio
    #url: jdbc:postgresql://localhost:5432/shared-database-example #Base de datos compartida
    #url: jdbc:postgresql://localhost:5432/db-per-service-example?currentSchema=pozos #Base de datos por servicio. Esquemas diferentes
    username: admin
    password: admin
```

Para el microservicio acciones
```
  datasource:
    type: com.zaxxer.hikari.HikariDataSource
    url: jdbc:postgresql://localhost:5432/acciones #Base de datos por servicio #Base de datos por servicio
    #url: jdbc:postgresql://localhost:5432/shared-database-example #Base de datos compartida    
    #url: jdbc:postgresql://localhost:5432/db-per-service-example?currentSchema=acciones #Base de datos por servicio. Esquemas diferentes
    username: admin
    password: admin
```
3. Editar en cada microservicio y el gateway el archivo *src/main/resources/config/liquibase/changelog/00000000000000_initial_schema.xml* reemplazando el atributo *author="jhipster"* por *author="jhipster_<nombre_servicio>"*. Esto es para evitar conflictos al usar la base de datos compartida, porque los changesets (archivos de liquibase) tienen el mismo id y autor, con cambios diferentes

## 2. Levantar el registry
1. Levantar el servicio de docker con el siguiente comando \
`dockerd`
2. Ir a la capreta del gateway y ejecutar\
`docker compose -f src/main/docker/jhipster-registry.yml up`


## 3. Levantar la base de datos postgres con docker
1. Descargar la imagen de postgres\
`docker pull postgres`
2. Levantar el contenedor\
`docker run --name postgresql -e POSTGRES_USER=admin -e POSTGRES_PASSWORD=admin -p 5432:5432 -v /data:/var/lib/postgresql/data -d postgres`
3. Conectarse a la base de datos con un gestor como DBeaver. En database dejar en blanco. El usuario es *admin* y la contraseña *admin*
4. En DBeaver ejecutar el siguiente script
```sql

CREATE DATABASE "shared-database-example";
CREATE DATABASE gateway;
CREATE DATABASE pozos;
CREATE DATABASE acciones;
CREATE DATABASE "db-per-service-example";

```

5. Conectarse a la base de datos *db-per-service-example* y crear los esquemas con el siguiente script
```sql
CREATE SCHEMA gateway;
CREATE SCHEMA pozos;
CREATE SCHEMA acciones;
```
NOTA: Para hacer rollback, ejecutar este script
```sql
-- Rollback
DROP DATABASE "shared-database-example";
DROP DATABASE "db-per-service-example";
DROP DATABASE gateway;
DROP DATABASE pozos;
DROP DATABASE acciones;
```

## 4. Levantar la aplicación
1. En las carpetas de los microservicios y el gateway ejecutar este comando\
`./mvnw -Pprod`
## 5. Cambiando la arquitectura de base de datos de la aplicación
Por defecto, la aplicación que genera jhipster usa una arquitectura de base de datos en la que cada microservicio tiene una propia. Es por esto que fue necesario crear las de cada aplicación en un paso anterior.

Para utilizar otras arquitecturas, se deben modificar los archivos applicaction-prod.yml de cada aplicación, como indica a continuación, según la arquitectura a implementar. 

### Base de datos compartida

1. Descomentar esta línea en los archivos applicaction-prod.yml de cada aplicación.\
`url: jdbc:postgresql://localhost:5432/shared-database-example`
3. Descomentar esta línea en el archivo applicaction-prod.yml del gateway\
`url: jdbc:postgresql://localhost:5432/shared-database-example `\
NOTA: La aplicación utiliza la base de datos *shared-database-example*

### Base de datos por microservicio con esquemas por servicio
1. Descomentar esta línea en los archivos applicaction-prod.yml de cada aplicación.\
`url: jdbc:postgresql://localhost:5432/db-per-service-example?currentSchema=<nombre_esquema>`
2. Descomentar esta línea en el archivo applicaction-prod.yml del gateway\
`url: jdbc:postgresql://localhost:5432/db-per-service-example?currentSchema=gateway `\
NOTA: La aplicación utiliza la base de datos *db-per-service-example* y los esquemas *pozos, acciones y gateway*

### Base de datos por servicio

1. Descomentar esta línea en los archivos applicaction-prod.yml de cada aplicación.\
`url: jdbc:postgresql://localhost:5432/<nombre_aplicación>`
2. Descomentar esta línea en el archivo applicaction-prod.yml del gateway\
`url: jdbc:postgresql://localhost:5432/gateway `\
NOTA: La aplicación utiliza las bases de datos *acciones, pozos y gateway*
