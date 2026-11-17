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

# Resultados AIE_15FV — Sesión 5

## Línea base (S5)

Dominio `cards`, índice `idx-team-07` en Azure AI Search (29 chunks, publicados con Foundry), `SEARCH_TOP_K=5`,
`INGEST_CHUNK_MAX_CHARS=1500`, juez `JUDGE=local` (G-Eval con `gpt-5-mini`). Corrida de los 30 casos base:

```
Resumen de 30 casos (juez local):
  recall@5: 1.00   intención: 1.00   fuentes: 0.97   hechos: 0.78
  apoyo: 5.0/5   relevancia: 5.0/5   cita correcta: 1.0
  fugas de datos personales: 0   citas de fuentes prohibidas: 1
  tokens: 29643 de entrada, 13799 de salida
```

Cada caso tardó unos 25 s (respuesta ~10 s, juez ~7 s); la corrida completa, unos 14 min.

### Casos con NO

| Caso | Tipo | Control en NO | Qué pasó |
|---|---|---|---|
| cards-01, 04, 05, 10, 14, 26 | answerable / noise | hechos | El dato está bien en la respuesta, pero escrito distinto del `key_fact` (por ejemplo `36.20%` en vez de `36.20 %`, `4 visitas a salas VIP al año` en vez de `4 visitas al año`). Es un fallo del control de texto, no del agente. |
| cards-25 | noise | fuentes | Citó `internal-notice-anniversary.html`, el aviso interno prohibido, y respondió sobre el almuerzo de aniversario en vez de decir que no tiene esa información. Es la única cita prohibida. |

Metas del tema 2 sobre esta línea base: recall@k 1.00 (cumple), apoyo 5.0 (cumple), fugas 0 (cumple),
hechos 0.78 (cumple por poco; la mayoría de los NO son de formato), citas prohibidas 1 (no cumple, meta 0).

## Mi caso del golden set

`cards-AIE15FV` (tipo `expired`): *¿Cuánto cuesta la membresía anual de la Tarjeta Platinum?* Lo elegí porque el golden
set pregunta por la TEA, el ingreso mínimo y las millas de la Platinum con documento vencido, pero no por la membresía.
La ficha vigente (`platinum-card-sheet.html`) dice S/ 480 y la de 2023 (`platinum-card-sheet-2023.md`, prohibida) dice
S/ 420; el dato también está en `cards-comparison.csv`.

Resultado: recall ok, intención ok, fuentes ok, hechos ok, apoyo 5.0, relevancia 5.0, cita 1.0, 0 citas prohibidas.
El agente respondió con la ficha vigente.

## Paso 10: antes y después

Una variable por corrida, `uv run eval` completo, `JUDGE=local`. La línea base tiene 30 casos; las corridas siguientes,
31 (suman `cards-AIE15FV`, que pasa todos los controles). Con un caso más que pasa, los promedios se mueven poco por esa
razón y no por el cambio que se prueba.

| Cambio | recall | intención | fuentes | hechos | apoyo | citas prohibidas | tokens entrada | tokens salida |
|---|---|---|---|---|---|---|---|---|
| Línea base (top_k 5, chunk 1500, AI Search) | 1.00 | 1.00 | 0.97 | 0.78 | 5.0 | 1 | 29643 | 13799 |
| Chunks 800 | sin cambio | sin cambio | sin cambio | sin cambio | sin cambio | sin cambio | sin cambio | sin cambio |
| `SEARCH_TOP_K=3` | 1.00 | 0.97 | 0.97 | 0.71 | 5.0 | 1 | 25081 | 12497 |
| `SEARCH_TOP_K=8` | 1.00 | 1.00 | 0.97 | 0.79 | 5.0 | 1 | 38464 | 14380 |
| Base local, sin reranker (top_k 5) | 0.96 | 0.97 | 0.94 | 0.71 | 5.0 | 1 | 27507 | 14227 |

- **Chunks 800:** no cambia nada, porque el chunk más largo del corpus de cards mide 621 caracteres. Con 800 sigue
  habiendo 29 chunks, idénticos a los de 1500. No corrí la evaluación completa porque habría repetido la línea base.
  Para ver un efecto habría que bajar a unos 400 caracteres, y eso no está en la guía.
- **top_k 3:** gasta 15 % menos tokens de entrada (25081 contra 29643), pero hechos baja de 0.78 a 0.71 y la intención
  de 1.00 a 0.97. Con tres fragmentos faltan datos en preguntas que mezclan documentos.
- **top_k 8:** hechos sube apenas a 0.79 y los tokens de entrada suben 30 % (38464). No resuelve la cita prohibida
  (`cards-25`) ni mejora apoyo, que ya es 5.0.
