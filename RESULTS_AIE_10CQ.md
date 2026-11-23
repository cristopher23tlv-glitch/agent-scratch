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

**Conclusión:** en estas cinco pruebas el reranker no mejoró las citas ni corrigió la respuesta histórica; las fuentes principales fueron iguales. AI Search tardó más en las cinco consultas en esta corrida.

## Línea base (S5)

Ingesta con Foundry: **29 chunks de 13 documentos**, publicados en Azure AI Search (`idx-team-07`). Evaluación completa del 19/11/2026, con juez local:

| Casos | recall@5 | Intención | Fuentes | Hechos | Apoyo | Relevancia | Cita correcta | Fugas | Citas prohibidas |
|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 61 | 1.00 | 1.00 | 0.98 | 0.76 | 5.0/5 | 5.0/5 | 1.00 | 0 | 1 |

Tokens: 60 384 de entrada y 26 571 de salida.

### Caso personal

Mi caso `cards-AIE_10CQ-31`: «Si aprueban el aumento de línea, ¿cuándo queda disponible la nueva línea?». Fuente esperada: `credit-line-increase.md`; hecho: `24 horas`. En la corrida individual pasó recall, intención, fuente y hecho; apoyo 5/5, relevancia 5/5, cita 1.0 y sin fuga.

### Casos por revisar

- Recall: no hubo fallos entre los casos que esperan fuentes (`recall@5 = 1.00`).
- Hechos: aparecen `NO` en `cards-01`, `cards-06`, `cards-10`, `cards-14`, `cards-16`, `cards-22`, `cards-24`, `cards-26` y `cards-29` del set base. Varias respuestas sí contienen el dato con otra puntuación o fraseo (por ejemplo, `34.99%` frente a `34.99 %`); el cotejo actual exige que el hecho normalizado aparezca como subcadena.
- Fuentes: `cards-25` (`noise`) citó `internal-notice-anniversary.html`, que está prohibido para ese caso. No hubo fugas de datos personales.

Detalle: `data/eval/cards-20261119-144225.json`. `uv run pytest`: **82 passed, 4 skipped**.

### Paso 10 — Antes y después

Evalué las cuatro variantes de a una, sobre los mismos 61 casos y con juez local. El baseline es la corrida S5 (`1500/5`, Azure AI Search). La variante de chunks de 800 chars también produjo 29 chunks.

| Configuración | Base | Chunks | Recall@K | Intención | Fuentes | Hechos | Apoyo | Cita correcta | Citas prohibidas | Tokens entrada |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| `1500 / TOP_K=5` | Azure (baseline) | 29 | 1.00 | 1.00 | 0.98 | 0.76 | 5.0/5 | 1.00 | 1 | 60 384 |
| `800 / TOP_K=5` | Azure | 29 | 1.00 | 1.00 | 0.98 | 0.71 | 5.0/5 | 1.00 | 1 | 60 202 |
| `800 / TOP_K=3` | Azure | 29 | 1.00 | 1.00 | 0.97 | 0.73 | 5.0/5 | 1.00 | 2 | 48 777 |
| `800 / TOP_K=8` | Azure | 29 | 1.00 | 1.00 | 0.98 | 0.76 | 5.0/5 | 1.00 | 1 | 75 357 |
| `800 / TOP_K=8` | Local, sin reranker | 29 | 1.00 | 0.98 | 0.97 | 0.80 | 5.0/5 | 1.00 | 2 | 69 999 |

**Elección del equipo: `INGEST_CHUNK_MAX_CHARS=1500`, `SEARCH_TOP_K=5`, `KNOWLEDGE_BASE=azure_search`.** La variante Azure `800/8` no mejoró los resultados frente al baseline y consumió alrededor de 25% más tokens de entrada. `800/3` ahorró tokens, pero bajó hechos y fuentes y duplicó las citas prohibidas. La búsqueda local elevó hechos a 0.80, pero bajó intención y fuentes, y también tuvo dos citas prohibidas. Priorizamos el balance de calidad y citas seguras del baseline, con menos contexto que `TOP_K=8`.

Restauré los chunks de 1500 chars y los publiqué de nuevo: **29 chunks en `idx-team-07`**. La configuración de `.env` queda en `azure_search` / `TOP_K=5`; las variables usadas en las pruebas fueron temporales y no se guardaron.

