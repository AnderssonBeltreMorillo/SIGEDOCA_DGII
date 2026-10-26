# ETL de Migración Histórica del Archivo Central

## 1. Propósito dentro de SIGEDOCA

Esta carpeta documenta y conserva el proceso ETL que construyó la fuente histórica utilizada por el módulo **Archivo Central** de SIGEDOCA.

El proceso no funciona como una automatización independiente. Su razón de ser es producir y depurar los registros que terminan en `../Base de Datos de Registros Migrados.xlsx`, cuya tabla `BD` es consultada posteriormente por el flujo de Power Automate `Búsqueda Automatizada en Registros del Archivo Central`.

Arquitectura completa:

`Excel históricos de transferencias -> Migrador.py -> validación/normalización -> tabla BD -> Power Automate -> búsqueda automatizada del Archivo Central`

La carpeta se ubica dentro de `Excel_BD` precisamente porque constituye el mecanismo de construcción histórica de esa base consolidada.

## 2. Componentes

### `Migrador.py`

Proceso ETL en Python encargado de explorar, validar, normalizar, extraer y cargar formularios Excel históricos a la estructura consolidada del Archivo Central.

### `Errores_Migracion_Formularios.xlsx`

Salida de control de calidad producida por el ETL. Registra filas que el migrador identifica como candidatas a registro, pero que no cumplen todos los requisitos necesarios para cargarse a la base consolidada.

El archivo incluido en el repositorio es una **plantilla estructural sin registros productivos**, porque las rutas y descripciones del reporte original revelan información sobre archivos y registros históricos.

## 3. Flujo ETL

### Exploración

`CapaExplorador` permite seleccionar un archivo individual o recorrer una carpeta recursivamente. Solo procesa `.xlsx` y excluye archivos temporales de Excel cuyo nombre comienza por `~$`.

### Clasificación de formularios

`CapaValidador` reconoce dos generaciones de portada de transferencia mediante posiciones conocidas:

- `PORTADA_RECIENTE`, identificada por `FORMULARIO PARA TRANSFERENCIA` en `D12`.
- `PORTADA_ANTIGUA`, identificada por el mismo texto en `D9`.

La portada aporta contexto que luego se incorpora a cada registro, como fecha de recepción, área, motivo y relación entre cajas y tipos documentales.

El script también clasifica anexos en:

- `1_CAJA`
- `VARIAS_CAJAS`

Los archivos sin portada reconocida o sin anexo válido no pasan a la fase de extracción.

## 4. Normalización

### Áreas

`normalizar_area()` compara el texto encontrado contra `CATALOGO_AREAS` mediante `difflib.get_close_matches()` con `cutoff=0.5`.

El Fuzzy Matching permite reconciliar variaciones de escritura de las unidades organizativas con una denominación normalizada antes de cargar los datos.

### Fechas

El migrador admite fechas ya interpretadas por Excel y también números seriales comprendidos aproximadamente entre `30000` y `80000`, transformándolos a `dd/mm/yyyy`.

### Caja y tipo documental

`coincidir_caja()` vincula el número de caja observado en un anexo con el mapeo construido desde la portada. Utiliza coincidencia exacta, coincidencia parcial y una regla para el caso en que solo existe una caja posible.

## 5. Extracción según tipo de anexo

### Una caja

El migrador extrae por fila:

- caja global;
- cédula/RNC;
- nombre;
- fecha;
- tipo documental asociado a la caja.

### Varias cajas

Extrae:

- caja de la fila;
- cédula/RNC;
- nombre;
- fecha;
- tipo documental resuelto mediante el mapeo de portada.

El script está deliberadamente adaptado a posiciones conocidas de los formularios históricos. No intenta inferir dinámicamente cualquier diseño Excel.

## 6. Reglas de aceptación del registro

Una fila solo es cargada cuando contiene los campos mínimos esperados para su tipo de anexo.

Cuando una fila presenta información parcial suficiente para ser considerada candidata, pero falta algún elemento obligatorio, la fila se envía al reporte de errores en lugar de incorporarse silenciosamente a `BD`.

