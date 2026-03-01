---
tags:
  - project
status: active
stack: "Obsidian, Claude Code, Markdown, YAML, Bash, JSON"
repo: "/Users/wintermute/src/experimentos/obsidian-project-hub"
date: 2026-02-25
---

# Obsidian Project Hub

## Qué es

Vault centralizado de conocimiento para todos los proyectos gestionados con Claude Code. Actúa como hub transversal que consolida documentación, decisiones arquitectónicas, patrones reutilizables y aprendizajes de debugging entre múltiples proyectos.

Incluye el plugin `vault-manager` para Claude Code que automatiza la inicialización de proyectos (`/vault-init`) y la sincronización de sesiones de trabajo (`/vault-sync`).

## Arquitectura

- **Vault template**: Estructura de directorios con plantillas estándar para 4 tipos de nota (project, task, session, knowledge)
- **vault-manager plugin**: Plugin de Claude Code con commands (`vault-init`, `vault-sync`), agents (`vault-analyzer`), skills (`vault-knowledge`) y hooks (recordatorio de sync al cerrar sesión)
- **Obsidian Skills**: Skills oficiales de Kepano integrados (obsidian-markdown, obsidian-bases, json-canvas, obsidian-cli, defuddle)
- **Vistas dinámicas**: Archivos `.base` para visualizar tareas como kanban/tabla sin duplicar datos
- **Dashboard central**: `dashboard.md` con tabla de proyectos y vistas embebidas

## Stack técnico

- **Obsidian** — Plataforma PKM con wikilinks, backlinks y graph view
- **Claude Code** — Integración via plugin architecture (commands, agents, skills, hooks)
- **Markdown/YAML** — Formato de notas con frontmatter estructurado
- **Obsidian Bases** — Vistas dinámicas tipo base de datos sobre notas
- **Blue Topaz Theme** — Tema preconfigurado

## Decisiones clave

- **4 tipos de nota** (v2.0): project, task, session, knowledge — reemplazando 7 tipos originales (project, task, epic, pattern, decision, session, exploration)
- **Conocimiento flat**: `conocimiento/` sin subdirectorios, diferenciado por tags
- **Tareas mínimas**: captura rápida con status `open`/`done`, sin priority ni type
- **Detección de proyecto via regla**: vault-sync lee `.claude/rules/vault-sync.md` en contexto en vez de depender solo de `vault-path.txt`
- **Frontmatter obligatorio**: Toda nota requiere YAML frontmatter con `tags`, `date` y `status` para filtrado automático
- **Wikilinks como grafo de conocimiento**: `[[nota]]` crea backlinks y permite descubrimiento de conexiones entre proyectos

## Comandos

```bash
# Conectar proyecto al vault
/vault-manager:vault-init

# Sincronizar sesión con el vault
/vault-manager:vault-sync

# Abrir en Obsidian
open /Users/wintermute/src/experimentos/obsidian-project-hub
```

## Contexto activo

- v2.0.0 — Simplificación mayor: 7 tipos → 4, taxonomía plana, tareas mínimas, fix detección de proyecto
- Plugin vault-manager actualizado con nueva taxonomía y detección de proyecto via regla
- 5 proyectos conectados: obsidian-project-hub, json-render-canvas, video-canvas, arginago, sharess
- 4 notas de conocimiento migradas a `conocimiento/` (flat)

## Próximos pasos

- Validar flujo vault-sync desde proyecto externo tras actualización a v2.0.0
- Re-ejecutar `/vault-init` en proyectos existentes para actualizar protocolo en reglas locales (opcional)
- Evaluar integración con Obsidian CLI para operaciones automatizadas
