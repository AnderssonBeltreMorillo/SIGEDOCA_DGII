# Datos de UDAD.xlsx

## 1. Rol dentro de SIGEDOCA

`Datos de UDAD.xlsx` es la capa histórica y analítica del módulo **Unidad de Almacenamiento de Datos y Digitalización (UDAD)**. Su estructura refleja los cuatro grandes estados del proceso documental que también existen operacionalmente en SharePoint/Microsoft Lists:

`Recepción de caja -> Preparación -> Digitalización -> Control de Calidad -> cierre/resultado de caja`

El libro no sustituye las listas operacionales. Las listas gestionan el trabajo vigente y disparan automatizaciones; Excel conserva eventos normalizados producidos durante ese trabajo para trazabilidad, medición de tiempos, reconstrucción histórica, métricas y control de cajas.

La arquitectura general es:

`Microsoft Lists / SharePoint -> Power Automate -> eventos del proceso -> tablas históricas de Datos de UDAD.xlsx`

Las tablas del libro son, por tanto, una proyección histórica de las etapas operacionales de UDAD.

## 2. Modelo del libro

El archivo contiene cuatro hojas y cuatro tablas:

| Hoja | Tabla | Función |
|---|---|---|
| `Preparacion` | `tblPreparacion` | Historial de preparación y no recepción de documentos. |
| `Digitalizacion` | `tblDigitalizacion` | Historial de digitalización, errores detectados y tareas relacionadas. |
| `Calidad` | `tblCalidad` | Historial de control de calidad, aprobaciones/rechazos y retrabajo. |
| `Cajas` | `tblCajas` | Seguimiento agregado de recepción, validación y cierre de cada caja. |

La versión productiva analizada contenía aproximadamente 47 mil eventos de preparación, 48 mil de digitalización, 41 mil de calidad y más de mil registros/eventos de cajas. El contenido evidencia que el libro almacena eventos del proceso, no únicamente el estado actual de cada documento. citeturn114search1

## 3. `tblPreparacion`

Columnas:

- `Evento_ID`
- `Fecha_Evento`
- `Fecha_Creacion`
- `Duracion_Minutos`
- `ID_SharePoint`
- `No_Caja`
- `No_Documento`
- `Orden`
- `RNC`
- `Razon_Social`
- `Area_Procedencia`
- `Tipo_Documento`
- `Prioridad`
- `Estado`
- `Usuario_Email`
- `Usuario_Nombre`
- `Tipo_Evento`

### Caso de uso

Cada fila representa un evento asociado a un documento durante preparación. `Evento_ID` funciona como identificador lógico del evento y `ID_SharePoint` permite relacionar el histórico Excel con el elemento operacional correspondiente.

El libro observado contiene eventos similares a `PREP|<ID>` y `NO-REC|<ID>`, diferenciando preparación normal de documentos no recibidos. Los registros conservan caja, documento, orden, contribuyente, procedencia, responsable y duración. citeturn114search1

La tabla permite responder posteriormente preguntas como:

- ¿quién preparó el documento?;
- ¿cuánto tardó desde la creación hasta el evento?;
- ¿a qué caja y orden pertenecía?;
- ¿el documento fue preparado o marcado como no recibido?;
- ¿qué área y tipo documental originaron el trabajo?

## 4. `tblDigitalizacion`

Comparte el núcleo identificador de `tblPreparacion` y agrega campos específicos de digitalización:

- `Tipo_Evento`
- `Tiene_Error`
- `Tipos_Error`
- `Usuario_Email`
- `Usuario_Nombre`
- `Planner_Task_ID`

### Caso de uso

Registra la ejecución de la etapa de digitalización y conserva evidencia de errores o incidencias detectadas. `Planner_Task_ID` permite mantener una referencia hacia gestión de tareas cuando el proceso genera o utiliza tareas de Microsoft Planner.

La combinación `No_Caja` + `No_Documento` + `Orden` mantiene continuidad con las demás etapas y permite reconciliar un mismo documento entre preparación, digitalización y calidad.

## 5. `tblCalidad`

Columnas específicas de control de calidad:

- `Tipo_Evento`
- `Es_Retrabajo`
- `Tipos_Error`
- `Cantidad_Errores`
- `Usuario_Email`
- `Usuario_Nombre`

### Caso de uso

