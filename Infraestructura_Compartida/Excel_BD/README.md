# Estadística de las solicitudes.xlsx - Arquitectura e integración SIGEDOCA

## 1. Rol del libro

`Estadística de las solicitudes.xlsx` es una pieza de infraestructura compartida entre los módulos **Archivo Central** y **Archivos de Gestión** de SIGEDOCA. No es un libro de captura manual aislado. Funciona como una capa persistente de telemetría operacional, históricos de solicitudes e interacciones y consolidación de métricas que recibe información de Power Automate y la transforma, mediante tablas y fórmulas internas, en información consumible por los tableros del propio libro.

El identificador del archivo utilizado por los flujos es `01PZOABV74QI5225HWENCL3FMRMJP7TLE5`.

Arquitectura lógica:

`SharePoint / procesos documentales -> Power Automate -> tablas Excel -> cálculos/series -> Metricas -> dashboard`

El mismo libro permite que procesos de dos módulos diferentes terminen en una capa estadística común, sin convertir sus listas operativas de SharePoint en una única lista compartida.

## 2. Estructura funcional del libro

| Hoja | Tabla principal | Función arquitectónica |
|---|---|---|
| `Metricas` | tablas auxiliares de cálculo | Dashboard y KPI actuales. Recibe el resultado lógico de las tablas operacionales mediante fórmulas y series de gráficos. |
| `RequestStats` | `RequestStats` | Registro normalizado de hitos de solicitudes de Archivo Central. Es la tabla común usada por los flujos estadísticos de Copia Digital, Préstamo, Eliminación y Transferencia. |
| `Interacciones con el portal` | `Interacciones` | Bitácora detallada de cambios intermedios de solicitudes de Archivo Central. |
| `Prestamo` | `Prestamo` | Histórico específico de préstamos documentales. |
| `Copia Digital` | `CopiaDigital` | Histórico específico de copias digitales. |
| `SolicitudesLaFeria` | `SolicitudesLaFeria` | Histórico consolidado de solicitudes atendidas por Archivo de Gestión La Feria. |
| `InteraccionesLaFeria` | `InteraccionesLaFeria` | Bitácora de estados y transiciones del proceso La Feria. |
| `Digitalización en AG` | `ConteoAGAutomatizado` | Estado agregado de volumen de PDF digitalizados por Archivo de Gestión. |
| `Metricas 2025` | tablas auxiliares + tabla dinámica | Dashboard histórico 2025. |
| `RequestStats 2025` | `RequestStats8` | Base histórica del modelo estadístico anterior. |

`Hoja1` no forma parte de la arquitectura funcional: era una copia redundante de `RequestStats 2025` y fue eliminada de la plantilla del repositorio.

## 3. Contratos técnicos principales

Los flujos no dependen solamente del nombre del archivo. Dependen de identificadores de tabla, nombres de columnas y, en algunos casos, valores clave usados para buscar o actualizar filas.

### 3.1 RequestStats

Los flujos exportados referencian la tabla mediante `tableId {33975DD1-14B7-415C-9DF7-792A779C6B7B}`.

Columnas escritas o utilizadas por los flujos:

- `ID`
- `Solicitud`
- `Creación`
- `Respuesta`
- `Gestor`

`ID` funciona además como clave lógica para evitar o detectar registros previos en los procesos de Copia Digital y Préstamo (`GDC<ID>` y `GDP<ID>`).

### 3.2 Interacciones

Los flujos exportados referencian `tableId {2311D215-F21D-4034-9DD9-EBF3F5BADE3D}`.

Además de los campos base de la solicitud, registra:

- `Estado`
- `Condición`
- `Situación solicitud`

Su función no es almacenar solamente el resultado final. Conserva estados intermedios y decisiones del ciclo de vida para análisis y trazabilidad.

### 3.3 InteraccionesLaFeria

Los flujos usan `tableId {98DB7551-8312-469B-A5DD-515DD0B63390}`.

Campos observados:

- `ID`
- `Creado`
- `RNC / Cédula`
- `Nombre o Razón Social`
- `Tipo de Contribuyente`
- `Asunto / Trámite`
- `Fecha del documento`
- `Fecha de recepción`
- `Tipo de Solicitud`
- `Solicitante`
- `Estado`
- `Condición`
- `Modificado`
- `Modificado por`

