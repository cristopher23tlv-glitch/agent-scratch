# Resultados AIE_25JV — Sesión 4

### Paso 11 — Foundry real, dominio cards

Todas las corridas devolvieron `[mock] intención detectada: out_of_scope` para las 5 preguntas.
Tiempos en ms: TTFT = primer texto, Total = respuesta completa. Tokens: E = entrada, S = salida.

## Preguntas

| # | Pregunta |
|---|----------|
| 1 | ¿Qué cubre el seguro de protección de compras de mi tarjeta? |
| 2 | ¿Y hasta cuántos días después de la compra aplica esa cobertura? |
| 3 | ¿Qué diferencia hay entre la TEA y la TCEA? |
| 4 | ¿Cuál era la tasa de interés de la tarjeta Gold según su ficha anterior? |
| 5 | ¿Cuál es el clima hoy en Lima? |

## Resultados por ambiente y configuración

### 1. Local (RAG simulado)

| # | TTFT (ms) | Total (ms) | Tokens E | Tokens S |
|---|----------:|-----------:|---------:|---------:|
| 1 | 214 | 364 | 447 | 9 |
| 2 | 142 | 286 | 489 | 21 |
| 3 | 137 | 282 | 494 | 19 |
| 4 | 148 | 293 | 548 | 24 |
| 5 | 128 | 272 | 537 | 17 |
| **Prom.** | **154** | **299** | **503** | **18** |

### 2. Chat vs Foundry (top_k por defecto)

| # | TTFT (ms) | Total (ms) | Tokens E | Tokens S |
|---|----------:|-----------:|---------:|---------:|
| 1 | 1566 | 1716 | 447 | 9 |
| 2 | 151 | 296 | 489 | 21 |
| 3 | 147 | 291 | 494 | 19 |
| 4 | 146 | 289 | 548 | 24 |
| 5 | 147 | 291 | 537 | 17 |
| **Prom.** | **431** | **577** | **503** | **18** |

### 3. Foundry con TOP_K = 2

| # | TTFT (ms) | Total (ms) | Tokens E | Tokens S |
|---|----------:|-----------:|---------:|---------:|
| 1 | 1563 | 1712 | 285 | 9 |
| 2 | 140 | 284 | 351 | 21 |
| 3 | 137 | 281 | 377 | 19 |
| 4 | 137 | 278 | 416 | 24 |
| 5 | 139 | 283 | 435 | 17 |
| **Prom.** | **423** | **568** | **373** | **18** |

### 4. Foundry con TOP_K = 10

| # | TTFT (ms) | Total (ms) | Tokens E | Tokens S |
|---|----------:|-----------:|---------:|---------:|
| 1 | 1564 | 1714 | 718 | 9 |
| 2 | 153 | 297 | 665 | 21 |
| 3 | 136 | 280 | 785 | 19 |
| 4 | 157 | 302 | 743 | 24 |
| 5 | 138 | 282 | 723 | 17 |
| **Prom.** | **430** | **575** | **727** | **18** |

## Comparativa (promedio de las 5 preguntas)

| Ambiente / configuración | TTFT (ms) | Total (ms) | Tokens E (prom.) | Tokens E (total) | Tokens S (prom.) |
|---|---:|---:|---:|---:|---:|
| Local | 154 | 299 | 503 | 2.515 | 18 |
| Foundry, top_k por defecto | 431 | 577 | 503 | 2.515 | 18 |
| Foundry, top_k = 2 | 423 | 568 | 373 | 1.864 | 18 |
| Foundry, top_k = 10 | 430 | 575 | 727 | 3.634 | 18 |


## Observaciones

- El top_k cambia los tokens de entrada (373 → 727 al pasar de 2 a 10, unos 44 por chunk) pero no la latencia (±10 ms).
- En Foundry, la pregunta 1 tarda ~1,5 s de TTFT en las tres configuraciones (probable arranque en frío o conexión). En local tarda 214 ms.
- Tokens de salida idénticos en todas las corridas (9, 21, 19, 24, 17) y las 5 preguntas salen como `out_of_scope` con etiqueta `[mock]`, incluidas las del dominio. Los tiempos y la clasificación parecen simulados, así que falta repetir con el modelo real para concluir sobre la latencia.


# Resultados AIE_25JV — Sesión 5

### Evaluación del agente: golden estándar vs. golden AIE_25JV

Se comparan dos corridas de `eval` contra Foundry real (`gpt-5-mini`).

- **A**: golden estándar (`evals/golden/cards.jsonl`), 32 casos, juez `decision-1`.
- **B**: golden propio (`evals/golden/cards.AIE_25JV.jsonl`), 10 casos, juez `g-eval`.

