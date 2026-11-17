# Resultados AIE_10CQ — Sesión 4

### Paso 11 — Foundry real, dominio cards

Ingesta con Foundry y `entra_id`: 29 chunks de 13 documentos publicados en Azure AI Search (`idx-team-07`).

| # | Pregunta | Intención | Fuente citada | ¿Fuente correcta? | Primer texto en | Respuesta completa en | Resultado |
|---:|---|---|---|---|---:|---:|---|
| 1 | ¿Qué cubre la protección contra fraude de mi tarjeta? | `product_question` | `card-insurance.docx` — Protección contra fraude | Sí | 6160 ms | 6345 ms | Indicó consumos no reconocidos hasta 72 horas antes del bloqueo y tope de S/ 10 000. |
| 2 | ¿Y hasta cuántas horas antes del bloqueo está cubierto? | `product_question` (seguimiento) | `card-insurance.docx` — Protección contra fraude | Sí | 8442 ms | 8490 ms | Conservó el contexto del turno anterior y respondió 72 horas. |
| 3 | ¿Qué significa la TCEA y qué incluye? | `product_question` | `tea-and-tcea.md` — TCEA y TEA | Sí | 5411 ms | 5646 ms | Explicó que la TCEA suma comisiones, gastos y seguros a la TEA y sirve para comparar tarjetas. |
| 4 | ¿Cuál era la TEA de compras de la tarjeta Platinum según su ficha de 2023? | `product_question` | `platinum-card-sheet.html` (vigente), no la ficha 2023 vencida | No para la consulta histórica | 4544 ms | 4702 ms | No citó el documento vencido, pero respondió 34.99 % como si fuera el dato de 2023; la ficha 2023 indica 39.99 %. |
| 5 | ¿Cuál es el precio del dólar hoy? | `out_of_scope` | Sin fuentes | Sí | 4063 ms | 4233 ms | Rechazó la consulta por estar fuera del dominio de tarjetas. |

### Paso 12 — Medición de `SEARCH_TOP_K` en local

Se usaron Qdrant local y el mock de Foundry, con `DOMAIN=cards`; no se llamaron servicios cloud. Se reconstruyeron y publicaron localmente los 29 chunks. Los tokens son el conteo aproximado por palabras del mock, no tokens de facturación de un modelo real.

| Pregunta | `SEARCH_TOP_K` | Tokens de entrada | Diferencia vs. 2 | Primer texto | Respuesta completa | ¿Faltó información? | Fuentes/contexto |
|---|---:|---:|---:|---:|---:|---|---|
| ¿Qué cubre la protección contra fraude de mi tarjeta? | 2 | 280 | — | 313 ms | 1190 ms | No; respondió 72 horas y S/ 10 000. | Citó `card-insurance.docx`; el segundo fragmento era sobre desgravamen. |
| ¿Qué cubre la protección contra fraude de mi tarjeta? | 10 | 590 | +310 | 327 ms | 1100 ms | No; misma respuesta y cita. | Citó `card-insurance.docx`; entre los fragmentos extra aparecieron TEA/TCEA, membresía, línea de crédito y millas, no pertinentes a la consulta. |
| ¿Y si los consumos son por fraude, cuántas horas previas están cubiertos? | 2 | 245 | — | 230 ms | 1142 ms | No; respondió 72 horas y S/ 10 000. | Citó `card-insurance.docx`; el segundo fragmento era sobre vencimiento de millas. |
| ¿Y si los consumos son por fraude, cuántas horas previas están cubiertos? | 10 | 546 | +301 | 189 ms | 966 ms | No; misma respuesta y cita. | Citó `card-insurance.docx`; entre los fragmentos extra aparecieron millas, aumento de línea, TEA/TCEA y beneficios de Signature/Oro, no pertinentes a la consulta. |

**Elección del equipo: `SEARCH_TOP_K=2` para estas consultas.** Conservó la respuesta y fuente necesarias, con 301–310 tokens de entrada menos por turno y menos contexto irrelevante. Como esta prueba usa el mock, conviene validar la elección con Foundry real antes de tomarla como resultado de calidad del modelo.

### Comparación — Qdrant local vs. Azure AI Search

Se repitieron las cinco preguntas del paso 11 con Foundry real y `SEARCH_TOP_K=5`. Para la corrida local se reconstruyó Qdrant con embeddings reales; para AI Search se reinició el agente con `KNOWLEDGE_BASE=azure_search`. El arranque confirmó `idx-team-07` con **29 chunks**. AI Search ejecuta búsqueda híbrida y semantic reranking; la columna indica si se observó una mejora en la fuente citada o en la respuesta.

| Pregunta | Fuentes en local | Fuentes en AI Search | ¿Mejoró con el reranker? | Respuesta completa (local / AI Search) |
|---|---|---|---|---:|
| ¿Qué cubre la protección contra fraude de mi tarjeta? | `card-insurance.docx` — Protección contra fraude | `card-insurance.docx` — Protección contra fraude | No observable: misma fuente y respuesta (72 horas, S/ 10 000). | 6695 / 8147 ms |
| ¿Y hasta cuántas horas antes del bloqueo está cubierto? | `card-insurance.docx` — Protección contra fraude | `card-insurance.docx` — Protección contra fraude | No observable: mismo seguimiento, fuente y dato (72 horas). | 8550 / 10513 ms |
| ¿Qué significa la TCEA y qué incluye? | `tea-and-tcea.md` — TCEA y TEA | `tea-and-tcea.md` — TCEA y TEA | No observable: mismas fuentes y explicación. | 6815 / 9180 ms |
| ¿Cuál era la TEA de compras de la tarjeta Platinum según su ficha de 2023? | `platinum-card-sheet.html` — ficha vigente | `platinum-card-sheet.html` — ficha vigente; `cards-comparison.csv` | No: ambos respondieron 34.99 % y no recuperaron la ficha 2023 (que indica 39.99 %). AI Search agregó la comparación, pero no resolvió la fecha histórica. | 4235 / 6084 ms |
| ¿Cuál es el precio del dólar hoy? | Sin fuentes (`out_of_scope`) | Sin fuentes (`out_of_scope`) | No aplica: ambas corridas la clasificaron fuera del dominio. | 4052 / 4692 ms |

**Conclusión:** en estas cinco pruebas el reranker no mejoró las citas ni corrigió la respuesta histórica; las fuentes principales fueron iguales. AI Search tardó más en las cinco consultas en esta corrida. Se mantiene `KNOWLEDGE_BASE=azure_search` en `.env` para la sesión 5.
