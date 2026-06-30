# ADR-0005 · Frontend: páginas renderizadas en servidor con refresco automático vía polling del navegador

**Estado:** aceptado  
**Fecha:** 2026-06-30

## Contexto y fuerza

US-01 exige que el dashboard se refresque automáticamente cada 60 segundos sin
recargar la página completa (`backlog.json:US-01:acceptance_criteria`). US-04 exige
un gráfico de audiencia horaria con comparación de rangos de fechas.

El MVP sirve a un equipo de operación pequeño (no más de 3–5 usuarios simultáneos
esperados: Administrador Técnico, Director de Emisora, Coordinador de Marketing).
No hay requisito de aplicación móvil nativa ni de UX altamente interactiva más allá
del refresco y la selección de rangos de fecha (`mvp:fuera-de-alcance`).

Fuerza principal: `us:US-01` (refresco automático ≤60 s) · `us:US-04` (gráfico
histórico) · ADR-0001 (monolito modular, sin infraestructura de despliegue separada
para el frontend).

## Decisión

Se adopta un enfoque de **páginas HTML renderizadas en servidor (SSR / server-side
rendering)** con un componente de refresco automático liviano en el navegador:

1. El backend expone rutas HTTP que devuelven HTML completo (plantillas Jinja2,
   Go html/template o equivalente según el lenguaje elegido para el monolito).
2. El dashboard principal (US-01, US-02) incluye un fragmento de JavaScript mínimo
   que realiza un fetch a un endpoint JSON `/api/snapshot` cada 60 segundos y
   actualiza solo los valores del DOM que cambian (contadores de oyentes y estado),
   sin recargar la página completa.
3. El gráfico de audiencia histórica (US-04) se renderiza en el navegador usando
   una librería de gráficos ligera (Chart.js o Recharts) que recibe los datos
   preprocesados desde la API.
4. La exportación CSV (US-05) se sirve como descarga directa desde el backend
   (`Content-Disposition: attachment`).

No se adopta un framework SPA (React, Vue, Angular) porque:
- El requisito de interactividad (refresco cada 60 s + selección de fechas) no
  requiere un grafo de componentes reactivo complejo.
- Un SPA añade un paso de compilación, gestión de rutas en cliente, y hace la
  aplicación más difícil de depurar para un equipo pequeño.

## Alternativas consideradas

- **SPA con React/Vue + API REST separada** — mayor interactividad y DX para equipos
  front especializados. Añade build pipeline (Vite/Webpack), dos despliegues
  separados (API + estáticos) y mayor complejidad de CORS. El nivel de interactividad
  del MVP no lo justifica. Rechazado.

- **WebSockets para refresco en tiempo real** — empuja datos del servidor al
  navegador sin polling. Requiere mantener conexiones persistentes. Para 3–5 usuarios
  simultáneos con datos que cambian cada 30–60 s, el overhead de WebSocket no
  aporta ventaja perceptible. Rechazado: sobre-dimensionado para el MVP.

- **HTMX** — permite refresco parcial de HTML sin JavaScript manual. Es una opción
  válida y se deja como alternativa de implementación dentro de esta decisión. No
  se prescribe el framework concreto: se acepta HTMX o fetch manual, según el
  lenguaje elegido para el monolito.

## Consecuencias

**Ganamos:**
- Frontend servido desde el mismo proceso del monolito: sin despliegue ni dominio
  separado, sin gestión de CORS.
- La lógica de autorización y la vista son co-ubicadas: menos superficie de ataque.
- Tiempo de construcción mínimo: no hay paso de compilación de activos obligatorio.

**Costo que aceptamos:**
- El JavaScript de refresco es manual (fetch + manipulación de DOM). Debe
  mantenerse acotado para no convertirse en un "SPA accidental". Se controla
  revisando que no supere unas pocas docenas de líneas.
- Si en el futuro se requiere una experiencia móvil nativa o una UX más dinámica,
  esta decisión deberá revisarse con un nuevo ADR para extraer la API y adoptar
  un cliente separado.
