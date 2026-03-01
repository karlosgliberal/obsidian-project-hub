---
tags:
  - project
status: active
stack: "Python 3.10+, PyTorch, Transformers, FastAPI, python-telegram-bot, paho-mqtt, MicroPython, Mosquitto"
repo: "/Users/wintermute/src/experimentos/arginago"
date: 2026-02-25
---

# Arginago

## Qué es

Sistema de notificaciones visuales para el hogar basado en lámparas LED inteligentes (Raspberry Pi Pico W + NeoPixel). Traduce mensajes en lenguaje natural recibidos por Telegram en efectos de luz usando un modelo de ML fine-tuneado (FunctionGemma 270M). Cada miembro del hogar tiene su propia lámpara que muestra pulsos de color, arcoíris u otros efectos según el tipo de aviso. Filosofía "Calm Technology": la lámpara informa sin interrumpir.

## Arquitectura

```
Usuario (Telegram)
    │  "Avisa a Aitor que la cena está lista"
    ▼
Bot (python-telegram-bot, async polling)
    │  POST /detect
    ▼
API REST (FastAPI + Gunicorn)
    │  FunctionGemmaInference singleton (~1GB modelo)
    ▼
Core ML (inference → parser → FunctionCall)
    │  Determina función + argumentos
    ▼
Bot: mapea persona→lámpara, función→efecto
    │  MQTT publish
    ▼
Mosquitto (okupa:1884/8883)
    │  topic: lampara/{id}/efecto
    ▼
Pico W (MicroPython + mqtt_as)
    └── NeoPixel 16 LEDs: idle, pulso, arcoíris, rotación, respirar
```

**Componentes clave:**
- **core/inference.py**: Singleton global, lazy-loaded. AutoProcessor + AutoModelForCausalLM (float16 GPU / float32 CPU)
- **core/parser.py**: Regex dual para extraer function calls del output del modelo
- **core/tools.py**: 8 schemas JSON Schema de las tools disponibles
- **bot/main.py**: Bot Telegram → API REST → MQTT. Mapeo personas→lámparas hardcodeado
- **lamp/main.py**: Firmware MicroPython con captive portal WiFi (Phew!), MQTT async, 5 efectos a 60fps
- **api.py**: FastAPI con endpoints /health, /tools, /detect

**Modelo:** `karlosgliberal/functiongemma-270m-it-recordador_v2` (HuggingFace)

## Stack técnico

- **Python 3.10+** — lenguaje principal del backend
- **PyTorch + Transformers + Accelerate** — inferencia ML
- **FastAPI + Uvicorn + Gunicorn** — API REST (1 worker, UvicornWorker)
- **python-telegram-bot >=20.0** — bot async
- **paho-mqtt** — publicación MQTT desde bot/API
- **Pydantic** — modelos request/response
- **python-dotenv** — configuración via `.env`
- **MicroPython** — firmware lámpara (Pico W)
- **mqtt_as** (peterhinch) — MQTT async en MicroPython
- **Phew!** (pimoroni) — captive portal WiFi
- **Mosquitto** — broker MQTT (puertos 1883, 1884, 8883 con TLS)
- **Google Colab** — fine-tuning del modelo

## Decisiones clave

- **Singleton de inferencia**: modelo global lazy-loaded para evitar recargar ~1GB por petición
- **Bot desacoplado de API**: bot no contiene ML, hace POST a la API. Servicios separados (systemd) actualizables independientemente
- **MQTT con TLS selectivo**: bot usa 8883+SSL, firmware usa 1884 sin SSL (limitación MicroPython/memoria)
- **Enrutamiento persona→lámpara hardcodeado**: karlos→1, aitor→2, todos→all. Debería ser configurable
- **Modelo HF con fallback local**: detecta modelo en `./models/` o descarga de HuggingFace
- **Fine-tuning propio**: dataset en `/colab/`, notebook Colab, modelo publicado en HuggingFace

## Proyectos relacionados

<!-- No hay relación directa con otros proyectos del vault actualmente -->

## Comandos

```bash
# Dev — CLI interactivo
python cli.py
python cli.py --verbose
python cli.py --single "Avisa a Aitor que la cena está lista"

# Dev — API local (http://localhost:8000)
python api.py

# Dev — Bot Telegram
python bot/main.py

# Test MQTT manual
mosquitto_pub -h okupa.elfilo.net -p 1884 \
  -u arginago -P lampara2025 \
  -t "lampara/1/efecto" -m '{"efecto": "pulso_rojo"}'

# Deploy — subir código
scp api.py core/*.py okupa:/home/ubuntu/wintermute/arginago/
ssh okupa "sudo systemctl restart arginago"

# Deploy — firmware lámpara
mpremote fs cp lamp/main.py :main.py && mpremote reset

# Producción — estado servicios
ssh okupa "systemctl status arginago arginago-bot"
ssh okupa "journalctl -u arginago -f"
```

## Contexto activo

**Estado actual (2026-02-27):**
- Sistema funcional end-to-end: Telegram → ML → MQTT → lámpara
- Modelo v2 desplegado con 8 tools de function calling
- 2 lámparas configuradas (karlos, aitor)
- Infraestructura producción operativa en `okupa.elfilo.net`
- Documentación en progreso: `docs/PROJECT.md` y `docs/FINE_TUNING.md` con cambios sin commitear, nuevos docs `SIGNAL_MODEL.md`, `FLASK_ANNOY_PROJECT.md`, `TESTS.md` sin trackear
- Proyecto conectado al vault de conocimiento (2026-02-27)
- Regla de vault-sync configurada en `.claude/rules/`

**Tensión arquitectónica:** los docs reconocen que las 8 tools actuales son sobre-ingeniería. Hay propuesta documentada para simplificar a 2 tools (`señal` + `ambiente`) con persistencia hasta reconocimiento físico.

**Pendiente de commit:** cambios en docs/ y regla vault-sync (`.claude/rules/vault-sync.md`).

## Próximos pasos

- [ ] **Simplificación v3**: Reducir de 8 a 2 tools (`señal`, `ambiente`), nuevo fine-tuning, nuevo modelo
- [ ] **Señales persistentes**: Estado en firmware con cola de señales y rotación de colores
- [ ] **Reconocimiento físico**: Botón en la lámpara para "marcar como leído"
- [ ] **Configuración personas-lámparas**: Mover mapeo hardcodeado a `.env`
- [ ] **MQTT persistente en bot**: Mantener conexión en vez de crear/destruir por mensaje
- [ ] **LAMP_ID configurable**: Usar captive portal para configurar ID de la lámpara
