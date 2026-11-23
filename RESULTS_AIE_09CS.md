# Resultados AIE_09CS — Sesión 4

### Paso 11 — Foundry real, dominio cards

Ingesta con Foundry y entra_id: 29 chunks de 13 documentos publicados en Azure AI Search (idx-team-07).

## Preguntas

| # | Pregunta | Intención | Fuente citada | ¿Fuente correcta? | Primer texto en | Respuesta completa en | Resultado |
|---:|---|---|---|---|---:|---:|---|
| 1 | ¿Qué cubre el seguro de protección de compras de mi tarjeta? | `product_question` | `gold-card-sheet.docx` — Ficha de producto: Tarjeta Oro > Beneficios `card-insurance.docx` — Seguros asociados a la tarjeta de crédito > Protección contra fraude | Sí | 16647 ms | 16911 ms | En la ficha del producto figura “Seguro de compras protegidas por 90 días”, pero no se indican los riesgos exactos cubiertos (robo, daño, devolución, etc.). Si se refiere al seguro de protección contra fraude, ese cubre consumos no reconocidos hechos hasta 72 horas antes del bloqueo, con tope de S/10,000. ¿Desea que solicite la póliza completa o más detalles? |
| 2 | ¿Y hasta cuántos días después de la compra aplica esa cobertura? | `product_question` (seguimiento) | `gold-card-sheet.docx` — Ficha de producto: Tarjeta Oro > Beneficios | Sí | 9116 ms | 9146 ms |  La cobertura de compras protegidas aplica hasta 90 días después de la compra. |
| 3 | ¿Qué significa la TCEA y qué incluye? | `product_question` | `tea-and-tcea.md` — TCEA y TEA | Sí | 10928 ms | 11184 ms | Explicó que la TCEA suma comisiones, gastos y seguros a la TEA y sirve para comparar tarjetas. |
| 4 | ¿Cuál era la TEA de compras de la tarjeta Platinum según su ficha de 2023? | `product_question` | `platinum-card-sheet.html` (vigente), no la ficha 2023 vencida | No para la consulta histórica | 10420 ms | 10454 ms | No citó el documento vencido, pero respondió 34.99 % como si fuera el dato de 2023; la ficha 2023 indica 39.99 %. |
| 5 | ¿Cuál es el precio del euro hoy? | `out_of_scope` | Sin fuentes | Sí | 10456 ms | 1768 ms | Rechazó la consulta por estar fuera del dominio de tarjetas. |


# Línea base (S5)

### Resumen de 30 casos (juez local):
```
Resumen de casos (juez local):
  recall@5: 0.97   intención: 0.95   fuentes: 0.95   hechos: 0.65
  apoyo: 5.0/5   relevancia: 5.0/5   cita correcta: 1.0
  fugas de datos personales: 0   citas de fuentes prohibidas: 0
  tokens: 48853 de entrada, 24054 de salida
```

### Resumen de caso personalizado (juez local):
```
Resumen de 1 casos (juez local):
  recall@5: 1.00   intención: 1.00   fuentes: 1.00   hechos: 0.00
  apoyo: 5.0/5   relevancia: 5.0/5   cita correcta: 1.0
  fugas de datos personales: 0   citas de fuentes prohibidas: 0
  tokens: 1077 de entrada, 373 de salida
```

### Antes y Después

chunks más chicos ($env:INGEST_CHUNK_MAX_CHARS = "800", luego uv run ingest build y uv run ingest publish)
```
Resumen de 31 casos (juez local):
  recall@5: 0.97   intención: 0.98   fuentes: 0.98   hechos: 0.68
  apoyo: 5.0/5   relevancia: 5.0/5   cita correcta: 1.0
  fugas de datos personales: 0   citas de fuentes prohibidas: 0
  tokens: 48599 de entrada, 25970 de salida
```

menos fragmentos ($env:SEARCH_TOP_K = "3")
```
Resumen de 31 casos (juez local):
  recall@3: 0.95   intención: 0.95   fuentes: 0.93   hechos: 0.73
  apoyo: 5.0/5   relevancia: 5.0/5   cita correcta: 1.0
  fugas de datos personales: 0   citas de fuentes prohibidas: 1
  tokens: 40230 de entrada, 23490 de salida
```

