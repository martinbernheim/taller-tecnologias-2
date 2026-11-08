# Taller de Tecnologías 2 — Obligatorio

Agentic RAG en LangGraph sobre tickets de soporte (Kaggle) y papers de la cátedra.
Entrega 16/11/26 21:00 (gestion.ort.edu.uy).

## Estructura

```
notebook/
  obligatorio.ipynb       el entregable: código, pruebas y documentación

docs/
  consigna-obligatorio.pdf  letra oficial
  plan-arquitectura.md      diseño del grafo, tools, memoria, anti-alucinación
  plan-de-accion.md         roadmap por fases hasta la entrega

data/
  papers/                 6 PDFs de la cátedra (RAG de documentos largos)
  dataset/                dataset de Kaggle (principal: dataset-tickets-multi-lang3-4k.csv)
    kaggle_raw/           otras variantes del dataset, no usadas por el notebook

CLAUDE.md                 guía de trabajo del proyecto
requirements.txt          dependencias (lock file)
.env                      OPENAI_API_KEY y PINECONE_API_KEY (no se versiona)
```

## Setup

```bash
python -m venv .venv
./.venv/Scripts/pip install -r requirements.txt
```

Crear `.env` en la raíz con `OPENAI_API_KEY=...` y `PINECONE_API_KEY=...`.
Abrir `notebook/obligatorio.ipynb` con el kernel del `.venv`.

## Por dónde arrancar

Leer `CLAUDE.md` y `docs/plan-arquitectura.md`. El notebook está dividido en partes
(setup, papers, tickets, tools, grafo, memoria, pruebas, documentación).
