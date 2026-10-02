# MusicalRent API

## Plan maestro del proyecto

Sistema académico para gestionar clientes, instrumentos musicales, reservas, prestamos, devoluciones y condiciones fisicas.

Este documento es la memoria principal del proyecto. Debe conservarse aunque se cierre el chat o se cambie de asistente.

---

## 1. Integrantes

- Steven Eraso Insuasty
- Sebastian Manchabajoy Rosero
- Hector Alejandro Riascos Insuasty

---

## 2. Idea del proyecto

MusicalRent sera una API REST para una tienda de alquiler de instrumentos musicales, principalmente guitarras.

El sistema permitira:

- Registrar clientes.
- Registrar instrumentos.
- Identificar el tipo concreto de instrumento.
- Consultar disponibilidad.
- Crear reservas.
- Confirmar y cancelar reservas.
- Crear prestamos a partir de reservas confirmadas.
- Registrar el estado fisico del instrumento al entregarlo.
- Registrar el estado fisico al devolverlo.
- Mantener historial de condiciones.
- Consultar prestamos activos y atrasados.
- Actualizar instrumentos parcialmente sin sobrescribir datos accidentalmente.
- Evitar reservas que se crucen en fechas.

El sistema debe permitir al dueño conocer:

- Que instrumentos existen.
- En que estado de disponibilidad se encuentran.
- En que condiciones fisicas se encuentran.
- Quien tiene cada instrumento.
- Cuando fue entregado.
- Cuando debe ser devuelto.
- Que ocurrio con el instrumento antes y despues del prestamo.

---

## 3. Objetivo academico

Aplicar en un solo proyecto los siguientes conceptos:

- Java 17.
- Programacion orientada a objetos.
- Encapsulamiento.
- Abstraccion.
- Herencia.
- Polimorfismo.
- Clases abstractas.
- Enums.
- Records.
- Objetos de valor.
- Reglas de negocio.
- Arquitectura por capas.
- Spring Boot.
- API REST.
- DTOs de entrada y salida.
- MapStruct.
- Spring Data JPA.
- Hibernate.
- PostgreSQL.
- Herencia JPA con `JOINED`.
- Relaciones entre entidades.
- Transacciones.
- Validacion.
- Manejo global de errores.
- Proyecciones anidadas sin recursion.
- Actualizacion parcial segura con `PATCH`.
- Pruebas unitarias.
- Pruebas de integracion.
- Documentacion tecnica.
- Trabajo colaborativo con Git.

---

## 4. Alcance del MVP

La primera version debe incluir unicamente lo necesario para demostrar el dominio del problema y los temas vistos en clase.

### Incluido

1. Clientes.
2. Instrumentos.
3. Guitarras acusticas.
4. Guitarras electricas.
5. Reservas.
6. Prestamos.
7. Devoluciones.
8. Condiciones fisicas.
9. Historial de condiciones.
10. Actualizacion parcial de instrumentos.
11. Filtros basicos de instrumentos.
12. Validacion y errores.
13. Pruebas automatizadas.
14. Documentacion y ejemplos de uso.

### No incluido inicialmente

- Pagos en linea.
- Usuarios y autenticacion.
- Roles avanzados.
- Facturacion.
- Notificaciones por correo.
- Aplicacion movil.
- Integracion con proveedores externos.
- Microservicios.
- Event sourcing.
- Inteligencia artificial.

Estas funcionalidades solo se evaluaran despues de terminar y probar el MVP.

---

## 5. Arquitectura

Se utilizara una arquitectura por capas:

```text
Cliente HTTP
    |
    v
Controller
    |
    v
Service
    |
    v
Repository
    |
    v
PostgreSQL
```

Los DTOs y mappers participan en los limites de la aplicacion:

```text
JSON de entrada
    |
    v
Request DTO
    |
    v
Controller
    |
    v
Service
    |
    v
Mapper
    |
    v
Entidad de dominio
    |
    v
Repository
```

Para una respuesta:

```text
Entidad de dominio
    |
    v
Mapper
    |
    v
Response DTO
    |
    v
JSON de salida
```

