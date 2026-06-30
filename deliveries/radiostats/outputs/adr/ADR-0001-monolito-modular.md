# ADR-0001 · Arquitectura general: monolito modular

**Estado:** aceptado  
**Fecha:** 2026-06-30

## Contexto y fuerza

El MVP de Radiostats debe servir a una sola emisora con entre 1 y 10 servidores de
streaming (supuesto:OQ-03, backlog.json). El equipo de operación es pequeño (Administrador
Técnico + Coordinador de Marketing + Director de Emisora) y el sistema no prevé
multi-tenancy ni carga variable extrema en el horizonte del MVP (mvp:fuera-de-alcance).

Las funcionalidades del MVP son cinco historias de usuario interconectadas
(US-01 a US-05) que comparten la misma fuente de datos (las APIs de Icecast/Shoutcast)
y el mismo almacén de historial. La separación en servicios independientes añadiría
complejidad operativa sin aportar beneficio técnico concreto en esta escala.

Fuerza principal: `req:R-09` — operación continua 24/7 sin ventanas de inactividad;
y `mvp:propuesta-de-valor` — entregar el MVP más rápido con riesgo aceptable.

## Decisión

Se adopta un **monolito modular**: una única aplicación desplegable con módulos
internos bien delimitados por responsabilidad (colector, almacén, alertas, API,
frontend). Los módulos se comunican mediante llamadas a funciones / interfaces internas,
no a través de red.

Los módulos son:
- **Collector**: polling a Icecast/Shoutcast cada ≤60 s.
- **Store**: persistencia del historial (base de datos local).
- **Alerter**: detección de caídas y envío de correo SMTP.
- **API**: endpoints HTTP que expone el backend al frontend.
- **Frontend**: interfaz web servida por el mismo proceso o un servidor estático adjunto.

## Alternativas consideradas

- **Microservicios** — cada módulo como servicio independiente. Aporta escalabilidad
  y despliegue independiente, pero multiplica la complejidad operativa (orquestación,
  red, observabilidad) sin que el volumen de datos de 1–10 servidores lo justifique.
  Rechazado: over-engineering para el MVP.

- **Serverless / funciones en la nube** — funciones desencadenadas por eventos o por
  scheduler. Requiere infraestructura en la nube y añade latencia variable; la
  métrica de detección ≤5 minutos (`req:R-03`, `mvp:metrica-exito-deteccion-5min`)
  es difícil de garantizar con cold starts. Rechazado: riesgo técnico innecesario.

## Consecuencias

**Ganamos:**
- Despliegue simple: un único proceso, un único repositorio, una única base de datos.
- Operación 24/7 directa sin orquestación de contenedores complejos.
- Ciclo de desarrollo más corto para las 5 historias del Sprint 1.

**Costo que aceptamos:**
- El escalado horizontal futuro requerirá extraer módulos. Si la emisora crece a
  decenas de servidores o la plataforma se convierte en SaaS multi-tenant, habrá
  que refactorizar. Ese momento es post-MVP y está documentado como open question.
