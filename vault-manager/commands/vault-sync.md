---
description: Sincronizar sesión actual con el vault de conocimiento
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git:*)
---

# /vault-sync — Sincronizar sesión con el vault

Actualiza el vault de conocimiento con el trabajo realizado en la sesión actual.

## Instrucciones

Sigue estos pasos en orden:

### 1. Configuración

Determina la ruta del vault:

1. Busca `${CLAUDE_PLUGIN_ROOT}/config/vault-path.txt`.
2. Si existe, lee la ruta del vault desde ese archivo. Usa esa ruta como `VAULT`.
3. Si NO existe, informa al usuario que el vault no está configurado y sugiere ejecutar `/vault-manager:vault-init` primero. Detén la ejecución.

Detecta el nombre del proyecto desde el directorio actual (kebab-case).

Verifica que existe `${VAULT}/proyectos/<nombre-proyecto>.md`. Si no existe, sugiere ejecutar `/vault-manager:vault-init` primero.

### 2. Analizar la sesión

Recopila información sobre lo que se hizo en esta sesión:

- **Cambios en código**: ejecuta `git diff --stat` y `git log --oneline -10` para ver commits recientes.
- **Archivos modificados**: `git diff --name-only` para el scope del trabajo.
- **Conversación**: revisa el contexto de la conversación actual para extraer:
  - Objetivo de la sesión
  - Qué se logró
  - Decisiones tomadas
  - Problemas encontrados y soluciones
  - Patrones identificados
  - Tareas nuevas surgidas
  - Próximos pasos

### 3. Actualizar proyecto en el vault

Edita `${VAULT}/proyectos/<nombre-proyecto>.md`:

- Actualiza la sección `## Contexto activo` con el estado actual del trabajo tras esta sesión.
- Actualiza `## Próximos pasos` si cambió.
- No sobreescribas información existente que siga siendo válida — acumula o actualiza.

### 4. Crear entrada de sesión

Crea `${VAULT}/sesiones/<fecha>-<nombre-proyecto>.md` usando el formato de `${VAULT}/plantillas/session.md`:

```yaml
---
tags:
  - session
project: "[[<nombre-proyecto>]]"
date: <fecha-hoy>
---
```

Rellena las secciones:
- `## Objetivo` — qué se pretendía hacer
- `## Qué se hizo` — resumen concreto de lo logrado
- `## Decisiones tomadas` — decisiones relevantes con justificación
- `## Descubrimientos` — aprendizajes, bugs, patrones identificados
- `## Próximos pasos` — qué queda pendiente

Si ya existe una sesión para hoy y este proyecto, actualízala en lugar de crear una nueva.

### 5. Gestionar tareas

Revisa las tareas existentes en `${VAULT}/tareas/` que estén vinculadas a este proyecto:

- Si alguna tarea se **completó** en esta sesión → cambia su `status` a `done`.
- Si alguna tarea está **en progreso** → cambia su `status` a `in_progress`.
- Si se **identificaron tareas nuevas** → crea notas nuevas en `${VAULT}/tareas/` con el formato de la plantilla.
- Si alguna tarea se **bloqueó** → cambia su `status` a `blocked` y documenta el motivo.

### 6. Capturar conocimiento (si aplica)

Si durante la sesión se identificaron:

- **Patrones reutilizables** → crea nota en `${VAULT}/conocimiento/patrones/` usando `${VAULT}/plantillas/pattern.md`.
- **Decisiones arquitectónicas** → crea nota en `${VAULT}/conocimiento/decisiones/` con formato ADR.
- **Aprendizajes de debugging** → crea nota en `${VAULT}/conocimiento/aprendizajes/`.

Solo crea estas notas si hay contenido genuinamente reutilizable. No fuerces la creación.

### 7. Resumen

Muestra al usuario un resumen de todo lo actualizado:

- Cambios en `proyectos/<nombre-proyecto>.md`
- Sesión creada/actualizada
- Tareas actualizadas (con status viejo → nuevo)
- Tareas nuevas creadas
- Conocimiento capturado (patrones, decisiones, aprendizajes)
