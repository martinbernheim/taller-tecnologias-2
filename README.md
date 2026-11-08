# Taller de Tecnologías 2 — Obligatorio

Agentic RAG en LangGraph. Entrega 16/11/26 21:00 (gestion.ort.edu.uy).

## Estructura del repo

```
CLAUDE.md                    guía de trabajo para el proyecto (stack, reglas, estado actual)
requirements.txt             dependencias (lock file, instalar con pip install -r requirements.txt)
.env                         OPENAI_API_KEY (no versionado, crear localmente a partir de .env.example)

docs/
  consigna-obligatorio.pdf   letra oficial del obligatorio
  plan-arquitectura.md       diseño del grafo, tools, memoria, anti-alucinación
  plan-de-accion.md          roadmap por fases hasta la entrega

data/
  papers/                    PDFs de la cátedra para RAG de documentos largos
  dataset/                   dataset de Kaggle elegido (tickets de soporte multi-idioma)
    kaggle_raw/               otras variantes del mismo dataset, no usadas por el notebook

notebook/ (o .ipynb en la raíz)   el desarrollo en sí — todavía no creado
```

## Setup local

```bash
python -m venv .venv
./.venv/Scripts/pip install -r requirements.txt
```

Crear un archivo `.env` en la raíz con `OPENAI_API_KEY=...` (la key del curso, pedírsela a Martín o Matías).

## Por dónde arrancar

Leer `CLAUDE.md` y `docs/plan-arquitectura.md` primero. El roadmap de trabajo
está en `docs/plan-de-accion.md`.
