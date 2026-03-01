---
tags:
  - session
project: "[[obsidian-project-hub]]"
date: 2026-03-01
---

# 2026-03-01 — Simplificación vault-manager v2.0.0

## Objetivo

Simplificar el vault-manager eliminando complejidad innecesaria: reducir de 7 a 4 tipos de nota, aplanar directorios de conocimiento, simplificar gestión de tareas, y arreglar bug de detección de proyecto en vault-sync.

## Qué se hizo

- **Migración de conocimiento**: Movidos 4 ficheros de subdirectorios (`patrones/`, `aprendizajes/`) a `conocimiento/` raíz. Re-etiquetados todos a tag `knowledge`. Eliminados subdirectorios vacíos (`patrones/`, `aprendizajes/`, `decisiones/`).
- **Taxonomía simplificada**: 7 tipos → 4 (project, task, session, knowledge). Actualizado CLAUDE.md, plantillas, SKILL.md.
- **Plantillas**: Creada `knowledge.md`. Simplificada `task.md` (sin priority/type). Eliminadas `epic.md`, `exploration.md`, `pattern.md`.
- **vault-sync fix**: Sección 1 ahora lee vault path y nombre de proyecto desde `.claude/rules/vault-sync.md` (ya en contexto de sesión) como método preferido, con fallback a `vault-path.txt`.
- **vault-sync simplificado**: Sección 5 de tareas cambiada de gestión automática de ciclo de vida a captura opt-in simple.
- **vault-init actualizado**: Template de regla apunta a `conocimiento/` (sin subdirectorio). Tareas opt-in con máximo 3-5 y aprobación del usuario.
- **vault-knowledge reducido**: SKILL.md de ~210 líneas a ~70. Eliminados esquemas obsoletos (epic, exploration, decision, learning, pattern).
- **vault-analyzer**: Tareas limitadas a máximo 5 con nota de que el usuario decide.
- **Dashboard**: Eliminadas secciones de épicas, exploraciones y knowledge base desagregada.

## Decisiones tomadas

- **4 tipos en vez de 7**: Los límites entre exploration/learning/pattern/decision eran difusos. Un solo tipo `knowledge` con tag adicional opcional cubre todos los casos.
- **Tareas opt-in**: vault-sync no debe gestionar ciclos de vida de tareas automáticamente. Captura simple, el usuario decide.
- **Detección via regla**: Leer de `.claude/rules/vault-sync.md` resuelve el bug donde vault-sync no encontraba el proyecto correcto (caso sharess con nombre de directorio diferente).
- **No borrar `tareas/epicas/`**: Queda como legacy. Tareas existentes con `priority`/`type` siguen funcionando.

## Descubrimientos

- El skill vault-knowledge duplicaba ~200 líneas de CLAUDE.md innecesariamente. Con la simplificación, ambos son concisos y complementarios.
- El mecanismo de "leer regla del contexto" es más robusto que parsear ficheros de config porque la regla ya está cargada por Claude Code automáticamente.

## Próximos pasos

- Push + actualizar plugin en proyectos existentes (`/plugin → Installed → vault-manager → Update`)
- Validar vault-sync desde un proyecto externo
- Considerar re-ejecutar vault-init en proyectos para actualizar protocolo en reglas locales