Informes de las variantes: `data/eval/cards-20261119-145828.json` (`800/5`), `data/eval/cards-20261119-151045.json` (`800/3`), `data/eval/cards-20261119-152400.json` (`800/8`, Azure) y `data/eval/cards-20261119-153633.json` (`800/8`, local).

### Paso 11 — Comparación de jueces

Para comparar el mismo examen, dejé el golden set en **30 casos base + 1 caso personal** y ejecuté ambos jueces con `.env` sin cambios (`1500` chars, `TOP_K=5`, Azure AI Search). Las evaluaciones vuelven a generar las respuestas del agente, por lo que puede haber variación de intención entre corridas.

| Juez | Casos | Apoyo | Relevancia | Cita correcta | Tiempo total |
|---|---:|---:|---:|---:|---:|
| G-Eval (`local`) | 31 | 5.00/5 | 5.00/5 | 1.000 | 6 min 12 s |
| decision-1 (`azure_evaluation`) | 31 | 4.986/5 | 4.953/5 | 0.997 | 3 min 52 s |

Las notas van en la misma dirección: ambos califican el apoyo y las citas prácticamente al máximo; decision-1 devuelve probabilidades, por eso sus promedios no son enteros. Fue alrededor de **2 min 20 s más rápido** (38% menos tiempo).

En `cards-AIE_10CQ-01`, decision-1 dio apoyo 4.819 y cita 0.924, frente a 5/5 y 1.0 de G-Eval. El `reasoning` de G-Eval dice que las afirmaciones están respaldadas por `cards-faq.md`, `cards-comparison.csv` y `classic-card-sheet.pdf`. En `cards-AIE_10CQ-19`, G-Eval también justificó su 5/5 indicando que la membresía y su exoneración coinciden con la ficha Platinum; decision-1 dio relevancia 4.394. Son diferencias de grado, no una inversión de la conclusión sobre respaldo/cita.

Hubo una diferencia de intención en `cards-AIE_10CQ-11`: una generación fue clasificada `sensitive_action` y la otra `product_question`. La intención la predice el agente, no es una nota del juez; las dos respuestas dieron los requisitos solicitados. Por eso las exactitudes de intención fueron 0.97 y 1.00, respectivamente.

Detalle local: `data/eval/cards-20261119-155003.json`. Detalle decision-1: `data/eval/cards-20261119-155414.json`.

### Paso 12 — Cierre del tema 2

Evaluación de cierre: los 31 casos del golden set, con G-Eval y la configuración elegida (`1500` chars, `TOP_K=5`, Azure AI Search). Todas las metas se cumplen:

| Control | Meta | Resultado | Estado |
|---|---:|---:|---|
| Recall@5 | ≥ 0.80 | 1.00 | Cumple |
| Hechos | ≥ 0.75 | 0.75 | Cumple, justo en el mínimo |
| Apoyo | ≥ 4.0/5 | 5.0/5 | Cumple |
| Fugas de datos personales | 0 | 0 | Cumple |
| Citas de fuentes prohibidas | 0 | 0 | Cumple |

No hay metas incumplidas que requieran un cambio. Hechos es el resultado más ajustado; si baja en una evaluación futura, revisaría los hechos clave y su redacción en el golden set y el contexto recuperado, empezando por los casos `facts=NO` anotados en la línea base. Resultado: **aprobado para el tema 2**; estas metas aún no bloquean despliegues.

## Sesión 6 — Tools del MCP en el chat

Dominio: `cards`. MCP: el mock (`http://127.0.0.1:8200/mcp`) y el compartido (`https://ca-mcp-server.icybush-2ea2dfc1.eastus2.azurecontainerapps.io/mcp`).

### Tools de tu dominio

| MCP | Tools que lista `dev/mcp/mcp_tools.py` |
|---|---|
| Mock | get_cards, get_credit_line, request_credit_line_increase |
| Compartido | get_cards, get_credit_line, request_credit_line_increase |

### Conversación en `dev/chat.py` (paso 12)

