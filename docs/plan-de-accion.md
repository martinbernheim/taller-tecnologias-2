# Plan de acción

Roadmap de ejecución para el obligatorio (consigna completa en
[`consigna-obligatorio.pdf`](consigna-obligatorio.pdf), arquitectura detallada
en [`plan-arquitectura.md`](plan-arquitectura.md)). Hoy es 23/09/2026, entrega
16/11/26 21:00 — quedan ~8 semanas.

## Verificación: ¿tenemos todo lo necesario para arrancar?

| Requisito de la consigna | Estado |
|---|---|
| LLM con tool calling | ✅ OpenAI (`gpt-4o-mini` generación, `gpt-4.1-nano` router), API key del curso cargada en `.env` |
| Embeddings | ✅ `text-embedding-3-small` (mismo proveedor) |
| Base vectorial (Pinecone/Qdrant/Chroma/FAISS) | ✅ Pinecone elegido; paquetes instalados, falta `PINECONE_API_KEY` en `.env` |
| Dataset elegido de Kaggle sobre una temática | ✅ Multilingual Customer Support Tickets, en `data/dataset/` |
| Papers/documentos largos de la cátedra | ✅ en `data/papers/` (6 PDFs) |
| LangGraph, sin `create_agent()`/`create_react_agent()` | ✅ instalado (v1.2.12), grafo se arma a mano |
| Entorno reproducible | ✅ `.venv` + `requirements.txt` (lock file) probado en limpio |
| Repo + equipo | ✅ repo privado, Matías invitado como colaborador |
| Notebook con el agentic RAG | ❌ todavía no existe — es el grueso del trabajo que sigue |
| Inscripción y equipo en gestion.ort.edu.uy | ⚠️ trámite administrativo — lo tienen que hacer ustedes, yo no tengo acceso a ese sistema |
| Fecha/tema de defensa oral | ⚠️ a coordinar con los docentes más adelante (16-27/11) |

Falta un solo bloqueante técnico: cuenta y API key de Pinecone (necesaria para
embeber e indexar papers/dataset). Lo demás — loaders, chunking, el grafo en
sí — se puede ir armando en paralelo sin esperarla. El resto de lo pendiente
es administrativo (inscripción/equipo en Gestión) y coordinar la defensa
cuando se acerque.

## Fases

### Fase 0 — Setup (hecho)
Repo, entorno, papers, dataset, plan de arquitectura.

### Fase 1 — RAG sobre papers
- Loader de PDFs (`pypdf`) + chunking (tamaño/overlap a decidir y documentar) —
  no depende de Pinecone, se puede arrancar ya.
- ⚠️ Embeddings + índice/namespace de Pinecone dedicado a papers — requiere
  cuenta y API key de Pinecone.
- Función/tool `buscar_papers(query)` con similarity search y umbral mínimo.
- Probar recuperación con preguntas de ejemplo antes de conectar al grafo.

### Fase 2 — Dataset + tool calling
- Carga del CSV elegido (`dataset-tickets-multi-lang3-4k.csv`) con pandas.
- Namespace/índice de Pinecone separado para el dataset (texto: subject+body+answer).
- Tool `buscar_dataset(query)` (semántica) y tool `filtrar_tickets(...)` (pandas
  puro: por queue/priority/language/type — la acción real de tool calling que
  pide la consigna, evita que el LLM "calcule" mal).

### Fase 3 — Grafo LangGraph (a mano)
- Estado del grafo (mensajes, resultados de retrieval, flags de memoria).
- Nodo router (clasifica: pregunta de documento / dataset / general / small-talk).
- Nodo LLM principal con bind_tools (papers, dataset, filtrado, memoria).
- Nodo `tools` manual (no `ToolNode` prearmado si cuenta como abstracción de
  alto nivel — revisar y documentar la decisión) con vuelta al LLM hasta que
  no haya más tool calls.
- Checkpointer (`MemorySaver` o equivalente en la API actual de LangGraph 1.x)
  por `thread_id` → memoria corto plazo, multi-conversación con pausa/retomo.

### Fase 4 — Memoria de largo plazo
- Store namespaced por `user_id`.
- Tools `guardar_memoria(hecho)` / `recuperar_memoria(query)`, decisión de
  cuándo usarlas queda en manos del LLM vía prompt.
- Probar que persiste entre distintos `thread_id` de un mismo usuario.

### Fase 5 — Anti-alucinación y multi-usuario/multi-sesión
- Prompt de sistema: responder solo con evidencia recuperada o memoria; admitir
  falta de información.
- Umbral de similaridad: sin contexto suficiente → no alucinar.
- Prueba explícita de múltiples `user_id`/`thread_id` corriendo en paralelo
  sobre el mismo grafo compilado.

### Fase 6 — Batería de pruebas
Como la consigna aclara que evalúan con **preguntas no conocidas**, conviene
armar variedad real antes de entregar:
- Preguntas sobre el dataset (factuales y de filtrado/agregación).
- Preguntas sobre cada paper.
- Preguntas generales / small-talk.
- Preguntas multi-turno que dependen de memoria corto plazo.
- Preguntas que deberían disparar memoria de largo plazo.
- Preguntas "trampa" sin respuesta en el material → debe admitir que no sabe.

### Fase 7 — Documentación (dentro del notebook, markdown cells)
Cubrir uno a uno los puntos de "Documentación mínima requerida" de la consigna:
arquitectura y justificación, estrategia de memoria, prompts usados, desafíos
y soluciones, pruebas y resultados, reflexión y experimentación opcional, y la
nota de uso de IA generativa (herramientas + para qué se usó cada una).

### Fase 8 — Opcionales (si el tiempo lo permite, no restan si no se hacen)
Orden sugerido por costo/beneficio: comparar estrategias de chunking →
comparar modelos de embeddings → comparar LLMs → API/Gradio.

### Fase 9 — Empaquetado y entrega
- Correr el notebook completo de punta a punta y dejar los outputs.
- Armar el zip/rar final **sin** los PDFs/CSV crudos (supera los 40MB permitidos
  por Gestión) — el profesor ya tiene los papers y el dataset es público en
  Kaggle; documentar el link de descarga en vez de adjuntarlo.
- Verificar tamaño final <40MB antes de subir a gestion.ort.edu.uy.

### Fase 10 — Defensa oral
Coordinar fecha/tema con los docentes apenas se acerque la ventana (16-27/11).
Repasar el notebook y las decisiones documentadas antes de la defensa —
es individual/eliminatoria, hay que poder explicar cualquier parte del código.

## Próximo paso inmediato

1. Conseguir cuenta + API key de Pinecone (bloquea todo lo que sea embeddings/indexado).
2. Mientras tanto, arrancar la parte de Fase 1 que no depende de eso: loader de
   PDFs y chunking de los papers.
