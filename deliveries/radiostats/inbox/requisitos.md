# Requisitos Candidatos — Radiostats

## Funcionales

- **[R-01]** El sistema debe agregar en una sola vista las métricas de oyentes de todos los servidores de streaming configurados (al menos Icecast y Shoutcast), eliminando la necesidad de navegar entre paneles separados.
  - Tipo: funcional
  - Origen: administradorTecnico.md · directorEmisora.md · Administrador Técnico / Director de Emisora

- **[R-02]** El sistema debe consultar automáticamente los servidores de streaming a intervalos regulares y almacenar el histórico de métricas (oyentes por servidor, marcas de tiempo).
  - Tipo: funcional
  - Origen: administradorTecnico.md · Administrador Técnico

- **[R-03]** El sistema debe emitir alertas automáticas cuando un servidor deja de responder o cuando sus métricas salen de parámetros normales, sin requerir intervención humana para detectar el fallo.
  - Tipo: funcional
  - Origen: administradorTecnico.md · Administrador Técnico

- **[R-04]** El sistema debe mostrar la audiencia con granularidad horaria y permitir comparar rangos de fechas (al menos día a día y semana a semana).
  - Tipo: funcional
  - Origen: coordinadorMarketing.md · Coordinador de Marketing

- **[R-05]** El sistema debe mostrar comparativas de audiencia por programa o franja horaria e indicar tendencias de crecimiento o descenso.
  - Tipo: funcional
  - Origen: directorEmisora.md · coordinadorMarketing.md · Director de Emisora / Coordinador de Marketing

- **[R-06]** El sistema debe generar y exportar reportes de audiencia en un formato presentable (visual, listo para enviar) a clientes y anunciantes.
  - Tipo: funcional
  - Origen: coordinadorMarketing.md · Coordinador de Marketing

- **[R-07]** El sistema debe mostrar el total de oyentes en tiempo real (suma de todos los servidores activos en ese momento).
  - Tipo: funcional
  - Origen: directorEmisora.md · coordinadorMarketing.md · Director de Emisora / Coordinador de Marketing

## No funcionales

- **[R-08]** El sistema debe obtener los datos de audiencia directamente de los servidores de streaming, sin depender de encuestas externas ni de proveedores de datos de terceros.
  - Tipo: no funcional
  - Origen: coordinadorMarketing.md · Coordinador de Marketing

- **[R-09]** El sistema debe operar de forma continua (24/7) y detectar caídas de servidor en tiempo real, sin ventanas de inactividad programadas que impidan el monitoreo nocturno.
  - Tipo: no funcional
  - Origen: administradorTecnico.md · Administrador Técnico
