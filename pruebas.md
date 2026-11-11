# Casos de Prueba - Proyecto Pedidos de Comedor

**Estudiante:** Jorge Luciano Barrientos Riveros  
**Curso:** 2.° curso - 3.ª sección - Turno mañana  

---

## Listado de Casos de Prueba

### CP01: Registro exitoso de comensal
* **Descripción:** Registrar un nuevo comensal con cédula única y nombre válido.
* **Datos de entrada:** Documento: `9988776`, Nombre: "Ana Benítez".
* **Resultado esperado:** Registro almacenado correctamente en la tabla `comensales`.
* **Estado:** Aprobado.

### CP02: Validación de documento duplicado en comensal
* **Descripción:** Intentar registrar un comensal con un número de documento ya existente.
* **Datos de entrada:** Documento ya registrado (ej: `1234567`).
* **Resultado esperado:** La base de datos rechaza la inserción por la restricción `UNIQUE`.
* **Estado:** Aprobado.

### CP03: Alta de menú con cupo y precio válido
* **Descripción:** Crear un nuevo menú diario con cupo positivo y precio mayor o igual a cero.
* **Datos de entrada:** Menú: "Guiso de Arroz", Cupo: 35, Precio: 10000.
* **Resultado esperado:** Inserción exitosa en la tabla `menus`.
* **Estado:** Aprobado.

### CP04: Restricción de cupo negativo en menú
* **Descripción:** Intentar registrar un menú con un cupo menor a cero.
* **Datos de entrada:** Cupo: -5.
* **Resultado esperado:** La restricción `CHECK (cupo >= 0)` bloquea la operación.
* **Estado:** Aprobado.

### CP05: Registro de pedido dentro del cupo disponible
* **Descripción:** Crear un pedido cuya cantidad no supere el cupo restante del menú.
* **Datos de entrada:** Menú ID 1 (Cupo 50), Cantidad: 2.
* **Resultado esperado:** Pedido registrado con éxito y estado 'Confirmado'.
* **Estado:** Aprobado.

### CP06: Bloqueo de pedido con cantidad no positiva
* **Descripción:** Intentar registrar un pedido con cantidad cero o negativa.
* **Datos de entrada:** Cantidad: 0 o -1.
* **Resultado esperado:** La restricción `CHECK (cantidad > 0)` rechaza el pedido.
* **Estado:** Aprobado.

### CP07: Cancelación de pedido y actualización de estado
* **Descripción:** Cambiar el estado de un pedido confirmado a 'Cancelado'.
* **Datos de entrada:** ID de pedido existente con estado 'Confirmado'.
* **Resultado esperado:** El estado cambia correctamente a 'Cancelado'.
* **Estado:** Aprobado.

### CP08: Registro automático en la bitácora (Audit Log)
* **Descripción:** Verificar que toda operación de alta o modificación genere un registro en la tabla de auditoría.
* **Datos de entrada:** Inserción de un nuevo pedido.
* **Resultado esperado:** Se añade una fila en la tabla `bitacora` detallando la acción y el usuario.
* **Estado:** Aprobado.