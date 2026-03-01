---
tags:
  - knowledge
projects:
  - "[[sharess]]"
status: validated
date: 2026-02-28
---

# Loop Arquitecto/Worker con flock + cron + evento

## Problema

Necesitas un proceso orquestador (Arquitecto) que coordine workers de larga duración, garantizando:
- Solo una instancia del orquestador a la vez (singleton)
- Los workers despiertan al orquestador al terminar (event-driven)
- Si un trigger falla, el sistema se recupera solo (safety net)
- Estado persistente entre ejecuciones
- Observable sin interferir con el sistema

## Solución

Tres mecanismos combinados: **flock** para singleton, **eventos de fichero** como triggers primarios, y **cron** como safety net.

### Componentes

1. **architect.sh** — Entry point con `flock -n` (non-blocking). Si no consigue el lock, sale silenciosamente. Si lo consigue, ejecuta health-check → brain → libera lock al salir.

2. **health-check.sh** — Check barato sin lógica pesada. Verifica: ¿hay eventos nuevos? ¿worker stuck? ¿quedan tareas? Retorna exit code 0/1. Esto es el patrón "cheap check then escalate".

3. **architect-brain.sh** — Lógica de decisión. Lee estado JSON + backlog, procesa eventos, decide siguiente acción, lanza worker. En producción este sería el LLM.

4. **worker.sh** — Ejecuta tarea en background. Al terminar: escribe resultado, crea evento (fichero marker), y dispara `architect.sh --trigger=event`.

5. **state.json** — Estado persistente con timeline de todas las decisiones.

6. **events/** — Directorio de markers. Ficheros `.done` o `.fail` que el health-check detecta.

### Flujo

```
[CRON cada minuto]──┐
                    ├──► architect.sh (flock -n)
[Worker completa]───┘         │
                         flock OK? ──no──► salir silencioso
                              │
                             yes
                              │
                         health-check
                              │
                         ¿hay trabajo? ──no──► salir
                              │
                             yes
                              │
                         brain: decide + lanza worker
                              │
                         libera flock → duerme
                              │
                         [worker corre en background]
                              │
                         worker termina → events/epic-N.done
                              │
                         architect.sh --trigger=event ──► (ciclo)
```

### Claves del diseño

- **flock es non-blocking** (`flock -n`): si otro proceso tiene el lock, sale inmediatamente con exit 0 (no error). Esto permite que cron y eventos disparen sin conflicto.
- **El lock se adquiere con fd**: `exec 9>"$LOCK_FILE" && flock -n 9`. Se libera al cerrar el fd (cuando el proceso sale).
- **El worker NO espera al arquitecto**: escribe el evento, dispara el trigger en background (`architect.sh &`), y sale.
- **Cron no necesita flock propio**: el propio `architect.sh` ya tiene el flock. Si el arquitecto está ocupado, cron rebota silenciosamente.
- **Health-check filtra antes de la lógica cara**: si no hay eventos ni tareas pendientes, no se ejecuta el brain. En producción esto evita llamadas innecesarias al LLM.

## Ejemplo

```bash
# Estructura mínima
├── architect.sh          # Entry point con flock
├── architect-brain.sh    # Decisiones (bash/LLM)
├── worker.sh             # Ejecuta tarea
├── health-check.sh       # Check barato
├── state.json            # Estado persistente
├── backlog.json          # Cola de tareas
├── events/               # Markers de completación
├── results/              # Output de workers
├── run/
│   ├── architect.lock    # flock file
│   └── worker.pid        # PID del worker actual
└── logs/

# Cron entry (safety net)
* * * * * /path/to/architect.sh --trigger=cron

# Lanzamiento manual
./architect.sh --trigger=manual

# Monitor event-driven (sin polling)
./monitor.sh --run
```

## Validación

Demo funcional en `sagaz@hero:/home/sagaz/sharess-demo/`. Ejecuta 3 épicas simuladas automáticamente:
- 4 wakeups del arquitecto, 3 épicas completadas, ~30s total
- flock bloquea correctamente instancias simultáneas
- El loop se auto-alimenta: Arquitecto → Worker → evento → Arquitecto
- Cron como safety net detecta workers muertos o stuck (>60s)
- Monitor event-driven (`tail -f`) sin polling ni interferencia con flock

## Cuándo usar

- Orquestación de procesos que deben ser singleton
- Workflows donde un paso dispara el siguiente
- Sistemas que necesitan ser resilientes a fallos de trigger
- Cuando quieres evitar daemons permanentes (proceso corto que despierta y duerme)

## Cuándo NO usar

- Si necesitas concurrencia real (múltiples workers simultáneos) — flock solo protege el orquestador, no los workers
- Si la latencia de 1 minuto del cron es inaceptable — usar systemd path units o inotifywait
- Si el estado es complejo y necesita transacciones — usar una base de datos en lugar de JSON + ficheros
