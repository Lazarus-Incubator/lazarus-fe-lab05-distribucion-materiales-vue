# Laboratorio 5: Sistema de Distribución de Libros y Materiales Académicos para Sedes del ICPNA en el Perú

## Contexto

El ICPNA cuenta con sedes y centros de atención en distintas zonas del Perú que requieren de forma periódica libros, workbooks, readers, diccionarios, materiales de evaluación y otros recursos académicos para atender a sus estudiantes en cada ciclo. Cuando la coordinación se realiza mediante correos dispersos, mensajes de WhatsApp y hojas de cálculo aisladas, se presentan problemas frecuentes: solicitudes duplicadas, falta de control de stock, lotes fuera de vigencia, demoras en los despachos y poca trazabilidad sobre qué materiales fueron aprobados, enviados y recibidos por cada sede.

## Objetivo del proyecto

Construir una aplicación Vue que permita registrar y gestionar sedes del ICPNA, materiales académicos, ingresos de lotes al almacén central, solicitudes de distribución, despachos y movimientos de inventario, manteniendo trazabilidad completa del proceso desde la solicitud hasta la recepción final.

## Alcance

Se trabajará con las siguientes entidades relacionadas:

- SedeIcpna
- MaterialAcademico
- LoteIngreso
- SolicitudDistribucion
- EntregaMaterial
- MovimientoInventario

---

# Entidad SedeIcpna

Representa una sede del ICPNA registrada en el sistema.

## Campos

- `id` numérico, lo maneja JSON Server
- `codigo` autogenerado con formato `SDE-YYYY-NNNN`
- `nombre` obligatorio
- `zona` lista fija: `Lima Centro`, `Lima Norte`, `Lima Sur`, `Costa Norte`, `Sierra Sur`, `Oriente`
- `direccion` obligatoria
- `responsable` obligatorio
- `contacto` obligatorio, teléfono o correo
- `estudiantesActivos` obligatorio, mayor a 0
- `estado` lista fija: `Activa`, `Inactiva`, `Suspendida`
- `fechaRegistro` formato `YYYY-MM-DD`

---

# Entidad MaterialAcademico

Representa un material disponible para solicitudes y despacho.

## Campos

- `id` numérico, lo maneja JSON Server
- `sku` autogenerado con formato `MAT-YYYY-NNNN`
- `nombre` obligatorio
- `categoria` lista fija: `Student Book`, `Workbook`, `Reader`, `Diccionario`, `Evaluacion`, `Otros`
- `unidadMedida` lista fija: `unidad`, `paquete`, `caja`, `kit`
- `stockMinimo` obligatorio, mayor o igual a 0
- `controlVigencia` booleano
- `activo` booleano

## Campo derivado

- `stockDisponible` se puede calcular sumando la `cantidadDisponible` de los lotes disponibles del material

---

# Entidad LoteIngreso

Representa el ingreso de un lote al almacén central.

## Campos

- `id` numérico, lo maneja JSON Server
- `codigoLote` autogenerado con formato `LT-YYYY-NNNN`
- `materialId` clave foránea hacia `MaterialAcademico`
- `fechaIngreso` formato `YYYY-MM-DD`
- `fechaFinVigencia` obligatoria si el material tiene `controlVigencia`
- `cantidadIngresada` obligatoria, mayor a 0
- `cantidadDisponible` obligatoria, inicia igual a `cantidadIngresada`
- `proveedor` obligatorio
- `costoUnitario` opcional
- `estado` lista fija: `Disponible`, `Agotado`, `Fuera de vigencia`

---

# Entidad SolicitudDistribucion

Representa una solicitud realizada por una sede del ICPNA. Esta entidad contiene un detalle de ítems.

## Campos

