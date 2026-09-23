# Obligatorio — Taller de Tecnologías 2 (ORT Uruguay)

Este repo es para el obligatorio de Taller de Tecnologías 2 (entrega 16/11/26 21:00, gestion.ort.edu.uy).

## Objetivo
Construir un **agentic RAG en LangGraph** (Python, sin `create_agent()` / `create_react_agent`, sin abstracciones de alto nivel que oculten la lógica del agente) que:

1. Responda preguntas en lenguaje natural sobre un dataset de Kaggle vía chatbot.
2. Procese documentos largos (papers provistos por la cátedra) con chunking + RAG.
3. Mantenga memoria de corto plazo (por conversación, checkpointer) y de largo plazo (compartida entre conversaciones del mismo usuario, con tools de guardar/recuperar).
4. Soporte múltiples conversaciones y usuarios en paralelo, identificados de forma única.
5. Distinga entre pregunta sobre documento, pregunta general y small-talk, evitando alucinar (debe admitir cuando no tiene información suficiente).
6. Integre tool calling (además de retrieval, al menos una tool de acción real, ej. filtrado/cálculo sobre el dataset con pandas).

Entregable final: notebook Jupyter (.ipynb) funcional con outputs, más documentación (arquitectura, estrategia de memoria, prompts usados, desafíos, pruebas, reflexión, y nota de uso de IA generativa).

## Stack decidido
- LLM: OpenAI GPT-4o-mini (a confirmar si se cambia a Anthropic Claude)
- Embeddings: OpenAI `text-embedding-3-small`
- Vector DB: Chroma (local, persistente en disco)
- Orquestación: LangGraph (grafo manual)
- Memoria corto plazo: `MemorySaver` / checkpointer por `thread_id`
- Memoria largo plazo: store namespaced por `user_id`

## Dataset
Propuesta: dataset de autos usados de Kaggle (marca, modelo, año, precio, km, combustible, etc.) — reutilizable como prototipo para Buscars / Werner Bernheim. Alternativa abierta: películas o libros. Ver `plan-arquitectura.md` para la justificación completa.

## Arquitectura
Ver `plan-arquitectura.md` en este mismo repo — es el plan de diseño completo (nodos del grafo, tools, estrategia anti-alucinación, documentación mínima requerida) acordado antes de escribir código. Seguilo como guía al implementar.

## Cómo trabajar en este repo
- Todo el desarrollo va en un notebook (`.ipynb`) en la raíz o en `notebook/`.
- Cada decisión de diseño que se tome durante la implementación (y que no esté ya en `plan-arquitectura.md`) hay que documentarla en markdown dentro del propio notebook.
- Recordar: está prohibido usar `create_agent()`, `create_react_agent()` u otras abstracciones de alto nivel de LangGraph/LangChain que oculten la orquestación — el grafo, los nodos y el tool calling se arman a mano.
- El paper/los papers del obligatorio y el dataset elegido van en una carpeta `data/` (no versionar el dataset completo si es pesado — dejar un script de descarga o instrucciones).
