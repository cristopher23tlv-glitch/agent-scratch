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

## Sesión 6 — Tools del MCP en el chat

Dominio: `cards`. MCP: el mock (`http://127.0.0.1:8200/mcp`). Todavía no tengo `MCP_URL` ni `MCP_SCOPE` del compartido
(`bank-mcp`), así que el `.env` queda con el mock y `MCP_SCOPE` vacío. Foundry real (`gpt-5-mini`) e
índice `idx-team-07` en Azure AI Search. Los pasos 1 a 11 dejan 110 tests en verde.

### Tools de tu dominio

| MCP | Tools que lista `dev/mcp/mcp_tools.py` |
|---|---|
| Mock | 3: `get_cards` (consulta), `get_credit_line` (consulta), `request_credit_line_increase` (acción sensible) |
| Compartido | pendiente: aún no tengo la URL ni el `MCP_SCOPE` |

Con `DOMAIN=loans` el mismo comando lista 3 tools de loans, y pedir `simulate_installment` con `cards` responde
`Unknown tool`. En MCP Inspector, con el header `X-Bank-Domain: cards` se ven las 3 tools (las dos consultas como solo
lectura y la acción como destructiva) y sin header se ven las 15. `customer_id=ABC` falla con "String should match
pattern '^CUS-\d{4}$'".

Llamadas directas del paso 7: `get_credit_line` da línea 3000, usada 1240.5, disponible 1759.5, máximo 4500.
`request_credit_line_increase` con 9000 falla las dos veces con "La línea máxima para esta tarjeta es 4500.00 PEN";
con 4000 devuelve `CLI-9001` en estado `registered`.

### Conversación en `dev/chat.py` (paso 12)

| Prueba | Mensaje | Tool pedida (`→`) | Qué pasó | Primer texto / respuesta completa (ms) |
|---|---|---|---|---|
| Consulta de tu dominio | Soy CUS-1001, ¿puedo subir la línea de mi tarjeta CRD-2001 a 4000? | `get_credit_line(card_id=CRD-2001)` | Línea 3000, máximo 4500. Pidió además `request_credit_line_increase(4000)` sin esperar confirmación | 28680 / 28948 |
| Acción confirmada | Sí, solicítalo, el motivo es mayor gasto familiar | `request_credit_line_increase(card_id=CRD-2001, requested_limit=4000)` | NO SE EJECUTA, pendiente de aprobación. Respuesta: "Solicitud registrada ... pendiente de aprobación" | 30060 / 30321 |
| Sin tool que la resuelva | ¿Cuál es el saldo de mi cuenta de ahorros? | ninguna | Dijo que no puede consultarlo y mandó a la app o a Atención al Cliente | 13454 / 13597 |
| Tool de otro dominio | ¿Cuánto pagaría al mes por un préstamo de 20000 soles a 24 meses? | ninguna (`simulate_installment` no existe para cards) | Pidió la TEA o el producto y no dio una cuota. Citó `tea-and-tcea.md` | 8436 / 8598 |

Repetí la consulta y la confirmación 2 veces más (corridas 2 y 3, sesiones nuevas):

| Corrida | Mensaje 1: tools | Mensaje 2: tools | Palabras de la respuesta del mensaje 2 |
|---|---|---|---|
| 1 | `get_credit_line` y luego `request_credit_line_increase(4000)` | `request_credit_line_increase(4000)` | "Solicitud registrada ... pendiente de aprobación" |
| 2 | `request_credit_line_increase(4000)` | `request_credit_line_increase(4000)` | "He registrado la solicitud ... pendiente de aprobación" |
| 3 | `request_credit_line_increase(4000)` | `request_credit_line_increase(4000)` | "Listo: he solicitado el aumento ... pendiente de aprobación" |

Lo que funciona: las 6 llamadas a `request_credit_line_increase` volvieron con `pending_approval` y el MCP no recibió
ninguna; el modelo eligió la tool y sus argumentos sin ayuda; las respuestas citan `credit-line-increase.md` junto con
el resultado de la tool.

Lo que no sale como espera la guía, en las 3 corridas:
- **Pide la acción en el primer mensaje.** Con "¿puedo subir ... a 4000?" ya llamó a `request_credit_line_increase`. En
  las corridas 2 y 3 ni consultó `get_credit_line` antes, así que no verificó el máximo de 4500.
- **Dice "registré" o "listo".** Aunque añade que queda pendiente de aprobación, la guía toma esas palabras como señal
  de que el modelo no siguió bien el resultado `pendiente_aprobacion`. No es un riesgo de seguridad, porque la tool no
  se ejecuta, pero el texto al cliente es ambiguo.