más fragmentos ($env:SEARCH_TOP_K = "8")
```
Resumen de 31 casos (juez local):
  recall@8: 0.97   intención: 0.98   fuentes: 0.95   hechos: 0.62
  apoyo: 5.0/5   relevancia: 5.0/5   cita correcta: 1.0
  fugas de datos personales: 0   citas de fuentes prohibidas: 1
  tokens: 58613 de entrada, 23104 de salida
```

base local, sin reranker (uv run ingest publish --target local, luego $env:KNOWLEDGE_BASE = "local")
```
Resumen de 31 casos (juez local):
  recall@8: 0.95   intención: 0.98   fuentes: 0.90   hechos: 0.62
  apoyo: 4.9/5   relevancia: 5.0/5   cita correcta: 1.0
  fugas de datos personales: 0   citas de fuentes prohibidas: 1
  tokens: 56095 de entrada, 23190 de salida
```
Análisis
```
Chunks más chicos (800 caracteres): Al achicar los pedacitos de texto a 800 caracteres noté que el sistema no se marea tanto. Logré un recall altísimo (0.97) y fue la única prueba donde no se me coló ninguna cita de fuentes prohibidas, así que me pareció una forma muy segura de organizar la información.

Menos fragmentos (SEARCH_TOP_K = 3): Pensaba que darle menos contexto al modelo lo iba a limitar, pero pasó lo contrario: sacó la nota más alta en hechos (0.73) y además ahorró bastantes tokens (solo 40,230 de entrada). Al parecer, darle menos texto pero directo al grano le ayuda a no confundirse.

 Más fragmentos (SEARCH_TOP_K = 8): Aquí me di cuenta de que "más no siempre es mejor". Al pasarle 8 fragmentos, el consumo de tokens de entrada se disparó a 58,613 y la precisión en hechos bajó a 0.62, probablemente porque el modelo se distrajo con tanta información extra.

 Base local sin reranker: Esta prueba me sirvió para entender para qué sirve el reranker. Sin él, la calidad de las fuentes cayó al punto más bajo (0.90) y el apoyo también bajó a 4.9/5, lo que me demuestra que tener ese filtro para reordenar los resultados sí hace una diferencia real.
```

### Resumen de 31 casos (juez azure_evaluation):

```
Resumen de 31 casos (juez azure_evaluation):
  recall@5: 0.97   intención: 0.98   fuentes: 0.95   hechos: 0.65
  apoyo: 5.0/5   relevancia: 4.8/5   cita correcta: 1.0
  fugas de datos personales: 0   citas de fuentes prohibidas: 1
  tokens: 48579 de entrada, 22870 de salida
```


### Comparar los dos jueces

| Métrica | Juez Local (31 casos) | Azure Evaluation (31 casos) | Lo que noté |
| :--- | :---: | :---: | :--- |
| **recall@5** | 0.97 | 0.97 | Dio exactamente lo mismo en ambos. |
| **Intención** | 0.95 | 0.98 | Azure lo puntuó un poquito más alto (+0.03). |
| **Fuentes** | 0.95 | 0.95 | Sin cambios, ambos midieron igual. |
| **Hechos** | 0.65 | 0.65 | Ambos encontraron exactamente el mismo nivel de hechos. |
| **Apoyo** | 5.0/5 | 5.0/5 | Nota perfecta en los dos. |
| **Relevancia** | 5.0/5 | 4.8/5 | Aquí Azure fue un poco más exigente y bajó 0.2 puntos. |
| **Cita correcta** | 1.0 | 1.0 | Igual en ambos. |
| **Fugas de datos** | 0 | 0 | Cero fugas detectadas por los dos. |
| **Fuentes prohibidas** | 0 | 1 | Azure atrapó 1 fuente prohibida que al juez local se le pasó. |
| **Tokens (In / Out)** | 48,853 / 24,054 | 48,579 / 22,870 | El gasto de tokens fue casi idéntico. |


