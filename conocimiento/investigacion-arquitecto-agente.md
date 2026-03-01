---
tags:
  - knowledge
project: "[[sharess]]"
status: complete
date: 2026-02-28
---


# Investigación: Patrones de Orquestación para el Arquitecto Agéntico

**Fecha:** 2026-02-28
**Proyecto:** [[sharess]]
**Objetivo:** Definir cómo implementar un agente "Arquitecto" que despierta, revisa, decide y vuelve a dormir.

---

## 1. Panorama: Quién ha resuelto esto y cómo

### 1.1 OpenHands (antes OpenDevin)

El proyecto más maduro en orquestación de agentes de desarrollo.

**Arquitectura clave:**
- **Event-sourced state**: Todo el estado se deriva de un log de eventos append-only. Permite replay determinista, auditoría y recuperación.
- **Agentes stateless, estado único mutable**: Los agentes son especificaciones inmutables (LLM + tools + prompts). Solo `ConversationState` muta.
- **Loop fundamental**: `Action → Execution → Observation` — todo (tools, delegación, MCP) sigue este contrato.
- **Delegación jerárquica**: Árbol padre-hijo, no mesh peer-to-peer. Un agente padre emite `AgentDelegateAction`, el controller spawna un hijo con su propio contexto.
- **StuckDetector**: Detecta 5 patrones de fallo (ciclos repetitivos, monólogos, errores repetidos, patrones alternantes, errores de context window). Usa comparación **semántica**, no exacta.
- **Pause/Resume**: Emite `PauseEvent`, persiste estado, detiene ejecución. En resume, carga `base_state.json` + replay de eventos.

**Persistencia en disco:**
```
workspace/conversations/<id>/
  base_state.json          # Metadata: status, stats, config, secrets
  events/
    event-00000-<id>.json  # Append-only, un fichero por evento
    event-00001-<id>.json
```

**Relevancia para Sharess:** El modelo event-sourced es robusto pero complejo. Lo que nos interesa es el patrón de StuckDetector y el concepto de delegación jerárquica.

---

### 1.2 Hermes Agent (Nous Research)

Agente CLI autónomo con sistema de memoria de 3 niveles, lanzado febrero 2026.

**Lo que resuelve:**
- **Gateway daemon**: Proceso único que gestiona sesiones, cron, mensajería. Instalable como servicio systemd (`hermes gateway install && hermes gateway start`).
- **Cron integrado**: El gateway tickea cada 60s y ejecuta jobs en sesiones aisladas.
- **Hooks lifecycle**: `gateway:startup`, `session:start`, `agent:start`, `agent:step`, `agent:end`, `command:*`.
- **Memoria de 3 niveles**:
  1. **Session**: Contexto conversacional dentro de una interacción
  2. **Persistent**: `MEMORY.md` + `USER.md` — hechos y preferencias cross-session
  3. **Skill**: Cuando resuelve algo complejo, escribe un `SKILL.md` reutilizable
- **Subagent delegation**: `delegate_task` spawna agentes aislados con su propio contexto y terminal.
- **Terminal backends**: Local, Docker, SSH, Singularity, Modal — cada uno con distinto nivel de aislamiento.

**Relevancia para Sharess:** El patrón gateway + cron + hooks es exactamente lo que necesitamos. La memoria de 3 niveles (session/persistent/skill) mapea bien a nuestro concepto de "alma + intenciones + memoria".

---

### 1.3 OpenClaw

Plataforma de agentes con 150K+ stars. Comparte ADN arquitectónico con Hermes.

**Patrón Heartbeat (supervisor wake/review/decide):**
1. Agente despierta por cron (cada 30min configurable)
2. Lee `HEARTBEAT.md` para instrucciones de monitorización
3. **Cheap checks primero**: Scripts deterministas rápidos (sin coste LLM)
4. **Escala a LLM** solo si algo significativo cambió
5. Envía notificaciones solo si es necesario

**Esto es clave**: El patrón "cheap check → escalate to LLM" ahorra costes enormes. No despiertas al LLM cada 5 minutos si no hay cambios.