### Responsabilidades

#### Controller

- Recibir solicitudes HTTP.
- Convertir parametros y JSON en DTOs.
- Delegar en el servicio.
- Devolver codigos HTTP y respuestas.
- No contener reglas complejas de negocio.
- No acceder directamente al repositorio.

#### Service

- Representar casos de uso.
- Coordinar entidades y repositorios.
- Ejecutar reglas de aplicacion.
- Controlar transacciones.
- Lanzar excepciones de negocio.

#### Domain

- Representar el negocio.
- Proteger invariantes.
- Contener estados y comportamientos.
- Evitar setters publicos innecesarios.

#### Repository

- Consultar y guardar en la base de datos.
- No contener reglas de negocio complejas.

#### Mapper

- Convertir request DTO a entidad.
- Convertir entidad a response DTO.
- Resolver el polimorfismo.
- Evitar asignaciones repetitivas.

#### DTO

- Definir el contrato de la API.
- Evitar exponer directamente entidades JPA.
- Controlar que datos entran y salen.

---

## 6. Estructura de paquetes

```text
src/main/java/com/musicalrent
├── MusicalRentApplication.java
├── controller
├── service
├── repository
├── domain
├── dto
│   ├── request
│   └── response
├── mapper
├── exception
├── config
└── specification
```

### Proposito de cada paquete

- `domain`: entidades, enums, objetos de valor y comportamiento.
- `dto/request`: datos recibidos por la API.
- `dto/response`: datos devueltos por la API.
- `mapper`: conversion entre DTOs y entidades.
- `repository`: persistencia mediante Spring Data JPA.
- `service`: casos de uso y transacciones.
- `controller`: endpoints HTTP.
- `exception`: excepciones y manejador global.
- `config`: configuraciones de Spring.
- `specification`: filtros dinamicos si llegan a ser necesarios.

---

## 7. Modelo de dominio

### 7.1 Cliente

Clase: `Cliente`

Atributos:

- `UUID id`.
- `String nombre`.
- `String numeroIdentificacion`.
- `String celular`.
- `String email`.
- `boolean activo`.

Reglas:

- El nombre es obligatorio.
- La identificacion es obligatoria y unica.
- El celular es obligatorio.
- El email debe tener formato valido si se solicita.
- Un cliente inactivo no puede crear reservas.
- No se debe eliminar fisicamente un cliente con historial.

### 7.2 Instrumento abstracto

Clase: `Instrumento`

Debe ser abstracta y usar herencia JPA con `JOINED`.

Atributos comunes:

- `UUID id`.
- `String codigoInventario`.
- `String marca`.
- `String modelo`.
- `BigDecimal precioDiario`.
- `EstadoDisponibilidad estadoDisponibilidad`.
- `CondicionFisica condicionActual`.
- `LocalDateTime fechaRegistro`.
- `Long version` para concurrencia optimista.

Reglas:

- El codigo de inventario es obligatorio y unico.
- Marca y modelo son obligatorios.
- El precio debe ser positivo o cero segun la politica definida.
- Un instrumento en reparacion no puede reservarse.
- Un instrumento alquilado no puede volver a alquilarse.
- Las transiciones deben hacerse mediante metodos de dominio.

### 7.3 Guitarra acustica

Clase: `GuitarraAcustica extends Instrumento`

Atributos propios:

- `int numeroCuerdas`.
- `String tipoMadera`.
- `boolean tienePastilla`.

### 7.4 Guitarra electrica

Clase: `GuitarraElectrica extends Instrumento`

Atributos propios:

- `int numeroCuerdas`.
- `String tipoCuerpo`.
- `int numeroPastillas`.

### 7.5 Periodo de alquiler

Clase: `PeriodoAlquiler`

Puede ser un `record` y un `@Embeddable`.

Atributos:

- `LocalDateTime fechaInicio`.
- `LocalDateTime fechaDevolucion`.

Reglas:

- Ninguna fecha puede ser nula.
- La fecha de devolucion no puede ser anterior a la fecha de inicio.
- Debe poder calcular los dias del periodo.

### 7.6 Reserva

