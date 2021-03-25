# R2DBC migration tool
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/name.nkonev.r2dbc-migrate/r2dbc-migrate-spring-boot-starter/badge.svg)](https://search.maven.org/search?q=g:name.nkonev.r2dbc-migrate%20AND%20a:r2dbc-migrate-spring-boot-starter)
[![Docker Image Version (latest semver)](https://img.shields.io/docker/v/nkonev/r2dbc-migrate)](https://hub.docker.com/r/nkonev/r2dbc-migrate/tags)
[![Build Status](https://github.com/nkonev/r2dbc-migrate/workflows/Java%20CI%20with%20Maven/badge.svg)](https://github.com/nkonev/r2dbc-migrate/actions)

Inspired by [this](https://spring.io/blog/2020/03/12/spring-boot-2-3-0-m3-available-now) announcement.

R2DBC [page](https://r2dbc.io/).

## Supported databases
* PostgreSQL
* Microsoft SQL Server
* MySQL
* H2
* MariaDB
* Oracle Database

It also supports user-provided dialect. You can pass implementation of `SqlQueries` interface to the `migrate()` method. If you use Spring Boot, just define a bean of type `SqlQueries`. Example [SimplePostgresqlDialect](https://github.com/nkonev/r2dbc-migrate/commit/86296acf0bbc6a7f4cbffe493cd2c3060d7885e2#diff-25735d05174bb55a45ca3d5986fc3ec1R369).

## Features
* Convention-based file names, for example `V3__insert_to_customers__split,nontransactional.sql`
* It waits until database have been started, there is test query, and validation result of. This can be useful to initial load data into database with docker-compose
* Supports migrations files larger than `-Xmx`: file will be splitted line-by-line (`split` modifier), then it will be loaded by chunks into database
* Lock support, that make you able to start number of replicas your microservice, without care of migrations will collide each other
* Each migration runs in the separated transaction
* It also supports `nontransactional` migrations, due to SQL Server prohibits `CREATE DATABASE` in the transaction
* Docker image
* First-class Spring Boot integration, see example below
* Also you can use this library without Spring (Boot) see library example below
* This library tends to be non-invasive, consequently it intentionally doesn't try to parse SQL and make some decisions relying on. So (in theory) you can freely update database and driver's version

All available configuration options are in [R2dbcMigrateProperties](https://github.com/nkonev/r2dbc-migrate/blob/master/r2dbc-migrate-core/src/main/java/name/nkonev/r2dbc/migrate/core/R2dbcMigrateProperties.java) class.
Their descriptions are available in your IDE Ctrl+Space help or in [spring-configuration-metadata.json](https://github.com/nkonev/r2dbc-migrate/blob/master/r2dbc-migrate-spring-boot-starter/src/main/resources/META-INF/spring-configuration-metadata.json) file.

## Download

### Docker
```
docker pull nkonev/r2dbc-migrate:latest
```

### Spring Boot Starter
```xml
<dependency>
  <groupId>name.nkonev.r2dbc-migrate</groupId>
  <artifactId>r2dbc-migrate-spring-boot-starter</artifactId>
  <version>VERSION</version>
</dependency>
```

### Only library
```xml
<dependency>
    <groupId>name.nkonev.r2dbc-migrate</groupId>
    <artifactId>r2dbc-migrate-core</artifactId>
    <version>VERSION</version>
</dependency>
```

If you use library, you need also use some implementation of `r2dbc-migrate-resource-reader-api`, for example:
```xml
<dependency>
    <groupId>name.nkonev.r2dbc-migrate</groupId>
    <artifactId>r2dbc-migrate-resource-reader-reflections</artifactId>
    <version>VERSION</version>
</dependency>
```
See `Library example` below.

### Standalone application

If you want to build your own docker image you will be able to do this
```bash
curl -Ss https://repo.maven.apache.org/maven2/name/nkonev/r2dbc-migrate/r2dbc-migrate-standalone/VERSION/r2dbc-migrate-standalone-VERSION.jar > /tmp/migrate.jar
```

## Spring Boot Example
https://github.com/nkonev/r2dbc-migrate-example

## Library example
https://github.com/nkonev/r2dbc-migrate-example/tree/library

## docker-compose v3 example
```yml
 migrate:
   image: nkonev/r2dbc-migrate:VERSION
   environment:
     _JAVA_OPTIONS: -Xmx128m
     spring.r2dbc.url: "r2dbc:pool:mssql://mssqlcontainer:1433"
     spring.r2dbc.username: sa
     spring.r2dbc.password: "yourSuperStrong(!)Password"
     r2dbc.migrate.resourcesPath: "file:/migrations/*.sql"
     r2dbc.migrate.validationQuery: "SELECT collation_name as result FROM sys.databases WHERE name = N'master'"
     r2dbc.migrate.validationQueryExpectedResultValue: "Cyrillic_General_CI_AS"
   depends_on:
     - mssqlcontainer
   volumes:
     - ./migrations:/migrations
```