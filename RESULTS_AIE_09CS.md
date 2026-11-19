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