---
tags:
  - pattern
projects:
  - "[[obsidian-project-hub]]"
status: validated
date: 2026-02-25
---

# Vault centralizado con plugin de sincronización

## Problema

Al trabajar con Claude Code en múltiples proyectos, el conocimiento generado (decisiones, patrones, aprendizajes, estado del trabajo) se pierde entre sesiones y queda aislado en cada proyecto. No hay forma nativa de acumular y cruzar conocimiento entre proyectos.

## Solución

Un vault de Obsidian centralizado que actúa como base de conocimiento transversal, combinado con un plugin de Claude Code que automatiza la conexión y sincronización entre proyectos y el vault.

### Componentes

1. **Vault de Obsidian** — estructura estandarizada con directorios para proyectos, tareas, sesiones, patrones, decisiones, aprendizajes y exploraciones
2. **Vistas `.base`** — Kanban de tareas, tabla por proyecto, tracker de épicas — se actualizan automáticamente al crear notas
3. **Plugin vault-manager** con:
   - `/vault-init` — analiza un proyecto y crea su entrada en el vault + regla de sync local
   - `/vault-sync` — al final de sesión, actualiza estado del proyecto, crea log de sesión, gestiona tareas
   - Skill `vault-knowledge` — se activa automáticamente cuando se menciona el vault
   - Agente `vault-analyzer` — análisis profundo del proyecto para la inicialización
   - Stop hook — recordatorio de sync al salir de una sesión

### Flujo

```
Proyecto A ──vault-init──► Vault centralizado ◄──vault-init── Proyecto B
     │                          │                        │
     └──vault-sync──►  sesiones/, tareas/,  ◄──vault-sync──┘
                        conocimiento/
```

## Ejemplo

```bash
# Desde un proyecto cualquiera
/vault-manager:vault-init    # Primera vez: pide ruta al vault, analiza proyecto
# ... trabajar normalmente ...
/vault-manager:vault-sync    # Registra sesión, actualiza tareas, captura patrones
```

El vault en Obsidian muestra todo visualmente: graph view conecta proyectos con tareas y sesiones, el dashboard agrupa tareas por proyecto en Kanban.

## Cuándo usar

- Trabajas con Claude Code en 2+ proyectos
- Quieres acumular patrones y decisiones entre proyectos
- Necesitas contexto de sesiones anteriores al retomar trabajo
- Usas Obsidian como herramienta de PKM (Personal Knowledge Management)

## Cuándo NO usar

- Proyecto único sin intención de escalar
- Equipo que ya tiene herramienta de knowledge management (Notion, Confluence)
- Si no usas Obsidian — el vault depende de sus features (wikilinks, .base views, graph)
