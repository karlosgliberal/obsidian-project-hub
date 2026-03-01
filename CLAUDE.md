# Obsidian Project Hub

Vault centralizado de conocimiento para todos los proyectos gestionados con Claude Code.

## Estructura del Vault

```
vault/
├── proyectos/        # Un fichero por proyecto
├── conocimiento/     # Conocimiento transversal (flat, tags para diferenciar)
├── tareas/           # Captura rápida de ideas (.md) + vistas (.base)
├── sesiones/         # Logs de sesiones
└── plantillas/       # Templates: project, task, session, knowledge
```

## Convenciones

### Frontmatter obligatorio

Toda nota debe tener frontmatter YAML con al menos:

```yaml
---
tags:
  - <tipo>           # project, task, session, knowledge
date: YYYY-MM-DD
status: <estado>     # depende del tipo (ver abajo)
---
```

### Taxonomía de tags

4 tipos primarios (exactamente uno por nota):

- `project` — Entrada de proyecto
- `task` — Captura rápida de idea/tarea vinculada a proyecto
- `session` — Log de sesión de trabajo
- `knowledge` — Patrones, aprendizajes, decisiones, investigaciones

Tags adicionales opcionales para categorización: `claude-code`, `obsidian`, `react`, `ai`, etc.

### Status por tipo

| Tipo | Valores posibles |
|------|-----------------|
| project | `active`, `paused`, `archived` |
| task | `open`, `done` |
| knowledge | `draft`, `validated` |
| session | (sin status, usa date) |

### Wikilinks

Usar `[[nombre-nota]]` para vincular notas entre sí. Especialmente:
- Tareas → Proyecto: `project: "[[nombre-proyecto]]"`
- Knowledge → Proyectos donde aplica: `projects: ["[[proyecto-1]]", "[[proyecto-2]]"]`
- Sesiones → Proyecto: `project: "[[nombre-proyecto]]"`

### Vistas .base

Los archivos `.base` son vistas dinámicas sobre las notas. No almacenan datos — filtran y muestran notas existentes por tags, frontmatter, carpetas. Embeber en markdown con `![[archivo.base]]`.

### Crear notas nuevas

1. Usar los templates en `plantillas/` como punto de partida
2. Nombrar archivos en kebab-case: `mi-nueva-tarea.md`
3. Siempre incluir frontmatter completo
4. Vincular a notas relacionadas con wikilinks

## Protocolo de sincronización

Cuando se trabaja desde un proyecto externo que apunta a este vault:

1. **Al inicio**: Leer `proyectos/<proyecto>.md` (incluye sección "Contexto activo")
2. **Durante**: Si se descubre conocimiento reutilizable, crear nota en `conocimiento/`
3. **Al final**:
   - Actualizar la sección "Contexto activo" en `proyectos/<proyecto>.md`
   - Crear entrada en `sesiones/` si la sesión fue significativa
