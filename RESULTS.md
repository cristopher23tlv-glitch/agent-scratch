| Mensaje | Intención | latency_ms | prompt_tokens | completion_tokens | response |
|---|---|---|---|---|---|
| `Hola, soy Cris` | greeting | 4180 | 381 | 175 | ¡Hola Cris! Soy el agente de tarjetas del Banco Demo. ¿En qué puedo ayudarte hoy con tus tarjetas de crédito? |
| `¿Qué diferencia hay entre la Clásica y la Platinum?` | product_question | 6329 | 389 | 498 | En general, la diferencia entre una tarjeta Clásica y una Platinum suele ser:<br><br>- Límites y requisitos: la Platinum exige mejores ingresos y ofrece límites de crédito más altos.<br>- Cuota anual: la Platinum tiene una cuota anual mayor que la Clásica.<br>- Beneficios y recompensas: la Platinum incluye más beneficios (mayores acumulaciones de millas/puntos, promociones exclusivas, concierge).<br>- Seguros y asistencias: la Platinum normalmente incorpora seguros de viaje, protección de compras y asistencia en viajes de mayor cobertura.<br>- Accesos y servicios: la Platinum puede ofrecer acceso a salas VIP en aeropuertos, prioridad en servicios y mayores topes para compras.<br>- TEA/TCEA y condiciones: las tasas (TEA/TCEA), comisiones y promociones pueden ser diferentespara cada tarjeta.<br><br>Si quieres, te envío la ficha detallada de ambas con beneficios, TEA/TCEA y anualidad para compararlas, o dime qué tipo de gastos haces y te recomiendo la más adecuada. |
| `¿Cuánta línea disponible tengo?` | data_query | 4737 | 384 | 439 | No puedo ver tu línea disponible desde aquí por seguridad. Para consultarla: 1) Abre la app Banco Demo > Tarjetas > selecciona la tarjeta > ver “Línea disponible”. 2) O ingresa a Banca por Internet > Mis productos > Tarjetas. 3) También figura en tu estado de cuenta o llamando a Atención al Cliente/visitando una sucursal. ¿Quieres quete guíe paso a paso en la app o web? |
| `Quiero un aumento de línea` | sensitive_action | 6267 | 383 | 636 | Entiendo: quieres un aumento de línea. Este trámite requiere confirmación y verificación antes de ejecutarse, y por seguridad no puedo hacerlo directamente aquí. Para continuar debes solicitarlo por un canal seguro (app/banca online, llamada al centro de atención o en una sucursal). Te pedirán actualizar y presentar documentación (comprobantes de ingresos, antigüedad laboral y autorización para consulta en centrales de riesgo). ¿Cómo prefieres que te indique los pasos: app, llamada o sucursal? |
| `¿A cuánto está el dólar hoy?` | out_of_scope | 3703 | 385 | 273 | Lo siento, no manejo cotizaciones de moneda. Para el tipo de cambio del dólar hoy consulta la web del Banco Central de Reserva del Perú, tu banco o servicios como Google/Reuters. ¿Deseas ayuda con algo sobre tarjetas de crédito, millas, seguros o línea de crédito? |

-----------
Session 3
-----------


| Tamaño máximo del chunk (caracteres) | Cantidad de chunks | Qué se gana | Qué se pierde |
|---:|---:|---|---|
| 250 | 36 | Precisión: cada chunk habla de una sola cosa. | Contexto: una regla y su excepción pueden quedar en chunks distintos. |
| 1500 | 29 | Precisión: cada chunk habla de una sola cosa. | Contexto: una regla y su excepción pueden quedar en chunks distintos. |