Esta tabla registra cada transición relevante del proceso La Feria, incluidas localización física, envío automatizado, préstamo, devolución, no localizado, eliminado y transferido al Archivo Central.

### 3.4 SolicitudesLaFeria

Los flujos usan `tableId {5126BC67-CFEE-4A0B-9B19-0F142AB3FCAD}`.

Utiliza prácticamente el mismo contexto documental de `InteraccionesLaFeria`, pero se escribe en puntos de cierre o consolidación. Su rol es representar la solicitud atendida, mientras `InteraccionesLaFeria` representa la secuencia de eventos que llevó a ese resultado.

### 3.5 ConteoAGAutomatizado

Los flujos usan `tableId {D6690CB6-8731-469D-A802-B57B98C741CD}`.

Contrato de actualización:

- columna clave: `ARCHIVO DE GESTIÓN`
- columna actualizada: `DIGITALIZACIÓN DE DOCUMENTOS`

Valores de clave observados: `Bavaro`, `El Seibo`, `Los Mina` y `Máximo Gómez`.

## 4. Integración con Archivo Central

### 4.1 `5. Estadística Solicitudes en Excel - Eliminación`

Disparador: elemento creado o modificado en la lista de eliminación de Archivo Central.

Condición: `Estado = Solucionado`.

Operación Excel: `AddRowV2` sobre la tabla estadística común.

Datos escritos explícitamente:

- `Solicitud` <- valor de la solicitud en SharePoint.
- `Creación` <- fecha de creación del elemento.
- `Gestor` <- persona que realizó la última modificación.

Uso: incorporar la terminación de solicitudes de eliminación al modelo consolidado de estadísticas.

### 4.2 `5. Estadística Solicitudes en Excel - Transferencia`

Condición: `Estado = Solucionado`.

Operación: agrega una fila a `RequestStats`.

Datos:

- `ID` <- prefijo `GDT` + ID de SharePoint.
- `Solicitud` <- tipo/valor de solicitud.
- `Gestor` <- modificador del elemento.

Uso: contabilizar transferencias concluidas e identificarlas de forma inequívoca dentro del histórico común.

### 4.3 `5. Registro de Solicitudes en Excel - Préstamo`

Este flujo implementa una bitácora de ciclo de vida, no solamente un alta final.

1. Consulta `RequestStats` filtrando `ID eq 'GDP<ID>'`.
2. Si determinados hitos representan un evento estadístico, agrega una fila a `RequestStats`.
3. Para estados intermedios o transiciones adicionales, agrega filas a `Interacciones`.

Campos en `RequestStats`:

- `ID = GDP<ID>`
- `Solicitud`
- `Creación`
- `Respuesta`
- `Gestor`

Las fechas de creación y respuesta se normalizan desde UTC hacia `SA Western Standard Time` y se formatean `dd/MM/yyyy HH:mm` en los registros principales.

Casos explícitos que generan registro estadístico incluyen:

- `Aprobado` + `Reasignación interna`.
- `Solucionado` + `Prestado`.
- `Rechazado` + `No localizado`.
- `Rechazado` + `Datos incompletos`.
- `Rechazado` + `Datos incorrectos`.

Para otras transiciones, `Interacciones` conserva `Estado`, `Condición` y `Situación solicitud` junto con timestamps y gestor.

Arquitectónicamente, esto permite separar **hitos cuantificables** de **eventos de trazabilidad**.

### 4.4 `5. Registros de Solicitudes en Excel - Copia Digital`

Es el equivalente más amplio para solicitudes `GDC`.

Primero consulta `RequestStats` con `ID eq 'GDC<ID>'`. Luego clasifica el estado/condición y decide entre registro estadístico e interacción.

Hitos observados que pueden escribir en `RequestStats`:

- `Solucionado` + `Entregado`.
- `Rechazado` + `No localizado`.
- `Rechazado` + `Datos incompletos`.
- `Rechazado` + `Datos incorrectos`.
- `Rechazado` + `Tipo de solicitud incorrecta`.
- `Aprobado` + `Reasignación interna`.
- `Aprobado` + `Digitalización en proceso`.

Campos principales:

- `ID = GDC<ID>`
- `Solicitud`
- `Creación`
- `Respuesta`
- `Gestor`

Cuando una transición no corresponde al hito principal que debe alimentar `RequestStats`, el flujo escribe una fila en `Interacciones` con el contexto ampliado.

