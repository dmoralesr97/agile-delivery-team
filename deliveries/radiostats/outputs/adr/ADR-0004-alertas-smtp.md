# ADR-0004 · Sistema de alertas: detección en el Collector y envío por SMTP

**Estado:** aceptado  
**Fecha:** 2026-06-30

## Contexto y fuerza

US-03 requiere que el sistema detecte la caída de un servidor y envíe una alerta
en ≤5 minutos desde el último dato válido (`mvp:metrica-exito-deteccion-5min`,
`req:R-03`, `req:R-09`). El canal de alerta acordado es correo electrónico via
SMTP configurable (supuesto:OQ-01). La alerta de recuperación también es obligatoria.

El sistema debe operar 24/7 sin ventanas de inactividad (`req:R-09`): el mecanismo
de detección no puede depender de interacción humana ni de un scheduler externo que
pueda detenerse.

Fuerza principal: `req:R-03` · `req:R-09` · `supuesto:OQ-01` ·
`mvp:metrica-exito-deteccion-5min`.

## Decisión

El módulo **Alerter** está integrado dentro del monolito (ADR-0001) y se activa
sincrónicamente al final de cada ciclo de polling del Collector (ADR-0002).

Flujo de detección y notificación:

1. Al finalizar cada ciclo de polling, el Collector pasa el resultado
   (status actual por servidor) al Alerter.
2. El Alerter compara el status actual con el estado previo almacenado en memoria
   (caché de estados).
3. Si un servidor pasa de `activo` a `no_responde` y no se ha enviado ya una alerta
   por esa caída, el Alerter encola un correo de caída.
4. Si un servidor pasa de `no_responde` a `activo`, el Alerter encola un correo de
   recuperación.
5. El correo se envía usando SMTP configurado mediante variables de entorno:
   `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASSWORD`, `ALERT_RECIPIENT`.
6. El estado de alerta enviada se persiste en la base de datos (tabla `alert_log`)
   para sobrevivir reinicios del proceso.

Con un intervalo de polling de 30 s (ADR-0002), la detección ocurre en ≤30 s y
el correo se envía en el mismo ciclo. El tiempo total de extremo a extremo es
inferior a 2 minutos en condiciones normales de red SMTP, cumpliendo el margen de
5 minutos con holgura.

## Alternativas consideradas

- **Cola de mensajes externa (RabbitMQ, Redis Streams)** — desacopla el Alerter
  del Collector y permite reintentos garantizados. Añade una dependencia de
  infraestructura significativa. El volumen de alertas esperado (decenas al mes,
  no miles por segundo) no justifica esta complejidad. Rechazado.

- **Webhook / Slack / SMS** — canales adicionales solicitados de forma informal en
  el discovery pero no respaldados como requisito aceptado; solo SMTP está acordado
  (supuesto:OQ-01). Se declaran como post-MVP. Rechazado para este ADR.

- **Servicio externo de monitoreo (Uptime Robot, Pingdom)** — depende de un
  proveedor de terceros y no integra con el dashboard propio ni con el historial
  de Radiostats. Contradice `req:R-08` (datos propios). Rechazado.

- **Scheduler externo (cron del sistema operativo)** — la detección se externaliza
  a cron. Introduce dependencia del entorno del sistema operativo y puede crear
  ventanas de inactividad si cron falla. Rechazado: `req:R-09` exige operación
  continua; el loop interno del monolito es más confiable.

## Consecuencias

**Ganamos:**
- La detección ocurre en cada ciclo de polling (≤30 s): margen amplio respecto al
  objetivo de ≤5 minutos.
- Sin dependencias externas para el camino crítico de alertas.
- La configuración vía variables de entorno es segura y compatible con cualquier
  entorno de despliegue (servidor propio, VPS, contenedor).

**Costo que aceptamos:**
- Si el proceso del monolito se reinicia durante una caída, la alerta en curso
  puede reenviarse. Se mitiga con `alert_log` en la base de datos que registra el
  estado de notificación.
- SMTP puede ser rechazado por el servidor de correo destino (spam, límites de
  envío). Se acepta como riesgo operativo conocido; el administrador debe usar un
  servidor SMTP de confianza. Es una responsabilidad de configuración, no de diseño.
- Canales adicionales (Slack, SMS) son post-MVP. Si la emisora los necesita antes
  de la segunda iteración, requerirán un nuevo ADR.
