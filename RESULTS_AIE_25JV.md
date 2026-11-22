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

## Sesión 6 — Tools del MCP en el chat

Dominio: `cards`. MCP: el mock (`http://127.0.0.1:8200/mcp`) y, si ya tienes `MCP_URL`, el compartido (`bank-mcp`).

### Tools de tu dominio

| MCP | Tools que lista `dev/mcp/mcp_tools.py` |
|---|---|
| Mock | `http://127.0.0.1:8200/mcp` · cards · 3 tools<br>`get_cards` (consulta): `customer_id`<br>`get_credit_line` (consulta): `card_id`<br>`request_credit_line_increase` (acción sensible): `card_id`, `requested_limit` |
| Compartido | `https://ca-mcp-server.icybush-2ea2dfc1.eastus2.azurecontainerapps.io/mcp` · cards · 3 tools<br>`get_cards` (consulta): `customer_id`<br>`get_credit_line` (consulta): `card_id`<br>`request_credit_line_increase` (acción sensible): `card_id`, `requested_limit` |


### Conversación en `dev/chat.py` (paso 12)

| Prueba | Mensaje | Tool pedida (`→`) | Qué pasó | Primer texto / respuesta completa (ms) |
|---|---|---|---|---|
| Consulta de tu dominio | Soy CUS-1001, ¿puedo subir la línea de mi tarjeta CRD-2001 a 4000? | `get_credit_line(card_id=CRD-2001)` y luego `request_credit_line_increase(card_id=CRD-2001, requested_limit=4000)` | `get_credit_line` falló: «El servicio del banco no responde». El agente siguió con la acción sensible, que **no se ejecutó**: quedó `pendiente_aprobacion`. Respondió (`sensitive_action`) que registró la solicitud de S/ 4,000 y citó `credit-line-increase.md`. | 43.590 / 45.019 |
| Acción confirmada | Sí, solicítalo, el motivo es mayor gasto familiar | `request_credit_line_increase(card_id=CRD-2001, requested_limit=4000)` | La tool volvió a responder `pendiente_aprobacion`: no se ejecuta hasta que lo apruebe un asesor. El agente informó que el aumento a S/ 4,000 quedó pendiente (respuesta en 2 días hábiles, línea disponible en 24 horas) y recordó los requisitos. | 29.716 / 30.251 |
| Sin tool que la resuelva | ¿Cuál es el saldo de mi cuenta de ahorros? | Ninguna | Intención `out_of_scope`. No inventó ningún saldo: dijo que no puede consultarlo y derivó a la banca móvil o por Internet, a atención al cliente o a una sucursal. | 11.980 / 12.145 |
| Tool de otro dominio (`fees`) | Soy CUS-1001, ¿por qué me cobraron comisiones este mes? ¿Alguna se puede devolver? | Ninguna: el MCP solo muestra las tools del dominio `cards`, así que las de `fees` no estaban disponibles | Intención `product_question`: respondió solo con RAG (`cardholder-cases.pdf`, `classic-card-sheet.pdf`, `card-insurance.docx`). Dio ejemplos de cargos de los documentos y aclaró que no tenía la política de devolución, por lo que no pudo confirmar si alguna comisión se devuelve. Tiempos: 16.447 / 16.832 ms. | — |


### ¿El modelo inventó algún dato cuando ninguna tool lo resolvía?

No. En las dos pruebas sin tool disponible no hubo datos inventados:

- **Saldo de ahorros:** no dio ninguna cifra. Declaró que no puede consultarlo y mandó al cliente a la banca móvil, a atención al cliente o a una sucursal.
- **Comisiones (tool de `fees`):** no inventó los cargos del cliente `CUS-1001`. Usó solo ejemplos de los documentos (membresía Clásica S/ 120, membresía Oro S/ 240, seguro de desgravamen 0.12 % mensual), que coinciden con el corpus. Dijo de forma explícita que los fragmentos no incluyen la política de devolución.


### Conclusión

- **Las acciones sensibles no se ejecutan solas.** En las dos pruebas con `request_credit_line_increase` la tool respondió `pendiente_aprobacion`, y el agente lo comunicó al cliente con los plazos y requisitos del corpus.
- **Sin tool, el agente no inventa.** Con saldo de ahorros respondió `out_of_scope`, y con comisiones (otro dominio) contestó solo con RAG y reconoció lo que no sabía.
- **La latencia es el punto débil.** Con tools tardó 30–45 s hasta la respuesta completa, y sin tools 12–17 s. Además, `get_credit_line` falló con «El servicio del banco no responde» en la primera prueba y habría que repetirla.

## Sesión 7 — Skills, memoria de trabajo y Agent Framework

Dominio: `cards`. La misma conversación de 4 turnos con los dos runtimes (paso 7), cambiando solo `AGENT_RUNTIME` en el `.env`. Modelo con razonamiento (`gpt-5-mini`). Una sola corrida por runtime.

Turnos: (1) «Soy CUS-1001, ¿puedo subir la línea de mi tarjeta CRD-2001 a 4000?» · (2) «Sí, solicítalo, el motivo es mayor gasto familiar» · (3) «¿Cuál es el saldo de mi cuenta de ahorros?» · (4) «¿Cuál era la línea máxima de mi tarjeta?»

