# Documentación de Análisis e Interacciones con IA (Google AI Studio)

**Proyecto:** Pedidos de comedor  
**Estudiante:** Jorge Luciano Barrientos Riveros  
**Curso:** 2.° curso - 3.ª sección - Turno mañana  

---

## 1. Análisis de Entidades, Relaciones y Requerimientos

### Prompt enviado:
> "Actúa como un analista de sistemas. Para un proyecto escolar de gestión de 'Pedidos de comedor', necesito que identifiques las entidades exactas (Menús, Comensales, Pedidos, Bitácora), sus atributos o campos principales (incluyendo claves primarias y foráneas), las relaciones entre ellas y las reglas de negocio necesarias."

### Respuesta y Análisis Estructurado:

#### Entidades y Atributos
* **Menús (`menus`):**
  * `id` (PK, Clave primaria)
  * `nombre` (Text, Nombre del menú)
  * `fecha` (Date, Fecha para la que aplica)
  * `cupo` (Integer, Cantidad máxima disponible)
  * `precio` (Numeric, Precio por porción)
* **Comensales (`comensales`):**
  * `id` (PK, Clave primaria)
  * `documento` (Text, Cédula o identificador único)
  * `nombre` (Text, Nombre y apellido)
* **Pedidos (`pedidos`):**
  * `id` (PK, Clave primaria)
  * `menu_id` (FK -> `menus.id`, Clave foránea)
  * `comensal_id` (FK -> `comensales.id`, Clave foránea)
  * `cantidad` (Integer, Cantidad solicitada)
  * `precio_aplicado` (Numeric, Precio al momento del pedido)
  * `estado` (Text, Ej: "Confirmado", "Cancelado")
* **Bitácora (`bitacora`):**
  * `id` (PK, Clave primaria)
  * `usuario` (Text, Identificador del usuario que ejecutó la acción)
  * `fecha` (Timestamp, Fecha y hora exacta)
  * `entidad` (Text, Tabla afectada)
  * `identificador` (Text, ID del registro afectado)
  * `accion` (Text, Alta, Modificación o Baja)

#### Relaciones
* Un **Menú** puede estar presente en muchos **Pedidos** (1 a N).
* Un **Comensal** puede realizar muchos **Pedidos** (1 a N).
* Cada **Pedido** pertenece a un solo **Menú** y a un solo **Comensal**.

#### Reglas de Negocio
* Bloquear pedidos si la `cantidad` supera el cupo restante o si no es positiva.
* Cancelar un pedido restituye/libera el cupo.
* Calcular el importe total con el `precio_aplicado`.
* Registrar automáticamente altas, modificaciones y bajas en la bitácora sin permitir su edición desde la interfaz.
---

## 2. Interacciones con IA y Registro de Decisiones

### Interacción 1: Estructura del Esquema SQL
* **Objetivo:** Definir las tablas y llaves foráneas para Supabase.
* **Prompt:** "Genera el script SQL para crear las tablas menus, comensales, pedidos y bitacora con sus restricciones de clave primaria y foránea."
* **Síntesis de respuesta:** La IA propuso el script de creación de tablas usando tipos `UUID` y claves foráneas en `pedidos`.
* **Decisión adoptada:** Se aceptó la estructura propuesta para mantener la integridad referencial.
* **Commit/Archivo:** `schema.sql` (Commit: `init database schema`).
* **Prueba realizada:** Ejecución exitosa de las sentencias SQL en el editor de Supabase.

### Interacción 2: Lógica de Validación de Cupos
* **Objetivo:** Validar que un pedido no supere el cupo disponible.
* **Prompt:** "¿Cómo puedo validar en el servidor que un pedido no supere el cupo restante del menú?"
* **Síntesis de respuesta:** La IA sugirió una consulta previa a la tabla `menus` para comparar el cupo contra los pedidos existentes.
* **Decisión adoptada:** Se implementó una función/trigger o validación en el servidor antes de insertar el pedido.
* **Commit/Archivo:** `services/pedidos.js` (Commit: `add cupo validation`).
* **Prueba realizada:** Intento de reserva de un pedido con cantidad mayor al cupo; el sistema respondió con error 400.

### Interacción 3: Registro Automático en Bitácora
* **Objetivo:** Auditar los cambios de alta, modificación y baja.
* **Prompt:** "Escribe la función en Supabase para registrar automáticamente en la tabla bitacora cada operación CRUD."
* **Síntesis de respuesta:** La IA proporcionó un Trigger de PostgreSQL que inserta usuario, fecha, entidad y acción.
* **Decisión adoptada:** Se aplicó el trigger a la tabla de `pedidos` y `menus`.
* **Commit/Archivo:** `triggers.sql` (Commit: `add audit log trigger`).
* **Prueba realizada:** Se insertó un registro de prueba y se verificó que apareciera en la tabla `bitacora`.

### Interacción 4: Regla de Cancelación de Pedidos
* **Objetivo:** Reintegrar el cupo al cancelar un pedido sin permitir múltiples devoluciones.
* **Prompt:** "Diseña la lógica para actualizar el estado a 'Cancelado' y liberar el cupo del menú solo una vez."
* **Síntesis de respuesta:** La IA sugirió actualizar el estado y verificar que no estuviera previamente cancelado.
* **Decisión adoptada:** Se agregó una restricción que impide procesar la cancelación de un pedido con estado 'Cancelado'.
* **Commit/Archivo:** `controllers/pedidos.js` (Commit: `implement cancellation logic`).
* **Prueba realizada:** Se canceló un pedido dos veces seguidas; la segunda operación fue denegada por el sistema.

### Interacción 5: Búsqueda y Filtros Combinados
* **Objetivo:** Implementar filtros por comensal, menú, fecha, estado e importe.
* **Prompt:** "¿Cómo construir una consulta con filtros dinámicos combinados y paginación de 10 en 10?"
* **Síntesis de respuesta:** La IA propuso usar parámetros opcionales en la consulta de Supabase junto con `.range(start, end)`.
* **Decisión adoptada:** Se adaptó la consulta del panel de administración para aceptar múltiples parámetros.
* **Commit/Archivo:** `services/busqueda.js` (Commit: `add multi-filter query`).
* **Prueba realizada:** Búsqueda combinada por fecha y estado 'Confirmado' obteniendo exactamente 10 resultados por página.

---

## 3. Propuesta de IA Corregida / Descartada

* **Propuesta recibida de la IA:** La IA sugirió eliminar físicamente el registro de la tabla `pedidos` cuando un usuario presione "Eliminar".
* **Motivo del descarte:** La consigna prohíbe borrar el historial o romper relaciones entre registros activos y pasados.
* **Decisión técnica tomada:** Se descartó el borrado físico (`DELETE`) y se implementó un borrado lógico mediante el cambio de estado o el bloqueo de baja en registros referenciados para garantizar la consistencia de los datos.