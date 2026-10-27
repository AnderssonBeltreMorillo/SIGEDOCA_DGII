# Plantilla de Matriz de Control de Documentos - Archivos de Gestión

## 1. Rol dentro de SIGEDOCA

`Matriz control de documentos - La Feria.xlsx` es la primera instancia implementada de una **plantilla de matriz de control documental reutilizable para los Archivos de Gestión**. La estructura de la matriz sirve como origen para migrar inventarios documentales heredados hacia Microsoft Lists / SharePoint dentro de SIGEDOCA. La Feria es el primer Archivo de Gestión donde se aplicó integralmente este patrón; el contenido histórico de esa instancia es específico de La Feria, pero el modelo de datos, el proceso de migración y el caso de uso están concebidos para reutilizarse en otras sucursales.

La matriz conserva el modelo institucional de **Matriz Control de Documentos y Expedientes** utilizado por Gestión Documental, pero incorpora una estructura más precisa para identificar procedencia, ubicación archivística, contribuyente, documento, trámite y estado. El objetivo de esa evolución fue facilitar consultas, automatización y una posible migración futura hacia una base de datos estructurada o relacional basada en SQL.

Arquitectura funcional reutilizable:

`Excel/listas históricos de una sucursal -> consolidación/homologación -> Matriz de Control de Documentos -> Power Automate -> Lista SharePoint de inventario de la sucursal -> botones JSON -> solicitudes -> procesos automatizados de Archivos de Gestión`

La referencia a **La Feria** en el nombre del archivo y en algunos flujos corresponde a la primera implementación concreta. No implica que la estructura sea exclusiva de La Feria. Para nuevas sucursales deben cambiarse los orígenes/destinos específicos del entorno, conservando el contrato de datos y el patrón funcional.

El Excel no es la base operacional final de los procesos actuales. Funciona como **fuente histórica consolidada y mecanismo de migración inicial** hacia la lista estructurada de SharePoint que posteriormente utilizan los flujos de atención documental.

**Estado actual:** componente histórico y **plantilla de incorporación para el módulo Archivos de Gestión**. En la primera implementación, la fuente operacional vigente de La Feria es la lista SharePoint resultante. Para cada nueva sucursal, el objetivo es repetir el patrón: usar su matriz heredada como fuente de migración y operar posteriormente desde su propia lista SharePoint configurada.

## 2. Relación con el modelo institucional de Gestión Documental

La estructura parte de la matriz de control ya utilizada en los procesos que abarcan:

- Archivo Central;
- Archivos de Gestión;
- Unidad de Almacenamiento de Datos y Digitalización.

La plantilla aplicada inicialmente en La Feria conserva esa lógica común, pero amplía/especifica campos para que cada registro tenga mayor granularidad y pueda trasladarse de una hoja de cálculo a estructuras con tipos de datos y relaciones más estrictas.

La matriz puede dividirse conceptualmente en cinco bloques:

1. Procedencia del documento o expediente.
2. Ubicación física dentro del archivo.
3. Identificación del contribuyente.
4. Descripción documental o del expediente.
5. Estado y condición del documento.

## 3. Estructura del libro

El libro contiene cuatro hojas:

### `Base de Datos`

Contiene la tabla principal `BDLaFeria`.

La versión analizada tenía aproximadamente 15 mil registros productivos. La estructura funcional de la tabla contiene 16 columnas:

1. `FECHA RECEPCIÓN (EN EL ARCHIVO)`
2. `AREA`
3. `CAJA #`
4. `ANAQUEL`
5. `NIVEL DEL ANAQUEL`
6. `POSICIÓN `
7. `TIPO CONTRIBUYENTE`
8. `CEDULA - RNC`
9. `NOMBRE`
10. `Descripción del Documento`
11. `FECHA`
12. `ASUNTO - TRAMITE`
13. `STATUS`
14. `FECHA DE PRESTAMO`
15. `Solicitante`
16. `OBSERVACIÓN`

La tabla combina información física y lógica, permitiendo representar tanto **dónde se encuentra un documento** como **a qué contribuyente/trámite pertenece y cuál es su situación documental**. citeturn107search1

### `Lista Desprendible`

Catálogo auxiliar del modelo institucional utilizado para mantener listas de valores y nomenclaturas controladas.

### `ListaSeleccion`

Catálogo auxiliar para áreas, tipos/asuntos documentales, estatus y otras opciones estandarizadas.

### `Datos #`

Hoja de control y conteo. Contiene fórmulas `COUNTIF` contra `Base de Datos` para resumir valores de estado, área, asunto/trámite y tipo de contribuyente.

Esta hoja demuestra que la matriz no se limita a almacenar filas: también proporciona una capa básica de control cuantitativo de la consistencia y distribución del inventario.

## 4. Diseño orientado a una futura base estructurada

La estructura ampliada separa información que en fuentes históricas podía encontrarse mezclada o implícita.

Ejemplos conceptuales:

- `AREA` identifica procedencia administrativa.
- `CAJA #`, `ANAQUEL`, `NIVEL DEL ANAQUEL` y `POSICIÓN ` describen ubicación física.
- `TIPO CONTRIBUYENTE`, `CEDULA - RNC` y `NOMBRE` forman la identidad documental del contribuyente.
- `Descripción del Documento`, `FECHA` y `ASUNTO - TRAMITE` describen el objeto documental.
- `STATUS`, `FECHA DE PRESTAMO`, `Solicitante` y `OBSERVACIÓN` describen estado/custodia.

Esta separación favorece una futura traducción a un modelo de datos relacional porque reduce la necesidad de interpretar texto libre para responder preguntas como:

- ¿qué documentos pertenecen a un contribuyente?;
- ¿dónde están físicamente?;
- ¿qué tipo de expediente es?;
- ¿en qué estado se encuentra?;
- ¿está prestado o bajo custodia?;

## 5. Migración de una matriz de Archivo de Gestión a SharePoint

La solución de Archivos de Gestión contiene el flujo:

`Migrar Excel a Lista SharePoint - Archivos de Gestión`

En la primera implementación, el flujo consume la instancia de La Feria mediante Excel Online (Business). En la definición exportada se registra `/Matriz control de documentos - La Feria.xlsx` y la tabla identificada como `{3708D9E8-F2F9-4E94-8104-297D9501619B}`. citeturn107file15

### Filtrado de registros

El flujo no migra indiscriminadamente todas las filas. Enumera la tabla utilizando:

`Migrar eq 'SI'`

Por tanto, existe un **marcador de control de migración** utilizado para determinar qué registros deben convertirse en elementos de SharePoint. citeturn107file15

### Mapeo Excel -> SharePoint

Para cada fila seleccionada, el flujo crea un elemento en el sitio:

`https://dgii.sharepoint.com/sites/ArchivodeGestionLaFeria`

y asigna los siguientes campos: citeturn107file15

| Excel | SharePoint |
|---|---|
| `CAJA #` | `OData__x0023_Caja` |
| `NOMBRE` | `Nombre` |
| `CEDULA - RNC` | `Title` |
| `FECHA RECEPCIÓN (EN EL ARCHIVO)` | `FechaRecepción` |
| `AREA` | `Área` |
| `TIPO CONTRIBUYENTE` | `Tipo de contribuyente` |
| `Descripción del Documento` | `Descripción del Documento` |
| `FECHA` | `Fecha del documento` |
| `ASUNTO - TRAMITE` | `Asunto / Tramite` |
| `STATUS` | `Status` |

Las fechas serializadas de Excel se transforman en el flujo mediante una operación basada en `1899-12-30` antes de enviarlas a SharePoint. citeturn107file15


## 6. Patrón reutilizable para incorporar Archivos de Gestión

La relevancia actual de la matriz es servir como **patrón de incorporación de cualquier Archivo de Gestión que todavía mantenga su inventario en Excel**. La Feria fue la primera sucursal en completar este recorrido y funciona como implementación de referencia, no como límite funcional del modelo.

Actualmente, dentro de este módulo, La Feria representa la implementación integral: inventario migrado a una lista de SharePoint, estructura de datos preparada para automatización y acciones de solicitud asociadas directamente a cada documento inventariado.

Para incorporar otra Administración Local o Archivo de Gestión que todavía conserve su inventario en una matriz Excel, el patrón previsto es:

1. Tomar la matriz de control documental existente de la sucursal.
2. Homologar su estructura con el modelo de datos requerido para la migración.
3. Crear previamente la lista de SharePoint de inventario documental de esa sucursal, incluyendo columnas, tipos de datos, opciones y configuración requeridos.
4. Configurar en la lista las columnas de acción que correspondan al modelo operativo.
5. Ejecutar/adaptar el flujo `Migrar Excel a Lista SharePoint - Archivos de Gestión` para trasladar las filas marcadas para migración.
6. Validar que los elementos creados mantienen la correspondencia entre inventario Excel y campos SharePoint.
7. A partir de ese momento, utilizar la lista SharePoint como fuente operacional del inventario. Las nuevas incorporaciones ya no se gestionan manteniendo el Excel como base activa.
8. Habilitar sobre los elementos de la lista los procesos automatizados de solicitud física y digital.

El patrón arquitectónico reutilizable es, por tanto:

`Matriz histórica de una sucursal -> homologación -> lista SharePoint configurada -> flujo de migración -> inventario operacional -> botones JSON -> flujos manuales -> solicitudes -> automatizaciones y notificaciones`

### Papel de los botones JSON

En la implementación de La Feria, la lista de control documental posee dos columnas configuradas mediante formato JSON avanzado. Los botones se muestran por elemento, por lo que la acción conserva el contexto de la fila/documento seleccionado.

- **Solicitar documento físico** inicia el flujo manual asociado al préstamo físico.
- **Solicitar documento digital** inicia el flujo manual asociado a la copia digital.

Los JSON incluyen el identificador del flujo que deben ejecutar. Estas columnas no son ornamentales: forman parte de la interfaz funcional entre el inventario SharePoint y Power Automate. Si se elimina una columna o se pierde su formato JSON, debe reconstruirse la asociación con el flujo correspondiente.

