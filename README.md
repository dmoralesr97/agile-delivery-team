# Agile Delivery Team

Equipo ágil de entrega impulsado por Claude Code que convierte el descubrimiento de un producto en un **plan de entrega listo para construir**: épicas, backlog priorizado, historias INVEST refinadas, arquitectura con ADRs y Sprint Plan.

No escribe código de producción. **Planifica el delivery** con criterio ágil.

---

## Cómo funciona

El equipo recibe como entrada las salidas de un **Discovery Agent** (MVP Canvas, user stories, requisitos, personas, mapa de evidencia) y produce de forma estructurada todos los artefactos que necesita un equipo de desarrollo para arrancar un sprint.

Cada paso invoca uno o más subagentes especializados y pasa por un **gate automático** que verifica calidad antes de escribir los artefactos críticos.

```
Discovery Agent (Unidad 1)
        │
        ▼
  inbox/ del delivery
        │
        ▼
 /delivery:generate-epics     → PO genera épicas + backlog.json
        │
        ▼
 /delivery:generate-stories   → Developer refina historias (gate INVEST/DoR)
        │
        ▼
 /delivery:architecture       → Architect propone arquitectura + ADRs
        │
        ▼
 /delivery:sprint-plan        → Scrum Master arma el Sprint Plan (gate DoR)
        │
        ▼
 /delivery:report             → reporte HTML visual
```

---

## El equipo (cuatro subagentes)

Los agentes viven en [.claude/agents/](.claude/agents/). Claude Code los invoca automáticamente al ejecutar cada comando de delivery.

| Rol | Archivo | Responsabilidad | Pregunta de vida |
|-----|---------|-----------------|-----------------|
| **Product Owner** | `product-owner.md` | Descompone el MVP en épicas y prioriza el backlog por valor | ¿Estamos construyendo lo correcto? |
| **Developer** | `developer.md` | Refina historias hasta INVEST, estima y resuelve dependencias | ¿Cómo lo construimos bien? |
| **Scrum Master** | `scrum-master.md` | Custodia la Definition of Ready y arma el Sprint Plan respetando la capacidad | ¿Qué frena al equipo? |
| **Architect** | `architect.md` | Diseña la arquitectura y registra decisiones en ADRs | ¿Qué estructura sostiene esto en el tiempo? |

### Product Owner

Lee el `inbox/` y descompone el MVP en **épicas orientadas a outcome** (no a módulos técnicos). Prioriza por valor: lo más riesgoso/valioso va primero. Genera `epics.md` con diagrama Mermaid y `backlog.json` con las historias candidatas.

### Developer

Toma el `backlog.json` y **refina cada historia** hasta que cumpla INVEST: escribe criterios de aceptación en Gherkin (Dado/Cuando/Entonces), estima en puntos de Fibonacci (1, 2, 3, 5, 8), divide las que superen 8 pts. Escribe `stories.md` y actualiza `backlog.json`.

### Scrum Master

Verifica que cada historia incluida en el sprint cumpla la **Definition of Ready**. Define el Sprint Goal, respeta la capacidad en puntos y escribe `sprint-plan.md` y `sprint-plan.json`. No sobrecompromete el sprint.

### Architect

Lee el `inbox/` y los artefactos del backlog para proponer la arquitectura más simple que funcione. Documenta cada decisión relevante en un **ADR** (formato MADR) con contexto, alternativas consideradas y consecuencias.

---

## Gate automático: `dor-invest-gate.py`

El hook [.claude/hooks/dor-invest-gate.py](.claude/hooks/dor-invest-gate.py) se ejecuta automáticamente como `PreToolUse` cada vez que un agente intenta escribir `stories.md` o `sprint-plan.md`. Si alguna historia no cumple los criterios, **bloquea la escritura** y devuelve el motivo al agente para que corrija.

Verifica por cada historia:

| Criterio | Campo | Regla |
|----------|-------|-------|
| **Formato** | `as_a`, `want`, `so_that` | No vacíos |
| **Testeable** | `acceptance_criteria` | Al menos 1 criterio verificable |
| **Estimable** | `estimate` | Entero > 0 |
| **Pequeña** | `estimate` | ≤ 8 pts (configurable con `DELIVERY_MAX_POINTS`) |
| **Independiente** | `dependencies` | Solo apunta a IDs que existen en el backlog |
| **Ready** | `open_questions` | Vacío (sin preguntas abiertas bloqueantes) |
| **No-vaga** | `want` | Sin términos como "rápido", "intuitivo" o "fácil" sin criterio medible |

El gate no se puede esquivar cambiando el nombre del archivo: el equipo debe corregir la causa raíz.

---

## Estructura del proyecto

