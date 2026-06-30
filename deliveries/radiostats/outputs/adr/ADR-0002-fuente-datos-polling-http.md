# ADR-0002 · Fuente de datos de streaming: polling HTTP periódico

**Estado:** aceptado  
**Fecha:** 2026-06-30

## Contexto y fuerza

El sistema debe obtener métricas (oyentes actuales, estado activo/caído) directamente
de los servidores de streaming Icecast y Shoutcast (`req:R-08`), a intervalos
regulares, almacenando el historial con marca de tiempo (`req:R-02`).

El criterio de aceptación de US-01 exige que los datos del dashboard no tengan más
de 60 segundos de antigüedad. El criterio de aceptación de US-03 exige detectar una
caída en ≤5 minutos (`mvp:metrica-exito-deteccion-5min`).

Icecast expone un endpoint HTTP de status en `/status-json.xsl` (JSON) y en
`/status.xsl` (XML). Shoutcast expone `/statistics` (XML) y `/7.html` (texto plano).
Ambos protocolos son HTTP sincrónicos: no ofrecen push nativo ni WebSocket estándar.

Fuerza principal: `req:R-02` · `req:R-08` · `req:R-09` · supuesto:OQ-02.

## Decisión

Se adopta **polling HTTP activo** desde el módulo Collector, con un intervalo
configurable de entre 30 y 60 segundos por servidor. El Collector ejecuta un ciclo
periódico (scheduler interno o proceso daemon) que:

1. Realiza una petición HTTP GET al endpoint de status de cada servidor configurado.
2. Parsea la respuesta (JSON para Icecast, XML/texto para Shoutcast) y extrae
   oyentes actuales y estado.
3. Persiste el registro en la base de datos con la marca de tiempo UTC.
4. Compara el resultado con el último estado conocido para activar el módulo Alerter
   si hay caída o recuperación.

El intervalo por defecto será de 30 segundos, garantizando un margen suficiente
para la detección de caída en ≤5 minutos (máximo 10 ciclos de 30 s antes de
escalar la alerta).

Las credenciales de acceso a cada servidor se configuran en variables de entorno
(supuesto:OQ-02).

## Alternativas consideradas

- **WebSocket / streaming push** — requeriría que los servidores de streaming
  soporten WebSocket como productores. Icecast y Shoutcast no ofrecen esta
  capacidad de forma estándar. Rechazado: no viable con la infraestructura existente.

- **SNMP** — protocolo de gestión de red. Icecast y Shoutcast no tienen agentes
  SNMP incorporados de forma estándar; requeriría instalar software adicional en
  los servidores del cliente. Rechazado: modifica la infraestructura del cliente,
  contradice el supuesto de acceso sin modificación (`mvp:supuestos-riesgosos-apis`).

- **Integración vía proveedor de CDN / tercero** — contradice explícitamente `req:R-08`
  (datos propios, sin depender de terceros). Rechazado.

## Consecuencias

**Ganamos:**
- Implementación directa y predecible; el comportamiento del polling es fácil de
  probar y depurar.
- Compatible con la infraestructura existente de Icecast y Shoutcast sin modificar
  los servidores del cliente.
- El intervalo de 30 s garantiza la métrica de detección ≤5 minutos.

**Costo que aceptamos:**
- El Collector genera carga HTTP en cada servidor cada 30 s. Con 10 servidores, son
  20 peticiones/minuto: carga despreciable para Icecast/Shoutcast estándar.
- Si un servidor de streaming bloquea las peticiones del Collector por reglas de
  firewall, el spike de iteración 0 (supuesto:OQ-02) debe detectarlo.
- El polling no es tiempo real estricto: hay un margen de latencia de hasta 30 s
  entre el evento real y su detección. Esto es aceptable para el MVP.