Los flujos toman del elemento seleccionado los campos documentales necesarios para crear la solicitud en nombre de la persona que ejecuta la acción. Desde ahí continúan las automatizaciones de atención, cambios de condición, Teams/Outlook y registro estadístico.

### Primera implementación y plantilla funcional de despliegue

La primera implementación en La Feria demuestra el patrón completo y funciona como **implementación de referencia para el módulo**, no como una solución exclusiva de esa sucursal. Para nuevas sucursales, la matriz documenta cómo llevar inventarios heredados hacia el modelo operacional moderno de SIGEDOCA y la lista SharePoint documenta el estado objetivo.

La secuencia de replicación no es `copiar el Excel y seguir trabajando en Excel`, sino:

`usar el Excel existente como fuente de incorporación -> migrar -> operar desde SharePoint`.

## 7. Evolución de la arquitectura

La matriz ocupa una posición transicional entre dos generaciones del sistema:

### Etapa histórica

`Excels y listas con estructuras similares -> consolidación manual -> Matriz La Feria`

### Etapa SIGEDOCA

`Matriz La Feria -> Power Automate -> Microsoft List / SharePoint -> automatizaciones de solicitudes`

### Evolución futura posible

`Modelo normalizado -> base estructurada/relacional -> servicios/automatizaciones`

La matriz mejorada resulta útil como puente porque sus columnas ya separan entidades y atributos que pueden posteriormente convertirse en campos tipados, claves e incluso entidades relacionadas.

## 8. Relación con las solicitudes del módulo Archivos de Gestión

Después de migrarse a SharePoint, el inventario deja de ser solamente un registro pasivo.

Sobre los elementos del inventario de La Feria se apoyan los flujos:

- `Solicitud de Préstamo Físico La Feria`;
- `Solicitud de Copia Digital La Feria`;
- `Solicitud de Documentos la Feria Atentidas`.

Los dos primeros pueden iniciarse desde un documento concreto del inventario y utilizar sus campos para crear una solicitud en nombre del usuario que ejecuta la acción.

Así, la cadena funcional pasa de:

`fila histórica Excel`

hacia:

`elemento estructurado SharePoint -> acción del usuario -> solicitud física/digital -> automatización -> notificación -> trazabilidad estadística`.

En otras palabras, el proceso de migración transforma un inventario histórico en un **catálogo documental accionable**.

## 9. Casos de uso posteriores a la migración

### Solicitud de documento físico desde el inventario SharePoint

Un usuario selecciona un registro del inventario -> la automatización toma el contexto documental -> crea una solicitud física -> registra la interacción -> envía notificaciones -> continúa el ciclo de localización/préstamo/devolución.

### Solicitud de copia digital desde el inventario SharePoint

Un usuario selecciona un registro -> se crea una solicitud digital -> la automatización busca material digital relacionado -> deriva hacia validación de enlaces o localización manual -> registra las transiciones.

### Atención y cierre

`Solicitud de Documentos la Feria Atentidas` interpreta cambios de `Condición` y procesa estados como localización física, envío automatizado, préstamo, devolución, no localizado, eliminado o transferido al Archivo Central.

La matriz es, por tanto, un **antecedente de datos** de estos procesos: el registro primero se estructura/migra al inventario SharePoint y luego puede participar en los workflows de atención.

## 10. Relación con infraestructura compartida y estadísticas

Los procesos posteriores de La Feria escriben históricos e interacciones en `Estadística de las solicitudes.xlsx`, ubicado en `Infraestructura_Compartida/Excel_BD`.

La arquitectura completa cruza así dos Excel con funciones distintas:

`Matriz La Feria -> migración al inventario SharePoint -> solicitudes y atención -> Estadística de las solicitudes.xlsx`

- `Matriz control de documentos - La Feria.xlsx`: **fuente histórica del inventario**.
- `Estadística de las solicitudes.xlsx`: **persistencia histórica/analítica de la actividad generada sobre ese inventario**.

No deben confundirse ambos roles.

## 11. Contratos técnicos que deben preservarse

Para reproducir correctamente el proceso deben preservarse:

- tabla `BDLaFeria`;
- columnas consumidas por Power Automate;
- marcador/columna `Migrar` utilizado por la condición `Migrar eq 'SI'`;
- correspondencia de fechas de Excel con campos de fecha de SharePoint;
- valores de elección compatibles con `Área`, `Tipo de contribuyente`, `Asunto / Tramite` y `Status`;
- nombres y tipos de columnas de la lista SharePoint destino;
- estructura de catálogos auxiliares si el libro vuelve a utilizarse para captura o mantenimiento.

Una diferencia entre los valores Excel y las opciones disponibles en SharePoint puede impedir la creación correcta de elementos, especialmente para campos tipo elección.

## 12. Plantilla del repositorio

La copia incluida en el repositorio conserva la estructura de `BDLaFeria`, encabezados, hojas auxiliares y capa de conteos, pero no los registros productivos con identificaciones, nombres o ubicaciones documentales reales.

El objetivo es conservar **el contrato de datos y su arquitectura de migración**, no publicar el inventario documental institucional.