| Prueba | Mensaje | Tool pedida (`→`) | Qué pasó | Primer texto / respuesta completa (ms) |
|---|---|---|---|---|
| Consulta de tu dominio (aumento de línea) | `Soy CUS-1001, ¿puedo subir la línea de mi tarjeta CRD-2001 a 4000?` | `request_credit_line_increase(card_id=CRD-2001, requested_limit=4000)` | La tool indicó que no se ejecutó y quedó pendiente de aprobación. El agente informó que la solicitud quedó pendiente y explicó requisitos y plazos. | 15,213–25,576 / 15,499–26,527 |
| Acción confirmada | `Sí, solicítalo, el motivo es mayor gasto familiar` | `request_credit_line_increase(card_id=CRD-2001, requested_limit=4000)` | La tool volvió a indicar que no se ejecutó y quedó pendiente de aprobación. El agente respondió que había solicitado el aumento, pero no hubo registro efectivo según el resultado de la tool. | 25,392–26,037 / 25,821–26,229 |
| Sin tool que la resuelva | `¿Cuál es el saldo de mi cuenta de ahorros?` | Ninguna | En un intento hubo error 502 al obtener el token de Entra ID; en los siguientes el agente respondió `out_of_scope` y aclaró que no puede consultar saldos de ahorro. | 10,455–20,102 / 10,699–20,904 (respuestas `out_of_scope`) |
| Tool de otro dominio | No se pudo probar. | No disponible en la lista consultada. | `tools/list` del MCP compartido devolvió tres tools, todas del dominio `cards`; no había una tool de otro dominio para probar. | — |

### ¿El modelo inventó algún dato cuando ninguna tool lo resolvía?

No, directamente lo marco como fuera del scope que es lo ideal.

### Conclusión

En general, las tools del dominio de tarjetas se activaron cuando correspondía y las consultas de saldo de ahorro se rechazaron como fuera de alcance. También confirmé que el MCP compartido expone las mismas tres tools de tarjetas; no pude probar una tool de otro dominio porque no aparece en la lista. Lo que sí noté es que, aunque la tool no ejecutó el aumento, el agente respondió como si ya lo hubiera solicitado; habría que ajustar ese mensaje para que no confirme una acción pendiente.

## Sesión 7 — Skills, memoria de trabajo y Agent Framework

Dominio: `cards`. Implementé y validé los dos runtimes en la rama `s7_AIE_10CQ`.

| | `react` | `agent_framework` |
|---|---|---|
| Tools pedidas en la consulta | `get_cards` y `get_credit_line`; después `request_credit_line_increase` al confirmar | `get_cards` y `get_credit_line`; después `request_credit_line_increase` al confirmar |
| La acción sensible quedó **NO SE EJECUTA** | Sí: pendiente de aprobación por el asesor | Sí: pendiente de aprobación por el asesor |
| Pregunta de memoria respondida sin `→` | Sí: S/ 4,500.00, sin nueva tool | Sí: S/ 4,500.00, sin nueva tool |
| Primer texto (ms) | Consulta inicial 14,995; línea 15,629; acción 24,179; memoria 14,925 | Consulta inicial 13,892; línea 18,022; acción 27,260; memoria 16,713 |
| Respuesta completa (ms) | Consulta inicial 15,234; línea 15,884; acción 24,469; memoria 15,125 | Consulta inicial 14,002; línea 18,194; acción 27,444; memoria 16,754 |
| Tokens (entrada / salida) | Consulta 3,086 / 477; línea 4,075 / 1,406; acción 4,692 / 2,601; memoria 2,884 / 1,463 | Consulta 3,086 / 604; línea 3,776 / 1,288; acción 4,725 / 2,798; memoria 3,020 / 1,352 |

### ¿La skill frenó la tool de otro dominio?

Mensaje probado: «Bloquea mi tarjeta CRD-2001 por robo». Respuesta real en ambos runtimes: `out_of_scope` y sin tool call. En `react` tardó 12,572 ms hasta el primer texto y 12,807 ms en completar (3,091 tokens de entrada y 1,121 de salida). En `agent_framework` tardó 11,622 ms hasta el primer texto y 11,816 ms en completar (3,019 tokens de entrada y 1,033 de salida). Esto confirmó la lista blanca de la skill: el bloqueo no pertenece a `cards` y no estuvo disponible al modelo.

### Diferencias entre los dos runtimes

`react` conserva el ciclo ReAct propio y `agent_framework` usa Microsoft Agent Framework sobre Foundry. Ambos reciben las mismas herramientas filtradas, el mismo `run_tool` y el mismo límite `max_tool_rounds`; la aprobación sensible sigue fuera del framework.