Clase: `Reserva`

Atributos:

- `UUID id`.
- `Cliente cliente`.
- `Instrumento instrumento`.
- `PeriodoAlquiler periodo`.
- `EstadoReserva estado`.
- `LocalDateTime fechaCreacion`.

Estados:

```text
PENDIENTE
CONFIRMADA
CANCELADA
FINALIZADA
```

Reglas:

- Debe existir cliente.
- Debe existir instrumento.
- El cliente debe estar activo.
- Las fechas deben ser validas.
- No debe existir otra reserva confirmada que se cruce.
- Una reserva cancelada no puede confirmarse.
- Al confirmar, el instrumento pasa a `RESERVADO`.

### 7.7 Prestamo

Clase: `Prestamo`

Atributos:

- `UUID id`.
- `Reserva reserva`.
- `LocalDateTime fechaEntregaReal`.
- `LocalDateTime fechaDevolucionReal`.
- `CondicionFisica condicionAlEntregar`.
- `CondicionFisica condicionAlDevolver`.
- `String observacionesEntrega`.
- `String observacionesDevolucion`.
- `EstadoPrestamo estado`.

Estados:

```text
ACTIVO
DEVUELTO
ATRASADO
```

Reglas:

- Solo una reserva confirmada puede generar un prestamo.
- Al entregar, el instrumento pasa a `ALQUILADO`.
- Un prestamo activo tiene fecha de entrega.
- Al devolver, se registra fecha y condicion.
- Un prestamo devuelto no puede devolverse otra vez.
- Si el instrumento vuelve danado, puede pasar a `EN_REPARACION`.

### 7.8 Registro de condicion

Clase: `RegistroCondicion`

Atributos:

- `UUID id`.
- `Instrumento instrumento`.
- `CondicionFisica condicion`.
- `String observaciones`.
- `LocalDateTime fechaRegistro`.
- `MomentoCondicion momento`.

Momentos:

```text
REGISTRO_INICIAL
ENTREGA
DEVOLUCION
INSPECCION
REPARACION
```

El historial debe ser acumulativo. No se debe borrar el estado anterior cuando cambia la condicion.

---

## 8. Enums

### TipoInstrumento

```text
GUITARRA_ACUSTICA
GUITARRA_ELECTRICA
```

### EstadoDisponibilidad

```text
DISPONIBLE
RESERVADO
ALQUILADO
EN_REPARACION
```

### CondicionFisica

```text
NUEVO
USADO
RAYADO
DANADO
```

### EstadoReserva

```text
PENDIENTE
CONFIRMADA
CANCELADA
FINALIZADA
```

### EstadoPrestamo

```text
ACTIVO
DEVUELTO
ATRASADO
```

### MomentoCondicion

```text
REGISTRO_INICIAL
ENTREGA
DEVOLUCION
INSPECCION
REPARACION
```

---

## 9. Polimorfismo

El cliente debe indicar el tipo concreto del instrumento.

Ejemplo de guitarra acustica:

```json
{
  "tipo": "GUITARRA_ACUSTICA",
  "codigoInventario": "GTR-001",
  "marca": "Yamaha",
  "modelo": "FG800",
  "precioDiario": 25000,
  "numeroCuerdas": 6,
  "tipoMadera": "ABETO",
  "tienePastilla": false
}
```

Ejemplo de guitarra electrica:

```json
{
  "tipo": "GUITARRA_ELECTRICA",
  "codigoInventario": "GTR-002",
  "marca": "Ibanez",
  "modelo": "GRX70",
  "precioDiario": 40000,
  "numeroCuerdas": 6,
  "tipoCuerpo": "SOLIDO",
  "numeroPastillas": 2
}
```

El mapper debe interpretar `tipo` y crear la clase correcta:

```text
GUITARRA_ACUSTICA -> GuitarraAcustica
GUITARRA_ELECTRICA -> GuitarraElectrica
```

No se deben crear controladores separados para cada subtipo.

---

## 10. DTOs

### Requests

