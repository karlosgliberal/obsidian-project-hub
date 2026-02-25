# Obsidian Project Hub

Vault centralizado de conocimiento para todos los proyectos gestionados con Claude Code.

## Estructura del Vault

```
vault/
├── proyectos/        # Un directorio por proyecto gestionado
├── conocimiento/     # Conocimiento transversal
│   ├── patrones/     # Patrones reutilizables entre proyectos
│   ├── decisiones/   # ADRs (Architecture Decision Records) globales
│   └── aprendizajes/ # Aprendizajes de debugging, investigación, etc.
├── tareas/           # Tareas cross-project (.md) + vistas (.base)
│   └── epicas/       # Épicas como .md
├── exploraciones/    # Investigaciones y research
├── sesiones/         # Logs de sesiones de trabajo
└── plantillas/       # Templates de Obsidian para notas nuevas
```

## Convenciones

### Frontmatter obligatorio

Toda nota debe tener frontmatter YAML con al menos:

```yaml
---
tags:
  - <tipo>           # task, epic, pattern, decision, session, exploration, project
date: YYYY-MM-DD
status: <estado>     # depende del tipo (ver abajo)
---
```

### Taxonomía de tags

- `project` — Entrada de proyecto
- `task` — Tarea individual
- `epic` — Épica con subtareas
- `pattern` — Patrón reutilizable
- `decision` — Decisión arquitectónica (ADR)
- `session` — Log de sesión de trabajo
- `exploration` — Investigación/research

Tags adicionales opcionales para categorización: `claude-code`, `obsidian`, `react`, `ai`, etc.

### Status por tipo

| Tipo | Valores posibles |
|------|-----------------|
| task | `open`, `in_progress`, `done`, `blocked` |
| epic | `open`, `in_progress`, `done` |
| project | `active`, `paused`, `archived` |
| exploration | `draft`, `in_progress`, `complete` |
| pattern | `draft`, `validated` |
| session | (sin status, usa date) |

### Wikilinks

Usar `[[nombre-nota]]` para vincular notas entre sí. Especialmente:
- Tareas → Proyecto: `project: "[[nombre-proyecto]]"`
- Patrones → Proyectos donde aplican: `projects: ["[[proyecto-1]]", "[[proyecto-2]]"]`
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
2. **Durante**: Si se descubre un patrón, crear nota en `conocimiento/patrones/`
3. **Al final**:
   - Actualizar la sección "Contexto activo" en `proyectos/<proyecto>.md`
   - Crear entrada en `sesiones/` si la sesión fue significativa
   - Actualizar tareas en `tareas/` si cambió su status
