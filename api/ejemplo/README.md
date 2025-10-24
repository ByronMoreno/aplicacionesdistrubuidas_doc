# Clase: Crear una API REST con Spring Boot y Gradle (H2 para desarrollo, PostgreSQL en producción)

> **Formato:** README profesional para presentar en clase, con instrucciones claras, ejemplos de código y comandos listos para copiar.

---

## :mortar_board: Resumen de la sesión


**Objetivo de la clase:**
- Entender las diferencias entre **Maven** y **Gradle**.
- Crear un proyecto Spring Boot con **Gradle**.
- Configurar y usar **H2** como base de datos en desarrollo y **PostgreSQL** en producción.
- Implementar un CRUD completo (Entity → Repository → Service → Controller).
- Ejecutar y probar la API (curl / Postman / H2 Console).

**Audiencia:** Estudiantes con conocimientos básicos de Java y HTTP.

---

## Índice

1. [Teoría rápida: Maven vs Gradle](#teoría-rápida-maven-vs-gradle)
2. [Estructura del proyecto y dependencias](#estructura-del-proyecto-y-dependencias)
3. [Archivo `build.gradle` (ejemplo)](#archivo-buildgradle-ejemplo)
4. [Configuración por perfiles (`application.yml`)](#configuración-por-perfiles-applicationyml)
5. [Implementación: ejemplo de dominio `Paciente` (Entity → Repository → Service → Controller)](#implementación-ejemplo-de-dominio-paciente)
6. [Datos iniciales (`data.sql`)](#datos-iniciales-datasql)
7. [Ejecución y pruebas](#ejecución-y-pruebas)
8. [Buenas prácticas y recomendaciones docentes](#buenas-prácticas-y-recomendaciones-docentes)
9. [Ejercicios y tareas para los estudiantes](#ejercicios-y-tareas-para-los-estudiantes)
10. [Recursos](#recursos)

---

## Teoría rápida: Maven vs Gradle

- **Maven**
  - Declarativo (XML `pom.xml`).
  - Opiniones y convenciones fuertes: estructura estándar.
  - Fácil de entender para proyectos convencionales.

- **Gradle**
  - DSL (Groovy o Kotlin): `build.gradle` / `build.gradle.kts`.
  - Más rápido por ejecución incremental y cache.
  - Muy flexible y extensible.

**Decisión para la clase:** Usamos **Gradle** por ser el más usado en nuevos proyectos y por su rendimiento.

---

## Estructura mínima del proyecto

```
mi-api-springboot/
├─ src/
│  ├─ main/
│  │  ├─ java/ec/unib/api/
│  │  │  ├─ controller/
│  │  │  ├─ model/
│  │  │  ├─ repository/
│  │  │  └─ service/
│  │  └─ resources/
│  │     ├─ application.yml
│  │     └─ data.sql
├─ build.gradle
└─ gradlew (wrapper)
```

---

## Archivo `build.gradle` (Groovy DSL) — ejemplo

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.0'
    id 'io.spring.dependency-management' version '1.1.0'
}

group = 'ec.unib'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '17'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'com.h2database:h2'
    runtimeOnly 'org.postgresql:postgresql'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

> **Nota docente:** Menciona la ventaja del *Gradle Wrapper* (`./gradlew`) porque garantiza reproducibilidad independientemente de la versión instalada en el equipo del alumno.

---

## Configuración por perfiles: `application.yml`

```yaml
spring:
  profiles:
    active: dev

---

spring:
  config:
    activate:
      on-profile: dev
  datasource:
    url: jdbc:h2:mem:db;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
    driver-class-name: org.h2.Driver
    username: sa
    password:
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
  h2:
    console:
      enabled: true
      path: /h2-console

---

spring:
  config:
    activate:
      on-profile: prod
  datasource:
    url: jdbc:postgresql://localhost:5432/mi_db
    driver-class-name: org.postgresql.Driver
    username: tu_usuario
    password: tu_password
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
```

**Puntos clave para explicar en clase:**
- `dev` usa **H2 en memoria** y `ddl-auto: update` para iterar rápido durante el desarrollo.
- `prod` usa **PostgreSQL** y `ddl-auto: validate` (o usar migraciones con Flyway/Liquibase).
- Cómo cambiar de perfil: `--spring.profiles.active=prod` al ejecutar la aplicación.

---

## Implementación: ejemplo de dominio `Paciente`

### 1) Entity — `Paciente.java`

```java
package ec.unib.api.model;

import jakarta.persistence.*;
import lombok.*;

@Entity
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Paciente {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nombre;
    private String apellido;
    private String email;
}
```

**Explicación:** usa `Lombok` para reducir boilerplate. Comentar que se puede enseñar sin Lombok para principiantes.

### 2) Repository — `PacienteRepository.java`

```java
package ec.unib.api.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import ec.unib.api.model.Paciente;

public interface PacienteRepository extends JpaRepository<Paciente, Long> {
}
```

### 3) Service — interfaz y implementación

**Interfaz:**
```java
package ec.unib.api.service;

import java.util.List;
import java.util.Optional;
import ec.unib.api.model.Paciente;

public interface PacienteService {
    Paciente crear(Paciente paciente);
    Optional<Paciente> obtenerPorId(Long id);
    List<Paciente> obtenerTodos();
    Paciente actualizar(Long id, Paciente paciente);
    void eliminar(Long id);
}
```

**Implementación:**
```java
package ec.unib.api.service.impl;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.List;
import java.util.Optional;
import ec.unib.api.model.Paciente;
import ec.unib.api.repository.PacienteRepository;
import ec.unib.api.service.PacienteService;

@Service
@Transactional
public class PacienteServiceImpl implements PacienteService {

    private final PacienteRepository repo;

    public PacienteServiceImpl(PacienteRepository repo) {
        this.repo = repo;
    }

    @Override
    public Paciente crear(Paciente paciente) {
        return repo.save(paciente);
    }

    @Override
    public Optional<Paciente> obtenerPorId(Long id) {
        return repo.findById(id);
    }

    @Override
    public List<Paciente> obtenerTodos() {
        return repo.findAll();
    }

    @Override
    public Paciente actualizar(Long id, Paciente paciente) {
        return repo.findById(id).map(p -> {
            p.setNombre(paciente.getNombre());
            p.setApellido(paciente.getApellido());
            p.setEmail(paciente.getEmail());
            return repo.save(p);
        }).orElseThrow(() -> new RuntimeException("Paciente no encontrado"));
    }

    @Override
    public void eliminar(Long id) {
        repo.deleteById(id);
    }
}
```

### 4) Controller — `PacienteController.java`

```java
package ec.unib.api.controller;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import java.net.URI;
import java.util.List;

import ec.unib.api.model.Paciente;
import ec.unib.api.service.PacienteService;

@RestController
@RequestMapping("/api/pacientes")
public class PacienteController {

    private final PacienteService service;

    public PacienteController(PacienteService service) {
        this.service = service;
    }

    @GetMapping
    public List<Paciente> all() {
        return service.obtenerTodos();
    }

    @GetMapping("/{id}")
    public ResponseEntity<Paciente> getById(@PathVariable Long id) {
        return service.obtenerPorId(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    public ResponseEntity<Paciente> create(@RequestBody Paciente paciente) {
        Paciente creado = service.crear(paciente);
        return ResponseEntity.created(URI.create("/api/pacientes/" + creado.getId())).body(creado);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Paciente> update(@PathVariable Long id, @RequestBody Paciente paciente) {
        try {
            Paciente actualizado = service.actualizar(id, paciente);
            return ResponseEntity.ok(actualizado);
        } catch (RuntimeException e) {
            return ResponseEntity.notFound().build();
        }
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        service.eliminar(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

## Datos iniciales (opcional): `data.sql`

```sql
INSERT INTO paciente (nombre, apellido, email) VALUES ('Juan','Pérez','juan@ejemplo.com');
INSERT INTO paciente (nombre, apellido, email) VALUES ('María','López','maria@ejemplo.com');
```

> Colocar `data.sql` en `src/main/resources` hará que Spring Boot lo ejecute al arrancar si `ddl-auto` crea las tablas.

---

## Ejecución y pruebas

### Comandos útiles
- Ejecutar con Gradle Wrapper (desarrolladores):
  - Linux/macOS: `./gradlew bootRun`
  - Windows: `gradlew.bat bootRun`
- Generar jar: `./gradlew clean build` → `java -jar build/libs/mi-api-0.0.1-SNAPSHOT.jar`

### Cambiar de perfil
- Ejecutar con perfil `prod` (Postgres):
  ```bash
  ./gradlew bootRun --args='--spring.profiles.active=prod'
  # o
  java -jar build/libs/mi-api-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod
  ```

### Probar endpoints (curl)
- Obtener todos:
  ```bash
  curl http://localhost:8080/api/pacientes
  ```
- Crear:
  ```bash
  curl -X POST -H "Content-Type: application/json" -d '{"nombre":"Ana","apellido":"Gómez","email":"ana@ej.com"}' http://localhost:8080/api/pacientes
  ```
- H2 Console: abrir en el navegador `http://localhost:8080/h2-console`, URL: `jdbc:h2:mem:db`, user: `sa`.

---

## Buenas prácticas y recomendaciones para la evaluación

- **Separar capas**: Controller / Service / Repository.
- **DTOs y validación**: Enseñar `jakarta.validation` (`@NotNull`, `@Email`) y usar `@Valid` en los controllers.
- **Manejo centralizado de errores**: `@ControllerAdvice` para respuestas consistentes.
- **Versionado de API**: `/api/v1/...`.
- **Migraciones para producción**: usar Flyway o Liquibase en vez de `ddl-auto`.
- **Pruebas**: agregar pruebas unitarias (Mockito) y pruebas de integración con H2.
- **Documentación**: integrar Swagger/OpenAPI para mostrar los endpoints.

---

## Ejercicios y tareas sugeridas

1. Agregar validaciones a `Paciente` (email obligatorio y formato válido). Implementar pruebas unitarias que validen el comportamiento.
2. Implementar paginación en `GET /api/pacientes` usando `Pageable`.
3. Añadir búsqueda por nombre (ej: `/api/pacientes?nombre=juan`) mediante query method en `PacienteRepository`.
4. Configurar Flyway y crear una migración SQL para la tabla `paciente`.
5. Escribir pruebas de integración que arranquen el contexto usando H2.
6. (Avanzado) Añadir autenticación básica (Spring Security) y proteger los endpoints.

---

## Recursos y lecturas recomendadas

- Spring Boot Reference Guide
- Gradle User Manual
- H2 Database Engine — Console and usage
- PostgreSQL official docs
- Flyway / Liquibase documentation

---

## Cierre

Este README está diseñado para presentarlo en clase y compartirlo con los estudiantes. Si quieres, puedo:

- Generar el proyecto completo (archivos fuente + `build.gradle`) y empaquetarlo en un ZIP para descargar.
- Añadir ejemplos de pruebas unitarias y de integración.
- Incluir ejemplos sin Lombok (para clases donde no se use Lombok en el curso).

