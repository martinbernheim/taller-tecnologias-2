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
- LLM: OpenAI — `gpt-4o-mini` para generación, `gpt-4.1-nano` para el router/clasificador (modelos habilitados por la key del curso)
- Embeddings: OpenAI `text-embedding-3-small`
- Vector DB: Chroma (local, persistente en disco)
- Orquestación: LangGraph (grafo manual)
- Memoria corto plazo: checkpointer de LangGraph por `thread_id`
- Memoria largo plazo: store namespaced por `user_id`

## Dataset
**Multilingual Customer Support Tickets** (Kaggle) — tickets de soporte multi-idioma (en/de/es/fr/pt) con texto (subject/body/answer) y columnas estructuradas (queue, priority, type, business_type, tags). Archivo principal: `data/dataset/dataset-tickets-multi-lang3-4k.csv`. Justificación completa en `docs/plan-arquitectura.md`.

## Arquitectura y plan
- `docs/plan-arquitectura.md` — diseño completo (nodos del grafo, tools, estrategia anti-alucinación, documentación mínima requerida), acordado antes de escribir código. Seguirlo como guía al implementar.
- `docs/plan-de-accion.md` — roadmap de ejecución por fases hasta la entrega.
- `docs/consigna-obligatorio.pdf` — letra oficial del obligatorio.

## Cómo trabajar en este repo
- Todo el desarrollo va en un notebook (`.ipynb`) en la raíz o en `notebook/`.
- Cada decisión de diseño que se tome durante la implementación (y que no esté ya en `docs/plan-arquitectura.md`) hay que documentarla en markdown dentro del propio notebook.
- Recordar: está prohibido usar `create_agent()`, `create_react_agent()` u otras abstracciones de alto nivel de LangGraph/LangChain que oculten la orquestación — el grafo, los nodos y el tool calling se arman a mano.
- Los papers del obligatorio y el dataset elegido viven en `data/` y sí se versionan (el repo es privado): `data/papers/` y `data/dataset/`. `.env` con la API key nunca se versiona.
- El paquete final que se sube a Gestión (límite 40MB) no debe incluir los PDFs/CSV crudos de `data/` — ver `docs/plan-de-accion.md`, fase de empaquetado.
