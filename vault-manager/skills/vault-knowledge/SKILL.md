---
description: >
  Conocimiento del vault de Obsidian para gestión de notas, tareas, patrones y conocimiento.
  Se activa cuando el usuario menciona vault, obsidian, tareas del vault, patrones,
  conocimiento, "guardar como patrón", "crear tarea en el vault", "registrar decisión",
  o cualquier referencia a la gestión de conocimiento centralizado.
---

# Vault Knowledge — Skill de gestión del vault

Tienes acceso a un vault de Obsidian centralizado que funciona como base de conocimiento transversal entre proyectos. Este skill te da las convenciones y herramientas para interactuar con él.

## Ubicación del vault

Lee la ruta desde `${CLAUDE_PLUGIN_ROOT}/config/vault-path.txt`. Si el archivo no existe, pide al usuario que ejecute `/vault-manager:vault-init` para configurar la ruta del vault.

## Estructura del vault

```
vault/
├── proyectos/        # Un directorio por proyecto gestionado
├── conocimiento/
│   ├── patrones/     # Patrones reutilizables entre proyectos
│   ├── decisiones/   # ADRs (Architecture Decision Records)
│   └── aprendizajes/ # Aprendizajes de debugging, investigación
├── tareas/           # Tareas (.md) + vistas (.base)
│   └── epicas/       # Épicas como .md
├── exploraciones/    # Investigaciones y research
├── sesiones/         # Logs de sesiones de trabajo
├── plantillas/       # Templates para notas nuevas
└── dashboard.md      # Navegación central
```

## Convenciones de frontmatter

Toda nota DEBE tener frontmatter YAML. Los campos obligatorios dependen del tipo:

### Proyecto
```yaml
---
tags:
  - project
status: active          # active | paused | archived
stack: "React, TypeScript"
repo: "/ruta/absoluta"
date: YYYY-MM-DD
---
```

### Tarea
```yaml
---
tags:
  - task
project: "[[nombre-proyecto]]"
status: open            # open | in_progress | done | blocked
priority: 2             # 1 (alta) a 4 (baja)
type: task              # task | feature | bug | exploration
created: YYYY-MM-DD
---
```

### Sesión
```yaml
---
tags:
  - session
project: "[[nombre-proyecto]]"
date: YYYY-MM-DD
---
```

### Patrón
```yaml
---
tags:
  - pattern
projects:
  - "[[proyecto-1]]"
  - "[[proyecto-2]]"
status: draft           # draft | validated
date: YYYY-MM-DD
---
```

### Épica
```yaml
---
tags:
  - epic
project: "[[nombre-proyecto]]"
status: open            # open | in_progress | done
progress: "0/5"
date: YYYY-MM-DD
---
```

### Exploración
```yaml
---
tags:
  - exploration
status: draft           # draft | in_progress | complete
date: YYYY-MM-DD
---
```

### Decisión (ADR)
```yaml
---
tags:
  - decision
projects:
  - "[[proyecto-1]]"
status: accepted        # proposed | accepted | deprecated
date: YYYY-MM-DD
---
```

## Taxonomía de tags

Tags primarios (exactamente uno por nota):
- `project` — Entrada de proyecto
- `task` — Tarea individual
- `epic` — Épica con subtareas
- `pattern` — Patrón reutilizable
- `decision` — Decisión arquitectónica (ADR)
- `session` — Log de sesión de trabajo
- `exploration` — Investigación/research

Tags adicionales opcionales para categorización: `claude-code`, `obsidian`, `react`, `ai`, `typescript`, etc.

## Convenciones de nombrado

- Archivos en **kebab-case**: `mi-nueva-tarea.md`
- Sesiones: `YYYY-MM-DD-nombre-proyecto.md`
- Sin espacios en nombres de archivo (excepto exploraciones existentes)

## Wikilinks

Usa `[[nombre-nota]]` (sin extensión .md) para vincular notas entre sí:

- Tarea → Proyecto: `project: "[[mi-proyecto]]"`
- Patrón → Proyectos: `projects: ["[[proyecto-1]]", "[[proyecto-2]]"]`
- Sesión → Proyecto: `project: "[[mi-proyecto]]"`
- En el cuerpo: `Ver [[nombre-nota]] para más detalles`

## Vistas .base

Los archivos `.base` son vistas dinámicas. **No los edites directamente** a menos que el usuario lo pida. Se embeben en markdown con `![[archivo.base]]`.

## Operaciones comunes

### Crear una tarea
1. Crea `${VAULT}/tareas/<nombre-en-kebab>.md`
2. Usa el frontmatter de tarea (ver arriba)
3. Incluye secciones: descripción, contexto, subtareas

### Crear un patrón
1. Crea `${VAULT}/conocimiento/patrones/<nombre-patron>.md`
2. Usa el frontmatter de patrón
3. Incluye secciones: problema, solución, ejemplo, cuándo usar, cuándo NO usar

### Registrar una decisión
1. Crea `${VAULT}/conocimiento/decisiones/<nombre-decision>.md`
2. Usa el frontmatter de decisión
3. Incluye: contexto, decisión, consecuencias, alternativas consideradas

### Crear una exploración
1. Crea `${VAULT}/exploraciones/<nombre-exploracion>.md`
2. Usa el frontmatter de exploración
3. Incluye: contexto del problema, estado del arte, hallazgos, decisiones, próximos pasos

### Registrar un aprendizaje
1. Crea `${VAULT}/conocimiento/aprendizajes/<nombre>.md`
2. Frontmatter mínimo: `tags: [learning]`, `date`, `projects`
3. Incluye: problema encontrado, investigación, solución, lección aprendida

## Plantillas

Las plantillas de referencia están en `${VAULT}/plantillas/`. Úsalas como guía para la estructura de cada tipo de nota:

- `project.md` — Proyecto
- `session.md` — Sesión
- `task.md` — Tarea
- `epic.md` — Épica
- `pattern.md` — Patrón
- `exploration.md` — Exploración

## Importante

- **No modifiques** archivos `.base` sin que el usuario lo pida explícitamente.
- **No modifiques** `CLAUDE.md` del vault.
- **Siempre** verifica que el frontmatter es YAML válido antes de escribir.
- **Siempre** usa la fecha de hoy para nuevas notas.
- Los placeholders `{{date}}` y `{{title}}` solo existen en las plantillas; en notas reales usa valores concretos.