**Mission Control**: Dashboard separado con gestión de lifecycle de agentes, workflows de aprobación, audit trails.

---

### 1.4 Claude Code Agent SDK

**Esta es la herramienta de implementación.** Anthropic ofrece un SDK oficial (Python y TypeScript) que da control programático total sobre Claude Code.

```bash
pip install claude-agent-sdk
# o
npm install @anthropic-ai/claude-agent-sdk
```

**API principal — `query()`:**

```python
from claude_agent_sdk import query, ClaudeAgentOptions

async for message in query(
    prompt="Resuelve la épica: implementar auth module",
    options=ClaudeAgentOptions(
        allowed_tools=["Read", "Edit", "Bash", "Glob", "Grep"],
        permission_mode="bypassPermissions",
        system_prompt="Eres un desarrollador senior. Tu épica es...",
        max_turns=20,
        cwd="/home/sagaz/proyecto",
    ),
):
    if hasattr(message, "session_id"):
        session_id = message.session_id
    if hasattr(message, "result"):
        print(message.result)
```

**Capacidades críticas:**
- **Session chaining**: Capturar `session_id` y pasar `resume=session_id` para mantener contexto
- **Subagents programáticos**: Definir agentes especializados que el principal puede spawnar
- **Hooks**: `PreToolUse`, `PostToolUse`, `Stop`, `SessionStart`, `SessionEnd` — para guardrails
- **MCP tools custom**: Crear tools in-process sin overhead de subproceso
- **Structured output**: `--json-schema` para output machine-parseable

**CLI headless equivalente (`-p`):**

```bash
# Ejecutar épica
claude -p "Resuelve esta épica: ..." \
  --dangerously-skip-permissions \
  --output-format json \
  --max-turns 20 \
  --allowedTools "Read,Edit,Bash,Glob,Grep" \
  --append-system-prompt "Contexto del proyecto: ..."

# Continuar sesión
session_id=$(claude -p "..." --output-format json | jq -r '.session_id')
claude -p "Continúa con..." --resume "$session_id"

# Budget limit como safety rail
claude -p "..." --max-budget-usd 5.00 --max-turns 30
```

**Dato crítico para containers/headless:**
```bash
# Bypass del wizard interactivo (obligatorio en headless)
echo '{"hasCompletedOnboarding": true}' > ~/.claude.json
```

---

### 1.5 Otros frameworks relevantes

| Framework | Patrón | Relevancia |
|-----------|--------|-----------|
| **CrewAI** | Flow (orquestador) + Crew (workers). El Flow gestiona estado macro; el Crew ejecuta micro. | Mapeo directo: Flow = Arquitecto, Crew = Worker |
| **LangGraph Supervisor** | Grafo dirigido donde nodos son agentes y edges son transiciones. Supervisor con edges a todos los workers. | El estado como grafo es inspectable y debuggeable |
| **AutoGen/Magentic** | Manager agent con task ledger. Delega, monitoriza, detecta stalls, adapta plan. | Exactamente nuestro Arquitecto |
| **Temporal** | Durable execution: Signals (eventos async), Schedules (cron), Singleton Workflows. | El más robusto para producción, pero añade infraestructura |
| **Restate** | "Virtual state machines that wake up, react, sleep". Cero compute mientras duerme. | El concepto ideal, implementación en TypeScript |

---

## 2. Patrones de Implementación

### 2.1 Singleton: Solo UN Arquitecto

**Recomendación: `flock`** (file locking del kernel)

```bash
#!/bin/bash
LOCKFILE=/var/run/rescate/architect.lock
exec 200>"$LOCKFILE"
flock -n 200 || { echo "Arquitecto ya corriendo"; exit 0; }

# Lock adquirido — somos la instancia única
echo $$ > "$LOCKFILE"
# ... lógica del arquitecto ...
# Lock se libera automáticamente al salir (incluso con SIGKILL)
```

**Por qué flock y no PID files:**