- `id` numérico, lo maneja JSON Server
- `codigo` autogenerado con formato `SOL-YYYY-NNNN`
- `sedeId` clave foránea hacia `SedeIcpna`
- `fechaSolicitud` formato `YYYY-MM-DD`
- `periodoAcademico` formato `YYYY-MM`
- `prioridad` lista fija: `Normal`, `Alta`, `Urgente`
- `estado` lista fija: `Borrador`, `Enviada`, `Observada`, `Aprobada`, `Despachada`, `Entregada`, `Parcial`, `Cancelada`
- `fechaLimiteRevision` formato `YYYY-MM-DD`, calculada según prioridad
- `comentarioSolicitud` opcional
- `comentarioRevision` opcional
- `detalles` arreglo de ítems

## Estructura de `detalles`

Cada ítem del detalle tendrá:

- `materialId`
- `cantidadSolicitada`
- `cantidadAprobada`
- `comentarioItem` opcional

## Regla de fecha límite de revisión

La `fechaLimiteRevision` se calcula automáticamente según la prioridad:

- `Urgente`: 1 día
- `Alta`: 2 días
- `Normal`: 4 días

---

# Entidad EntregaMaterial

Representa el despacho y la entrega de una solicitud aprobada.

## Campos

- `id` numérico, lo maneja JSON Server
- `codigo` autogenerado con formato `ENT-YYYY-NNNN`
- `solicitudId` clave foránea hacia `SolicitudDistribucion`
- `fechaProgramada` formato `YYYY-MM-DD`
- `fechaDespacho` opcional
- `fechaEntrega` opcional
- `estadoEntrega` lista fija: `Programada`, `Despachada`, `En ruta`, `Entregada`, `Con incidencia`
- `responsableAlmacen` obligatorio
- `responsableRecepcion` opcional
- `comentario` opcional
- `itemsEntregados` arreglo de ítems

## Estructura de `itemsEntregados`

Cada ítem entregado tendrá:

- `materialId`
- `loteId`
- `cantidad`

---

# Entidad MovimientoInventario

Permite mantener trazabilidad de todos los ingresos y salidas de almacén.

## Campos

- `id` numérico, lo maneja JSON Server
- `materialId` clave foránea hacia `MaterialAcademico`
- `loteId` clave foránea hacia `LoteIngreso`
- `fecha` formato `YYYY-MM-DD`
- `tipoMovimiento` lista fija: `Ingreso`, `Salida`, `Ajuste`
- `cantidad` obligatoria
- `referenciaTipo` lista fija: `Lote`, `Solicitud`, `Entrega`, `Ajuste`
- `referenciaId` id relacionado
- `comentario` opcional

---

# Funcionalidades

## 1. Dashboard principal

La aplicación debe mostrar un dashboard con indicadores generales.

### Indicadores sugeridos

- total de sedes activas
- total de estudiantes activos atendidos
- solicitudes pendientes de revisión
- solicitudes vencidas
- entregas realizadas en el mes
- materiales con stock bajo
- lotes por vencer o quedar fuera de vigencia en los próximos 15 días

### Visualizaciones sugeridas

- gráfico por estado de solicitudes
- gráfico por categoría de materiales
- ranking de sedes con mayor cantidad de solicitudes

---

## 2. Módulo de sedes ICPNA

### Listado de sedes

Tabla con columnas sugeridas:

- código
- nombre
- zona
- responsable
- estudiantesActivos
- estado
- fechaRegistro

### Funcionalidades

- buscador por código, nombre o responsable
- filtros por zona
- filtros por estado
- botón Nueva sede
- acciones por fila:
  - Ver detalle
  - Editar
  - Eliminar con confirmación

### Formulario de sede

Mismo formulario para crear y editar.

---

## 3. Módulo de materiales académicos

### Listado de materiales

Tabla con columnas sugeridas:

- sku
- nombre
- categoría
- unidadMedida
- stockMinimo
- stockDisponible
- controlVigencia
- activo

### Funcionalidades

- buscador por sku o nombre
- filtros por categoría
- filtros por controlVigencia
- filtros por activo
- indicador visual de stock bajo
- botón Nuevo material
- acciones por fila:
  - Ver detalle
  - Editar
  - Eliminar con confirmación

### Formulario de material

Mismo formulario para crear y editar.

---

## 4. Módulo de lotes de ingreso

### Listado de lotes

Tabla con columnas sugeridas:

- codigoLote
- material
- proveedor
- fechaIngreso
- fechaFinVigencia
- cantidadIngresada
- cantidadDisponible
- estado

### Funcionalidades

- buscador por codigoLote o proveedor
- filtros por material
- filtro de lotes próximos a vencer o salir de vigencia
- filtro por estado
- botón Nuevo lote
- acciones por fila:
  - Ver detalle
  - Editar
  - Eliminar con confirmación

### Formulario de lote

Al crear un lote se debe:

- guardar el lote
- registrar automáticamente un `MovimientoInventario` con:
  - `tipoMovimiento`: `Ingreso`
  - `referenciaTipo`: `Lote`
  - `referenciaId`: id del lote creado

---

## 5. Módulo de solicitudes de distribución

### Listado de solicitudes

Tabla con columnas sugeridas:

- código
- sede ICPNA
- periodoAcademico
- prioridad
- estado
- fechaSolicitud
- fechaLimiteRevision
- total de ítems

### Funcionalidades

- buscador por código o nombre de sede
- filtros por estado
- filtros por prioridad
- filtros por periodoAcademico
- filtros por zona de la sede
- botón Nueva solicitud
- acciones por fila:
  - Ver detalle
  - Editar
  - Eliminar con confirmación

### Formulario de solicitud

Mismo formulario para crear y editar mientras el estado sea `Borrador` u `Observada`.

Debe incluir:

- selección de sede ICPNA
- fecha de solicitud
- periodoAcademico
- prioridad
- comentario de solicitud
- detalle dinámico de ítems usando arreglos reactivos

### Cada ítem del detalle debe permitir

- seleccionar material
- ingresar cantidad solicitada
- ingresar cantidad aprobada
- comentario opcional
- eliminar ítem

### Al crear una solicitud

Se debe:

- generar código `SOL-YYYY-NNNN`
- calcular `fechaLimiteRevision`
- guardar la solicitud
- iniciar con estado `Borrador` o `Enviada` según la estrategia definida

### Reglas del detalle

- debe existir al menos 1 ítem
- no se permiten materiales repetidos
- `cantidadSolicitada` debe ser mayor a 0
- `cantidadAprobada` no puede ser negativa
- `cantidadAprobada` no puede ser mayor a `cantidadSolicitada`

---

## 6. Revisión de solicitudes

Pantalla de detalle de la solicitud con tarjeta resumen y tabla de ítems.

### Debe mostrar

- datos de la sede ICPNA
- datos de la solicitud
- estado actual
- fecha límite de revisión
- detalle de materiales solicitados y aprobados
- stock disponible por material
- botón Aprobar
- botón Observar
- botón Cancelar
- botón Generar entrega, solo si la solicitud fue aprobada

### Reglas de revisión

#### Al aprobar

- la solicitud cambia a estado `Aprobada`
- debe existir al menos un ítem con `cantidadAprobada > 0`
- no se puede aprobar una cantidad mayor al stock disponible actual del material

#### Al observar

- la solicitud cambia a estado `Observada`
- `comentarioRevision` es obligatorio

#### Al cancelar

- la solicitud cambia a estado `Cancelada`
- `comentarioRevision` recomendado

---

## 7. Módulo de entregas

### Flujo general

Una entrega representa el proceso de despacho y recepción de una solicitud aprobada.

### Estados sugeridos

- `Programada`
- `Despachada`
- `En ruta`
- `Entregada`
- `Con incidencia`

### Listado de entregas

Tabla con columnas sugeridas:

- código
- solicitud
- sede ICPNA
- fechaProgramada
- fechaDespacho
- fechaEntrega
- estadoEntrega
- responsableAlmacen

### Funcionalidades

- buscador por código o solicitud
- filtros por estadoEntrega
- filtros por rango de fechas
- acciones por fila:
  - Ver detalle
  - Editar
  - Registrar despacho
  - Registrar entrega

### Crear entrega

Solo se puede crear desde una solicitud `Aprobada`.

Al crear la entrega se debe:

- generar código `ENT-YYYY-NNNN`
- guardar la entrega con estado `Programada`