El flujo también contiene una validación por área antes de ciertas ramas. Esto indica que el registro estadístico forma parte de una lógica de negocio dependiente del origen de la solicitud, no de una exportación indiscriminada de todos los cambios.

## 5. Integración con Archivos de Gestión

### 5.1 Conteos automáticos de digitalización

Los cuatro flujos de conteo ejecutan la misma función arquitectónica sobre distintos sitios de SharePoint:

- `Conteo Digitalización Bavaro`
- `Conteo Digitalización El Seibo`
- `Conteo Digitalización Los Mina`
- `Conteo Digitalización Máximo Gómez`

Los flujos se ejecutan periódicamente cada 15 minutos según las definiciones exportadas.

Proceso común:

1. Consultar el repositorio documental del Archivo de Gestión correspondiente.
2. Contar archivos PDF, con variaciones de recorrido según la estructura del sitio.
3. Obtener/acceder a `ConteoAGAutomatizado`.
4. Actualizar la fila cuyo `ARCHIVO DE GESTIÓN` coincide con la sucursal.
5. Escribir el total en `DIGITALIZACIÓN DE DOCUMENTOS`.

Casos:

- Bávaro: cuenta PDF mediante las propiedades de archivos y actualiza la clave `Bavaro`.
- El Seibo: recorre `Documentos compartidos/Digitalizados`, tipos y cajas; filtra extensiones PDF e incrementa `Total PDFS`; actualiza `El Seibo`.
- Los Mina: cuenta archivos PDF no carpeta (`FSObjType eq 0`) y actualiza `Los Mina`.
- Máximo Gómez: cuenta PDF con paginación y actualiza `Máximo Gómez`.

Esto hace de `Digitalización en AG` un **snapshot agregado de producción digital por ubicación**, en lugar de un log fila a fila.

### 5.2 `Solicitud de Préstamo Físico La Feria`

Se inicia desde un elemento seleccionado del inventario documental de La Feria.

Secuencia relevante:

1. Obtiene el documento seleccionado.
2. Crea una solicitud de tipo `Documento Físico` en la lista de solicitudes.
3. Registra inmediatamente una fila en `InteraccionesLaFeria` con identificador `ALLF<ID>` y los datos documentales/contextuales.
4. Notifica por correo y Teams.

El registro Excel enlaza el inventario del documento con la nueva solicitud y establece el primer evento de trazabilidad del caso.

### 5.3 `Solicitud de Copia Digital La Feria`

Parte también de un documento seleccionado, pero crea una solicitud de `Documento Digital`.

El flujo registra el evento inicial en `InteraccionesLaFeria`; después busca archivos digitales que coincidan con el documento, incluyendo una regla especial para contribuyentes `sucesoral`.

Según el resultado:

- Si encuentra archivos, cambia la condición a `Validar enlaces encontrados`, notifica al equipo y registra otra interacción.
- Si no encuentra archivos, cambia a `Localización Manual`, notifica al equipo y registra esa transición.

Por tanto, `InteraccionesLaFeria` permite reconstruir desde el Excel la ruta seguida por una solicitud digital antes de quedar resuelta.

### 5.4 `Solicitud de Documentos la Feria Atentidas`

Este flujo reacciona a modificaciones en la lista de solicitudes y solo procesa cuando cambia la columna `Condición`.

El valor de `Condición` funciona como máquina de estados mediante un `Switch`.

Casos documentados:

#### `Localización Física`

Registra `ALLFC<ID>` en `InteraccionesLaFeria` y conserva todos los datos documentales, estado, condición, timestamps y responsable.

#### `Envío Automatizado`

- Responde al solicitante con el enlace al documento digital.
- Registra la interacción.
- Agrega además una fila a `SolicitudesLaFeria`.

Esto representa un cierre exitoso de solicitud digital.

#### `Prestado`

- Notifica que el expediente físico puede retirarse.
- Registra la interacción.
- Agrega la solicitud a `SolicitudesLaFeria`.

Esto representa el cierre/hito de entrega física.

#### `Devuelto`

Confirma la devolución y registra una nueva interacción. El Excel conserva así el ciclo posterior al préstamo, no solamente la entrega.

#### `No localizado`

- Notifica el resultado al solicitante/equipo.
- Registra interacción.
- Registra también la solicitud consolidada en `SolicitudesLaFeria`.

#### `Eliminado`

