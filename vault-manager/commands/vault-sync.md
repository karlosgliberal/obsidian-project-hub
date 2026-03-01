---
description: Sincronizar sesión actual con el vault de conocimiento
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git:*)
---

# /vault-sync — Sincronizar sesión con el vault

Actualiza el vault de conocimiento con el trabajo realizado en la sesión actual.

## Instrucciones

Sigue estos pasos en orden:

### 1. Configuración

Determina la ruta del vault y el nombre del proyecto:

1. **Preferido**: Busca en el contexto actual la regla `.claude/rules/vault-sync.md` (ya cargada en sesión). Extrae:
   - Línea `Ruta: <path>` → usa como `VAULT`
   - Línea `[[<nombre>]]` → usa como nombre del proyecto
2. **Fallback**: Si no hay regla en contexto, busca `${CLAUDE_PLUGIN_ROOT}/config/vault-path.txt` y usa el nombre del directorio actual como proyecto.
3. Si ninguno existe, informa al usuario y sugiere ejecutar `/vault-manager:vault-init`. Detén la ejecución.

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
  - Conocimiento reutilizable identificado
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
- `## Descubrimientos` — aprendizajes, conocimiento identificado
- `## Próximos pasos` — qué queda pendiente

Si ya existe una sesión para hoy y este proyecto, actualízala en lugar de crear una nueva.

### 5. Capturar tareas (opt-in)

Pregunta al usuario si quiere registrar ideas o tareas surgidas durante la sesión.

- Si dice sí: crea notas simples en `${VAULT}/tareas/` con frontmatter mínimo (`tags: [task]`, `project`, `status: open`, `created`).
- Si dice no: salta este paso.
- NO busques ni modifiques tareas existentes. NO hagas transiciones automáticas de status.

### 6. Capturar conocimiento (si aplica)

Si durante la sesión se identificó conocimiento reutilizable (patrones, aprendizajes, decisiones, investigaciones):

- Crea nota en `${VAULT}/conocimiento/` usando `${VAULT}/plantillas/knowledge.md` con tag `knowledge`.
- Solo crea si hay contenido genuinamente reutilizable. No fuerces la creación.

### 7. Resumen

Muestra al usuario un resumen de todo lo actualizado:

- Cambios en `proyectos/<nombre-proyecto>.md`
- Sesión creada/actualizada
- Tareas nuevas creadas (si las hubo)
- Conocimiento capturado (si lo hubo)