| Método           | Race-safe   | Auto-cleanup en crash | Funciona con cron |
| ---------------- | ----------- | --------------------- | ----------------- |
| PID file solo    | No (TOCTOU) | No (PID huérfano)     | Sí                |
| **flock**        | **Sí**      | **Sí**                | **Sí**            |
| systemd oneshot  | Sí          | Sí                    | Via timer         |
| DB advisory lock | Sí          | Sí                    | Para distribuido  |

### 2.2 Triggers: Cron + Evento (Dual)

Ambos intentan adquirir el mismo flock. Es elegante y race-condition-free:

```
┌─────────────────┐     ┌──────────────────────┐
│  Cron            │     │  Worker completado    │
│  (cada 5 min)    │     │  (trigger evento)     │
└────────┬────────┘     └──────────┬───────────┘
         │                         │
         ▼                         ▼
    ┌────────────────────────────────────┐
    │  flock -n /run/rescate/arch.lock   │
    │  -c '/path/to/architect.sh'        │
    └────────────────────────────────────┘
         │
         │  Lock acquired? → SÍ: ejecutar
         │                 → NO: salir silenciosamente
```

**Cron:**
```cron
*/5 * * * * flock -n /var/run/rescate/architect.lock -c '/usr/local/bin/architect.sh --trigger=cron'
```

**Worker completion hook:**
```bash
# Al final del worker:
echo '{"epic_id":42,"status":"done"}' > /var/run/rescate/events/epic-42.done
flock -n /var/run/rescate/architect.lock -c '/usr/local/bin/architect.sh --trigger=event'
```

### 2.3 Estado del Arquitecto (el "Alma" persistente)

El Arquitecto es un **proceso corto** (no un daemon). Despierta, carga estado, decide, actúa, persiste estado, sale.

```json
{
  "version": 1,
  "current_state": "sleeping",
  "last_wake": "2026-02-28T10:30:00Z",

  "soul": {
    "project_vision": "Sistema agéntico de desarrollo...",
    "immutable_intent": "Crear una máquina de loops autónoma...",
    "architecture_decisions": []
  },

  "completed_epics": [
    {"id": 1, "summary": "...", "outcome": "success", "learnings": "..."}
  ],

  "current_worker": {
    "epic_id": 3,
    "pid": 12345,
    "session_id": "uuid-...",
    "started_at": "2026-02-28T10:30:15Z"
  },

  "backlog": [
    {"id": 4, "description": "...", "priority": 1},
    {"id": 5, "description": "...", "priority": 2}
  ],

  "stuck_detection": {
    "max_worker_duration_minutes": 60,
    "consecutive_failures": 0,
    "max_consecutive_failures": 3
  },

  "timeline": [
    {"ts": "...", "event": "woke_up", "trigger": "cron", "action": "checked_worker_health"},
    {"ts": "...", "event": "woke_up", "trigger": "event", "action": "reviewed_epic_42"}
  ]
}
```

### 2.4 Máquina de Estados del Arquitecto

```
          ┌──────────┐
          │ SLEEPING  │◄──────────────────────┐
          └────┬─────┘                        │
               │ trigger (evento/cron)        │
          ┌────▼─────┐                        │
          │ WAKING   │                        │
          │ (acquire │                        │
          │  flock)  │                        │
          └────┬─────┘                        │
               │ lock acquired                │
          ┌────▼──────┐                       │
          │ REVIEWING │                       │
          │ • cargar state.json               │
          │ • si cron: check worker health    │
          │ • si event: revisar epic result   │
          └────┬──────┘                       │
               │                              │
          ┌────▼──────┐                       │
          │ DECIDING  │                       │
          │ • evaluar si épica OK             │
          │ • elegir siguiente épica          │
          │ • puede crear épicas nuevas       │
          │ • puede reordenar backlog         │
          └────┬──────┘                       │
               │                              │
          ┌────▼──────────┐                   │
          │ LAUNCHING     │                   │
          │ • spawn worker│                   │
          │ • persist state                   │
          └────┬──────────┘                   │
               │                              │
               └──────────────────────────────┘
                 (release flock → sleep)
```