### Registrar despacho

Al registrar despacho se debe:

- seleccionar los lotes que abastecerán cada material
- registrar `fechaDespacho`
- actualizar `estadoEntrega` a `Despachada`
- descontar `cantidadDisponible` de cada lote usado
- si la `cantidadDisponible` de un lote llega a 0, cambiar estado del lote a `Agotado`
- registrar un `MovimientoInventario` tipo `Salida` por cada ítem o lote afectado
- actualizar la solicitud a estado `Despachada`

### Registrar entrega final

Al registrar recepción se debe:

- guardar `fechaEntrega`
- actualizar `responsableRecepcion`
- registrar comentario si corresponde
- si todo fue entregado correctamente:
  - `estadoEntrega = Entregada`
  - solicitud pasa a `Entregada`
- si hubo faltantes o problemas:
  - `estadoEntrega = Con incidencia`
  - solicitud pasa a `Parcial`

---

## 8. Módulo de movimientos de inventario

### Listado de movimientos

Tabla con columnas sugeridas:

- fecha
- material
- lote
- tipoMovimiento
- cantidad
- referenciaTipo
- referenciaId
- comentario

### Funcionalidades

- filtro por material
- filtro por tipoMovimiento
- filtro por rango de fechas
- vista de historial o kardex simple

---

# Autogeneración de códigos

## Formatos

### Sede ICPNA

`SDE-YYYY-NNNN`

Ejemplos:

- `SDE-2026-0001`
- `SDE-2026-0002`

### Material académico

`MAT-YYYY-NNNN`

Ejemplos:

- `MAT-2026-0001`
- `MAT-2026-0002`

### Lote

`LT-YYYY-NNNN`

Ejemplos:

- `LT-2026-0001`
- `LT-2026-0002`

### Solicitud

`SOL-YYYY-NNNN`

Ejemplos:

- `SOL-2026-0001`
- `SOL-2026-0002`

### Entrega

`ENT-YYYY-NNNN`

Ejemplos:

- `ENT-2026-0001`
- `ENT-2026-0002`

## Regla general

El correlativo incrementa dentro del mismo año. Al cambiar el año, vuelve a `0001`.

## Implementación sugerida

Al presionar Guardar en crear:

1. obtener el último registro desde la API ordenado por `id` descendente
2. revisar si el código corresponde al mismo año actual
3. si es del mismo año, sumar 1 al correlativo
4. si no existe o es de otro año, iniciar en `0001`

### Ejemplo de endpoint sugerido

```http
GET /solicitudes?_sort=id&_order=desc&_limit=1
```

La misma lógica aplica para las demás entidades que manejan código.

---

# Reglas de negocio

## Reglas generales

1. una sede con estado `Inactiva` o `Suspendida` no puede crear nuevas solicitudes
2. una solicitud debe tener al menos un ítem
3. no se permiten materiales repetidos dentro del detalle de una solicitud
4. no se puede aprobar más cantidad de la solicitada
5. no se puede despachar más stock del disponible
6. un lote fuera de vigencia no puede usarse en un despacho
7. si el material tiene `controlVigencia`, el lote debe tener `fechaFinVigencia`
8. si `cantidadDisponible` llega a 0, el lote pasa a estado `Agotado`
9. si la fecha actual supera la fecha de fin de vigencia, el lote debe considerarse `Fuera de vigencia`
10. no se puede crear una solicitud duplicada para la misma sede y el mismo periodo académico si ya existe una solicitud no cancelada
11. una entrega solo puede generarse desde una solicitud `Aprobada`
12. una solicitud `Cancelada` ya no debe permitir nuevas acciones operativas

## Regla FEFO simplificada adaptada a vigencia

Para materiales con control de vigencia, al momento del despacho el sistema debe sugerir primero los lotes con fecha de fin de vigencia más próxima.

---

# Validaciones

## Validaciones de SedeIcpna

- `nombre` obligatorio
- `zona` obligatoria
- `direccion` obligatoria, mínimo 5 caracteres
- `responsable` obligatorio
- `contacto` obligatorio
- `estudiantesActivos` obligatorio, mayor a 0
- `estado` obligatorio
- `fechaRegistro` obligatoria