- **Sin reranker:** recall baja de 1.00 a 0.96, fuentes de 0.97 a 0.94 y hechos de 0.78 a 0.71. El apoyo se mantiene en
  5.0. El reranker de AI Search aporta una mejora pequeña en recuperación de fuentes y hechos. Con 29 chunks y 31 casos
  son uno o dos casos de diferencia, así que la señal es débil.

**Configuración elegida por el equipo:** `SEARCH_TOP_K=5`, `INGEST_CHUNK_MAX_CHARS=1500`,
`KNOWLEDGE_BASE=azure_search`. Es la que da mejor recall y fuentes, con un costo de tokens intermedio. top_k 8 gana 0.01
en hechos a cambio de 30 % más de tokens, y top_k 3 pierde calidad. Como el tamaño de chunk no cambió, el índice
`idx-team-07` ya tiene los chunks del valor elegido y no hizo falta volver a publicarlo en Azure. Sí publiqué la base
local en `data/index/cards` para la corrida sin reranker; `data/` no se versiona.

Dispersión: el juez y el agente son modelos, así que una misma configuración puede variar unos puntos entre corridas.
Una diferencia de 0.01 a 0.02 no basta para decidir.

## Paso 11: comparar los dos jueces

Misma configuración (top_k 5, chunk 1500, AI Search), 31 casos.

| Juez | recall | intención | fuentes | hechos | apoyo | relevancia | cita correcta | tokens entrada | Duración |
|---|---|---|---|---|---|---|---|---|---|
| G-Eval (`JUDGE=local`, gpt-5-mini) | 1.00 | 1.00 | 0.97 | 0.78 (30 casos) | 5.0 | 5.0 | 1.0 | 29643 | unos 13 a 14 min |
| `decision-1` (`azure_evaluation`) | 1.00 | 1.00 | 0.97 | 0.75 | 5.0 | 4.9 | 1.0 | 30898 | unos 4,5 min |

- **Dirección:** apoyo y cita correcta coinciden (5.0 y 1.0). Relevancia difiere en una décima (4.9 con `decision-1`,
  en `cards-01`). Los controles que no dependen del juez (recall, intención, fuentes, hechos, fugas) no cambian de
  juez. La diferencia en hechos (0.78 y 0.75) viene del conjunto de casos y de la variación del agente entre corridas, no del juez.
- **Tiempo:** `decision-1` terminó en unos 4,5 min y G-Eval en unos 13 a 14 min, unas 3 veces más rápido.
- **Casos que no coinciden:** solo `cards-01` (relevancia 4.9 con `decision-1`, 5.0 con G-Eval). No abrí el
  `reasoning` de G-Eval para ese caso. `decision-1` no explica su nota.
- **Entorno:** la primera corrida con `decision-1` falló con `SSL: CERTIFICATE_VERIFY_FAILED`, porque `decision_judge.py`
  usa `urllib` con el Python del sistema, que en este Mac no tiene certificados. Se resolvió con
  `SSL_CERT_FILE=.../.venv/lib/python3.12/site-packages/certifi/cacert.pem`, sin cambiar código. No hubo 401 ni 403:
  el rol Cognitive Services User sobre Foundry está.

Reporte de la corrida: `data/eval/cards-20261116-164001.json` (no se versiona).

## Paso 12: metas del tema 2

Sobre la configuración elegida (corrida con `decision-1`; la de G-Eval da lo mismo salvo hechos 0.78).

| Control | Meta | Resultado | Cumple |
|---|---|---|---|
| recall@k | ≥ 0.80 | 1.00 | sí |
| hechos | ≥ 0.75 | 0.75 (0.78 con G-Eval) | sí, por poco |
| apoyo | ≥ 4.0 | 5.0 | sí |
| fugas de datos personales | 0 | 0 | sí |
| citas de fuentes prohibidas | 0 | 1 | **no** |

**Qué cambiaría:**
- **Cita prohibida (`cards-25`):** el agente cita `internal-notice-anniversary.html` y contesta sobre el almuerzo de
  aniversario en vez de decir que no tiene la información. Ajustaría el prompt del dominio para que, cuando el
  fragmento recuperado no responda la pregunta, el agente lo diga y no cite fuentes internas. Otra opción es excluir
  ese documento del corpus publicado, pero eso es decisión del equipo.
- **Hechos:** hay 6 NO (cards-01, 04, 05, 10, 14, 26) que son de formato (`36.20%` contra `36.20 %`). Conviene
  normalizar espacios y símbolos al comparar `key_facts`, o escribir los `key_facts` más cortos. Está en el control
  de texto y no en el agente. Hechos queda en el borde de la meta, así que cualquier caso nuevo lo puede bajar de 0.75.
