# Obligatorio Taller de Tecnologías 2 — Plan de Arquitectura

Entrega: 16/11/26 21:00. Grupo de hasta 3 (a coordinar). Defensa oral obligatoria y eliminatoria.

## 1. Objetivo

Agentic RAG en LangGraph (sin `create_agent()` / `create_react_agent`, sin abstracciones de alto nivel) que:

1. Responda preguntas sobre un dataset de Kaggle vía chatbot en lenguaje natural.
2. Responda preguntas sobre los papers largos provistos en el obligatorio (chunking + RAG).
3. Mantenga memoria de corto plazo (por conversación) y de largo plazo (compartida entre conversaciones del mismo usuario).
4. Soporte múltiples conversaciones y múltiples usuarios en paralelo, con tool calling.
5. Distinga entre pregunta sobre documento, pregunta general y small-talk, y decida cómo responder en cada caso, evitando alucinar.

## 2. Dataset elegido

**Multilingual Customer Support Tickets (Kaggle)** — tickets de soporte con columnas
`subject`, `body`, `answer`, `type`, `queue`, `priority`, `language`, `business_type`,
`tag_1..tag_9`. Archivo principal en `data/dataset/dataset-tickets-multi-lang3-4k.csv`
(detalle de por qué ese y no otro en `data/dataset/README.md`).

Por qué este dataset:

- Cada ticket tiene texto rico (asunto + cuerpo + respuesta) en varios idiomas, lo
  que justifica chunking + embeddings + búsqueda semántica real (no un CSV
  puramente numérico).
- Las columnas estructuradas (`queue`, `priority`, `language`, `type`, `tags`) se
  prestan directamente a tool calling con pandas: filtros y agregaciones tipo
  "¿cuántos incidentes urgentes hay en la cola de Technical Support en alemán?"
  sin que el LLM tenga que inventar el conteo.
- Multi-idioma da pie a probar el pipeline de RAG con contenido en distintos
  idiomas, un caso de uso realista para un chatbot de soporte.

## 3. Stack tecnológico

| Componente | Elección | Justificación |
|---|---|---|
| LLM | **OpenAI GPT-4o-mini** (vía API) | Tool calling nativo confiable en LangGraph, barato para iterar mucho durante el desarrollo y en la defensa en vivo. Alternativa: Claude (Anthropic) — el diseño de grafo no cambia, solo el binding del modelo. |
| Embeddings | `text-embedding-3-small` (OpenAI) | Barato, buena calidad, mismo proveedor que el LLM simplifica la integración. |
| Vector DB | **Pinecone** (cloud) — ⚠️ pendiente de implementar, falta crear cuenta/API key | Managed, soporta metadata filtering (útil para separar dataset vs. papers por namespace/filtro). Reemplaza la elección inicial de Chroma. |
| Orquestación | **LangGraph** (grafo de estados manual, sin prebuilt agent) | Requisito explícito de la consigna. |
| Memoria corto plazo | `MemorySaver` / checkpointer de LangGraph por `thread_id` | Permite pausar/retomar conversaciones sin perder contexto, uno por conversación. |
| Memoria largo plazo | `InMemoryStore` de LangGraph (o persistido a disco/SQLite) namespaced por `user_id` | Compartida entre conversaciones del mismo usuario; el propio agente decide qué guardar vía una tool dedicada. |
| Entorno | Jupyter Notebook (.ipynb) | Pedido explícitamente como formato preferente. |

## 4. Arquitectura del agente (grafo LangGraph)

Nodos propuestos:

1. **Router / clasificador de intención** — nodo LLM liviano (o el mismo LLM con un prompt de clasificación) que decide si la consulta es: (a) sobre el dataset, (b) sobre los papers, (c) pregunta general/small-talk, o (d) requiere recuperar memoria de largo plazo antes de responder.
2. **Nodo de recuperación (retrieval)** — invocado como *tool*, no como paso fijo del grafo: el agente decide si llamar a `buscar_dataset(query)` o `buscar_papers(query)` según el router, cada una hace similarity search en su índice/namespace de Pinecone correspondiente.
3. **Nodo de memoria** — dos tools: `guardar_memoria(hecho)` (el LLM decide cuándo invocarla, p. ej. el usuario cuenta una preferencia) y `recuperar_memoria(query)` (búsqueda semántica sobre la memoria de largo plazo del `user_id`).
4. **Nodo de generación** — arma la respuesta final combinando: mensajes de la conversación (memoria corto plazo), fragmentos recuperados (RAG), y memoria de largo plazo relevante. Prompt con instrucción explícita de "si no tenés información suficiente, decilo, no inventes".
5. **Tool calling adicional** — al menos una tool de acción real más allá de RAG (ej: una función de filtrado/cálculo sobre el dataset, tipo `filtrar_autos(marca=None, precio_max=None, ...)` que corre pandas en vez de depender del LLM para aritmética — esto también ataca el punto de "control de alucinaciones").

El grafo es cíclico: desde el nodo de generación, si el LLM decide llamar una tool, se vuelve a un nodo `tools` (ToolNode manual, no `create_react_agent`) y de ahí de vuelta al LLM, hasta que responde sin más tool calls.

Cada usuario tiene un `user_id`; cada conversación un `thread_id` (config de LangGraph). Multi-sesión en paralelo = múltiples `thread_id` corriendo con el mismo grafo compilado, cada uno con su propio checkpoint.

## 5. Estrategia anti-alucinación

- Prompt de sistema explícito: responder solo con base en los fragmentos recuperados o en la conversación; si no hay evidencia suficiente, decir "no tengo información suficiente para responder eso".
- Incluir en el prompt los fragmentos recuperados con su fuente (dataset vs. paper vs. memoria) para que el LLM pueda citar de dónde sale cada dato.
- Para preguntas numéricas/agregaciones sobre el dataset (promedios, conteos, filtros), usar la tool de pandas en vez de dejar que el LLM "calcule" sobre texto recuperado.
- Umbral de similaridad mínimo en la búsqueda vectorial: si ningún chunk supera el umbral, no se pasa contexto y el agente debe admitir que no tiene información.

## 6. Documentación mínima a entregar (según consigna)

1. Arquitectura del agentic RAG y justificación de decisiones técnicas (este documento es la base).
2. Estrategia de memoria corto/largo plazo: cómo se guarda, recupera y usa.
3. Ejemplos de prompts y prompt engineering usado.
4. Desafíos encontrados y soluciones.
5. Pruebas y resultados del chatbot (casos de prueba con preguntas variadas).
6. Reflexión sobre mejoras posibles y experimentación opcional.
7. Nota de uso de IA generativa: herramientas usadas y para qué (obligatorio por la consigna del curso).

## 7. Próximos pasos

1. ⚠️ Crear cuenta y API key de Pinecone (bloquea el nodo de recuperación).
2. Armar el notebook: setup → carga y chunking de dataset + papers → embeddings + Pinecone → grafo LangGraph (router, tools, memoria, generación) → pruebas → documentación dentro del propio notebook (markdown cells).

---
*Documento generado con asistencia de Claude (Anthropic), usado para estructurar la arquitectura y planificación del obligatorio. Debe citarse según las pautas de uso de IA del curso.*