## Validaciones de MaterialAcademico

- `nombre` obligatorio
- `categoria` obligatoria
- `unidadMedida` obligatoria
- `stockMinimo` obligatorio, mayor o igual a 0

## Validaciones de LoteIngreso

- `materialId` obligatorio
- `fechaIngreso` obligatoria
- `cantidadIngresada` obligatoria, mayor a 0
- `cantidadDisponible` obligatoria, mayor o igual a 0
- `cantidadDisponible` no puede ser mayor a `cantidadIngresada`
- si el material tiene `controlVigencia`, `fechaFinVigencia` es obligatoria
- `fechaFinVigencia` debe ser mayor o igual a `fechaIngreso`
- `proveedor` obligatorio

## Validaciones de SolicitudDistribucion

- `sedeId` obligatorio
- `fechaSolicitud` obligatoria
- `periodoAcademico` obligatorio
- `prioridad` obligatoria
- `estado` obligatorio
- debe existir al menos 1 ítem en `detalles`
- cada ítem debe tener `materialId` obligatorio
- `cantidadSolicitada` mayor a 0
- `cantidadAprobada` mayor o igual a 0
- no repetir `materialId` dentro de `detalles`

## Validaciones de EntregaMaterial

- `solicitudId` obligatorio
- `fechaProgramada` obligatoria
- `responsableAlmacen` obligatorio
- para registrar despacho debe existir al menos 1 ítem entregado
- no permitir usar lotes agotados
- no permitir usar lotes fuera de vigencia

## Validaciones de MovimientoInventario

- `materialId` obligatorio
- `loteId` obligatorio
- `fecha` obligatoria
- `tipoMovimiento` obligatorio
- `cantidad` obligatoria, mayor a 0
- `referenciaTipo` obligatorio
- `referenciaId` obligatorio

---

# Regla anti duplicados

## Solicitudes duplicadas

No permitir crear si ya existe una solicitud con el mismo:

- `sedeId`
- `periodoAcademico`

y cuyo estado sea distinto de `Cancelada`.

## Implementación sugerida

Consultar primero:

```http
GET /solicitudes?sedeId=...&periodoAcademico=...
```

Si retorna uno o más registros no cancelados, mostrar error y no guardar.

---

# Instalación y ejecución

## Instalar dependencias

```bash
npm install
```

## Levantar la API JSON Server

```bash
npx json-server --watch db.json --port 3000
```

API disponible en:

```text
http://localhost:3000
```

## Levantar Vue

```bash
npm run dev
```

Aplicación disponible en:

```text
http://localhost:5173
```

---

# Endpoints esperados

## Sedes

- `GET /sedes`
- `GET /sedes/:id`
- `POST /sedes`
- `PUT /sedes/:id`
- `PATCH /sedes/:id`
- `DELETE /sedes/:id`
- `GET /sedes?_sort=id&_order=desc&_limit=1`

## Materiales académicos

- `GET /materiales`
- `GET /materiales/:id`
- `POST /materiales`
- `PUT /materiales/:id`
- `PATCH /materiales/:id`
- `DELETE /materiales/:id`
- `GET /materiales?_sort=id&_order=desc&_limit=1`

## Lotes

- `GET /lotes`
- `GET /lotes/:id`
- `GET /lotes?materialId=:id`
- `POST /lotes`
- `PUT /lotes/:id`
- `PATCH /lotes/:id`
- `DELETE /lotes/:id`
- `GET /lotes?_sort=id&_order=desc&_limit=1`

## Solicitudes

- `GET /solicitudes`
- `GET /solicitudes/:id`
- `GET /solicitudes?sedeId=:id`
- `POST /solicitudes`
- `PUT /solicitudes/:id`
- `PATCH /solicitudes/:id`
- `DELETE /solicitudes/:id`
- `GET /solicitudes?_sort=id&_order=desc&_limit=1`

## Entregas