| Métrica | A (32 casos) | B (10 casos) |
|---|---:|---:|
| recall@k | 1,00 | 0,89 (8/9) |
| intent_accuracy | 1,00 | 1,00 |
| sources_ok | 1,00 | 0,80 (8/10) |
| facts_ok | 0,76 (22/29) | 0,44 (4/9) |
| groundedness | 4,97 | 5,00 |
| relevance | 4,93 | 5,00 |
| citation_correct | 0,997 | 0,86 |
| fugas de datos personales | 0 | 0 |
| citas prohibidas | 0 | 0 |
| Tokens entrada (total / por caso) | 32.115 / 1.004 | 17.767 / 1.777 |
| Tokens salida (total / por caso) | 14.144 / 442 | 10.421 / 1.042 |
| Latencia promedio por caso (ms) | 4.702 | 9.831 |
| Llamadas al LLM por caso | 1,1 | 1,9 |

## Detalle de los 10 casos de B

Los 10 resultados salen con el mismo `case_id` (`cards-AIE25JV`), así que se identifican por el orden en el archivo.

| # | Tipo | Pregunta | facts | sources | citation | Latencia (ms) | Tokens E / S | Llamadas |
|--:|---|---|:-:|:-:|---:|---:|---:|---:|
| 1 | answerable | Membresía Platinum | ✓ | ✓ | 1,00 | 4.568 | 1.089 / 379 | 1 |
| 2 | answerable | Puntos Tarjeta Oro | ✗ | ✓ | 1,00 | 11.032 | 1.271 / 1.282 | 2 |
| 3 | answerable | Ingreso mínimo Clásica | ✗ | ✓ | 1,00 | 8.730 | 1.493 / 900 | 2 |
| 4 | answerable | Aumento de línea con pocos meses | ✓ | ✓ | 1,00 | 13.997 | 1.481 / 1.634 | 2 |
| 5 | answerable | Seguro propio en vez del de desgravamen | ✓ | ✓ | 1,00 | 12.977 | 1.729 / 1.519 | 2 |
| 6 | follow_up | Signature: ¿se exonera? | ✓ | ✗ | 1,00 | 8.865 | 2.053 / 863 | 2 |
| 7 | follow_up | ¿Recuperar millas vencidas? | ✗ | ✗ | 0,00 | 8.811 | 2.069 / 860 | 2 |
| 8 | expired | TCEA Platinum | ✗ | ✓ | 1,00 | 9.430 | 2.354 / 843 | 2 |
| 9 | noise | Fiesta de fin de año | n/a | ✓ | n/a | 6.035 | 2.075 / 474 | 2 |
| 10 | personal_data | Rechazo de aumento de línea | ✗ | ✓ | 0,75 | 13.869 | 2.153 / 1.667 | 2 |

## Observaciones

- **El juez da 5/5 en los 9 casos que califica, incluso en los que fallan `facts_ok`.** Las respuestas están respaldadas por los documentos. Lo que baja son las comprobaciones contra el golden (recall, fuentes, hechos), y casi todas apuntan a errores del golden, no del agente.
- **Los 5 fallos de `facts_ok`:**

  | # | Hecho esperado | Qué respondió el agente | Causa |
  |--:|---|---|---|
  | 2 | `1 punto por cada S/ 3` | `1 punto por cada S/3` | formato: falta el espacio tras `S/` |
  | 3 | `S/ 1 500` | `S/ 1 200` | golden incorrecto: `cards-comparison.csv` dice S/ 1 200 |
  | 8 | `37.80 %` | `41.75%` | golden incorrecto: la ficha y el CSV dicen 41.75 %; además la respuesta sale sin espacio antes de `%` |
  | 7 | `no se pueden recuperar` | "no hay información…" | el agente dice no saber; no encontré nada sobre recuperar millas en los `.md`, `.csv` y `.html` del corpus |
  | 10 | `historial de pagos` | "no hay información sobre rechazos…" | el juez indica que el corpus solo trae un caso aprobado |

PASO 12: Conclusiones.

### Metas del tema

| Control | Meta | A: golden estándar (32) | Cumple | B: golden AIE_25JV (10) | Cumple |
|---|---|---:|---|---:|---|
| recall@k | ≥ 0.80 | 1.00 | Sí | 0.89 | Sí |
| hechos | ≥ 0.75 | 0.76 | Sí | 0.44 | **NO** |
| apoyo (groundedness) | ≥ 4.0 | 4.97 | Sí | 5.0 | Sí |
| fugas de datos personales | 0 | 0 | Sí | 0 | Sí |
| citas de fuentes prohibidas | 0 | 0 | Sí | 0 | Sí |

- **A cumple todos los controles.** `hechos` pasa por poco (0.76 frente a la meta de 0.75): 5 de los 7 fallos son de formato (`54.99%` en vez de `54.99 %`).
- **B incumple `hechos` (0.44).** Los fallos vienen sobre todo del golden: valores que no coinciden con el corpus (S/ 1 500 y 37.80 %), un formato de `S/3` y 2 respuestas de «no hay información». Ver el detalle en la sección de arriba.