El reporte recibido confirma que el error predominante es `DATO INCOMPLETO`. Las descripciones identifican ausencias de **Cédula, Nombre, Fecha o Mapeo de Caja** y, para determinados anexos de varias cajas, también **Caja**. citeturn104search13

## 7. Carga en `Base de Datos de Registros Migrados.xlsx`

El ETL escribe sobre la hoja `Base de Datos` y la tabla `BD`.

Correspondencia principal:

| Resultado ETL | Tabla `BD` |
|---|---|
| `FECHA_RECEPCION` | `FECHA RECEPCIÓN (EN EL ARCHIVO)` |
| `AREA` | `AREA` |
| `CAJA` | `CAJA #` |
| `N/A` | `ANAQUEL` |
| `N/A` | `NIVEL DEL ANAQUEL` |
| `N/A` | `POSICIÓN ` |
| vacío | `TIPO CONTRIBUYENTE` |
| `CEDULA` | `CEDULAxRNC` |
| `NOMBRE` | `NOMBRE` |
| `TIPO_DOC` | `Descripción del Documento` |
| `FECHA` | `FECHA` |
| `MOTIVO` | `ASUNTO / TRAMITE` |

Las columnas posteriores de estado, préstamo, solicitante y observación quedan disponibles para el ciclo de gestión documental después de la migración.

Al terminar una carga, el migrador amplía el rango de la tabla `BD` hasta la última fila escrita.

## 8. Reporte `Errores_Migracion_Formularios.xlsx`

El reporte generado contiene cinco campos:

- `RUTA_ARCHIVO`
- `HOJA`
- `FILA`
- `TIPO_ERROR`
- `DESCRIPCION` citeturn104search13

Su función arquitectónica es proporcionar **trazabilidad de rechazo del ETL**.

Esto permite distinguir:

`registro válido -> BD`

frente a:

`registro candidato incompleto -> reporte de errores -> revisión/corrección`

El archivo productivo analizado contiene más de 173 mil filas de incidencias registradas. El volumen de incidencias no equivale necesariamente al número de archivos rechazados, porque un mismo archivo puede producir errores en múltiples hojas y filas.

## 9. Escala histórica

Según la documentación profesional proporcionada por el autor, la ejecución histórica del ETL:

- analizó **2,378 archivos Excel**;
- identificó **980 archivos válidos**;
- extrajo y estructuró **70,352 registros**.

El libro consolidado resultante observado posee una tabla histórica de más de 70 mil filas, lo que es coherente con el volumen de carga descrito. Las cifras sobre cantidad de archivos procesados y válidos corresponden a la ejecución histórica indicada por el autor, no a métricas reconstruidas únicamente desde los archivos finales.

## 10. Relación con Power Automate

El ETL construye la fuente que después consume SIGEDOCA:

1. `Migrador.py` transforma el legado Excel.
2. La tabla `BD` concentra los registros aceptados.
3. `Búsqueda Automatizada en Registros del Archivo Central` recibe un RNC/Cédula de un proceso actual.
4. El flujo elimina guiones para normalizar el identificador.
5. Excel Online consulta `CEDULAxRNC` en la base consolidada.
6. Si existe coincidencia, el flujo recupera información como RNC y nombre/razón social.
7. Si no existe coincidencia, el proceso deriva hacia búsqueda física manual.

El subsistema convierte así miles de fuentes históricas heterogéneas en un índice único consumible por automatización.

## 11. Dependencias técnicas

`Migrador.py` utiliza:

- Python 3;
- `openpyxl`;
- `tkinter`;
- `difflib` de la biblioteca estándar.

También depende de la estructura histórica de los formularios. Si las coordenadas, nombres de hojas o encabezados cambian, deben revisarse las funciones de identificación de portada/anexo y los rangos de extracción.

## 12. Archivos que deben preservarse juntos

Para documentar correctamente este subsistema deben conservarse juntos:

- `Migrador.py`, lógica ETL.
- `Errores_Migracion_Formularios.xlsx`, esquema de control de calidad y rechazo.
- `../Base de Datos de Registros Migrados.xlsx`, estructura consolidada resultante.
- `../README.md`, integración del libro con Power Automate y Archivo Central.

Separar estos componentes haría perder la relación causal entre migración histórica, persistencia consolidada y automatización operativa.