- `CrearClienteRequest`.
- `ActualizarClienteRequest`.
- `CrearInstrumentoRequest`.
- `ActualizarInstrumentoRequest`.
- `CrearReservaRequest`.
- `CrearPrestamoRequest`.
- `DevolverPrestamoRequest`.

### Responses

- `ClienteResponse`.
- `InstrumentoResponse`.
- `ReservaResponse`.
- `ReservaDetalleResponse`.
- `PrestamoResponse`.
- `ClienteResumenResponse`.
- `InstrumentoResumenResponse`.
- `HistorialCondicionResponse`.

No se deben devolver entidades JPA directamente desde los controllers.

---

## 11. Proyecciones anidadas sin recursion

Una respuesta detallada de reserva puede incluir resumen de cliente e instrumento:

```json
{
  "id": "uuid",
  "estado": "CONFIRMADA",
  "cliente": {
    "id": "uuid",
    "nombre": "Laura Gomez",
    "numeroIdentificacion": "123456"
  },
  "instrumento": {
    "id": "uuid",
    "codigoInventario": "GTR-001",
    "tipo": "GUITARRA_ACUSTICA",
    "marca": "Yamaha",
    "modelo": "FG800"
  },
  "fechaInicio": "2026-10-10",
  "fechaDevolucion": "2026-10-15"
}
```

Reglas:

- `ClienteResumenResponse` no contiene reservas.
- `InstrumentoResumenResponse` no contiene prestamos.
- No serializar entidades JPA con relaciones completas.
- Evitar ciclos `Cliente -> Reserva -> Cliente`.
- Definir claramente la profundidad de cada respuesta.

---

## 12. Actualizacion parcial segura

Endpoint:

```text
PATCH /api/instrumentos/{id}
```

Ejemplo:

```json
{
  "precioDiario": 30000,
  "condicionActual": "RAYADO"
}
```

Los campos no enviados deben conservarse.

El request de actualizacion debe usar campos anulables:

- `String marca`.
- `String modelo`.
- `BigDecimal precioDiario`.
- `EstadoDisponibilidad estadoDisponibilidad`.
- `CondicionFisica condicionActual`.

No se deben permitir desde el PATCH:

- `id`.
- `tipo`.
- `codigoInventario`.
- `fechaRegistro`.
- `version`.
- Historial.

MapStruct debe configurarse con:

```java
NullValuePropertyMappingStrategy.IGNORE
```

Si cambia la condicion fisica, se debe crear un nuevo `RegistroCondicion`.

---

## 13. Endpoints

### Clientes

```text
POST   /api/clientes
GET    /api/clientes
GET    /api/clientes/{id}
PATCH  /api/clientes/{id}
```

### Instrumentos

```text
POST   /api/instrumentos
GET    /api/instrumentos
GET    /api/instrumentos/{id}
PATCH  /api/instrumentos/{id}
PATCH  /api/instrumentos/{id}/estado
GET    /api/instrumentos/{id}/historial-condiciones
```

Filtros:

```text
GET /api/instrumentos?estado=DISPONIBLE
GET /api/instrumentos?tipo=GUITARRA_ELECTRICA
GET /api/instrumentos?condicion=RAYADO
```

### Reservas

```text
POST   /api/reservas
GET    /api/reservas
GET    /api/reservas/{id}
POST   /api/reservas/{id}/confirmar
POST   /api/reservas/{id}/cancelar
```

### Prestamos

```text
POST   /api/prestamos
GET    /api/prestamos
GET    /api/prestamos/{id}
POST   /api/prestamos/{id}/devolver
GET    /api/prestamos/activos
GET    /api/prestamos/atrasados
```

---

## 14. Codigos HTTP

- `200 OK`: consulta u operacion exitosa.
- `201 Created`: recurso creado.
- `204 No Content`: operacion exitosa sin cuerpo.
- `400 Bad Request`: datos invalidos.
- `404 Not Found`: recurso inexistente.
- `409 Conflict`: duplicados o conflicto de disponibilidad.
- `500 Internal Server Error`: error inesperado.

Al crear recursos, devolver tambien la cabecera `Location`.

---

## 15. Reglas de negocio