---

## 3. Arquitectura Recomendada para Sharess

### 3.1 Composición de ficheros

```
/home/sagaz/sharess/
├── architect/
│   ├── architect.sh          # Entry point (flock + dispatch)
│   ├── architect-brain.py    # Lógica LLM (usa Claude Agent SDK)
│   ├── health-check.sh       # Cheap check sin LLM
│   ├── state.json            # El "alma" persistente
│   └── prompts/
│       ├── soul.md            # Intención inmutable del proyecto
│       ├── review-epic.md     # Prompt para revisar una épica
│       └── decide-next.md     # Prompt para decidir siguiente paso
├── worker/
│   ├── worker.sh             # Lanza claude -p con contexto
│   └── templates/
│       └── epic-prompt.md    # Template de prompt por épica
├── events/                   # Completion markers
├── results/                  # Output de cada épica
│   ├── epic-1/
│   └── epic-2/
└── logs/
    ├── architect.log
    └── workers/
```

### 3.2 Flujo concreto

```bash
# architect.sh — Entry point
#!/bin/bash
set -euo pipefail
LOCKFILE=/var/run/rescate/architect.lock
STATEFILE=/home/sagaz/sharess/architect/state.json
TRIGGER="${1:---trigger=manual}"

exec 200>"$LOCKFILE"
flock -n 200 || exit 0

# Cheap check primero (sin LLM)
source /home/sagaz/sharess/architect/health-check.sh
if ! needs_attention; then
    exit 0  # Nada que hacer, volver a dormir
fi

# Despertar al cerebro (con LLM)
python3 /home/sagaz/sharess/architect/architect-brain.py \
    --state "$STATEFILE" \
    --trigger "$TRIGGER"
```

```python
# architect-brain.py — Lógica con Claude Agent SDK
import asyncio, json
from claude_agent_sdk import query, ClaudeAgentOptions

async def main(state_file, trigger):
    state = json.load(open(state_file))
    soul = open("prompts/soul.md").read()

    # Construir prompt contextual
    prompt = f"""
    {soul}

    ## Estado actual
    {json.dumps(state, indent=2)}

    ## Trigger: {trigger}

    Revisa el estado, decide la siguiente acción, y responde en JSON.
    """

    async for msg in query(
        prompt=prompt,
        options=ClaudeAgentOptions(
            allowed_tools=["Read", "Glob", "Grep", "Bash"],
            max_turns=5,
            cwd="/home/sagaz/proyecto",
        ),
    ):
        if hasattr(msg, "result"):
            decision = json.loads(msg.result)

            # Actualizar estado
            state["timeline"].append({"trigger": trigger, "decision": decision})
            json.dump(state, open(state_file, "w"), indent=2)

            # Ejecutar decisión
            if decision["action"] == "launch_epic":
                launch_worker(decision["epic_id"], state)
            elif decision["action"] == "kill_stuck_worker":
                kill_worker(state["current_worker"]["pid"])
```

### 3.3 Despliegue en el servidor

```bash
# Cron del Arquitecto (cada 5 min)
*/5 * * * * flock -n /var/run/rescate/architect.lock -c '/home/sagaz/sharess/architect/architect.sh --trigger=cron' >> /home/sagaz/sharess/logs/architect.log 2>&1

# Systemd user service para el worker (opcional, para lingering)
# ~/.config/systemd/user/sharess-worker@.service
[Unit]
Description=Sharess Worker Epic %i

[Service]
Type=oneshot
ExecStart=/home/sagaz/sharess/worker/worker.sh %i
TimeoutStartSec=3600
```

---

## 4. Decisiones Clave y Recomendaciones

### Qué USAR:

| Decisión | Recomendación | Por qué |
|----------|---------------|---------|
| **SDK** | Claude Agent SDK (Python) | Control programático completo, session chaining, subagents, hooks |
| **Singleton** | flock | Atómico, auto-cleanup, trivial con cron |
| **Triggers** | Cron + event, ambos via flock | Race-condition-free, elegante |
| **Estado** | JSON file (state.json) | Simple, versionable, debuggeable |
| **Worker** | `claude -p` o Agent SDK | Headless, con `--max-turns` y `--max-budget-usd` como safety rails |
| **Stuck detection** | Cheap check (bash) + LLM escalation | Ahorra costes: no despertar LLM si no hay cambios |
| **Proceso** | Corto (no daemon) | Despierta → actúa → duerme. Cero compute mientras duerme |

### Qué NO hacer:

| Anti-patrón | Por qué |
|-------------|---------|
| Daemon permanente | Coste innecesario, complejidad de estado en memoria |
| PID files sin flock | Race conditions, PIDs huérfanos |
| Despertar LLM en cada tick de cron | Costoso. Usar cheap checks primero |
| Múltiples workers simultáneos | Riesgo de pisar código. Un worker a la vez (validar hipótesis primero) |
| Guardar estado solo en memoria | Se pierde en crash. Siempre persistir a disco |

---

## 5. Roadmap de Implementación

### Fase 1: Mínimo viable (1-2 días)
1. Script `architect.sh` con flock + cron
2. `state.json` básico
3. Worker como `claude -p` con prompt estático
4. Completion marker (fichero) como trigger

### Fase 2: Cerebro con LLM (2-3 días)
5. Instalar Claude Agent SDK en el servidor
6. `architect-brain.py` que lee estado y decide
7. Prompts del Arquitecto (soul.md, review-epic.md)
8. Health check bash (cheap check sin LLM)

### Fase 3: Integración Beats (3-5 días)
9. Conectar con Beats CLI como fuente de verdad
10. El Arquitecto lee/escribe épicas en Beats
11. Panel de visualización básico

### Fase 4: Robustez (ongoing)
12. Stuck detection avanzado
13. Rollback vía Git checkpoints
14. Timeline y señales rojas en panel
15. Métricas y logging estructurado

---

## Fuentes

### Proyectos analizados
- [OpenHands](https://github.com/OpenHands/OpenHands) — Agent orchestration, StuckDetector, event-sourced state
- [Hermes Agent](https://github.com/NousResearch/hermes-agent) — Gateway daemon, cron, 3-tier memory
- [OpenClaw](https://dev.to/entelligenceai/inside-openclaw-how-a-persistent-ai-agent-actually-works-1mnk) — Heartbeat pattern, cheap-check-then-escalate
- [Claude Agent SDK](https://platform.claude.com/docs/en/agent-sdk/overview) — Python/TypeScript SDK oficial
- [CrewAI](https://github.com/crewAIInc/crewAI) — Flow/Crew separation
- [LangGraph Supervisor](https://github.com/langchain-ai/langgraph-supervisor-py) — Graph-based orchestration
- [AutoGen/Magentic](https://learn.microsoft.com/en-us/agent-framework/overview/) — Task ledger pattern
- [Temporal](https://temporal.io/blog/orchestrating-ambient-agents-with-temporal) — Durable execution, signals, schedules
- [Restate](https://www.restate.dev/blog/persistent-serverless-state-machines-with-xstate-and-restate) — Virtual state machines

### Documentación Claude Code
- [Claude Code Authentication](https://code.claude.com/docs/en/authentication)
- [Claude Code Headless Mode](https://code.claude.com/docs/en/headless)
- [Claude Agent SDK Quickstart](https://platform.claude.com/docs/en/agent-sdk/quickstart)
- [Agent Teams](https://code.claude.com/docs/en/agent-teams)
- [Hosting Guide](https://platform.claude.com/docs/en/agent-sdk/hosting)

### Patrones de infraestructura
- [flock — File Locking](https://gavv.net/articles/file-locks/)
- [Erlang/OTP Supervisors](https://learnyousomeerlang.com/supervisors)
- [Azure AI Agent Design Patterns](https://learn.microsoft.com/en-us/azure/architecture/ai-ml/guide/ai-agent-design-patterns)
- [Anthropic Multi-Agent Research System](https://www.anthropic.com/engineering/multi-agent-research-system)