Informa que el documento agotó su ciclo de retención y registra tanto interacción como consolidación en `SolicitudesLaFeria`.

#### `Transferido al Archivo Central`

Informa que el expediente ya no pertenece físicamente al Archivo de Gestión La Feria y debe solicitarse al Archivo Central; registra interacción y consolidación.

Este caso representa explícitamente una frontera entre los dos módulos documentales y explica por qué el libro estadístico compartido es útil: crea continuidad histórica aunque la custodia física cambie de módulo.

## 6. Diferencia entre las tablas de solicitud y de interacción

La arquitectura utiliza dos niveles de persistencia:

### Nivel de hechos consolidados

- `RequestStats`
- `SolicitudesLaFeria`
- historiales específicos `Prestamo` y `CopiaDigital`

Representan eventos que deben contarse, resumirse o considerarse resultados relevantes.

### Nivel de trazabilidad

- `Interacciones`
- `InteraccionesLaFeria`

Representan cambios intermedios de estado, decisiones y contexto operacional.

Esta separación permite calcular métricas de volumen y cumplimiento sin perder el recorrido de cada caso.

## 7. Capa analítica

### `Metricas`

La hoja principal de dashboard contiene fórmulas, tablas auxiliares y 30 gráficos en la versión analizada. Consume la información estructurada del libro para presentar métricas por período, tipo de solicitud, gestión, cumplimiento y tiempos.

### `Metricas 2025`

Mantiene el modelo histórico anterior, con 13 gráficos y una tabla dinámica. `RequestStats 2025` conserva la base histórica asociada.

La coexistencia de las hojas actuales e históricas muestra que el libro también cumple una función de continuidad temporal del modelo de indicadores.

## 8. Casos de uso arquitectónicos

### Caso A: medir tiempo de respuesta de una solicitud de Archivo Central

SharePoint registra `Created` y `Modified` -> Power Automate normaliza las fechas -> `RequestStats` recibe `Creación`, `Respuesta`, `Gestor` y `Solicitud` -> fórmulas calculan duración/clasificación -> `Metricas` presenta KPI y gráficos.

### Caso B: auditar el ciclo de una solicitud

Cambio de `Estado` o `Condición` -> flujo evalúa la rama -> `Interacciones` o `InteraccionesLaFeria` recibe el evento -> queda registrada la secuencia de estados, situación, responsable y fechas.

### Caso C: conocer producción digital de Archivos de Gestión

Sitio SharePoint de cada Archivo de Gestión -> flujo cuenta PDF -> `ConteoAGAutomatizado` actualiza una fila por ubicación -> la capa estadística puede comparar volúmenes entre archivos.

### Caso D: solicitud de La Feria que termina bajo custodia de Archivo Central

Solicitud La Feria -> condición `Transferido al Archivo Central` -> comunicación al usuario -> registro en `InteraccionesLaFeria` + `SolicitudesLaFeria` -> continuidad estadística aun cuando cambia la unidad responsable de custodiar el documento.

## 9. Dependencias que deben preservarse

Al reconstruir o modificar esta integración se deben tratar como contratos los siguientes elementos:

- nombre y ubicación del libro;
- nombres de las tablas;
- identificadores de tabla usados por los flujos;
- nombres exactos de columnas;
- prefijos `GDC`, `GDP`, `GDT`, `ALLF` y `ALLFC` utilizados para correlación;
- claves `ARCHIVO DE GESTIÓN` de la tabla de conteos;
- valores de `Estado` y `Condición` que activan las ramas;
- fórmulas y fuentes de gráficos en `Metricas` y `Metricas 2025`;
- estructura de timestamps, incluida la conversión a `SA Western Standard Time` donde está implementada.

Cambiar cualquiera de estos contratos sin modificar simultáneamente los flujos puede provocar fallos de escritura, pérdida de trazabilidad, duplicación de registros o indicadores incompletos.

## 10. Límites de este documento

`Base de Datos de Registros Migrados.xlsx` y `Matriz control de documentos - La Feria.xlsx` son libros distintos con responsabilidades propias. Los flujos incluidos en el inventario permiten ver sus conexiones, pero su arquitectura debe documentarse en sus respectivos README y no confundirse con la responsabilidad de `Estadística de las solicitudes.xlsx`.

La copia del libro incluida en el repositorio conserva la arquitectura necesaria para estudiar y reconstruir la integración, pero no pretende funcionar como respaldo de la información productiva.