Representa las decisiones de Control de Calidad y permite distinguir una revisión normal de un retrabajo posterior.

`Tipos_Error` y `Cantidad_Errores` permiten cuantificar no solo cuántos documentos pasan por calidad, sino qué problemas fueron encontrados y cuántas incidencias estuvieron asociadas al evento.

La arquitectura de UDAD requiere interpretar el historial de calidad por secuencia de eventos. Un rechazo histórico no equivale por sí mismo al estado final si después existe una corrección y una aprobación posterior. Por eso esta tabla debe tratarse como **bitácora de eventos**, no como una tabla donde la primera ocurrencia determina el resultado definitivo.

## 6. `tblCajas`

Columnas:

- `Evento_ID`
- `Fecha_Evento`
- `Fecha_Recepcion`
- `Fecha_Finalizacion`
- `Duracion_Total_Minutos`
- `ID_Recepcion`
- `No_Caja`
- `Area_Procedencia`
- `Tipo_Documento`
- `Estado_Validacion`
- `Cierre_Comunicado`
- `Cantidad_Documentos`
- `Cantidad_Documentos_Rechazados`
- `Cantidad_Eventos_Error`
- `Tuvo_Errores`
- `Usuario_Email`
- `Usuario_Nombre`
- `Tipo_Evento`
- `Motivo_Rechazo`

### Caso de uso

Esta tabla representa la capa agregada del proceso por caja. Mientras Preparación, Digitalización y Calidad registran eventos por documento, `tblCajas` permite conocer el ciclo general de la unidad física/documental recibida.

Permite conservar:

- recepción y finalización;
- duración total;
- cantidad de documentos;
- cantidad de rechazados;
- cantidad de eventos de error;
- si la caja tuvo errores;
- validación/cierre;
- comunicación del cierre;
- responsable;
- motivo de rechazo cuando corresponda.

Esta separación es fundamental: **documentos** y **cajas** tienen ciclos relacionados pero no idénticos.

## 7. Correspondencia con SharePoint/Microsoft Lists

El diseño del libro replica conceptualmente las principales listas operacionales de UDAD:

`Lista Preparación -> tblPreparacion`

`Lista Digitalización -> tblDigitalizacion`

`Lista Control de Calidad -> tblCalidad`

`Recepción / ciclo de caja -> tblCajas`

La palabra "replica" no significa que Excel sea la fuente maestra operacional. SharePoint administra elementos actuales, estados y triggers. Excel mantiene una representación histórica orientada a eventos y métricas.

Los identificadores `ID_SharePoint` e `ID_Recepcion` sirven como puentes de correlación entre ambas capas.

## 8. Semántica común del documento

Las primeras tres tablas comparten un núcleo de atributos:

- caja;
- número de documento;
- orden;
- RNC;
- razón social;
- área de procedencia;
- tipo de documento;
- prioridad;
- estado;
- usuario;
- timestamp del evento.

Esto permite reconstruir el recorrido de un documento:

`Preparación -> Digitalización -> Calidad`

utilizando principalmente `No_Caja`, `No_Documento` y `Orden` para reconciliar registros del mismo contexto documental.

## 9. Integración con los flujos UDAD

En las soluciones exportadas se detectó que `Datos de UDAD.xlsx` es utilizado por:

- `UDAD - 01 Ingesta de Cajas v1`
- `UDAD - 02 Transición a Digitalización v1`
- `UDAD - 03 Transición a Control de Calidad v1`
- `UDAD - 04 Gestión de Calidad y Retrabajo v1`
- `UDAD - 06 Agregar documento adicional v1`
- `UDAD - 08 Reparación integral y cierre de cajas v1`

El archivo se identifica en las definiciones mediante `01PZOABV2YLP37AQJRENC2M4F6YXSFMQJO`.

### UDAD-01: Ingesta de cajas

Recibe el Excel adjunto asociado al formulario/lista de recepción. Como el adjunto no puede procesarse directamente en el contexto original, el flujo obtiene sus propiedades y genera un archivo temporal que puede ser leído por Excel Online. El adjunto debe cumplir el formato `.xlsx`.

Después del procesamiento de la carga, `Datos de UDAD.xlsx` conserva el histórico de los documentos/caja incorporados al proceso.

### UDAD-02: transición a Digitalización

Representa el paso entre preparación y digitalización. La continuidad de campos comunes permite registrar el evento de digitalización manteniendo la identidad del documento que ya existía en preparación.