1. No registrar instrumentos sin codigo.
2. No permitir codigos duplicados.
3. No permitir identificaciones duplicadas.
4. No permitir precios negativos.
5. No permitir clientes inactivos.
6. No reservar instrumentos en reparacion.
7. No reservar instrumentos alquilados en un periodo incompatible.
8. Validar siempre las fechas.
9. No permitir reservas cruzadas.
10. No confirmar reservas canceladas.
11. No crear prestamos sin reservas confirmadas.
12. No crear dos prestamos activos para el mismo instrumento.
13. Cambiar el instrumento a alquilado al entregar.
14. Cambiarlo a disponible o reparacion al devolver.
15. Registrar condicion de entrega.
16. Registrar condicion de devolucion.
17. Mantener historial de condiciones.
18. No modificar el historial desde un PATCH.
19. Usar metodos de dominio para cambiar estados.
20. No exponer entidades completas en JSON.

---

## 16. Validacion

Usar Bean Validation:

- `@NotBlank`.
- `@NotNull`.
- `@Email`.
- `@Positive`.
- `@Size`.
- `@Pattern` cuando sea necesario.
- `@Valid` en los controllers.

La validacion HTTP no reemplaza la validacion del dominio. Las entidades tambien deben proteger sus invariantes.

---

## 17. Excepciones

Crear excepciones especificas:

- `ClienteNoEncontradoException`.
- `InstrumentoNoEncontradoException`.
- `ReservaNoEncontradaException`.
- `PrestamoNoEncontradoException`.
- `ReglaDeNegocioException`.
- `RecursoDuplicadoException`.
- `InstrumentoNoDisponibleException`.

Crear un `@RestControllerAdvice` que devuelva:

```json
{
  "timestamp": "2026-10-02T10:00:00",
  "status": 409,
  "error": "CONFLICT",
  "message": "El instrumento ya esta reservado para ese periodo",
  "path": "/api/reservas"
}
```

---

## 18. Persistencia

Usar:

- `@Entity`.
- `@Table`.
- `@Id`.
- `@Column`.
- `@Enumerated(EnumType.STRING)`.
- `@OneToMany`.
- `@ManyToOne`.
- `@Embedded`.
- `@Inheritance(strategy = InheritanceType.JOINED)`.
- `@Version`.

Recomendaciones:

- Usar `BigDecimal` para dinero.
- Usar `LAZY` cuando corresponda.
- No usar cascadas indiscriminadamente.
- No exponer relaciones JPA directamente.
- No depender de `ddl-auto=update` en produccion.
- Considerar Flyway o Liquibase en una fase posterior.
- Guardar credenciales mediante variables de entorno.

---

## 19. Patrones y principios

### Patrones presentes

- Arquitectura por capas.
- MVC adaptado a REST.
- Repository Pattern.
- Service Layer.
- Data Mapper.
- Dependency Injection.
- Domain Model.
- Value Object.
- Unit of Work implicito de Hibernate.
- Identity Map implicito de Hibernate.

### Principios

- Responsabilidad unica.
- Bajo acoplamiento.
- Alta cohesion.
- Encapsulamiento.
- Abstraccion.
- Polimorfismo.
- Inversion de dependencias.
- Sustitucion de Liskov.
- Fail fast.
- Separacion de responsabilidades.
- Inmutabilidad en DTOs cuando sea apropiado.

### No agregar inicialmente

- Microservicios.
- CQRS.
- Event Sourcing.
- Pagos.
- Seguridad avanzada.
- Arquitectura distribuida.
- Patrones que no resuelvan un problema real del proyecto.

---

## 20. Plan de trabajo por fases

### Fase 1: analisis

Entregables:

- Problema.
- Actores.
- Reglas de negocio.
- Diagrama de clases.
- Diagrama entidad-relacion.
- Lista de endpoints.
- Alcance y limitaciones.

### Fase 2: configuracion

- Crear proyecto Spring Boot.
- Configurar Java 17.
- Configurar Maven.
- Configurar PostgreSQL.
- Configurar MapStruct.
- Configurar Git.
- Crear README.

### Fase 3: dominio

