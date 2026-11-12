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
