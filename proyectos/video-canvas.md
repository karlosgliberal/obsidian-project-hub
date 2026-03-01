---
tags:
  - project
status: active
stack: Python 3.11, OpenAI SDK, Kimi K2.5, OpenCV, yt-dlp, Faster-Whisper, ffmpeg
repo: /Users/wintermute/src/experimentos/video-canvas
date: 2026-02-25
---

# video-canvas

## Qué es

Transforma URLs de video (TikTok, YouTube y cualquier fuente soportada por yt-dlp) en archivos `.canvas` de Obsidian — mapas visuales navegables del contenido. No es un resumen de texto: es una recomposición espacial donde cada idea clave del video se convierte en un grupo con keyframes, resumen y conexiones narrativas hacia otras ideas.

## Arquitectura

Pipeline de 6 pasos:

```
URL
 → 1. extract_video_info()              [tiktok-ingest] — título y slug
 → 2. prepare_video()                   [video.py]      — descarga mp4 + audio wav + duración
 → 3. transcribe()                      [tiktok-ingest] — Faster-Whisper → texto
 → 4. analyze()                         [analyzer.py]   — 3 llamadas Kimi K2.5 → frames + ideas + layout
 → 5. extract_keyframes_at_timestamps() [video.py]      — OpenCV → JPEGs
 → 6. build_canvas() + save_canvas()    [canvas_builder.py] → .canvas JSON
```

### Patrón de 3 roles LLM

El video se codifica una sola vez en base64 y se envía en las llamadas:

1. **Editor** — Selecciona 8-15 timestamps con descripción. Regla anti-caras: máximo 2 frames de "talking head", prioriza diagramas, objetos, texto en pantalla. Devuelve `List[Frame]`.
2. **Guionista** — Agrupa frames en 3-7 ideas fuerza, asigna colores Obsidian (1-6). Devuelve `List[KeyIdea]`.
3. **Director** — Solo recibe texto (sin video). Decide posiciones de grupos y conexiones narrativas. Devuelve `CanvasComposition`.

### Generación de canvas

`build_canvas()` recalcula posiciones reales ignorando las coordenadas del Director. Cada grupo contiene: nodo group (contenedor con color), nodo text (título + summary), N nodos file (keyframes JPEG en grid 2 columnas), N nodos text (timestamp + descripción). Edges entre grupos con labels descriptivos.

## Stack técnico

| Componente | Tecnología |
|------------|-----------|
| Lenguaje | Python 3.11 |
| LLM | Kimi K2.5 (Moonshot AI) via OpenAI SDK |
| API endpoint | `https://api.moonshot.ai/v1` |
| Transcripción | Faster-Whisper (via [[tiktok-ingest]]) |
| Keyframes | OpenCV |
| Descarga video | yt-dlp |
| Audio | ffmpeg / ffprobe |
| Tests | pytest (61 tests, 100% mocked) |

## Decisiones clave

- **Migración Gemini → Kimi K2.5:** Backend original usaba `google-genai` SDK con Gemini Files API. Migrado a OpenAI SDK compatible apuntando a Moonshot. Video enviado como base64 `video_url` en cada llamada. Sin gestión de archivos remotos.
- **Director no necesita video:** Optimización deliberada — solo recibe texto de las ideas ya resumidas.
- **Posiciones del Director son orientativas:** `_layout_groups()` respeta el orden pero recalcula posiciones absolutas con tamaños reales, evitando solapamientos.
- **ffprobe como método primario de duración:** OpenCV devuelve 0 para muchos videos TikTok. ffprobe es fiable. Último recurso: duración del audio de Whisper.
- **Fallback a análisis básico:** Si Kimi falla o duración es 0, genera frames uniformes + 1 idea genérica. El pipeline nunca falla — produce un canvas degradado pero válido.

## Proyectos relacionados

- [[tiktok-ingest]] — Dependencia en runtime (`sys.path`). Aporta `slugify()`, `OBSIDIAN_FOLDER`, `extract_video_info()`, `transcribe()`
- [[json-render-canvas]] — Renderer de canvas JSON (TypeScript/Vite/React Flow)

## Comandos

```bash
# Uso principal (CLI bash wrapper — crea venv automáticamente)
./canvas "https://www.tiktok.com/@usuario/video/123"

# Equivalente directo con Python
python -m video_canvas.pipeline "https://youtu.be/xxx"

# Con directorio de salida personalizado
./canvas "https://..." --output-dir /ruta/al/vault

# Tests
python -m pytest tests/ -v

# Test de conectividad con API de Kimi
python test_kimi_api.py                  # solo conexión texto
python test_kimi_api.py video.mp4        # + análisis multimodal

# Variables de entorno requeridas
export MOONSHOT_API_KEY="sk-..."
export TIKTOK_INGEST_DIR="/ruta/a/tiktok-ingest"  # si no está en ../

# Variables opcionales
export KIMI_MODEL="kimi-k2.5"
export KIMI_BASE_URL="https://api.moonshot.ai/v1"
export OBSIDIAN_FOLDER="/ruta/vault"
export MEDIA_BASE_DIR="/ruta/media"
```

## Contexto activo

**Branch:** `feature/kimi-k25`
**Último commit:** `292f905 feat: migrate video analysis backend from Gemini to Kimi K2.5`
**Estado:** Migración a Kimi K2.5 completada. Queda limpieza de referencias legacy a Gemini en documentación y `dist/`.

## Próximos pasos

- [ ] Actualizar `CONTEXT.md` para reflejar Kimi K2.5 en lugar de Gemini
- [ ] Regenerar `dist/video-canvas-skill/` con dependencias actualizadas (`openai` en lugar de `google-genai`)
- [ ] Limpiar variables legacy `GEMINI_API_KEY`/`GEMINI_MODEL` de `config.py`
- [ ] Decidir si `tres_roles_flujo.canvas` se commitea o se añade a `.gitignore`
- [ ] Corregir docstring de `prepare_video()` que menciona Gemini
- [ ] Evaluar si implementar o eliminar el flag `--layout` del CLI wrapper
- [ ] Implementar selección de backend LLM (`--model gemini|kimi|...`) — permitir elegir el modelo/proveedor desde CLI, con config por defecto y validación de API keys disponibles
