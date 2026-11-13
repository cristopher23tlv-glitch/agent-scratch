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
