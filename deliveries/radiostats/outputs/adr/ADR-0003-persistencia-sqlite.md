# ADR-0003 · Persistencia del historial: SQLite con tabla de series temporales

**Estado:** aceptado  
**Fecha:** 2026-06-30

## Contexto y fuerza

El sistema debe almacenar el historial de métricas (oyentes por servidor, marca de
tiempo) durante los últimos 90 días (`supuesto:OQ-05`, `req:R-02`). US-04 exige
consultas con granularidad horaria y comparación de rangos de fechas (`req:R-04`).

El MVP opera para una sola emisora con 1–10 servidores de streaming. Estimando el
volumen: 10 servidores × 2 lecturas/minuto × 60 min × 24 h × 90 días = ~25 millones
de filas en el peor caso. Con agregación horaria (1 fila/hora/servidor), son
10 × 24 × 90 = 21 600 filas, una carga trivial para cualquier base de datos relacional.

Fuerza principal: `req:R-02` · `req:R-04` · `supuesto:OQ-05` · ADR-0001
(monolito modular: la base de datos es local al proceso).

## Decisión

Se adopta **SQLite** como motor de persistencia, con dos tablas principales:

- `snapshots`: registros crudos de cada polling (server_id, timestamp_utc, listeners,
  status). Se retienen 90 días; una tarea de mantenimiento diaria purga registros
  más antiguos.
- `hourly_aggregates`: agregados por hora calculados mediante tarea programada o
  vista materializada (promedio y máximo de oyentes por servidor por hora). Esta
  tabla alimenta directamente US-04 y US-05.

SQLite se elige porque:
1. No requiere servidor separado: encaja directamente en la arquitectura monolítica
   (ADR-0001) sin complejidad operativa adicional.
2. El volumen de datos es manejable (decenas de miles de filas, no millones, con
   agregación horaria).
3. Las consultas necesarias (GROUP BY hora, filtro por rango de fechas, SUM de
   oyentes) son estándar SQL que SQLite ejecuta eficientemente con índices sobre
   `timestamp_utc` y `server_id`.
4. El despliegue es un único fichero: backup trivial, sin gestión de usuarios ni
   conexiones de red que asegurar.

## Alternativas consideradas

- **PostgreSQL** — motor robusto con soporte nativo para extensiones de series
  temporales (TimescaleDB). Añade un proceso de servidor, gestión de conexiones,
  usuarios y respaldos más complejos. El volumen del MVP no justifica esta
  complejidad. Rechazado para el MVP; es la migración natural si la plataforma
  escala a multi-tenant.

- **InfluxDB / TimescaleDB** — bases de datos especializadas en series temporales.
  Óptimas para millones de métricas por segundo. El volumen del MVP (decenas de
  peticiones por minuto) no alcanza el umbral donde aportan ventaja real. Añaden
  una dependencia de infraestructura nueva. Rechazado: sobre-dimensionado.

- **Almacenamiento en ficheros CSV/JSON** — simple de implementar, pero sin
  capacidad de consulta eficiente por rango de fechas ni de agregación. Rechazado:
  no cumpliría los criterios de aceptación de US-04 con rendimiento aceptable.

## Consecuencias

**Ganamos:**
- Despliegue de cero dependencias externas: la base de datos viaja con la aplicación.
- Backup trivial: copiar un fichero.
- Consultas SQL estándar que cualquier desarrollador del equipo puede escribir y
  optimizar sin conocimiento especializado.

**Costo que aceptamos:**
- SQLite no soporta concurrencia de escritura multi-proceso real (WAL mode lo
  mitiga para lecturas concurrentes). El monolito (ADR-0001) garantiza que solo
  un proceso escribe, eliminando este riesgo en el MVP.
- La migración a PostgreSQL será necesaria si el producto evoluciona a SaaS
  multi-emisora. Ese costo es post-MVP y está declarado como open question.
- La purga de 90 días debe ejecutarse de forma fiable; un fallo en la tarea de
  mantenimiento puede hacer crecer el fichero indefinidamente. Se mitiga con una
  alerta de tamaño de fichero (fuera del alcance del MVP: open question).
