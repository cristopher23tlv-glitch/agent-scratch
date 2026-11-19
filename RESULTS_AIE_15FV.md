# Resultados AIE_15FV (equipo 07, dominio cards): Sesión 4

## Paso 10: chat contra el mock

Con el mock de Foundry y la base local (29 chunks, vectores del mock) el agente responde en streaming, cita la sección
del fragmento y un saludo no trae fuentes. En el log del agente aparece `llm SearchQuery` en el segundo turno, es decir,
hubo rewrite. Como los vectores del mock no tienen semántica, la fuente citada en esta prueba no es relevante; solo
se valida el flujo.

| Turno | Primer texto (ms) | Respuesta completa (ms) | Tokens entrada | Tokens salida |
|---|---:|---:|---:|---:|
| Pregunta de producto | 218 | 1030 | 393 | 48 |
| Seguimiento (con rewrite) | 171 | 898 | 538 | 48 |
| Saludo (sin fuentes) | 138 | 268 | 591 | 11 |

## Paso 11: chat contra Foundry (gpt-5-mini) con la base local

Base local armada con embeddings reales (`text-embedding-3-small`), `SEARCH_TOP_K=5`. Todas las preguntas son del dominio cards.

| # | Tipo | Pregunta | Fuente citada | ¿Correcta? | Primer texto (ms) | Respuesta completa (ms) | Tokens entrada |
|---|---|---|---|---|---:|---:|---:|
| 1 | regla del corpus | ¿Cuál es la TEA de compras de la Tarjeta Platinum? | `platinum-card-sheet.html` > Condiciones financieras | Sí (34.99 %) | 6690 | 6843 | 905 |
| 2 | seguimiento | ¿y la de la Signature? | `signature-card-sheet.md` y `cards-comparison.csv` | Sí (29.99 %); entendió que seguía hablando de la TEA | 10847 | 10870 | 1290 |
| 3 | término exacto | ¿Qué es la TCEA? | `tea-and-tcea.md` > TCEA | Sí | 10041 | 10194 | 1223 |
| 4 | documento vencido | ¿Qué decía la ficha de la Platinum de 2023 sobre la membresía? | `platinum-card-sheet.html` (vigente) | Sí: no cita `platinum-card-sheet-2023.md`; responde con la membresía vigente S/ 480 | 9572 | 9705 | 1329 |
| 5 | fuera del corpus | ¿A cuánto está el dólar hoy? | ninguna | Sí: `out_of_scope`, sin fuentes | 9965 | 10121 | 1490 |

Observaciones:

- Las cinco respuestas traen su fuente correcta, y el documento vencido nunca llega al contexto porque se filtra antes de buscar.
- En el seguimiento el log muestra `llm SearchQuery`: el rewrite convirtió "¿y la de la Signature?" en una pregunta completa.
- El primer texto tarda entre 6.7 y 10.8 s, casi igual que la respuesta completa. gpt-5-mini razona antes de escribir
  (los tokens de salida, 845 a 995, incluyen ese razonamiento) y el JSON estructurado no sale hasta que empieza `answer`.
  El streaming funciona, pero con este modelo el beneficio es menor que con el mock (150 a 450 ms).

## Paso 12: `SEARCH_TOP_K` (2, 5 y 10)

Base local, Foundry real, dos preguntas: **A** "¿Cuál es la TEA de compras de la Tarjeta Platinum?" (un dato en un solo
fragmento) y **B** "¿Qué diferencia hay entre la tarjeta Clásica y la Platinum?" (necesita varios fragmentos).

| top_k | Pregunta | Fuentes citadas | Primer texto (ms) | Respuesta completa (ms) | Tokens entrada | Tokens salida |
|---:|---|---:|---:|---:|---:|---:|
| 2 | A | 1 | 4262 | 4511 | 685 | 234 |
| 2 | B | 2 | 5913 | 6081 | 620 | 601 |
| 5 | A | 1 | 6690 | 6843 | 905 | 170 |
| 5 | B | 3 | 7004 | 7305 | 961 | 738 |
| 10 | A | 1 | 8076 | 8221 | 1392 | 298 |
| 10 | B | 5 | 10010 | 10324 | 1396 | 819 |

