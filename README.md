# SIGEDOCA - DGII

**Sistema Integral de Gestión Documental Automatizada de la DGII**

SIGEDOCA es un sistema de gestión documental desarrollado mediante tecnologías de Microsoft 365 y Microsoft Power Platform.

El sistema integra automatización de procesos, gestión documental, solicitudes de usuarios, digitalización, control de calidad y otros procesos relacionados con la administración documental.

## Módulos

SIGEDOCA está compuesto actualmente por tres módulos principales:

### 1. Archivo Central

Módulo orientado a la gestión de procesos y solicitudes correspondientes al Archivo Central.

### 2. Archivos de Gestión

Módulo destinado a procesos asociados a los Archivos de Gestión y solicitudes realizadas desde las diferentes áreas o sucursales.

### 3. UDAD

Módulo de la Unidad de Almacenamiento de Datos y Digitalización.

Gestiona procesos relacionados con recepción, preparación, digitalización, control de calidad, solicitudes urgentes, corrección controlada de datos y cierre de cajas documentales.

## Tecnologías principales

SIGEDOCA utiliza diferentes servicios del ecosistema Microsoft, entre ellos:

- Microsoft Power Automate
- Microsoft Power Apps
- Microsoft SharePoint
- Microsoft Lists
- Microsoft Excel
- Microsoft Teams
- Microsoft Planner
- Microsoft Outlook
- Microsoft Power BI

## Estructura del repositorio

```text
SIGEDOCA_DGII/
├── Archivo_Central/
│   ├── Documentacion/
│   ├── Excel_BD/
│   ├── SharePoint/
│   └── Solucion/
│
├── Archivos_de_Gestion/
│   ├── Documentacion/
│   ├── Excel_BD/
│   ├── SharePoint/
│   └── Solucion/
│
├── UDAD/
│   ├── Documentacion/
│   ├── Excel_BD/
│   ├── Power_Apps/
│   ├── SharePoint/
│   └── Solucion/
│
└── Infraestructura_Compartida/
    └── SharePoint_Central/