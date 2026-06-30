# Historias de Usuario Refinadas — Radiostats

Generado por: Developer  
Fecha: 2026-06-30  
Delivery: radiostats  
Estado: todas las historias cumplen INVEST + Definition of Ready

---

## Supuestos adoptados (preguntas abiertas resueltas)

Los siguientes supuestos se adoptaron para desbloquear las historias del backlog.
Quedan registrados como supuestos aceptados, no como hechos confirmados. Deben
validarse con los interesados al inicio del proyecto.

| ID | Supuesto adoptado | Historias desbloqueadas |
|----|-------------------|------------------------|
| OQ-02/OQ-03 | El MVP soporta entre 1 y 10 servidores de streaming (Icecast/Shoutcast). Si la emisora tiene más, se priorizarán los primeros 10. | US-01 |
| OQ-02 | Las APIs de Icecast/Shoutcast son accesibles sin autenticación en red interna, o con credenciales configuradas en variables de entorno durante el setup inicial. Se realizará un spike de 1 día en la iteración 0. | US-01 |
| OQ-01 | Las alertas se envían por correo electrónico (SMTP configurable). Canal adicional (Slack/SMS) queda como mejora post-MVP. | US-03 |
| OQ-05 | El sistema retiene datos de los últimos 90 días. No se importa historial previo al MVP. | US-04 |
| OQ-04 | El MVP genera CSV como formato prioritario (menor complejidad técnica). PDF queda como mejora post-MVP. | US-05 |

---

## Épica E-01 · Dashboard unificado: visibilidad operativa en tiempo real

---

### US-01 · Dashboard unificado de servidores   ·   épica E-01   ·   5 pts

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

### US-02 · Total consolidado de oyentes en tiempo real   ·   épica E-01   ·   3 pts

**Como** Director de Emisora, **quiero** ver el total acumulado de oyentes activos en este momento, sumando todos los servidores activos, con desglose por servidor en la misma vista, **para** poder responder consultas de anunciantes de forma inmediata sin hacer cálculos manuales entre paneles.

Criterios de aceptación (Gherkin):
- Dado que estoy en el dashboard, cuando lo consulto, entonces el total de oyentes refleja datos de hace menos de 60 segundos.
- Dado que hay más de un servidor activo, cuando veo el dashboard, entonces el número muestra la suma total de oyentes y el desglose por servidor en la misma vista.
- Dado que un servidor está caído y no aporta oyentes, cuando veo el total, entonces el total excluye ese servidor y lo indica visualmente.

Origen: us:US-02 · req:R-07 · pain:vista-consolidada-inexistente · pain:perdida-oportunidades-comerciales · persona:director-de-emisora · persona:coordinador-de-marketing

---

## Épica E-02 · Monitoreo proactivo: alertas automáticas de fallos de servidor

---

### US-03 · Alerta automática de caída y recuperación de servidor   ·   épica E-02   ·   5 pts

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

## Épica E-03 · Inteligencia comercial: historial y exportación de reportes de audiencia

---

### US-04 · Historial de audiencia con granularidad horaria   ·   épica E-03   ·   8 pts

**Como** Coordinador de Marketing, **quiero** consultar la audiencia histórica con granularidad horaria y comparar rangos de fechas (al menos día a día y semana a semana), **para** poder identificar tendencias y medir el impacto real de campañas con datos propios, sin esperar encuestas externas.

Criterios de aceptación (Gherkin):
- Dado un rango de fechas dentro de los últimos 90 días seleccionado, cuando lo consulto, entonces veo un gráfico con el número de oyentes por hora para ese periodo.
- Dado dos rangos de fechas distintos dentro de los últimos 90 días, cuando los selecciono, entonces puedo compararlos en la misma vista con visualización lado a lado o superpuesta.
- Dado que una campaña estuvo activa en un periodo, cuando lo consulto, entonces el pico de audiencia de ese periodo es visible en el gráfico sin cálculo adicional.
- Dado que el sistema lleva menos de 90 días operando, cuando consulto el historial, entonces se muestran los datos disponibles desde la fecha de inicio del sistema sin error.

Supuestos aceptados:
- El sistema retiene datos de los últimos 90 días. No se importa historial previo al MVP.

Origen: us:US-04 · req:R-02 · req:R-04 · pain:datos-audiencia-desactualizados · pain:datos-llegan-tarde · persona:coordinador-de-marketing

---

### US-05 · Exportación de reporte de audiencia en CSV   ·   épica E-03   ·   5 pts

**Como** Coordinador de Marketing, **quiero** exportar un reporte de audiencia en formato CSV listo para presentar a clientes, con datos de tendencia horaria, totales y nombre del periodo analizado, **para** no tener que construir el reporte manualmente cada vez que un anunciante solicita justificación de la pauta.

Criterios de aceptación (Gherkin):
- Dado un rango de fechas y métricas seleccionadas, cuando activo la exportación, entonces obtengo un archivo CSV con columnas: hora, oyentes por servidor, total acumulado y nombre del periodo analizado.
- Dado que el reporte CSV se genera, entonces incluye una fila de totales y el nombre de la emisora en el encabezado, sin requerir edición adicional antes de enviarlo al cliente.
- Dado que el archivo CSV se descarga, cuando lo abro en una hoja de cálculo, entonces los datos están ordenados cronológicamente y las columnas tienen encabezados legibles en español.

Supuestos aceptados:
- El MVP genera CSV como formato prioritario (menor complejidad técnica). PDF queda como mejora post-MVP.

Origen: us:US-05 · req:R-06 · mvp:metrica-exito-reporte-exportado · pain:reportes-manuales-clientes · pain:perdida-oportunidades-comerciales · persona:coordinador-de-marketing

---

## Resumen del backlog refinado

| Historia | Título | Épica | Pts | Estado DoR |
|----------|--------|-------|-----|-----------|
| US-01 | Dashboard unificado de servidores | E-01 | 5 | Lista |
| US-02 | Total consolidado de oyentes en tiempo real | E-01 | 3 | Lista |
| US-03 | Alerta automática de caída y recuperación | E-02 | 5 | Lista |
| US-04 | Historial de audiencia con granularidad horaria | E-03 | 8 | Lista |
| US-05 | Exportación de reporte de audiencia en CSV | E-03 | 5 | Lista |

**Total backlog:** 26 puntos · 5 historias · 0 preguntas abiertas bloqueantes
