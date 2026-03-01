---
tags:
  - knowledge
projects:
  - "[[obsidian-project-hub]]"
date: 2026-02-25
---

# Ciclo de actualización de plugins en Claude Code

## Problema encontrado

Después de modificar ficheros del plugin (hooks, skills) y hacer push al repo de GitHub, ejecutar `/plugin update vault-manager@obsidian-project-hub` no descargaba los cambios. El plugin seguía usando la versión cacheada.

## Investigación

- Los plugins instalados se copian a `~/.claude/plugins/cache/<marketplace>/<plugin>/<version>/`
- El sistema usa el campo `version` de `plugin.json` para detectar si hay cambios
- Sin bump de versión, Claude Code asume que no hay nada nuevo y usa el cache
- El marketplace local (en `~/.claude/plugins/marketplaces/`) es un clone del repo de GitHub

## Solución

**El flujo correcto para publicar una actualización es:**

1. Hacer los cambios en los ficheros del plugin
2. **Bumpar la versión** en `vault-manager/.claude-plugin/plugin.json` (semver: patch/minor/major)
3. **Bumpar también** en `.claude-plugin/marketplace.json` para consistencia
4. Commit + push a GitHub
5. Desde Claude Code: `/plugin` → pestaña **Installed** → seleccionar el plugin → update
6. **Reiniciar la sesión** de Claude Code para que aplique los cambios

### Ejemplo de bump

```json
// plugin.json
{ "version": "1.0.0" }  →  { "version": "1.1.0" }

// marketplace.json
{ "version": "1.0.0" }  →  { "version": "1.1.0" }
```

## Lección aprendida

- Los plugins de Claude Code usan **versionado semántico obligatorio** para detectar actualizaciones
- Sin bump de versión → sin actualización, sin importar cuántos commits hagas
- El comando `/plugin update <name>` no es equivalente a `git pull` — solo descarga si detecta versión nueva
- Después de actualizar, **hay que reiniciar la sesión** para que los cambios apliquen
