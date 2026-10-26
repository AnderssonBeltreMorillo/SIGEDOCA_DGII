# Base de Datos de Registros Migrados.xlsx

## 1. Rol dentro de SIGEDOCA

`Base de Datos de Registros Migrados.xlsx` es la base histórica normalizada utilizada por el módulo **Archivo Central** de SIGEDOCA para realizar búsquedas automatizadas sobre registros documentales que originalmente estaban distribuidos en archivos Excel independientes.

El libro funciona como la capa de consulta consolidada de un proceso previo de migración/ETL: documentos históricos dispersos fueron analizados, normalizados y transformados a una estructura tabular común para que Power Automate pudiera consultar el inventario desde un único origen.

La relación arquitectónica es:

`Excel históricos dispersos -> ETL Python -> Base de Datos de Registros Migrados.xlsx -> Power Automate -> búsqueda automatizada -> resultado al proceso de Archivo Central`

Según la descripción histórica del proyecto proporcionada por el autor, el proceso ETL en Python analizó **2,378 archivos Excel**, identificó **980 archivos válidos** y estructuró **70,352 registros**, utilizando reglas de normalización y coincidencia difusa (Fuzzy Matching). Estas cifras describen el proceso ETL original y no se deducen del libro actual.

## 2. Estructura del libro

El libro está compuesto por tres hojas:

### `Base de Datos`

Es la hoja principal y contiene la tabla `BD`.

La versión analizada contenía más de 70 mil filas y una estructura de 16 columnas orientada a localizar física y documentalmente expedientes del Archivo Central.

#### Tabla `BD`

Columnas:

1. `FECHA RECEPCIÓN (EN EL ARCHIVO)`
2. `AREA`
3. `CAJA #`
4. `ANAQUEL`
5. `NIVEL DEL ANAQUEL`
6. `POSICIÓN `
7. `TIPO CONTRIBUYENTE`
8. `CEDULAxRNC`
9. `NOMBRE`
10. `Descripción del Documento`
11. `FECHA`
12. `ASUNTO / TRAMITE`
13. `STATUS`
14. `FECHA DE PRESTAMO`
15. `Solicitante`
16. `OBSERVACIÓN`

La estructura combina cuatro dimensiones funcionales:

- **procedencia**, mediante fecha de recepción y área;
- **ubicación física**, mediante caja, anaquel, nivel y posición;
- **identidad del contribuyente**, mediante tipo, RNC/cédula y nombre;
- **descripción y estado documental**, mediante descripción, fecha, asunto/trámite, estatus, préstamo, solicitante y observación.

### `Lista Desprendible`

Hoja auxiliar utilizada como catálogo para listas desplegables y valores normalizados del libro.

Forma parte de la estructura de captura/normalización del documento y no debe tratarse como base de registros operacionales.

### `ListaSeleccion`

Hoja auxiliar de referencia que contiene categorías como:

- `Área`
- `Tipo de Documento (Asunto / Tramite)`
- `Estatus`

La hoja incluye valores normalizados de administraciones locales, trámites documentales y estados como `En Archivo Central` y `Prestado`.

Estas listas ayudan a mantener coherencia semántica entre registros que originalmente provenían de fuentes heterogéneas.

## 3. Origen: proceso ETL histórico

Este libro debe entenderse como **salida de un proceso ETL**, no como un Excel creado únicamente para Power Automate.

El proceso descrito para la migración realizó conceptualmente tres etapas:

### Extracción

Lectura masiva de archivos Excel históricos provenientes de diferentes unidades y estructuras documentales.

### Transformación

Normalización de registros que no tenían una estructura uniforme. El proceso incluyó reglas de limpieza y coincidencia difusa para reconciliar variaciones de texto y convertir información heterogénea a campos compatibles con la matriz del Archivo Central.

### Carga

Los registros normalizados se consolidaron en la estructura de `Base de Datos`, creando un origen único que posteriormente puede ser consultado por automatizaciones.

Por tanto, existe una línea arquitectónica entre el proyecto ETL y SIGEDOCA:

`Fuentes históricas -> normalización -> tabla BD -> automatización de búsqueda`

## 4. Integración con Power Automate

El flujo que consume este libro es:

`Búsqueda Automatizada en Registros del Archivo Central`

En la solución exportada, Power Automate identifica el archivo mediante:

`01PZOABV3JARKTGCP7MZDZKHAQBMNAY5C7`

Ruta registrada:

`/Base de Datos de Registros Migrados.xlsx`

## 5. Caso de uso del flujo de búsqueda

El flujo realiza una búsqueda automática en el histórico consolidado cuando necesita determinar si existen registros asociados al RNC o cédula de una solicitud.

### Paso 1. Normalización del identificador

El flujo genera `LimpiarRNC` eliminando guiones del valor recibido en la solicitud.

Esto resuelve una diferencia básica de representación:

`identificador recibido -> eliminar '-' -> valor comparable con CEDULAxRNC`

### Paso 2. Consulta de Excel

El flujo consulta el libro mediante Excel Online (Business) y aplica un filtro sobre:

`CEDULAxRNC`

con la condición lógica:

`CEDULAxRNC eq <RNC/Cédula normalizado>`

