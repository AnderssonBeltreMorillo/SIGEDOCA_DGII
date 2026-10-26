# Excel compartido de SIGEDOCA

Esta carpeta contiene libros de Excel utilizados como infraestructura compartida por distintos módulos de SIGEDOCA.

## Estadística de las solicitudes.xlsx

### Propósito

`Estadística de las solicitudes.xlsx` funciona como repositorio histórico y estadístico compartido entre los módulos **Archivo Central** y **Archivos de Gestión**.

Los flujos de Power Automate agregan o consultan registros en diferentes tablas del libro. Estos datos alimentan históricos, logs, cálculos y tableros de seguimiento.

### Relación detectada con Power Automate

Los archivos fuente exportados de Power Automate identifican este libro mediante el ID:

`01PZOABV74QI5225HWENCL3FMRMJP7TLE5`

#### Archivo Central

Los siguientes flujos utilizan el libro:

- `5. Estadística Solicitudes en Excel - Eliminación`
- `5. Estadística Solicitudes en Excel - Transferencia`
- `5. Registro de Solicitudes en Excel - Préstamo`
- `5. Registros de Solicitudes en Excel - Copia Digital`

#### Archivos de Gestión

Los siguientes flujos utilizan el mismo libro:

- `Conteo Digitalización Bavaro`
- `Conteo Digitalización El Seibo`
- `Conteo Digitalización Los Mina`
- `Conteo Digitalización Máximo Gómez`
- `Solicitud de Copia Digital La Feria`
- `Solicitud de Documentos la Feria Atentidas`
- `Solicitud de Préstamo Físico La Feria`

Esta dependencia compartida es la razón por la que el libro se almacena en `Infraestructura_Compartida/Excel_BD` y no dentro de un único módulo.

## Estructura del libro

La versión analizada contiene las siguientes hojas funcionales:

1. `Metricas`
2. `RequestStats`
3. `Interacciones con el portal`
4. `Prestamo`
5. `Copia Digital`
6. `SolicitudesLaFeria`
7. `InteraccionesLaFeria`
8. `Digitalización en AG`
9. `Metricas 2025`
10. `RequestStats 2025`

La hoja `Hoja1` fue eliminada de la copia destinada al repositorio porque era una copia redundante de los datos de `RequestStats 2025` y no formaba parte funcional del sistema.

## Tablas operacionales

El libro conserva, entre otras, las siguientes tablas que sirven como estructuras de datos para automatizaciones y cálculos:

| Hoja | Tabla |
|---|---|
| RequestStats | `RequestStats` |
| Interacciones con el portal | `Interacciones` |
| Prestamo | `Prestamo` |
| Copia Digital | `CopiaDigital` |
| SolicitudesLaFeria | `SolicitudesLaFeria` |
| InteraccionesLaFeria | `InteraccionesLaFeria` |
| Digitalización en AG | `ConteoAGAutomatizado` |
| RequestStats 2025 | `RequestStats8` |

La nomenclatura de estas tablas debe considerarse parte de la configuración técnica de SIGEDOCA. Los nombres no deben cambiarse sin revisar previamente las referencias de los flujos de Power Automate y las fórmulas del libro.

## Dashboards y métricas

### Metricas

Es el tablero principal correspondiente a la estructura estadística actual. Contiene:

- fórmulas de cálculo;
- tablas auxiliares de métricas;
- gráficos;
- indicadores por período y tipo de solicitud;
- métricas de cumplimiento y tiempos de atención.

En el archivo analizado se detectaron **30 gráficos** en esta hoja.

### Metricas 2025

Conserva la estructura histórica del tablero correspondiente a 2025.

Contiene:

- fórmulas;
- tablas auxiliares;
- **13 gráficos**;
- una tabla dinámica.

La estructura de 2025 se mantiene para preservar la evolución histórica del diseño y de los cálculos del sistema.

## Sanitización para el repositorio

El archivo almacenado en este repositorio es una copia estructural sanitizada. No debe confundirse con el archivo productivo.

Para preparar la versión pública se eliminaron los registros operacionales de las siguientes tablas:

- `RequestStats`
- `Interacciones`
- `Prestamo`
- `CopiaDigital`
- `SolicitudesLaFeria`
- `InteraccionesLaFeria`
- `ConteoAGAutomatizado`
- `RequestStats8`

Cada tabla conserva:

- su nombre técnico;
- encabezados;
- estructura;
- formato;
- una fila vacía para mantener válida la definición de tabla.

También se conservaron:

- nombres de hojas;
- dashboards;
- gráficos;
- fórmulas estructurales;
- tablas auxiliares de métricas;
- configuración histórica necesaria para comprender el funcionamiento del libro.

Se eliminaron los registros que contenían información operacional, solicitudes reales, datos históricos identificables y demás contenido de producción.

## Advertencias de mantenimiento

Antes de modificar este libro se debe comprobar el impacto en:

1. Flujos de Power Automate que escriben o consultan sus tablas.
2. Fórmulas de `Metricas` y `Metricas 2025`.
3. Gráficos y tablas auxiliares.
4. Procesos históricos o de auditoría que dependan de nombres de tablas u hojas.

No se recomienda:

- renombrar tablas;
- renombrar hojas utilizadas por fórmulas o automatizaciones;
- eliminar columnas sin revisar dependencias;
- sustituir el libro productivo por esta copia sanitizada.

## Alcance

Este README documenta la estructura técnica conocida a partir de:

- las definiciones JSON de los flujos de Power Automate exportados en las soluciones de SIGEDOCA;
- el análisis estructural de la copia de `Estadística de las solicitudes.xlsx`;
- la función descrita para el libro dentro de Archivo Central y Archivos de Gestión.

La documentación podrá ampliarse conforme se incorporen al repositorio los componentes de SharePoint, Power BI y demás dependencias del sistema.
