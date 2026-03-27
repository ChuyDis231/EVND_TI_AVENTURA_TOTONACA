## Asistencia de IA – Punto 8: Actualización del Procedimiento de Simulación
---
#### Descripción:
Uso del asistente de inteligencia artificial Claude (Anthropic) como apoyo para analizar, comprender y mejorar los procedimientos almacenados `simula_carritos` y `simula_pedido` de la base de datos del equipo `ecommerce_230438`.

#### Objetivo:
Modificar el procedimiento almacenado que simula los carritos de compra y su detalle, asegurando:
- Congruencia en la línea temporal de los datos
- Coherencia entre la duración de la sesión y las acciones del usuario
- Lógica realista en el tiempo de selección de productos
- Secuencia adecuada de inserciones y actualizaciones

---

## Prompt 1 — Análisis de la base de datos

**Prompt enviado:**
> "Hola, esta vez para no trabajar uno solo decidimos trabajar todo el equipo con una misma base de datos, aquí te la mando para que la analices. En este caso me tocó hacer el punto 8. Necesito de tu ayuda para completar el punto 8."

**Respuesta obtenida:**
Claude analizó el archivo SQL de la base de datos `ecommerce_230438` e identificó los procedimientos almacenados existentes, en particular:
- `simula_carritos` — procedimiento principal a modificar
- `simula_pedido` — procedimiento con fechas incorrectas (usando `DEFAULT`)
- `genera_fechahora_lapso` — función auxiliar de tiempos
- Comentario encontrado en el código original: *"las fechas deberán ser corregidas para el examen práctico, deben ser posteriores a las del carrito"*

---

## Prompt 2 — Solicitud de mejoras con congruencia temporal

**Prompt enviado:**
> "Necesito que modifiques el procedimiento `simula_carritos` asegurando: congruencia en la línea temporal, coherencia entre la duración de sesión y las acciones del usuario, lógica realista en el tiempo de selección de productos, y secuencia adecuada de inserciones y actualizaciones."

**Respuesta obtenida:**
Claude generó la versión mejorada con las siguientes mejoras técnicas:
- El carrito se crea entre **5 y 60 segundos después** del inicio de sesión
- Cada producto se agrega con un intervalo de **2 a 8 minutos** entre uno y otro, acorde a una boutique de ropa
- Si un producto es eliminado, su `fecha_actualizacion` es **posterior** al `fecha_registro` del ítem
- La `fecha_actualizacion` del carrito refleja el **último cambio real** en el detalle
- Ningún ítem puede registrarse después del fin de sesión

---

## Prompt 3 — Corrección de simula_pedido

**Prompt enviado:**
> "También necesito que corrijas `simula_pedido` para que las fechas del pedido sean posteriores a las del carrito."

**Respuesta obtenida:**
Claude corrigió `simula_pedido` sustituyendo los valores `DEFAULT` por fechas calculadas dinámicamente:
- El pedido se registra entre **30 segundos y 3 minutos** después del último ítem del carrito
- La `fecha_actualizacion` del pedido es **posterior** al registro, entre 1 y 5 minutos después
- Se leen las fechas reales del carrito para garantizar la secuencia correcta

---

## Prompt 4 — Ajuste de intervalos de tiempo

**Prompt enviado:**
> "En el tiempo de cada producto, ¿no es muy poco? El profe dijo que mínimo la gente tarda 5 minutos para agregar 3 productos. ¿Cómo lo corrijo?"

**Respuesta obtenida:**
Claude identificó que el intervalo era demasiado corto y propuso el siguiente cambio:

```sql
-- Antes (muy rápido):
SET v_intervalo_seg = FLOOR(30 + RAND() * 150);

-- Después (realista para boutique de ropa):
SET v_intervalo_seg = FLOOR(120 + RAND() * 360);
```

---

## Prompt 5 — Limpieza de datos y verificación

**Prompt enviado:**
> "¿Cuáles serían los comandos para eliminar los datos y ver cómo se genera la información?"

**Respuesta obtenida:**

```sql
-- Limpiar datos anteriores
SET FOREIGN_KEY_CHECKS = 0;
DELETE FROM carrito_detalle;
DELETE FROM carritos;
SET FOREIGN_KEY_CHECKS = 1;

-- Generar nuevos carritos
CALL simula_carritos(5, NULL, NULL);

-- Verificar orden temporal
SELECT cd.carrito_id, cd.fecha_registro, cd.estatus,
       c.fecha_creacion, c.fecha_actualizacion
FROM carrito_detalle cd
JOIN carritos c ON c.id = cd.carrito_id
ORDER BY cd.carrito_id, cd.fecha_registro ASC;
```

---

#### Evidencia

![Imagen 1 - Creación de carritos](./8_Actualizacion_procedimiento_simulacion/generacion_carritos_comandos.png)
![Imagen 2 - Visualizacion de los detalles de los carritos](./8_Actualizacion_procedimiento_simulacion/detalles_carritos.png)

---

## Resultado final
Se obtuvieron los procedimientos `simula_carritos` y `simula_pedido` corregidos y mejorados, los cuales fueron revisados, comprendidos e implementados por la estudiante en la base de datos del equipo `ecommerce_230438`.

**Herramienta utilizada:** Claude (Anthropic) — https://claude.ai