Esta columna es, por tanto, el **contrato principal de búsqueda entre Power Automate y el histórico migrado**.

### Paso 3. Proyección del resultado

Cuando encuentra coincidencias, el flujo proyecta información proveniente de la tabla, incluyendo explícitamente:

- `RNC` <- `CEDULAxRNC`
- `Nombre Razón Social` <- `NOMBRE`

La consulta no utiliza el libro como simple archivo adjunto. Utiliza la matriz como un índice de búsqueda estructurado para enriquecer o responder al proceso documental.

### Paso 4. Caso sin coincidencias

Si el identificador no aparece en el archivo maestro, el flujo genera una notificación indicando que el RNC/Cédula no fue encontrado y que debe realizarse una búsqueda manual en los registros físicos.

Esta rama muestra el objetivo funcional completo:

`búsqueda automática primero -> búsqueda física manual como excepción`

El Excel reduce así la necesidad de revisar manualmente el archivo físico cuando el expediente puede identificarse mediante el histórico migrado.

## 6. Papel de la tabla BD en el Archivo Central

La tabla `BD` funciona simultáneamente como:

### Índice de contribuyentes

`CEDULAxRNC` y `NOMBRE` permiten asociar documentos con una identidad tributaria o razón social.

### Índice de ubicación física

`CAJA #`, `ANAQUEL`, `NIVEL DEL ANAQUEL` y `POSICIÓN ` permiten conservar la ubicación física del material cuando dicha información estaba disponible en la fuente histórica.

### Índice documental

`Descripción del Documento`, `FECHA` y `ASUNTO / TRAMITE` describen qué documentación debería encontrarse asociada al contribuyente.

### Estado de custodia

`STATUS`, `FECHA DE PRESTAMO`, `Solicitante` y `OBSERVACIÓN` permiten representar el estado o contexto posterior del documento dentro del Archivo Central.

## 7. Relación entre ETL y automatización

La utilidad de la automatización depende directamente del trabajo previo de normalización.

Sin ETL, Power Automate tendría que consultar miles de archivos heterogéneos, con columnas, formatos y convenciones potencialmente distintas.

Con el ETL:

1. Los archivos originales se procesan una vez.
2. Los registros válidos se homologan.
3. La identidad se concentra en `CEDULAxRNC`.
4. La descripción documental se lleva a un esquema común.
5. Power Automate consulta una sola fuente estructurada.

Esto convierte un problema de exploración de archivos en una consulta tabular determinística.

## 8. Contratos que deben preservarse

Para que `Búsqueda Automatizada en Registros del Archivo Central` continúe funcionando al reconstruir SIGEDOCA, deben preservarse especialmente:

- la existencia de la tabla `BD`;
- la columna `CEDULAxRNC`;
- la columna `NOMBRE`;
- los nombres de los demás campos cuando se amplíe la proyección o la lógica del flujo;
- la estructura de hojas auxiliares si se reutiliza el libro como plantilla de captura/mantenimiento;
- la relación de la conexión Excel Online (Business) con el archivo correspondiente.

Cambiar `CEDULAxRNC` rompe la consulta filtrada que realiza actualmente el flujo. Cambiar `NOMBRE` afecta la proyección `Nombre Razón Social`.


## Proceso ETL asociado

La población inicial de esta base histórica fue generada por el proceso ETL conservado en `ETL_Migracion_Historica/`. Ese subdirectorio forma parte de este mismo componente porque documenta cómo los formularios Excel históricos fueron explorados, clasificados, normalizados y transformados antes de cargarse en la tabla `BD`.

El subsistema completo debe leerse como una cadena única:

`Formularios Excel históricos -> ETL_Migracion_Historica/Migrador.py -> validación y normalización -> Base de Datos de Registros Migrados.xlsx / tabla BD -> Búsqueda Automatizada en Registros del Archivo Central -> procesos actuales de SIGEDOCA`

El ETL también produce `Errores_Migracion_Formularios.xlsx` como salida de control de calidad. Las filas que no cumplen los requisitos de carga se registran con ruta de origen, hoja, fila, tipo de error y descripción, permitiendo separar la carga aceptada de las excepciones que requieren revisión. La documentación técnica del proceso se encuentra en `ETL_Migracion_Historica/README.md`.

## 9. Diferencia respecto a Estadística de las solicitudes.xlsx

Este libro no cumple el mismo propósito que `Estadística de las solicitudes.xlsx`.

- `Estadística de las solicitudes.xlsx` registra actividad generada por SIGEDOCA y alimenta históricos/métricas.
- `Base de Datos de Registros Migrados.xlsx` contiene un **legado documental transformado** que SIGEDOCA consulta para automatizar búsquedas sobre información histórica.

En términos arquitectónicos:

- uno es una salida/telemetría de los procesos actuales;
- el otro es una fuente histórica de consulta creada mediante migración.

## 10. Plantilla incluida en el repositorio

La versión incluida en el repositorio mantiene la arquitectura del libro, incluida la tabla `BD`, encabezados, hojas auxiliares, listas y configuración necesaria para comprender el modelo, pero no contiene los registros históricos productivos con identificadores y nombres reales.

El objetivo de la copia es permitir comprender y reproducir el contrato técnico del componente sin convertir el repositorio en una copia de la base documental histórica.