### UDAD-03: transición a Control de Calidad

Proyecta la etapa siguiente del ciclo y genera la información necesaria para que el control de calidad quede ligado al mismo documento/caja/orden.

### UDAD-04: Calidad y Retrabajo

Gestiona los resultados del control, incluidos errores y retrabajos. La estructura de `tblCalidad` está preparada precisamente para conservar `Es_Retrabajo`, `Tipos_Error` y `Cantidad_Errores`.

### UDAD-06: documento adicional

Permite incorporar un documento que no formaba parte de la carga original. La automatización utiliza la identidad de caja/documento/orden para integrarlo en las mismas estructuras, evitando crear un proceso histórico paralelo.

### UDAD-08: reparación integral y cierre de cajas

Actúa como mecanismo de reconciliación/cierre. La existencia de `tblCajas` junto a las tres tablas documentales permite contrastar eventos, errores y cantidades antes de considerar una caja cerrada.

La definición del flujo contiene mensajes explícitos que indican que, si no se localiza la fila `Caja recibida` correspondiente en `Datos de UDAD.xlsx`, una incorporación puede cancelarse. Esto demuestra que determinadas operaciones de reparación/cierre utilizan el histórico Excel como referencia complementaria del estado de caja.

## 10. Botón de corrección controlada

La lista SharePoint de Preparación contiene una columna con formato JSON avanzado que actúa como botón por elemento para iniciar el proceso de corrección.

La secuencia funcional es:

`Elemento de Preparación -> botón JSON -> flujo manual -> usuario de solo ejecución -> formulario Power Apps -> campos corregidos -> flujo automatizado -> reparación por No_Caja + Orden en las listas relacionadas`

Este caso es importante porque el libro comparte las mismas claves de negocio que el proceso de corrección. Aunque la operación se ejecuta sobre SharePoint, la consistencia histórica debe mantener la misma identidad de caja/documento/orden para que las métricas y procesos posteriores sigan siendo reconciliables.

## 11. Arquitectura de eventos, no de estado único

`Datos de UDAD.xlsx` debe interpretarse como un **event store operacional simplificado**.

No se limita a guardar "cómo está ahora" un documento. Conserva qué ocurrió, cuándo ocurrió, quién ejecutó el evento y cuánto duró.

Ejemplos de identificadores y tipos de evento observados muestran esta intención, como eventos de preparación y no recepción. citeturn114search1

Este diseño posibilita:

- medir productividad por proceso;
- reconstruir tiempos;
- atribuir trabajo a usuarios;
- identificar errores y retrabajos;
- reconciliar cantidades por caja;
- detectar inconsistencias entre etapas;
- producir dashboards sin depender únicamente del estado actual de las listas.

## 12. Dependencias y contratos técnicos

Deben preservarse cuidadosamente:

- nombres `tblPreparacion`, `tblDigitalizacion`, `tblCalidad` y `tblCajas`;
- nombres de columnas;
- semántica de `Evento_ID`;
- `ID_SharePoint` e `ID_Recepcion`;
- claves `No_Caja`, `No_Documento` y `Orden`;
- campos de error/retrabajo;
- valores usados para identificar eventos de caja;
- referencia del libro configurada en Excel Online (Business).

Cambiar estos contratos sin modificar los flujos rompe la escritura, consulta o reconciliación histórica.

## 13. Papel dentro de una futura evolución

El modelo ya separa entidades que podrían migrarse posteriormente a una base relacional:

- Documento
- Caja
- Evento de preparación
- Evento de digitalización
- Evento de calidad
- Error
- Usuario/ejecutor

Los identificadores y timestamps permitirían conservar trazabilidad si la persistencia histórica se traslada de Excel a SQL/Dataverse u otra plataforma.


## 14. Integración con Power BI: Dashboard Gestión UDAD

`Datos de UDAD.xlsx` también funciona como **fuente analítica del informe Power BI `Dashboard Gestión UDAD`**. Esta relación completa la arquitectura del módulo: las listas de SharePoint soportan la operación actual; Power Automate registra los eventos relevantes en Excel; y Power BI utiliza ese histórico para construir medidas, indicadores, estados de avance y visualizaciones de gestión.

La cadena analítica queda así:

`SharePoint / Microsoft Lists -> Power Automate -> Datos de UDAD.xlsx -> modelo semántico / medidas Power BI -> Dashboard Gestión UDAD`

Por tanto, el Excel cumple dos responsabilidades simultáneas:

1. **Persistencia histórica de eventos:** conserva qué ocurrió en Preparación, Digitalización, Calidad y Cajas.
2. **Fuente analítica:** proporciona las tablas desde las cuales Power BI obtiene los datos necesarios para calcular y mostrar indicadores de gestión.

### Relación conceptual entre tablas Excel y el dashboard

#### `tblPreparacion`

Aporta eventos de preparación, documentos no recibidos, responsables, fechas y duración. Permite construir indicadores sobre volumen preparado, trabajo por usuario, tiempos y excepciones de recepción/preparación.

#### `tblDigitalizacion`

Aporta actividad de digitalización, responsables, duración, errores y referencias de Planner. Permite medir avance de digitalización y analizar incidencias asociadas a esta etapa.

#### `tblCalidad`

Aporta aprobaciones/rechazos, retrabajos, tipos de error y cantidad de errores. Es la fuente histórica necesaria para métricas de control de calidad y para diferenciar un rechazo intermedio de la aprobación final posterior.

#### `tblCajas`

Aporta la perspectiva agregada por caja: recepción, finalización, cantidades, documentos rechazados, incidencias, validación y comunicación de cierre. Es la estructura natural para KPIs de avance y cierre de cajas.

### Qué presenta el informe Power BI

El informe conectado se denomina `Dashboard Gestión UDAD`. Su descripción estructural indica que presenta el progreso operacional de cajas y distingue estados como completadas, en progreso o pendientes. También incluye indicadores relacionados con cajas registradas, preparadas, digitalizadas, controladas y rechazadas, además de alertas, motivos de pendientes y responsables.

Esto confirma que `Datos de UDAD.xlsx` no debe verse solamente como un archivo de logs. Forma parte de la **capa de inteligencia operacional** de SIGEDOCA para UDAD.

### Arquitectura de doble capa: estado actual e histórico

La separación es importante:

- **SharePoint/Microsoft Lists:** contiene la operación y el estado vigente de los elementos que los flujos procesan.
- **Datos de UDAD.xlsx:** conserva los eventos históricos y agregados necesarios para análisis longitudinal.
- **Power BI:** transforma ese histórico en indicadores y visualizaciones de seguimiento.

Esta estructura evita depender exclusivamente del estado actual de una lista para explicar qué ocurrió antes. Por ejemplo, un documento puede haber sido rechazado en Calidad, corregido y aprobado posteriormente. El historial de `tblCalidad` permite reconstruir esa secuencia y las medidas de Power BI pueden evaluarla con criterios de resultado final.

### Dependencia crítica para replicación

Al replicar el módulo UDAD debe considerarse también la cadena de Power BI. No basta con reproducir las listas y los flujos. Deben conservarse o reconstruirse:

- las cuatro tablas del libro y sus nombres;
- las columnas consumidas por el modelo analítico;
- la ubicación/conexión de `Datos de UDAD.xlsx` usada por Power BI;
- las transformaciones de Power Query, si existen;
- relaciones del modelo semántico;
- medidas DAX;
- páginas, visuales, filtros y reglas del informe.

Cambiar nombres o semántica de columnas en Excel puede romper tanto los flujos de Power Automate como el modelo Power BI. El libro actúa así como un **contrato compartido entre automatización y analítica**.

### Artefacto Power BI

Informe: `Dashboard Gestión UDAD`  
Artifact ID: `883a9719-8b03-4d80-8736-a62c2610eb09`  
Workspace: `My workspace`

La documentación detallada de medidas DAX, Power Query, relaciones y visuales debe incorporarse cuando se exporte o documente el modelo de Power BI. La información disponible actualmente permite establecer con certeza el papel arquitectónico del informe, pero no debe utilizarse para inventar medidas específicas que todavía no hayan sido extraídas.

## 15. Plantilla del repositorio

La copia incluida en el repositorio conserva las cuatro hojas, las cuatro tablas, encabezados, formatos y contratos estructurales, pero no contiene los registros productivos de documentos, contribuyentes o usuarios.

El propósito es documentar y permitir reconstruir la arquitectura de persistencia histórica de UDAD sin publicar información operacional.