Orden:

1. Enums.
2. `PeriodoAlquiler`.
3. `Cliente`.
4. `Instrumento` abstracto.
5. `GuitarraAcustica`.
6. `GuitarraElectrica`.
7. `Reserva`.
8. `Prestamo`.
9. `RegistroCondicion`.

Antes de pasar a Spring, probar las reglas del dominio.

### Fase 4: persistencia

- Agregar anotaciones JPA.
- Crear tablas y relaciones.
- Configurar herencia `JOINED`.
- Crear repositorios.
- Revisar restricciones.
- Probar consultas.

### Fase 5: DTOs y mappers

- Crear requests.
- Crear responses.
- Crear respuestas resumidas.
- Crear mapper polimorfico.
- Crear mapper de actualizacion parcial.
- Revisar warnings de MapStruct.

### Fase 6: servicios

Implementar casos de uso:

- Crear cliente.
- Crear instrumento.
- Consultar instrumentos.
- Crear reserva.
- Confirmar reserva.
- Cancelar reserva.
- Crear prestamo.
- Devolver instrumento.
- Consultar historial.
- Actualizar instrumento.

### Fase 7: controllers

Crear endpoints y asignar codigos HTTP correctos.

### Fase 8: errores y validacion

- Bean Validation.
- Excepciones especificas.
- `@RestControllerAdvice`.
- Respuesta de error uniforme.

### Fase 9: pruebas

- Pruebas de dominio.
- Pruebas de mappers.
- Pruebas de servicios.
- Pruebas de repositorios.
- Pruebas de controllers.
- Pruebas de integracion.

### Fase 10: documentacion

- README.
- Guia de instalacion.
- Variables de entorno.
- Diagramas.
- Coleccion de Postman.
- Ejemplos JSON.
- Decisiones tecnicas.
- Division del trabajo.
- Limitaciones.

---

## 21. Distribucion sugerida del equipo

### Steven: instrumentos y polimorfismo

- `Instrumento`.
- `GuitarraAcustica`.
- `GuitarraElectrica`.
- Enums de instrumentos.
- DTO polimorfico.
- Mapper polimorfico.
- Repositorio de instrumentos.
- Controller de instrumentos.

### Sebastian: clientes y reservas

- `Cliente`.
- `Reserva`.
- `PeriodoAlquiler`.
- Reglas de fechas.
- Verificacion de reservas cruzadas.
- DTOs de reserva.
- Proyecciones anidadas.
- Controller de reservas.

### Hector: prestamos, historial y calidad

- `Prestamo`.
- `RegistroCondicion`.
- Flujo de entrega y devolucion.
- PATCH seguro.
- Mapper de actualizacion.
- Excepciones.
- Pruebas de integracion.
- Documentacion final.

### Responsabilidad compartida

- Revisar el modelo.
- Revisar nombres.
- Integrar ramas.
- Ejecutar pruebas.
- Preparar la presentacion.
- Poder explicar todo el sistema, no solo el modulo propio.

---

## 22. Pruebas obligatorias

1. Crear cliente valido.
2. Rechazar cliente invalido.
3. Registrar guitarra acustica.
4. Registrar guitarra electrica.
5. Verificar que se conserva el subtipo.
6. Rechazar precio negativo.
7. Rechazar codigo duplicado.
8. Crear reserva valida.
9. Rechazar fechas invalidas.
10. Rechazar reserva cruzada.
11. Confirmar reserva.
12. Cambiar instrumento a reservado.
13. Crear prestamo.
14. Cambiar instrumento a alquilado.
15. Devolver instrumento en buen estado.
16. Devolver instrumento danado.
17. Cambiar instrumento a reparacion.
18. Consultar historial.
19. Actualizar parcialmente instrumento.
20. Comprobar que null no sobrescribe datos.
21. Impedir modificar campos protegidos.
22. Verificar que no existe recursion JSON.
23. Verificar codigos HTTP.
24. Verificar excepciones.
25. Verificar consultas de repositorio.
26. Verificar el arranque de Spring.

---

## 23. Criterios de aceptacion

El proyecto se considera funcional cuando:

- La aplicacion inicia correctamente.
- PostgreSQL esta configurado.
- Se pueden registrar clientes.
- Se pueden registrar ambos tipos de guitarra.
- La API conserva el tipo concreto del instrumento.
- Se pueden consultar instrumentos.
- Se pueden crear reservas.
- Se rechazan fechas invalidas.
- Se rechazan cruces de reservas.
- Se pueden confirmar y cancelar reservas.
- Se pueden crear prestamos.
- Se puede devolver un instrumento.
- Se registra condicion de entrega y devolucion.
- Existe historial de condiciones.
- PATCH no sobrescribe campos con null.
- Las respuestas no tienen ciclos JSON.
- Los errores tienen formato uniforme.
- Las pruebas principales pasan.
- El README permite instalar y ejecutar el proyecto.
- Cada integrante puede explicar arquitectura, dominio y flujo completo.

---

## 24. Flujo de una operacion completa

```text
1. Cliente registrado.
2. Instrumento registrado.
3. Cliente solicita reserva.
4. Service valida cliente y fechas.
5. Repository revisa disponibilidad.
6. Se crea Reserva pendiente.
7. Se confirma Reserva.
8. Instrumento pasa a RESERVADO.
9. Se entrega instrumento.
10. Se registra condicion de entrega.
11. Instrumento pasa a ALQUILADO.
12. Cliente devuelve instrumento.
13. Se registra condicion de devolucion.
14. Se agrega evento al historial.
15. Instrumento pasa a DISPONIBLE o EN_REPARACION.
16. Prestamo queda DEVUELTO.
```

---

## 25. Git y trabajo colaborativo

Ramas sugeridas:

```text
main
 develop
 feature/clientes
 feature/instrumentos
 feature/reservas
 feature/prestamos
 feature/pruebas
 feature/documentacion
```

Convencion de commits:

```text
feat: agrega entidad instrumento
feat: agrega mapper polimorfico
feat: implementa creacion de reservas
fix: evita reservas cruzadas
 test: agrega pruebas de devolucion
 docs: agrega guia de instalacion
 refactor: separa excepciones de dominio
```

Reglas:

- No trabajar directamente sobre `main`.
- Hacer commits pequenos.
- Describir claramente cada cambio.
- Actualizar la rama antes de integrar.
- Resolver conflictos revisando el dominio.
- No subir contrasenas reales.
- No borrar cambios de otro integrante.

---

## 26. Riesgos tecnicos

- Confundir estado de disponibilidad con condicion fisica.
- Exponer entidades JPA directamente.
- Crear recursion entre Cliente y Reserva.
- Permitir reservas cruzadas.
- Usar `double` para dinero.
- Permitir setters para estados criticos.
- No registrar la condicion anterior.
- Usar `ddl-auto=update` en produccion.
- No manejar conflictos de codigo duplicado.
- No probar el mapper polimorfico.
- Crear demasiadas funcionalidades antes de terminar el MVP.
- Agregar patrones que no resuelvan problemas reales.

---

## 27. Orden de aprendizaje

Estudiar y construir en este orden:

1. Clases y objetos.
2. Encapsulamiento.
3. Constructores y validaciones.
4. Enums.
5. Records y DTOs.
6. Herencia.
7. Polimorfismo.
8. Objetos de valor.
9. Entidades JPA.
10. Relaciones.
11. Repositorios.
12. Servicios.
13. Controllers REST.
14. MapStruct.
15. Transacciones.
16. Validacion HTTP.
17. Excepciones.
18. Proyecciones.
19. PATCH seguro.
20. Pruebas.
21. Documentacion.

En cada paso se debe responder:

- Que problema resuelve.
- Por que existe el archivo.
- Por que esta en ese paquete.
- Que principio aplica.
- Como se prueba.
- Que error ocurriria si se implementa mal.

---

## 28. Prompt maestro para continuar con un asistente

