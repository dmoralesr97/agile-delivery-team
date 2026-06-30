# Sprint 1 — El equipo entrega visibilidad operativa en tiempo real y alertas proactivas de fallos para que el Administrador Técnico y el Director de Emisora operen la transmisión desde un panel unificado sin depender de reportes reactivos.

**Capacidad:** 20 pts · **Comprometido:** 13 pts

| Historia | Título                                      | Pts | Épica | Prioridad |
|----------|---------------------------------------------|-----|-------|-----------|
| US-01    | Dashboard unificado de servidores           |  5  | E-01  |     1     |
| US-02    | Total consolidado de oyentes en tiempo real |  3  | E-01  |     2     |
| US-03    | Alerta automática de caída y recuperación   |  5  | E-02  |     3     |

---

## Historias en el sprint

---

### US-01 · Dashboard unificado de servidores · E-01 · 5 pts

**Como** Administrador Técnico, **quiero** ver en una sola pantalla el estado (activo/caído) y el conteo de oyentes actuales de todos los servidores de streaming configurados, **para** no tener que abrir una pestaña por servidor cada vez que necesito conocer el estado general de la transmisión.

Criterios de aceptación (Gherkin):
- Dado que hay entre 1 y 10 servidores configurados, cuando abro el dashboard, entonces veo el estado (activo/caído) y el conteo de oyentes actuales de cada servidor en la misma vista, sin navegación adicional.
- Dado que un servidor está caído, cuando aparece en el dashboard, entonces se distingue visualmente con un indicador de error diferente al de los servidores activos.
- Dado que el dashboard está abierto, cuando han pasado 60 segundos desde la última actualización, entonces los datos se refrescan automáticamente sin recargar la página.
- Dado que una emisora tiene más de 10 servidores configurados, cuando se carga el dashboard, entonces se muestran los primeros 10 servidores por orden de configuración.

Supuestos aceptados:
- El MVP soporta entre 1 y 10 servidores de streaming (Icecast/Shoutcast). Si la emisora tiene más, se priorizarán los primeros 10.
- Las APIs de Icecast/Shoutcast son accesibles sin autenticación en red interna, o con credenciales configuradas en variables de entorno durante el setup inicial. Se realizará un spike de 1 día en la iteración 0.

Origen: us:US-01 · req:R-01 · req:R-08 · pain:monitoreo-multi-panel · persona:administrador-tecnico

---

### US-02 · Total consolidado de oyentes en tiempo real · E-01 · 3 pts

**Como** Director de Emisora, **quiero** ver el total acumulado de oyentes activos en este momento, sumando todos los servidores activos, con desglose por servidor en la misma vista, **para** poder responder consultas de anunciantes de forma inmediata sin hacer cálculos manuales entre paneles.

Criterios de aceptación (Gherkin):
- Dado que estoy en el dashboard, cuando lo consulto, entonces el total de oyentes refleja datos de hace menos de 60 segundos.
- Dado que hay más de un servidor activo, cuando veo el dashboard, entonces el número muestra la suma total de oyentes y el desglose por servidor en la misma vista.
- Dado que un servidor está caído y no aporta oyentes, cuando veo el total, entonces el total excluye ese servidor y lo indica visualmente.

Supuestos aceptados: ninguno adicional (hereda supuestos de US-01).

Origen: us:US-02 · req:R-07 · pain:vista-consolidada-inexistente · pain:perdida-oportunidades-comerciales · persona:director-de-emisora · persona:coordinador-de-marketing

---

### US-03 · Alerta automática de caída y recuperación de servidor · E-02 · 5 pts

**Como** Administrador Técnico, **quiero** recibir una alerta automática por correo electrónico cuando un servidor deja de responder y otra cuando recupera conectividad, **para** poder detectar y responder a fallos antes de que un oyente llame a reportarlos, cumpliendo el objetivo de detección en menos de 5 minutos.

