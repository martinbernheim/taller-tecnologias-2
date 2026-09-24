# Dataset

**Multilingual Customer Support Tickets** (Kaggle). Se versiona en el repo
(privado, solo el grupo tiene acceso) para que cloná y funciona sin pasos
manuales adicionales.

## Archivo principal

**`dataset-tickets-multi-lang3-4k.csv`** — 4.000 tickets, 5 idiomas reales
(en, de, es, fr, pt), esquema más completo: `subject`, `body`, `answer`,
`type`, `queue`, `priority`, `language`, `business_type`, `tag_1`..`tag_9`.
Es el que usa el notebook.

Por qué este y no una de las variantes más grandes: el texto (asunto + cuerpo
+ respuesta) justifica chunking/embeddings de verdad, `business_type` y los
tags dan más dimensiones para la tool de filtrado con pandas, y el tamaño es
manejable para generar embeddings sin gastar de más de la cuota de la API key
del curso.

## `kaggle_raw/`

Otras variantes del mismo dataset tal como vienen en el ZIP de Kaggle —
quedan como referencia/respaldo, el notebook no las usa:

| Archivo | Filas | Columnas | Idiomas |
|---|---|---|---|
| `aa_dataset-tickets-multi-lang-5-2-50-version.csv` | 28.587 | 16 (sin `business_type`, con `version`) | en, de |
| `dataset-tickets-multi-lang-4-20k.csv` | 20.000 | 15 (sin `business_type`) | en, de |
| `dataset-tickets-german_normalized_50_5_2.csv` | 13.178 | 5 (solo subject/body/queue/priority/language) | de |
| `dataset-tickets-german_normalized.csv` | 2.125 | 5 | de |