| | `react` | `agent_framework` |
|---|---|---|
| Tools pedidas en la consulta (turno 1) | `get_cards(customer_id=CUS-1001)` y `get_credit_line(card_id=CRD-2001)` | Las mismas dos, en el mismo orden |
| Tool pedida en la acción confirmada (turno 2) | Solo `request_credit_line_increase(card_id=CRD-2001, requested_limit=4000)`, sin repetir `get_cards` ni `get_credit_line` | Igual |
| La acción sensible quedó **NO SE EJECUTA** | Sí: `pendiente_aprobacion`; el agente lo comunicó como solicitud pendiente de un asesor | Sí: igual |
| Pregunta de memoria respondida sin `→` | Sí (turno 4): «S/4,500», sin ninguna tool | Sí (turno 4): «S/ 4,500.00», sin ninguna tool |
| Primer texto (ms), turnos 1 / 2 / 3 / 4 | 23.501 / 18.811 / 9.302 / 10.996 (suma 62.610) | 21.336 / 26.021 / 9.496 / 13.308 (suma 70.161) |
| Respuesta completa (ms), turnos 1 / 2 / 3 / 4 | 23.786 / 19.079 / 9.445 / 11.011 (suma 63.321) | 21.614 / 26.167 / 9.559 / 13.317 (suma 70.657) |
| Tokens entrada (por turno / total) | 4.466 / 4.620 / 3.018 / 3.173 → **15.277** | 4.466 / 4.713 / 3.009 / 3.181 → **15.369** |
| Tokens salida (por turno / total) | 2.154 / 2.189 / 864 / 1.165 → **6.372** | 1.881 / 3.145 / 897 / 1.414 → **7.337** |

### ¿La skill frenó la tool de otro dominio?

Mensaje: «¿Cuál es el saldo de mi cuenta de ahorros?» (fuera del dominio `cards`; como mi dominio es `cards`, uso esta pregunta en lugar de la de `fees` de la guía).

Respuesta: sí, la frenó. Con los dos runtimes la intención fue `out_of_scope` y no hubo ninguna llamada `→` a tools. El agente dijo que no puede consultar saldos de ahorros, no inventó ninguna cifra y derivó a banca móvil/web, cajero o servicio al cliente. Ni el MCP ni la skill dejan pasar la consulta.

### Diferencias entre los dos runtimes

- **Comportamiento: prácticamente idéntico.** Mismas tools, mismo orden, misma intención en cada turno (`sensitive_action`, `sensitive_action`, `out_of_scope`, `data_query`), mismas fuentes (`credit-line-increase.md`) y los mismos 4.466 tokens de entrada en el turno 1, lo que indica que el prompt es el mismo. El harness es nuestro: el cambio fue una línea del `.env`.
- **Latencia: sin diferencia concluyente.** `react` fue ~10 % más rápido en total (63,3 s frente a 70,7 s), pero `agent_framework` ganó el turno 1 (21,6 s frente a 23,8 s) y perdió por 7 s el turno 2. Con una sola corrida es ruido del modelo, no del runtime. En ambos, el primer texto tarda 9–26 s y casi todo ese tiempo es razonamiento y tools.
- **Streaming poco visible.** Entre el primer texto y la respuesta completa pasan solo 0,01–0,28 s. El texto sí llega en pedazos, pero casi al final, cuando el modelo ya terminó de razonar.
- **Tokens de salida: `agent_framework` generó ~15 % más** (7.337 frente a 6.372), sobre todo en el turno 2 (3.145 frente a 2.189), probablemente por tokens de razonamiento. La entrada es casi igual (15.369 frente a 15.277).
- **Estilo de la respuesta:** `react` trata de «usted»; `agent_framework`, de «tú». Es un efecto del modelo, no del runtime.
- **Matices en la acción confirmada (turno 2).** La tool devolvió que la acción **no se registró todavía**. `react` dijo «He registrado la solicitud», lo que se pasa de lo que ocurrió. `agent_framework` dijo «He solicitado…» (más fiel), pero añadió «registré el motivo: mayor gasto familiar» (la tool solo recibe `card_id` y `requested_limit`, así que el motivo no se guarda en ninguna parte) y «Te avisaremos por los canales registrados» (no lo respalda la tool). Ambos citan los plazos y requisitos del corpus correctamente.

### Conclusión

- **Cambiar de runtime no cambia el resultado.** Con `react` o `agent_framework` el agente pide las mismas tools, la acción sensible no se ejecuta (`pendiente_aprobacion`) y lo comunica con los plazos del corpus.
- **La memoria de trabajo funciona.** En el turno 2 no hubo `get_cards` ni `get_credit_line` previos: `CRD-2001` y `4000` salieron del turno anterior. En el turno 4 la línea máxima (S/ 4.500) se respondió sin ninguna tool, con lo que ya se había consultado.
- **La latencia sigue siendo el punto débil:** 9–26 s hasta el primer texto con cualquier runtime, por el modelo que razona. Elegir runtime no la arregla.