- `GET /entregas`
- `GET /entregas/:id`
- `GET /entregas?solicitudId=:id`
- `POST /entregas`
- `PUT /entregas/:id`
- `PATCH /entregas/:id`
- `DELETE /entregas/:id`
- `GET /entregas?_sort=id&_order=desc&_limit=1`

## Movimientos

- `GET /movimientos`
- `GET /movimientos/:id`
- `GET /movimientos?materialId=:id`
- `GET /movimientos?loteId=:id`
- `POST /movimientos`

---

# Estructura sugerida del proyecto

```text
src/
  assets/
  components/
    shared/
      ConfirmDialog.vue
      StatCard.vue
      EmptyState.vue
      TableFilters.vue
  views/
    dashboard/
      DashboardView.vue
    sedes/
      SedesListView.vue
      SedeFormView.vue
      SedeDetailView.vue
    materiales/
      MaterialesListView.vue
      MaterialFormView.vue
      MaterialDetailView.vue
    lotes/
      LotesListView.vue
      LoteFormView.vue
      LoteDetailView.vue
    solicitudes/
      SolicitudesListView.vue
      SolicitudFormView.vue
      SolicitudDetailView.vue
      components/
        SolicitudItemsForm.vue
    entregas/
      EntregasListView.vue
      EntregaFormView.vue
      EntregaDetailView.vue
      components/
        DespachoForm.vue
    movimientos/
      MovimientosListView.vue
  router/
    index.ts
  stores/
    sedes.store.ts
    materiales.store.ts
    lotes.store.ts
    solicitudes.store.ts
    entregas.store.ts
    movimientos.store.ts
  services/
    api.ts
    sedes.service.ts
    materiales.service.ts
    lotes.service.ts
    solicitudes.service.ts
    entregas.service.ts
    movimientos.service.ts
  composables/
    useCodigos.ts
    useFechas.ts
    useInventario.ts
    useValidators.ts
  types/
    sede-icpna.ts
    material-academico.ts
    lote-ingreso.ts
    solicitud-distribucion.ts
    entrega-material.ts
    movimiento-inventario.ts
  App.vue
  main.ts
```

---

# Criterios de aceptación

- se puede crear una sede y se guarda en JSON Server
- el sistema autogenera código de sede con formato `SDE-YYYY-NNNN`
- se puede crear un material académico y se guarda en JSON Server
- el sistema autogenera SKU con formato `MAT-YYYY-NNNN`
- se puede crear un lote de ingreso
- al crear un lote se registra automáticamente un movimiento de ingreso
- se puede listar materiales con indicador de stock disponible
- se puede identificar materiales con stock bajo
- se puede crear una solicitud con múltiples ítems
- la solicitud usa detalle dinámico con arreglos reactivos
- no se permiten materiales repetidos en una misma solicitud
- se calcula `fechaLimiteRevision` según prioridad
- se aplica la regla anti duplicados por sede y periodo académico
- se puede revisar una solicitud y cambiar su estado
- no se puede aprobar más cantidad que la disponible
- se puede generar una entrega desde una solicitud aprobada
- al registrar despacho se descuenta inventario de lotes
- al agotarse un lote, su estado cambia a `Agotado`
- no se pueden usar lotes fuera de vigencia en despacho
- se registran movimientos de salida automáticamente
- se puede registrar la entrega final
- la solicitud cambia a `Entregada` o `Parcial` según corresponda
- existe una pantalla de dashboard con indicadores
- se puede buscar y filtrar en los módulos principales

---

# Recomendaciones

## Vue

Se recomienda trabajar con:

- Vue 3
- Composition API
- Vue Router
- Pinia
- Axios
- componentes reutilizables por feature
- `reactive` y `ref` para formularios y detalle dinámico
- composables para códigos, cálculos y validaciones

## Sugerencias de implementación

- centralizar reglas de inventario en un composable o servicio
- usar `Promise.all` cuando se necesite cargar varias entidades relacionadas
- separar vistas, componentes, stores, servicios y composables para mantener orden
- reutilizar componentes de formulario para crear y editar
- manejar validaciones de campos y del detalle dinámico desde el formulario y la lógica compartida