### ¿El modelo inventó algún dato cuando ninguna tool lo resolvía?

No. Con el saldo de ahorros no dio ningún monto. Con la cuota del préstamo, que en cards no tiene tool, no calculó
ni estimó: pidió la TEA. Los montos de la línea (3000, máximo 4500) salieron de `get_credit_line` y coinciden con lo
que dio el mock en el paso 7. Lo único que añade el modelo por su cuenta son plazos y requisitos del aumento de
línea (2 días hábiles, 24 horas, tarjeta activa 6 meses), y vienen del corpus (`credit-line-increase.md`), citado.

### Conclusión

El MCP simulado cumple el contrato (15 tools, filtro por dominio, errores legibles), y el agente usa las tools de
cards desde el chat con las acciones sensibles bloqueadas. El punto flojo es el comportamiento del modelo: pide la
acción antes de que el cliente confirme y la describe como "registrada". Cambiaría `TOOLS_HEADER` (o el prompt de
cards) para exigir consultar `get_credit_line` primero, esperar la confirmación del cliente y decir "la pedí, queda
pendiente de un asesor". Lo dejo para después, porque esta sesión no lo pide. Falta probar con el MCP compartido
cuando tenga `MCP_URL` y `MCP_SCOPE`: solo cambian esas dos líneas del `.env`.

## Sesión 7 — Skills, memoria de trabajo y Agent Framework

Dominio: `cards`. Esta vez usé el MCP compartido (`bank-mcp`, con `MCP_SCOPE` y token de Entra): al arrancar lista 3 tools
(`get_cards`, `get_credit_line`, `request_credit_line_increase`), las mismas del mock. Foundry real (`gpt-5-mini`), índice
`idx-team-07` y `SEARCH_TOP_K=5`. `uv run pytest` da 119 passed y 5 skipped (124 en total, como pide la guía). El runtime
se cambia solo con `AGENT_RUNTIME` en el `.env`.

Conversación de 4 turnos, la misma con los dos runtimes: (1) «Soy CUS-1001, ¿puedo subir la línea de mi tarjeta CRD-2001
a 4000?» · (2) «Sí, solicítalo, el motivo es mayor gasto familiar» · (3) «¿Cuál es el saldo de mi cuenta de ahorros?» ·
(4) «¿Cuál era la línea máxima de mi tarjeta?». Los tiempos y tokens salen de los mismos eventos que muestra `dev/chat.py`
(corrí el agente en proceso, con el mismo `build_chat_service` que usa la API).

### Una corrida de cada runtime

| | `react` (corrida 2) | `agent_framework` (corrida 1) |
|---|---|---|
| Turno 1: tools pedidas | `get_cards(CUS-1001)` y `get_credit_line(CRD-2001)` | Las mismas, en el mismo orden |
| Turno 2: tool pedida | Solo `request_credit_line_increase(CRD-2001, 4000)` | Igual |
| La acción sensible quedó **NO SE EJECUTA** | Sí: `pendiente_aprobacion` | Sí: igual |
| Turno 2 sin `get_cards` ni `get_credit_line` previos | Sí | Sí |
| Turno 3 (ahorros): tools | Ninguna, `out_of_scope` | Ninguna, `out_of_scope` |
| Turno 4: respondió sin `→` | **No**: dijo que en los fragmentos no aparecía la línea máxima y ofreció consultarla | Sí: «S/ 4,500», sin ninguna tool |
| Primer texto (ms), turnos 1 / 2 / 3 / 4 | 22.996 / 23.786 / 12.494 / 12.801 (suma 72.077) | 13.623 / 25.060 / 11.401 / 13.748 (suma 63.832) |
| Respuesta completa (ms), suma | 72.929 | 64.451 |
| Tokens de entrada (turnos 1 / 2 / 3 / 4) | 4.466 / 4.659 / 3.005 / 3.185 → **15.315** | 4.466 / 4.617 / 3.045 / 3.196 → **15.324** |
| Tokens de salida (turnos 1 / 2 / 3 / 4) | 2.167 / 2.801 / 1.159 / 1.186 → **7.313** | 1.140 / 2.940 / 905 / 1.370 → **6.355** |

### Corridas extra (3 de `react`, 2 de `agent_framework`)

Repetí la conversación porque `react` no se comportó igual en las tres. Con `agent_framework` las dos corridas fueron iguales.