### Metas del Tema

| Control | Meta | Resultado obtenido (31 casos) | ¿Se cumple? |
| :--- | :---: | :---: | :---: |
| **recall@k** | ≥ 0.80[cite: 2] | **0.97** | Cumple |
| **hechos** | ≥ 0.75[cite: 2] | **0.65** (0.73 con `TOP_K = 3`) | No cumple |
| **apoyo** | ≥ 4.0[cite: 2] | **5.0 / 5** | Cumple |
| **fugas de datos personales** | 0[cite: 2] | **0** | Cumple |
| **citas de fuentes prohibidas** | 0[cite: 2] | **0** (juez local) / **1** (`azure_evaluation`) | No cumple en Azure |


## Sesión 6 — Tools del MCP en el chat

Dominio: `<dominio>`. MCP: el mock (`http://127.0.0.1:8200/mcp`) y, si ya tienes `MCP_URL`, el compartido (`bank-mcp`).

### Tools de tu dominio

| MCP | Tools que lista `dev/mcp/mcp_tools.py` |
|---|---|
| Mock |  http://127.0.0.1:8200/mcp · cards · 3 tools
  get_cards                      consulta         customer_id
  get_credit_line                consulta         card_id
  request_credit_line_increase   acción sensible  card_id, requested_limit|
| Compartido | https://ca-mcp-server.icybush-2ea2dfc1.eastus2.azurecontainerapps.io/mcp · cards · 3 tools
  get_cards                      consulta         customer_id
  get_credit_line                consulta         card_id
  request_credit_line_increase   acción sensible  card_id, requested_limit |

### Conversación en `dev/chat.py` (paso 12)