- **Con 2:** A queda completa. B pierde información: la respuesta menciona TEA, millas, salas VIP e ingreso mínimo, pero
  omite la membresía anual y la TCEA, que están en `cards-comparison.csv` y en la ficha de la Platinum, fuera del top 2.
  Los dos fragmentos que recuperó vienen del mismo documento (`cards-faq.md`).
- **Con 10:** cada turno cuesta entre 430 y 490 tokens de entrada más que con 5 (905 → 1392 en A; 961 → 1396 en B), unos
  90 a 100 tokens por fragmento extra. En A el modelo igual cita una sola fuente, así que los otros 9 fragmentos no aportan.
  En B las fuentes extra sí vienen al caso (beneficios de la Platinum y ficha de la Clásica) y la respuesta es más
  completa, pero ninguna fuente citada está fuera de tema.
- **Latencia:** sube con el contexto, pero también varía entre corridas (el rango de A, de 4.3 a 8.1 s, mezcla ambos efectos).
  Con tres mediciones no se puede separar bien el efecto del top_k del ruido.
- **Elección del equipo:** `SEARCH_TOP_K=5`. Con 2 se pierde información en las preguntas que combinan fragmentos;
  con 10 se paga casi 50 % más de tokens de entrada sin ganar nada en las preguntas de un solo dato. En la sesión 5 esto se
  mide con el golden set (recall@k) en vez de a ojo.

## Paso 13: Azure AI Search (`idx-team-07`)

`uv run ingest publish --target azure_search` → `29 chunks en https://srch-workshop-jsu3.search.windows.net (índice idx-team-07)`.
Los 29 son los mismos chunks que la base local, así que el conteo coincide. Al levantar el agente, el log dice
`Knowledge base: azure_search · https://srch-workshop-jsu3.search.windows.net (índice idx-team-07) · 29 chunks`.
(El conteo en el portal, paso 13.2, queda por revisar a mano.)

| # | Pregunta | Fuentes en local | Fuentes en AI Search | ¿Mejoró con el reranker? | Respuesta completa (ms), local / AI Search |
|---|---|---|---|---|---|
| 1 | TEA de compras de la Platinum | `platinum-card-sheet.html` | `platinum-card-sheet.html` | Igual | 6843 / 8804 |
| 2 | ¿y la de la Signature? | `signature-card-sheet.md`, `cards-comparison.csv` | `signature-card-sheet.md`, `cards-comparison.csv` | Igual | 10870 / 9940 |
| 3 | ¿Qué es la TCEA? | `tea-and-tcea.md` | `tea-and-tcea.md` | Igual | 10194 / 12598 |
| 4 | Ficha de la Platinum de 2023 (vencido) | `platinum-card-sheet.html` | `platinum-card-sheet.html`, `cards-comparison.csv` | Igual de correcta: no cita el documento de 2023; suma una fuente vigente que también tiene la membresía | 9705 / 13650 |
| 5 | Dólar hoy | ninguna | ninguna | Igual (`out_of_scope`) | 10121 / 14674 |

- Las respuestas y fuentes son equivalentes. Con solo 29 chunks, la búsqueda local ya pone el fragmento correcto en el
  top 5, así que el reranker no tiene mucho que mejorar. No se nota una mejora medible; se verá mejor con el golden set de la sesión 5.
- AI Search tardó más en cuatro de cinco preguntas (entre 2 y 4.6 s más; la 2 fue 0.9 s más rápida) y usó entre 29 y 189
  tokens de entrada más por turno. Con gpt-5-mini la variación entre corridas ya es de varios segundos, así que no se
  atribuye todo a la nube.
- `deploy-agent.yaml` actualizado (paso 13.4): el agente desplegado usa `KNOWLEDGE_BASE=azure_search` y exige `DOMAIN`,
  `SEARCH_ENDPOINT` y `SEARCH_INDEX` en las Variables del repo.
