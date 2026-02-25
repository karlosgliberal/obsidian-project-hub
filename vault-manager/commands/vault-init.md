---
description: Inicializar proyecto actual en el vault de conocimiento
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(git log:*, cat:*, ls:*)
---

# /vault-init — Inicializar proyecto en el vault

Conecta el proyecto del directorio de trabajo actual al vault de conocimiento centralizado.

## Instrucciones

Sigue estos pasos en orden:

### 1. Configuración inicial

Determina la ruta del vault:

1. Busca `${CLAUDE_PLUGIN_ROOT}/config/vault-path.txt`.
2. Si existe, lee la ruta del vault desde ese archivo. Usa esa ruta como `VAULT` en todo el proceso.
3. Si NO existe:
   - Pregunta al usuario la ruta absoluta a su vault de Obsidian.
   - Valida que el directorio proporcionado existe y contiene `CLAUDE.md` y `plantillas/`.
   - Si la validación falla, informa al usuario y vuelve a preguntar.
   - Si la validación pasa, crea `${CLAUDE_PLUGIN_ROOT}/config/vault-path.txt` con la ruta proporcionada.
   - Usa esa ruta como `VAULT`.

### 2. Detectar proyecto

- Nombre del proyecto: usa el nombre del directorio actual (último segmento de `$PWD`), en kebab-case.
- Si existe `CLAUDE.md` en el directorio actual, léelo para obtener contexto adicional.

### 3. Verificar existencia

Comprueba si ya existe `${VAULT}/proyectos/<nombre-proyecto>.md`.

- Si **ya existe**: informa al usuario y pregunta si quiere actualizarlo o abortar.
- Si **no existe**: continúa al paso 4.

### 4. Analizar el proyecto

Lanza el agente `vault-analyzer` para que haga un análisis profundo del proyecto. Este agente devolverá un informe estructurado con:

- Descripción del proyecto
- Stack técnico
- Arquitectura y decisiones clave
- Estado actual y actividad reciente
- Tareas/issues pendientes
- Comandos útiles

### 5. Crear entrada del proyecto en el vault

Con los datos del agente, crea `${VAULT}/proyectos/<nombre-proyecto>.md` siguiendo el formato de la plantilla `${VAULT}/plantillas/project.md`. Asegúrate de:

- Frontmatter completo: `tags: [project]`, `status: active`, `stack`, `repo` (ruta absoluta al proyecto), `date` (hoy)
- Sección `## Qué es` con descripción
- Sección `## Arquitectura` con componentes principales
- Sección `## Stack técnico` con lenguajes, frameworks, dependencias
- Sección `## Decisiones clave` con decisiones arquitectónicas relevantes
- Sección `## Comandos` con comandos de dev/build/test
- Sección `## Contexto activo` con el estado actual del trabajo (esto se actualiza con `/vault-sync`)
- Sección `## Próximos pasos` con lo que queda por hacer

Usa wikilinks `[[nombre]]` para referenciar otros proyectos o notas del vault cuando sea relevante.

### 6. Crear tareas si se detectaron

Si el análisis detectó issues, TODOs, o tareas pendientes, crea una nota por tarea en `${VAULT}/tareas/<nombre-tarea>.md` usando el formato de `${VAULT}/plantillas/task.md`:

```yaml
---
tags:
  - task
project: "[[<nombre-proyecto>]]"
status: open
priority: 2
type: task
created: <fecha-hoy>
---
```

Nombra cada archivo en kebab-case descriptivo. Vincula cada tarea al proyecto con wikilink.

### 7. Crear regla de sincronización en el proyecto local

Crea `.claude/rules/vault-sync.md` en el directorio del proyecto con:

```markdown
---
description: Sincronización con vault de conocimiento
globs: *
---

# Conexión al Vault de Conocimiento

Este proyecto está conectado al vault centralizado.

## Vault
Ruta: ${VAULT}

## Proyecto en vault
[[<nombre-proyecto>]]

## Protocolo
- **Al inicio de sesión**: Lee `${VAULT}/proyectos/<nombre-proyecto>.md` para contexto.
- **Durante el trabajo**: Si descubres un patrón reutilizable, créalo en `${VAULT}/conocimiento/patrones/`.
- **Al final de sesión**: Ejecuta `/vault-sync` para sincronizar.
```

Reemplaza `${VAULT}` con la ruta real y `<nombre-proyecto>` con el nombre real.

### 8. Actualizar dashboard del vault

Edita `${VAULT}/dashboard.md`:

- Añade una fila a la tabla de proyectos: `| [[<nombre-proyecto>]] | Active | <stack> |`
- Si hay tareas nuevas, ya aparecerán automáticamente via los `.base` embebidos.

### 9. Resumen

Muestra al usuario:

- Ficheros creados en el vault (con rutas)
- Ficheros creados en el proyecto local
- Tareas detectadas y registradas
- Cómo usar `/vault-sync` al final de las sesiones