```
agile-delivery-team/
├── .claude/
│   ├── agents/                  ← definición de los cuatro subagentes
│   │   ├── architect.md
│   │   ├── developer.md
│   │   ├── product-owner.md
│   │   └── scrum-master.md
│   ├── hooks/
│   │   └── dor-invest-gate.py   ← gate automático INVEST + DoR
│   ├── scripts/
│   │   └── build-report.py      ← generador del reporte HTML (determinista)
│   ├── skills/delivery/         ← formatos canónicos de todos los artefactos
│   │   ├── SKILL.md             ← formatos: épica, historia, backlog.json, ADR, sprint plan
│   │   ├── generate-epics.md
│   │   ├── generate-stories.md
│   │   ├── architecture.md
│   │   ├── sprint-plan.md
│   │   └── report.md
│   └── settings.json            ← hook PreToolUse configurado
├── deliveries/
│   └── <nombre-del-delivery>/
│       ├── inbox/               ← entradas del Discovery Agent (solo lectura)
│       │   ├── mvp-canvas.md
│       │   ├── user-stories.md
│       │   ├── requisitos.md
│       │   ├── personas.md
│       │   └── evidence-map.json
│       └── outputs/             ← artefactos producidos por el equipo
│           ├── epics.md
│           ├── backlog.json
│           ├── stories.md
│           ├── architecture.md
│           ├── adr/
│           │   └── ADR-0001-*.md
│           ├── sprint-plan.md
│           ├── sprint-plan.json
│           └── report.html
└── CLAUDE.md                    ← constitución del proyecto (reglas del equipo)
```

---

## Uso paso a paso

### 1. Crear un delivery y agregar el inbox

```bash
mkdir -p deliveries/mi-producto/inbox
# Copiar aquí las salidas del Discovery Agent:
# mvp-canvas.md, user-stories.md, requisitos.md, personas.md, evidence-map.json
```

### 2. Generar épicas y backlog

```
/delivery:generate-epics mi-producto
```

El **Product Owner** lee el `inbox/` y produce:
- `outputs/epics.md` — épicas con diagrama Mermaid
- `outputs/backlog.json` — historias candidatas estructuradas

### 3. Refinar historias (con gate automático)

```
/delivery:generate-stories mi-producto
```

El **Developer** refina cada historia y el **Scrum Master** la valida. El gate `dor-invest-gate.py` verifica INVEST + DoR antes de escribir `stories.md`. Si alguna historia no cumple, bloquea y el Developer corrige.

### 4. Diseñar la arquitectura

```
/delivery:architecture mi-producto
```

El **Architect** produce:
- `outputs/architecture.md` — diagrama de componentes Mermaid
- `outputs/adr/ADR-NNNN-*.md` — una decisión por archivo

### 5. Armar el Sprint Plan (con gate automático)

```
/delivery:sprint-plan mi-producto
```

El **Scrum Master** define el Sprint Goal, respeta la capacidad y mete solo historias "ready". El gate vuelve a verificar antes de escribir `sprint-plan.md`.

### 6. Generar el reporte HTML

```
/delivery:report mi-producto
```

El script `build-report.py` (no el modelo) genera un dashboard visual determinista con el estado INVEST de cada historia a color y el Sprint Plan.

---

## Reglas no negociables

1. **Cero invención.** Toda épica, historia o ADR traza a algo del `inbox/`. Si el descubrimiento no lo respalda, va a `open_questions`, no se afirma como hecho.
2. **Trazabilidad.** Campo `origin` en épicas e historias; cada ADR cita la fuerza que lo motiva.
3. **INVEST + DoR es ley.** El gate lo exige programáticamente; no hay atajos.
4. **Valor antes que solución.** Prioridad por output → outcome → impact, nunca por facilidad técnica.
5. **Aislamiento.** Cada delivery es independiente bajo `deliveries/<nombre>/`. No se mezclan datos entre deliveries.
6. **Idioma:** español.

---

## Formato de artefactos clave

### Historia en `backlog.json`

```json
{
  "id": "US-01",
  "epic": "E-01",
  "as_a": "usuario registrado",
  "want": "ver mis estadísticas de escucha",
  "so_that": "descubrir mis géneros más escuchados",
  "acceptance_criteria": [
    "Dado que el usuario tiene historial, cuando accede a su perfil, entonces ve un gráfico con el top 5 de géneros."
  ],
  "estimate": 3,
  "dependencies": [],
  "open_questions": [],
  "priority": 1,
  "origin": ["us:estadisticas-perfil"]
}
```

### ADR (formato MADR)

```markdown
# ADR-0001 · Título de la decisión
**Estado:** aceptado
**Fecha:** 2026-06-30

## Contexto y fuerza
<problema que obliga a decidir; cita el origen>

## Decisión
<lo que se decide>

## Alternativas consideradas
- <alternativa> — <por qué no>

## Consecuencias
<lo que ganamos y el costo que aceptamos>
```

---

## Configuración del gate

La variable de entorno `DELIVERY_MAX_POINTS` controla el tamaño máximo de una historia antes de que el gate la rechace como "no pequeña". Por defecto: **8 puntos**.

```bash
DELIVERY_MAX_POINTS=5 # para sprints con historias más pequeñas
```
