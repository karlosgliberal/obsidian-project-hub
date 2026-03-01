---
tags:
  - project
status: active
stack: "Ubuntu 24.04 LTS, Hetzner Dedicated, RAID1, LVM, Claude Code, Beats CLI"
repo: "/Users/wintermute/src/experimentos/rescate"
date: 2026-02-28
---

# Sharess

## Qué es

Sistema agéntico de desarrollo de software. Una "máquina de loops" que encadena agentes de IA para construir software de forma autónoma. El objetivo es crear un sistema donde un **Arquitecto** (agente orquestador con IA) coordina múltiples sesiones de Claude Code, cada una resolviendo una épica completa, con un panel de tareas (Beats) como única fuente de la verdad.

El proyecto nace de una sesión de diseño entre 3 personas (Silvetti/Cilveti, Isma, Carlos) el 28 de febrero de 2026, y se implementará sobre un servidor dedicado en Hetzner.

## Arquitectura

### Componentes principales

1. **El Arquitecto** — Agente orquestador supremo. Se despierta por triggers (épica completada) o por cron. Lee el alma del proyecto + estado actual, decide qué épica ejecutar a continuación, puede crear nuevas épicas o reordenar prioridades. Solo una instancia simultánea.

2. **Agente Ejecutor** — Sesión de Claude Code que resuelve una épica completa (1 loop = 1 épica). Valida su propio trabajo (tests, navegador, etc.). No necesita supervisión.

3. **Beats CLI** — Sistema de gestión de tareas basado en Git. Soporta épicas, tareas, subtareas, dependencias, estados. Es la fuente de la verdad absoluta del proyecto.

4. **Panel de visualización** — Lee de Beats, modo solo lectura ("modo héroe": los humanos observan pero no tocan). Muestra señales rojas si el Arquitecto diverge del plan.

5. **Sistema de triggers** — Eventos que despiertan al Arquitecto: épica completada, sesión rota/ciclada, cron periódico.

### Flujo

```
[HUMANO] → Especificación + alma → Descomposición en épicas (Beats) → INICIAR
    ↓
[EJECUTOR] → Sesión Claude Code → Épica → Subtareas → Validación → Done
    ↓
[TRIGGER] → Épica completada → Despierta al Arquitecto
    ↓
[ARQUITECTO] → Lee alma + estado → Revisa resultado → Decide siguiente → Lanza nueva sesión
    ↓
[EJECUTOR] → Nueva épica → (ciclo se repite)

[CRON] → Cada X min → Chequea sesiones rotas → Actúa si necesario
```

### Dos capas de contrato

- **Alma/Intención (inmutable):** Propósito fundamental del proyecto. No se puede cambiar automáticamente.
- **Especificaciones (mutables por el Arquitecto):** El cómo. El Arquitecto puede cambiarlas pero debe documentar y señalizar.

## Stack técnico

- **Infraestructura:** Servidor dedicado Hetzner (Intel i7-7700, 64GB RAM, 2x256GB SSD RAID1)
- **SO:** Ubuntu 24.04.4 LTS (Noble Numbat), kernel 6.8.0-101
- **Almacenamiento:** LVM sobre RAID1 (root 50G, swap 8G, home 20G, var 100G, ~60G libre)
- **Orquestación:** Claude Code CLI, scripts bash, cron
- **Gestión de tareas:** Beats CLI (base de datos basada en Git)
- **Agentes:** Claude Code sessions (una por épica)

## Decisiones clave

1. **El panel (Beats) es la única fuente de la verdad** — Todo gira en torno al estado de las tareas en Beats.
2. **Modo héroe** — Los humanos no intervienen una vez arrancado. Si la máquina muere, se para todo y se rebobina.
3. **Un loop = una épica = una sesión de Claude Code** — Hipótesis a validar.
4. **El agente ejecutor valida su propio trabajo** — No hay segunda capa de revisión dentro del loop.
5. **Solo UN Arquitecto, nunca múltiples instancias** — Decisiones contradictorias serían catastróficas.
6. **El Arquitecto trabaja con tools abstractas** — No lanza CLIs directamente, usa herramientas como `lanzar_nueva_epica`.
7. **RAID1 + LVM en el servidor** — Redundancia de disco + flexibilidad de particionado.
8. **SSH solo por clave** — Password deshabilitado. 3 usuarios autorizados.

## Infraestructura

### Servidor Hetzner

| Componente | Detalle |
|-----------|---------|
| IP | `94.130.50.146` |
| Acceso SSH | `ssh hero` (alias configurado) |
| CPU | Intel Core i7-7700 @ 3.60GHz (8 threads) |
| RAM | 64 GB DDR4 |
| Discos | 2x Micron 1100 SSD 256GB (RAID1) |
| OS | Ubuntu 24.04.4 LTS |
| Kernel | 6.8.0-101-generic |

### Usuarios SSH autorizados

- `karlosgliberal@gmail.com` (ed25519)
- `cilveti@MacBook-Pro-de-IT.local` (rsa)
- `ismael@mp002` (rsa)

### Particionado LVM

| LV | Mount | Size |
|----|-------|------|
| root | / | 50G |
| swap | swap | 8G |
| home | /home | 20G |
| var | /var | 100G |
| (libre) | — | ~60G |

