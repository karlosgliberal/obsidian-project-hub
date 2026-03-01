---
description: >
  Conocimiento del vault de Obsidian para gestión de notas, tareas, patrones y conocimiento.
  Se activa cuando el usuario menciona vault, obsidian, tareas del vault, patrones,
  conocimiento, "guardar como patrón", "crear tarea en el vault", "registrar decisión",
  o cualquier referencia a la gestión de conocimiento centralizado.
  También se activa con indicaciones de fin de sesión: "hemos terminado", "cerramos",
  "fin de sesión", "wrap up", "ya está", "por hoy vale", "dejamos aquí",
  "terminamos por hoy", "session end".
---

# Vault Knowledge — Skill de gestión del vault

Tienes acceso a un vault de Obsidian centralizado que funciona como base de conocimiento transversal entre proyectos.

## Ubicación del vault

1. **Preferido**: Lee la ruta desde `.claude/rules/vault-sync.md` (línea `Ruta: <path>`). El nombre del proyecto está en la línea `[[<nombre>]]`.
2. **Fallback**: Lee `${CLAUDE_PLUGIN_ROOT}/config/vault-path.txt`. Usa el nombre del directorio actual como proyecto.
3. Si ninguno existe, pide al usuario que ejecute `/vault-manager:vault-init`.

## 4 tipos de nota

### Project
```yaml
tags: [project]
status: active | paused | archived
stack: "..."
repo: "/ruta/absoluta"
date: YYYY-MM-DD
```
Ubicación: `${VAULT}/proyectos/`

### Task
```yaml
tags: [task]
project: "[[nombre-proyecto]]"
status: open | done
created: YYYY-MM-DD
```
Ubicación: `${VAULT}/tareas/`

### Session
```yaml
tags: [session]
project: "[[nombre-proyecto]]"
date: YYYY-MM-DD
```
Ubicación: `${VAULT}/sesiones/`

### Knowledge
```yaml
tags: [knowledge]
projects: ["[[proyecto-1]]"]
status: draft | validated
date: YYYY-MM-DD
```
Ubicación: `${VAULT}/conocimiento/`

## Operaciones

- **Crear tarea**: nota en `${VAULT}/tareas/<nombre-kebab>.md` con frontmatter de task
- **Crear conocimiento**: nota en `${VAULT}/conocimiento/<nombre-kebab>.md` con frontmatter de knowledge

## Sync de fin de sesión

Cuando el usuario indique que quiere terminar la sesión (frases como "hemos terminado",
"cerramos", "por hoy vale", "dejamos aquí", etc.):

1. Si hubo trabajo significativo, sugiere ejecutar `/vault-manager:vault-sync`
2. Si no hubo trabajo significativo, simplemente despídete

## Importante

- **No modifiques** archivos `.base` sin que el usuario lo pida.
- **No modifiques** `CLAUDE.md` del vault.
- **Siempre** verifica que el frontmatter es YAML válido antes de escribir.
- Los placeholders `{{date}}` y `{{title}}` solo existen en plantillas; en notas reales usa valores concretos.
