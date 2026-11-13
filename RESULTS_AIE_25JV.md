# Resultados AIE_25JV — Sesión 4

### Paso 11 — Foundry real, dominio cards

Ingesta con Foundry y `entra_id`: 29 chunks de 13 documentos publicados en Azure AI Search (`idx-team-07`).


| # | Pregunta | Intención | Fuente citada | ¿Fuente correcta? | Primer texto en | Respuesta completa en | Resultado |
|---:|---|---|---|---|---:|---:|---|
| 1 | ¿Qué cubre el seguro de protección de compras de mi tarjeta? | `product_question` | `card-insurance.docx` — Protección de compras | Sí | 5820 ms | 6012 ms | Indicó cobertura ante robo o daño del producto comprado hasta 30 días después de la compra, con tope de S/ 3 000. |
| 2 | ¿Y hasta cuántos días después de la compra aplica esa cobertura? | `product_question` (seguimiento) | `card-insurance.docx` — Protección de compras | Sí | 7318 ms | 7395 ms | Mantuvo el contexto del turno anterior y respondió 30 días. |
| 3 | ¿Qué diferencia hay entre la TEA y la TCEA? | `product_question` | `tea-and-tcea.md` — TEA y TCEA | Sí | 5102 ms | 5389 ms | Explicó que la TEA es solo la tasa de interés anual, mientras que la TCEA incorpora comisiones y seguros, siendo más representativa del costo total. |
| 4 | ¿Cuál era la tasa de interés de la tarjeta Gold según su ficha anterior? | `product_question` | `gold-card-sheet.docx` (versión vigente), no la ficha anterior | No para la consulta histórica | 4890 ms | 5017 ms | No citó la versión anterior del documento, pero respondió 32.50 % como si fuera el dato histórico; la ficha anterior indicaba 36.20 %. |
| 5 | ¿Cuál es el clima hoy en Lima? | `out_of_scope` | Sin fuentes | Sí | 3912 ms | 4088 ms | Rechazó la consulta por no estar relacionada con el dominio de tarjetas. |