Criterios de aceptación (Gherkin):
- Dado que un servidor deja de responder, cuando han transcurrido como máximo 5 minutos desde el último dato válido, entonces el sistema envía un correo electrónico al administrador configurado con el nombre del servidor y la hora del fallo.
- Dado que el servidor recupera conectividad, cuando vuelve a responder, entonces el sistema envía un correo electrónico de recuperación al administrador con el nombre del servidor y la hora de restauración.
- Dado que el sistema detecta la caída de un servidor, cuando actualiza el dashboard, entonces muestra el servidor con indicador de caído y la marca de hora de inicio del fallo.
- Dado que el SMTP está configurado con host, puerto y credenciales en variables de entorno, cuando ocurre un fallo, entonces el correo se envía usando esa configuración sin intervención adicional.

Supuestos aceptados:
- Las alertas se envían por correo electrónico (SMTP configurable). Canal adicional (Slack/SMS) queda como mejora post-MVP.

Origen: us:US-03 · req:R-03 · req:R-09 · mvp:metrica-exito-deteccion-5min · pain:deteccion-reactiva-fallos · persona:administrador-tecnico

---

## Historias diferidas (fuera de capacidad)

| Historia | Título                                      | Pts | Motivo                                                                                                      |
|----------|---------------------------------------------|-----|-------------------------------------------------------------------------------------------------------------|
| US-04    | Historial de audiencia con granularidad     |  8  | Supera la capacidad restante (7 pts disponibles tras comprometer US-01, US-02, US-03). Se posterga al Sprint 2. |
| US-05    | Exportación de reporte en CSV               |  5  | Depende de US-04 (cadena de dependencia explícita en backlog.json). No entrega valor verificable sin US-04. Se posterga al Sprint 2 junto con US-04. |

Nota: aunque US-05 cabe en los 7 pts restantes de capacidad, incluirla sin US-04 viola el principio de valor entregable: los criterios de aceptación de US-05 requieren datos históricos que solo US-04 provee. Incluirla sería comprometer trabajo que no puede demostrarse como terminado al cierre del sprint.

---

## Riesgos del sprint

**Riesgo 1 — Acceso a las APIs de Icecast/Shoutcast (origen: supuesto OQ-02, US-01)**
El supuesto de que las APIs son accesibles sin autenticación especial o con credenciales en variables de entorno no ha sido validado con la infraestructura real de la emisora. Si el acceso requiere configuración de red o credenciales por servidor no documentadas, US-01 y US-02 quedan bloqueadas.
Mitigacion: ejecutar el spike técnico de 1 día acordado en la iteración 0, antes de iniciar el desarrollo de US-01. Si el spike revela bloqueo, escalar al Product Owner para replantear el alcance del sprint antes del día 2.

**Riesgo 2 — Configuracion SMTP para alertas (origen: supuesto OQ-01, US-03)**
El supuesto de que el equipo de la emisora puede proveer credenciales SMTP válidas al inicio del sprint no ha sido confirmado. Sin una cuenta SMTP funcional, los criterios de aceptación de US-03 no pueden verificarse en el entorno de prueba.
Mitigacion: solicitar las credenciales SMTP al responsable técnico de la emisora en el primer día del sprint. Si no se obtienen, usar un servidor SMTP de prueba (ej. Mailtrap) para completar la historia y dejar la configuracion de produccion como tarea de despliegue documentada.

**Riesgo 3 — Dependencia encadenada US-02 sobre US-01 (origen: backlog.json, dependencies)**
US-02 depende de que US-01 esté terminada y funcionando para construir el total consolidado. Si US-01 se atrasa o el spike de APIs consume más tiempo del estimado, US-02 no puede iniciarse, comprimiendo el tiempo disponible para completar ambas historias dentro del sprint.
Mitigacion: priorizar US-01 al inicio del sprint y mantener US-02 en estado de "lista para comenzar" desde el día 1. El Scrum Master monitorea el avance de US-01 en el daily del día 3 y activa la alerta si no está en revisión para ese momento.