| Corrida | Qué pasó |
|---|---|
| `react` 1 | `get_cards` falló en el turno 1 con «El servicio del banco no responde» (error del MCP compartido). El modelo lo dijo y no inventó ningún dato. En el turno 2 volvió a consultar las tarjetas y la línea antes de pedir el aumento. Suma de respuestas: 79.286 ms |
| `react` 2 | La de la tabla. En el turno 4 no usó la memoria y se ofreció a consultar |
| `react` 3 | En el turno 1 pidió `request_credit_line_increase(4000)` sin esperar mi confirmación (quedó `pendiente_aprobacion`). Turno 4 respondió S/ 4.500 sin tool. Suma: 80.331 ms |
| `agent_framework` 1 | La de la tabla |
| `agent_framework` 2 | Mismas tools que la corrida 1, y turno 4 con S/ 4.500 sin tool. Suma: 67.180 ms |

### ¿La skill frenó la tool de otro dominio?

Mensaje: «¿Cuál es el saldo de mi cuenta de ahorros?» (pregunta de otro dominio; no usé una de `fees` porque el mío es `cards`).
En las 5 corridas, con los dos runtimes, la intención fue `out_of_scope` y no hubo ninguna llamada `→`. El agente dijo que
no puede consultar saldos de ahorros, no dio ningún monto y mandó a la app, la banca por internet, el cajero o Atención al
Cliente. La skill de `cards` solo le ofrece 3 tools; si el modelo pide otra, `_run` la devuelve como error
(lo cubre `test_skill_decides_which_tools_the_model_sees_and_adds_its_instructions`, no lo probé contra el MCP real).

### Memoria de trabajo

Después de cada turno con tools se guarda un mensaje `system` con cada tool, sus argumentos y su resultado. En la
conversación de `agent_framework` 2 quedó guardado, entre otras cosas:
`get_credit_line({"card_id": "CRD-2001"}) → ok: {... "max_eligible": 4500.0 ...}`. Por eso el turno 2 pidió el aumento con
`CRD-2001` y `4000` sin consultar de nuevo, y el turno 4 contestó la línea máxima sin ninguna tool. Solo falló en
`react` 2, donde el dato estaba en la memoria pero el modelo no lo usó.

### Diferencias entre los dos runtimes

- **Tools y aprobación: iguales.** Mismas tools en el turno 1 (salvo `react` 1, por el error del MCP, y `react` 3), la acción sensible siempre `NO SE EJECUTA`
  y la misma intención en cada turno (`sensitive_action`, `sensitive_action`, `out_of_scope`, `data_query`). El turno 1
  gasta los mismos 4.466 tokens de entrada, así que el prompt es el mismo. El cambio fue una línea del `.env`.
- **Latencia: `agent_framework` salió más rápido, pero con pocas corridas.** Sumas de respuesta: `react` 72,9 s y 80,3 s
  (sin contar la corrida con el error del MCP), `agent_framework` 64,5 s y 67,2 s. Los rangos no se cruzan, pero con 2 o 3
  corridas cada uno y un modelo que razona (turnos de 11 a 29 s) lo trato como una tendencia, no como un resultado.
- **Cuándo pide la acción sensible.** Con `agent_framework` pidió `request_credit_line_increase` solo después de mi
  confirmación en las 2 corridas. Con `react` lo hizo ya en el turno 1 en 1 de 3 corridas, que es el mismo problema que
  vi en la sesión 6.
- **Streaming poco visible.** Entre el primer texto y la respuesta completa pasan entre 10 y 640 ms: el texto llega en
  pedazos, pero casi al final, cuando el modelo ya terminó de razonar.
- **Cómo describen la acción.** En `react` 2 el agente dijo «He registrado la solicitud» y «Le avisaremos por los canales
  registrados». La tool respondió que no se registró todavía, así que el texto se pasa. En `agent_framework` 1 dijo «He
  solicitado ... quedó pendiente de aprobación», más fiel. En `agent_framework` 2 y `react` 3 dijeron que anotaron el
  motivo («mayor gasto familiar»), pero la tool solo recibe `card_id` y `requested_limit`: el motivo no se guarda.

### Conclusión

- **Cambiar de runtime no cambia las reglas.** Con `react` o con `agent_framework` el agente pide las mismas tools, la acción
  sensible no se ejecuta y el cliente recibe la explicación con los plazos del corpus. La aprobación vive en `ChatService`,
  no en el framework.
- **La skill y la memoria funcionan, pero el modelo manda.** La lista blanca dejó solo 3 tools y la pregunta de ahorros no
  llamó a ninguna. La memoria de trabajo guardó los datos, pero en 1 de 5 corridas el modelo no los usó (`react` 2).
- **Pendiente de mejorar:** la latencia (10 a 30 s hasta el primer texto con cualquier runtime) y el texto al cliente, que
  a veces dice «registrado» o «anoté el motivo» cuando la tool no lo hizo. Ajustaría `skills/cards.md` para pedir la
  acción solo después de la confirmación y para decir «la pedí, queda pendiente».