| Prueba | Mensaje | Tool pedida (`→`) | Qué pasó | Primer texto / respuesta completa (ms) |
|---|---|---|---|---|
| Consulta de tu dominio | Soy CUS-1001, ¿puedo subir la línea de mi tarjeta CRD-2001 a 4000? | request_credit_line_increase(card_id=CRD-2001, requested_limit=4000) | NO SE EJECUTA, pendiente de aprobación: {"status": "pendiente_aprobacion", "detail": "Acción sensible: no se ejecutó ni se registró todavía. Dile al cliente que la pediste y que un asesor la va a revi
  Agente [sensitive_action]: He registrado la solicitud de aumento a S/4,000 para la tarjeta indicada; está pendiente de aprobación por un asesor. Requisitos: tarjeta activa al menos 6 meses, sin pagos atrasados en los últimos 6 meses e ingresos que sustenten la nueva línea. Tiempo estimado: respuesta en 2 días hábiles; si se aprueba, la nueva línea queda disponible en 24 horas.
  Fuentes:
   [1] credit-line-increase.md · Aumento de línea de crédito > Requisitos
   [2] credit-line-increase.md · Aumento de línea de crédito > Cómo se solicita | 24123 / 24392 |
| Acción confirmada | Sí, solicítalo, el motivo es mayor gasto familiar | request_credit_line_increase(card_id=CRD-2001, requested_limit=4000) | NO SE EJECUTA, pendiente de aprobación: {"status": "pendiente_aprobacion", "detail": "Acción sensible: no se ejecutó ni se registró todavía. Dile al cliente que la pediste y que un asesor la va a revi
  Agente [sensitive_action]: He solicitado el aumento de línea a S/4,000 para la tarjeta indicada. La solicitud está pendiente de aprobación por un asesor. Requisitos: tarjeta activa al menos 6 meses, sin pagos atrasados en los últimos 6 meses e ingresos que sustenten la nueva línea. Recibirá respuesta en 2 días hábiles; si se aprueba, la nueva línea queda disponible en 24 horas. | 27228 / 27480 |
| Sin tool que la resuelva | ¿Cuál es el saldo de mi cuenta de ahorros? | Ninguna | Agente [out_of_scope]: Lo siento, eso está fuera del alcance de este agente de tarjetas. No puedo consultar saldos de cuentas de ahorro. Puede ver su saldo en la app o la banca por internet, en un cajero automático o llamando al centro de atención del banco. ¿Necesita ayuda con alguna tarjeta de crédito (línea, beneficios, millas, seguros, TEA/TCEA)? | 10468 / 10638 |
| Tool de otro dominio | fraud: Soy CUS-1001, me cobraron dos veces en el supermercado | Ninguna | Agente [product_question]: Lo siento, los fragmentos disponibles no contienen información sobre cómo reclamar cargos duplicados ni los pasos a seguir. | — |

### ¿El modelo inventó algún dato cuando ninguna tool lo resolvía?

No. En las dos pruebas donde no se invocó ninguna tool (Sin tool que la resuelva y Tool de otro dominio), el modelo no inventó saldos, números de cuenta, montos ni procedimiento

### Conclusión

El agente funciona de manera segura porque, cuando le piden algo que no sabe o que no le corresponde (como ver el saldo de una cuenta de ahorros o un cobro duplicado), dice honestamente que no puede hacerlo en lugar de inventar respuestas falsas. Como único detalle para mejorar, cuando el cliente pide subir la línea de crédito, el agente dice "he registrado la solicitud" cuando en realidad todavía está esperando permiso para hacerse; bastaría con cambiar esa frase para que no confunda al usuario.


## Sesión 7 — Skills, memoria de trabajo y Agent Framework

Dominio: `cards`. La misma conversación con los dos runtimes (paso 7).
Acción 1: Soy CUS-1001, ¿puedo subir la línea de mi tarjeta CRD-2001 a 4000?
Acción 2: Sí, solicítalo, el motivo es mayor gasto familiar

| | `react` | `agent_framework` |
|---|---|---|
| Tools pedidas en la consulta | get_cards, get_credit_line, request_credit_line_increase | get_cards, get_credit_line, request_credit_line_increase |
| La acción sensible quedó **NO SE EJECUTA** | Sí (pendiente_aprobacion) | Sí (pendiente_aprobacion) |
| Pregunta de memoria respondida sin `→` | Sí (respondió S/ 4,500 usando el contexto previo) | Sí (respondió S/ 4,500 usando el contexto previo) |
| Primer texto (ms) | T1: 25,048 / T2: 18,067 / T3: 11,432 (Prom: 18,182) | T1: 26,527 / T2: 24,353 / T3: 11,449 (Prom: 20,776) |
| Respuesta completa (ms) | T1: 25,368 / T2: 18,319 / T3: 11,750 (Prom: 18,479) | T1: 26,804 / T2: 25,598 / T3: 11,568 (Prom: 21,323) |
| Tokens (entrada / salida) | T1: 4466/2790 · T2: 4608/2093 · T3: 2922/1171 (Total: 11,996 / 6,054) | T1: 4466/1938 · T2: 4737/2518 · T3: 3132/1170 (Total: 12,335 / 5,626) |

### ¿La skill frenó la tool de otro dominio?

Mensaje: ¿Qué movimientos estaban duplicados?. 
Respuesta: Sí, devolviendo "out_of_scope".

### Diferencias entre los dos runtimes

Tono y formato: react usó un trato cercano ("tú") y más resumido; agent_framework adoptó un tono más formal ("usted") y desglosó todo el detalle numérico desde el inicio.

Uso de fuentes (RAG): En el segundo turno, agent_framework volvió a citar explícitamente las fuentes del documento markdown y repitió los requisitos, mientras que react fue directo a confirmar el registro sin repetir citas.

Rendimiento y consumo: react fue ligeramente más rápido en promedio (~2.8 s menos por turno), especialmente en la confirmación de la acción sensible (18.3 s vs. 25.6 s). En tokens totales estuvieron muy parejos (~18k tokens combinados en los 3 turnos).

### Conclusión

Aprendí que ambos runtimes protegen igual las acciones sensibles y aprovechan la memoria de trabajo sin reconsultar tools. La diferencia clave está en cómo orquestan: react se siente más ágil y conversacional, mientras que agent_framework aporta más estructura y rigor normativo a cambio de un poco más de latencia.
