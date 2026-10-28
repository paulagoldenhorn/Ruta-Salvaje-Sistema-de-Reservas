# **Ruta Salvaje – Sistema de reservas**

Sistema desarrollado en equipo como Proyecto Integrador de la carrera Certified Tech Developer (Digital House). 

Trabajo realizado siguiendo historias de usuario organizadas en sprints con Scrum.

El backend expone una API REST para una plataforma de reserva de paquetes de experiencias. Cubre la gestión del catálogo de productos, la autenticación de usuarios, las reservas con control de disponibilidad por fechas, las valoraciones y el envío de notificaciones por email.

[Video demostrativo del sitio web (YouTube)](https://youtu.be/Uv0zT3bsGfo?si=ejvpVPINYe7DpIsm)

## Estructura del repositorio
```
├── Proyecto-Integrador-Back/       # Proyecto Backend
├── Proyecto-Integrador-Front/      # Proyecto Frontend
├── Testing manual/                 # Planillas de casos de prueba
└── PRESENTACION_DEMO.pdf           # Presentación del proyecto para la Demo
```

## Tabla de contenidos

1. [El equipo](#el-equipo)
2. [Stack tecnológico](#stack-tecnológico)
3. [Funcionalidades](#funcionalidades)
4. [Modelo de datos](#modelo-de-datos)
5. [Endpoints](#endpoints)
6. [Seguridad y permisos](#seguridad-y-permisos)

## El Equipo

**Backend y Base de Datos**

- Paula Goldenhorn
- Tomás Davite
- Gabriel Olivos

**Frontend y UX/UI**

- Leandro Martinez
- Pablo Nicolás Merino
- Leonardo Cerda
- Magalí Orellana
- Matías Stewart Usher
- Milagros Castro

## Stack tecnológico

| Área | Tecnologías |
| --- | --- |
| Lenguaje y framework | Java 17, Spring Boot 2.7 |
| Persistencia | Spring Data JPA / Hibernate, MySQL |
| Seguridad | Spring Security, JWT |
| Notificaciones | JavaMail |
| Documentación | Springdoc OpenAPI (Swagger) |
| Utilidades | Lombok, Log4j, Maven |

## Funcionalidades

- **Autenticación y autorización:** registro e inicio de sesión con tokens JWT. Dos roles, `BASIC` y `ADMIN`, con permisos por endpoint (el catálogo se consulta sin login, las operaciones de gestión son solo para administradores y las reservas requieren usuario autenticado). Un administrador puede otorgar o quitar el rol a otros usuarios.
- **Catálogo:** CRUD de productos con nombre único, múltiples imágenes, categorías y características asociadas, y listado paginado.
- **Reservas:** creación con validación de disponibilidad por rango de fechas mediante una consulta SQL de superposición de períodos. Se rechaza la reserva si hay fechas ocupadas o si el producto no existe. También hay consulta de fechas reservadas por producto, búsqueda de productos disponibles en un rango, y historial de reservas del usuario autenticado.
- **Valoraciones:** puntuación de productos por usuarios autenticados, expuesta mediante DTOs.
- **Notificaciones por email:** correo automático al completar una reserva y confirmación de registro, con endpoint para reenviarla.
- **Manejo de errores:** excepciones propias (`BadRequestException`, `ResourceNotFoundException`) y un `@ControllerAdvice` global que devuelve respuestas HTTP coherentes (400 y 404).
- **Documentación:** endpoints documentados con Swagger/OpenAPI.

## **Modelo de datos**

Base de datos relacional **MySQL** con las tablas `users`, `product`, `category`, `feature`, `image`, `reservation` y `score`. Incluye claves foráneas, una relación N:M entre producto y característica, y restricciones de unicidad (nombre de producto, nombre de categoría y email de usuario).

## **Endpoints**

### Autenticación

| Método | Ruta | Descripción |
| --- | --- | --- |
| POST | `/auth/register` | Registra un usuario y devuelve un token JWT |
| POST | `/auth/authenticate` | Inicia sesión y devuelve un token JWT |
| POST | `/registerConfirm` | Reenvía el email de confirmación de registro |

### Usuarios

| Método | Ruta | Descripción |
| --- | --- | --- |
| GET | `/user` | Datos del usuario autenticado |
| GET | `/user/all` | Lista de usuarios (paginada) |
| PUT | `/user` | Actualiza los datos del usuario |
| PUT | `/user/limited` | Actualiza los datos del usuario (versión limitada) |
| PUT | `/user/role` | Actualiza el rol de un usuario |
| DELETE | `/user` | Elimina al usuario autenticado |

### Productos

| Método | Ruta | Descripción |
| --- | --- | --- |
| POST | `/product` | Crea un producto (nombre único) |
| GET | `/product/{id}` | Detalle de un producto |
| GET | `/product/all` | Lista de productos (paginada) |
| GET | `/product/reservedDates/{id}` | Fechas reservadas de un producto |
| GET | `/product/startDate/{startDate}/endDate/{endDate}` | Productos disponibles en un rango (`yyyy-MM-dd`) |
| PUT | `/product` | Actualiza un producto |
| DELETE | `/product/{id}` | Elimina un producto |

### Categorías y características

| Método | Ruta | Descripción |
| --- | --- | --- |
| POST | `/category` · `/feature` | Crea una categoría / característica |
| GET | `/category/all` · `/feature/all` | Lista todas |
| GET | `/category/{id}` · `/feature/{id}` | Detalle |
| PUT | `/category` · `/feature` | Actualiza |
| DELETE | `/category/{id}` · `/feature/{id}` | Elimina |

### Reservas

| Método | Ruta | Descripción |
| --- | --- | --- |
| POST | `/reservation` | Crea una reserva (valida disponibilidad y envía email) |
| GET | `/reservation/all` | Lista todas las reservas |
| GET | `/reservation/{id}` | Detalle de una reserva |
| GET | `/reservation/user` | Historial de reservas del usuario autenticado |
| GET | `/reservation/product/{productId}` | Reservas de un producto |
| DELETE | `/reservation/{id}` | Elimina una reserva |

### Valoraciones

| Método | Ruta | Descripción |
| --- | --- | --- |
| POST | `/score` | Puntúa un producto |
| GET | `/score/{id}` | Detalle de una valoración |
| GET | `/score/product/{product_id}` | Valoraciones de un producto |
| GET | `/score/all` | Lista todas las valoraciones |
| DELETE | `/score/{id}` | Elimina una valoración |

## Seguridad y permisos

La API es stateless: cada request protegido lleva el token en el header `Authorization: Bearer <token>`.

| Acceso | Rutas |
| --- | --- |
| **Público** | `/auth/**`, `/registerConfirm`, Swagger, y todos los `GET` de `/product`, `/category`, `/feature` y `/score` |
| **`ADMIN`** | `POST`, `PUT` y `DELETE` de `/product`, `/category` y `/feature`; `GET /user/all` |
| **`ADMIN` o `BASIC`** | `/user`, `/reservation` (GET, POST, DELETE) y `/score` (POST, DELETE) |

---

![image](https://github.com/paulagoldenhorn/Rutas-Salvajes/assets/101194303/689adc26-4aea-4686-81d2-b89abf49825a)