```text
Actua como docente de ingenieria de software y arquitecto senior.

Estamos construyendo MusicalRent API, un proyecto academico de tres estudiantes:
- Steven Eraso Insuasty.
- Sebastian Manchabajoy Rosero.
- Hector Alejandro Riascos Insuasty.

El sistema gestiona clientes, instrumentos musicales, guitarras acusticas y electricas, reservas, prestamos, devoluciones y condiciones fisicas.

Usamos Java 17, Spring Boot, Spring Web, Spring Data JPA, Hibernate, PostgreSQL, MapStruct, Maven, JUnit 5, Mockito y Bean Validation.

La arquitectura obligatoria es:
Controller -> Service -> Repository -> PostgreSQL.
Usamos tambien Domain, DTO request, DTO response, Mapper y Exception.

Reglas de trabajo:
1. No generes todo de una sola vez.
2. Trabaja fase por fase.
3. Antes de escribir codigo explica el problema.
4. Explica por que cada archivo existe.
5. Explica por que pertenece a su paquete.
6. Explica el concepto de programacion aplicado.
7. Espera confirmacion antes de avanzar.
8. No modifiques archivos sin autorizacion.
9. No borres cambios existentes.
10. Usa las convenciones del proyecto.
11. Ejecuta una validacion pequena despues de cada cambio.
12. Si hay un error, explica primero la causa.
13. No agregues dependencias innecesarias.
14. No inventes funcionalidades fuera del MVP.
15. El objetivo es que los estudiantes entiendan y puedan explicar el codigo.

Orden obligatorio:
1. Analisis y diagramas.
2. Configuracion.
3. Enums.
4. Objetos de valor.
5. Cliente.
6. Instrumento abstracto.
7. GuitarraAcustica.
8. GuitarraElectrica.
9. Reserva.
10. Prestamo.
11. RegistroCondicion.
12. Repositorios.
13. DTOs.
14. Mappers.
15. Servicios.
16. Controllers.
17. Validacion.
18. Excepciones.
19. Proyecciones anidadas.
20. PATCH seguro con MapStruct.
21. Pruebas.
22. Documentacion.

Conceptos que deben aparecer:
- Encapsulamiento.
- Abstraccion.
- Herencia.
- Polimorfismo.
- Domain Model.
- Value Object.
- DTO.
- Data Mapper.
- Repository Pattern.
- Service Layer.
- Dependency Injection.
- SOLID.
- REST.
- JPA.
- Hibernate.
- PostgreSQL.
- JOINED.
- Transacciones.
- Validacion.
- Excepciones.
- Proyecciones sin recursion.
- Actualizacion parcial segura.
- Pruebas.

No expongas entidades JPA directamente.
Usa BigDecimal para dinero.
Usa UUID.
Usa EnumType.STRING.
Usa metodos del dominio para cambios de estado.
Usa @Transactional en servicios.
Usa @RestControllerAdvice.
Usa NullValuePropertyMappingStrategy.IGNORE para PATCH.
Usa respuestas anidadas resumidas sin ciclos.

Para cada fase entrega:
- Objetivo.
- Conceptos.
- Archivos.
- Codigo.
- Explicacion linea por linea cuando sea necesario.
- Errores comunes.
- Prueba.
- Resultado esperado.
- Siguiente paso.
```

---

## 29. Primera tarea al retomar el proyecto

No crear todas las entidades aun.

Primero hacer una sesion de analisis:

1. Confirmar nombre del proyecto.
2. Confirmar alcance.
3. Dibujar las entidades.
4. Dibujar las relaciones.
5. Confirmar estados.
6. Confirmar reglas de negocio.
7. Confirmar ejemplos JSON.
8. Confirmar responsabilidad de cada integrante.
9. Crear el proyecto Spring Boot.
10. Ejecutar la aplicacion vacia.

La primera implementacion debe ser un enum sencillo y una prueba pequena, no todo el sistema.

---

## 30. Estado actual

- Este documento contiene el plan inicial.
- La carpeta del nuevo proyecto es `MusicalRent`.
- El proyecto original del hotel no debe modificarse para empezar MusicalRent.
- Todavia no se ha generado codigo de aplicacion en esta carpeta.
- El siguiente paso es iniciar la fase de analisis y luego crear la estructura base.
