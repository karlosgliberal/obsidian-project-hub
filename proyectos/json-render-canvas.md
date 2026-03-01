---
tags:
  - project
status: active
stack: "Vite 6, React 18, TypeScript 5, @json-render/core 0.8, @xyflow/react 12, react-markdown 9, JSZip, Zod 4"
repo: "/Users/wintermute/src/experimentos/json_render_canvas"
date: 2026-02-25
---

# json-render-canvas

## Qué es

Transformer que convierte archivos `.canvas` de Obsidian (JSON Canvas) en vistas web interactivas con React Flow. Soporta tres pipelines de entrada: carga directa de `.canvas`, upload de ZIP con imágenes, y generación desde lenguaje natural con Gemini (streaming JSONL). Usa `@json-render/core` como formato intermedio (spec declarativa con catálogo Zod) que actúa como contrato entre el origen del canvas y el renderer.

## Arquitectura

Tres pipelines convergen en un visor único:

```
.canvas (JSON Canvas)  →  transformer  →  CanvasSpec
ZIP (.canvas + images) →  zip-utils + transformer  →  CanvasSpec
User prompt            →  catalog.prompt() + Gemini streaming  →  CanvasSpec

CanvasSpec  →  specToFlow()  →  layoutPipeline()  →  React Flow  →  CanvasViewer
```

Módulos principales:
- **`catalog/`** — 5 schemas Zod para componentes canvas, `promptTemplate` custom para Gemini
- **`transformer/`** — `.canvas` → spec: parse + containment por bbox + posiciones relativas
- **`renderer/`** — spec → React Flow: nodos custom (Text, Group, Frame, Link), edges con smooth step, layout pipeline (compact → auto-size → de-overlap)
- **`generator/`** — Pipeline AI: streaming Gemini, `createSpecStreamCompiler`, `autoFixSpec`, refinamiento iterativo
- **`embed.ts`** — API pública `mountCanvasViewer()` para embeber en páginas externas

## Stack técnico

| Librería | Versión | Rol |
|---|---|---|
| Vite | ^6.0.0 | Build tool + dev server (proxy Gemini API) |
| React | ^18.3.0 | UI |
| TypeScript | ^5.5.0 | Tipado |
| @json-render/core | ^0.8.0 | Catálogo Zod, spec intermedia, compiler streaming |
| @xyflow/react | ^12.10.1 | Grafo interactivo (nodos, edges, drag, zoom) |
| react-markdown | ^9.0.0 | Markdown en TextNode |
| JSZip | ^3.10.1 | Extracción ZIP en browser |
| Zod | ^4.0.0 | Validación schemas |

## Decisiones clave

- **Edges en `state.edges`**: json-render es árbol, edges son grafo → externalizados al campo `state`
- **Containment por bounding box**: `.canvas` no tiene jerarquía explícita → se resuelve geométricamente
- **Posiciones relativas al padre**: React Flow con `parentId` usa coords relativas, transformer las convierte
- **Tipo "canvas-group"**: evita colisión con CSS built-in de RF (`.react-flow__node-group`)
- **8 handles duales por nodo**: source+target en cada lado para edges bidireccionales
- **`push().result` en streaming**: `getResult()` borra buffer interno → solo usar al final del stream
- **Layout en post-procesamiento**: `layoutPipeline()` opera sobre nodos RF, no modifica la spec

## Proyectos relacionados

- [[video-canvas]] — Pipeline Python: Video → Gemini (3 roles) → `.canvas`
- Pipeline URL → Canvas: visión futura que unifica los 3 proyectos

## Comandos

```bash
# Dev server
npm run dev

# Build producción
npm run build

# Preview build
npm run preview
```

## Contexto activo

**Última actividad**: 21-24 febrero 2026 — Sprint de UI redesign + AI generation + documentación.

**Estado**: Pipeline completo funcional (carga .canvas, ZIP upload, generación AI streaming, refinamiento iterativo, layout automático, viewer interactivo). Sin tests automatizados. Sin remoto git configurado.

**Datos de test**: 2 `.canvas` reales (butter-chicken, tres-roles-flujo) + 4 specs JSON generadas con AI.

## Próximos pasos

1. **Integrar `embed.ts` en karlosgliberal.com** — `mountCanvasViewer()` ya existe, falta build e integración en el jardín digital
2. **Mejorar layout compactor** — mejor espaciado, gaps, alineación de filas (prerequisito para visual feedback)
3. **Visual feedback loop** (VASCAR-like) — screenshot canvas + evaluación multimodal Gemini + iteración automática
4. **Explorar MCP Server** — exponer canvas como dato versionable
5. **Pipeline URL → Canvas** — unificación de los 3 proyectos (largo plazo)