## Proyectos relacionados

Proyecto piloto para validar el sistema: app de "localizar gente para jugar a rol".

## Comandos

```bash
# Acceso al servidor
ssh hero

# Verificar RAID
ssh hero 'cat /proc/mdstat'

# Ver estado LVM
ssh hero 'lvs'

# Ver espacio libre en LVM
ssh hero 'vgdisplay vg0 | grep Free'
```

## Contexto activo

### Sesión 2026-02-28

**Infraestructura completada:**
- Servidor Hetzner provisionado e instalado con Ubuntu 24.04.4 LTS
- RAID1 configurado y sincronizado
- LVM configurado (root 50G, swap 8G, home 20G, var 100G, ~60G libre)
- Sistema actualizado (kernel 6.8.0-101)
- 3 claves SSH autorizadas (karlos, cilveti, ismael)
- Firewall ufw activo (SSH, HTTP, HTTPS)
- Usuario `sagaz` creado con sudo NOPASSWD, grupos docker/adm/systemd-journal
- Docker 29.2.1 + Compose v5.1.0 instalado
- Nginx 1.24.0 instalado
- Claude Code 2.1.63 instalado (para usuario sagaz)
- Lingering habilitado (servicios corren sin sesión)
- build-essential, tmux, jq, ripgrep, git instalados

**Investigación de orquestación completada:**
- Analizados: OpenHands, Hermes Agent, OpenClaw, Claude Agent SDK, CrewAI, LangGraph, AutoGen, Temporal, Restate
- Documento completo: `investigacion-arquitecto-agente.md` en el repo
- Decisión clave: usar Claude Agent SDK (Python) + flock para singleton + cron+evento dual trigger
- El Arquitecto será un proceso corto (no daemon): despierta → revisa → decide → lanza worker → duerme
- Patrón "cheap check then escalate": no despertar LLM si no hay cambios (bash primero, LLM solo si necesario)

## Demo del loop (validada)

Demo funcional del patrón flock + cron + evento en `sagaz@hero:/home/sagaz/sharess-demo/`.

- **Patrón**: [[flock-cron-event-loop]]
- **Resultado**: 3 épicas simuladas ejecutadas automáticamente, 4 wakeups, singleton verificado
- **Componentes validados**: flock singleton, event-driven triggers, cron safety net, cheap health-check, estado persistente JSON, monitor event-driven (`tail -f`)
- **Scripts**: `architect.sh`, `architect-brain.sh`, `worker.sh`, `health-check.sh`, `monitor.sh`, `reset.sh`

```bash
# Ejecutar demo completa con monitor visual
ssh sagaz@hero '/home/sagaz/sharess-demo/monitor.sh --run'
```

## Integración Claude Code en Arquitecto (validada)

**Fecha:** 2026-02-28

Se integró Claude Code CLI (`claude -p`) directamente dentro del `architect-brain.sh` como cerebro real del Arquitecto. Validaciones:

- **Claude Code autenticado** en el servidor como usuario `sagaz` (path: `/home/sagaz/.local/bin/claude`, v2.1.63)
- **Llamada síncrona**: `architect-brain.sh` llama a `claude -p` de forma síncrona — el flock se retiene durante toda la sesión de Claude Code
- **Singleton verificado con LLM real**: mientras Claude Code trabaja dentro del Arquitecto, un segundo intento de lanzar el Arquitecto recibe `SKIP: otro arquitecto ya está corriendo`
- **Cron configurado y probado**: `* * * * *` en crontab de `sagaz` (desactivado tras la prueba para no interferir)
- **Output de Claude**: guardado en `logs/claude-session.txt` — Claude fue capaz de analizar el proyecto y proponer acciones

### Arquitectura actual del brain

```
architect.sh (flock singleton)
  └── health-check.sh (cheap, sin LLM)
       └── architect-brain.sh
            └── claude -p "prompt" --output-format text --max-turns 5
                (síncrono, flock retenido durante toda la sesión)
```

## Repositorio unificado

**Repo:** https://github.com/cilvet/panel-hero
**Local:** `/Users/wintermute/src/experimentos/rescate/panel-hero`

Repositorio donde se unifica el trabajo del proyecto.

## Próximos pasos

- [x] ~~Configurar firewall (ufw)~~
- [x] ~~Crear usuario no-root para operaciones~~
- [x] ~~Investigar OpenHands/Hermes para patrones de orquestación~~
- [x] ~~Crear script architect.sh con flock + cron (Fase 1 MVP — demo sin LLM)~~
- [x] ~~Integrar Claude Code CLI en architect-brain.sh (llamada síncrona con flock)~~
- [x] ~~Verificar singleton con sesión real de Claude Code~~
- [x] ~~Configurar cron en servidor~~
- [ ] Instalar Claude Agent SDK (Python) en el servidor
- [ ] Instalar y configurar Beats CLI
- [ ] Investigar hooks nativos de Beats
- [ ] Crear architect-brain.py con Claude Agent SDK (Fase 2 — reemplazar bash por Python)
- [ ] Diseñar prompts del Arquitecto (soul.md, review-epic.md)
- [ ] Definir alma/intención del proyecto piloto
- [ ] Crear especificación/PRD del proyecto piloto