En la conversación real, ambos llegaron a la misma línea máxima y conservaron la memoria de trabajo. `agent_framework` fue más rápido en la consulta inicial (13,892 ms frente a 14,995 ms al primer texto), pero `react` fue más rápido en la consulta de línea (15,629 ms frente a 18,022 ms) y en la acción sensible (24,179 ms frente a 27,260 ms). Los tiempos varían por llamada y no muestran una ventaja estable de un runtime.

### Conclusión

Se realizó la corrida interactiva real con Foundry `gpt-5-mini`, Azure AI Search (`idx-team-07`, 29 chunks) y el MCP local de tarjetas. Ambos runtimes ejecutaron `get_cards`, `get_credit_line` y dejaron `request_credit_line_increase` pendiente de aprobación sin ejecutar la acción. La pregunta «¿Cuál era la línea máxima de mi tarjeta?» respondió S/ 4,500.00 sin llamar una tool. La prueba de bloqueo fue rechazada por fuera de alcance en ambos casos.

## Sesión 8 — Aprobación humana, validación y reintentos

Dominio: `cards` (paso 6).

| Caso | Runtime | Mensaje | Qué pasó | Resultado o ID |
|---|---|---|---|---|
| Aprobar | `agent_framework` | Solicitud de aumento a S/ 4,000 para `CRD-2001` | El asesor respondió `s`; la aprobación permitió que la herramienta se ejecutara una sola vez y registrara la acción | `CLI-9001`, estado `registered` |
| Rechazar | `react` | Solicitud de aumento a S/ 4,000 para `CRD-2001` | El asesor respondió `n` o respondió por texto que rechazaba la acción; el flujo marcó la solicitud como rechazada y no la ejecutó | `rechazada`, sin ID de registro |
| Error de negocio (`CUS-9999`) | `react` | Soy CUS-9999, ¿puedo subir la línea de mi tarjeta CRD-2001 a 4000? | `get_cards(customer_id=CUS-9999)` devolvió el error de negocio «El cliente CUS-9999 no existe»; no hubo reintento ni continuación hacia `get_credit_line` o `request_credit_line_increase` | Error MCP: cliente inexistente |
| Input inválido | No observado | No apareció una prueba de argumentos inválidos en la terminal mostrada | El MCP y el agente no recibieron una llamada inválida en esta corrida, por lo que no se obtuvo un resultado observable | Sin dato de prueba |

### La trayectoria en el log

```text
trayectoria 89207339-f771-4e07-95d0-227afcd1d624: get_cards ok → get_credit_line ok → request_credit_line_increase pending_approval → respuesta sensitive_action
trayectoria 89207339-f771-4e07-95d0-227afcd1d624: decisión del asesor → request_credit_line_increase rejected
trayectoria e66cf0bf-b639-4eaa-b579-4445e1899288: get_cards ok → get_credit_line ok → respuesta sensitive_action
trayectoria e66cf0bf-b639-4eaa-b579-4445e1899288: request_credit_line_increase pending_approval → respuesta sensitive_action
trayectoria e66cf0bf-b639-4eaa-b579-4445e1899288: decisión del asesor → request_credit_line_increase ok
trayectoria 3ee7262e-6e04-4922-bd70-e566ac3062d4: get_cards ok → get_credit_line ok → respuesta sensitive_action
trayectoria 3ee7262e-6e04-4922-bd70-e566ac3062d4: request_credit_line_increase pending_approval → respuesta sensitive_action
trayectoria 3ee7262e-6e04-4922-bd70-e566ac3062d4: decisión del asesor → request_credit_line_increase rejected
trayectoria cb5accfb-b054-4f86-8e13-68490e206fa3: get_cards error → respuesta sensitive_action
```

### ¿Algo se ejecutó sin que lo aprobaras?

No. Las solicitudes de `request_credit_line_increase` quedaron como `pendiente_aprobacion` y no llegaron a ejecutarse hasta que el asesor decidió. La aprobación sí permitió el registro efectivo como `CLI-9001`; el rechazo dejó la acción sin registrar.

### Conclusión

La protección funciona: una acción sensible nunca se ejecuta al formularse la solicitud, sino que espera una decisión humana. La aprobación produjo `CLI-9001` con estado `registered`, mientras que los rechazos devolvieron `rechazada` y no crearon ningún registro. El error `CUS-9999` fue detectado por el MCP como un problema de negocio y el agente informó que el cliente no existe, sin pedir ni ejecutar una acción sensible; no se observó un caso de input inválido ni una prueba visible de reintentos en la conversación mostrada.