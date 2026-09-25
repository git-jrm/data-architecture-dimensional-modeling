# 🏗️ Arquitectura de datos y Modelado dimensional

Este repositorio documenta dos casos de arquitectura y modelado de datos aplicados a escenarios inspirados en empresas reales: diseño de una arquitectura de datos por capas (Data Lake, Data Warehouse, gobernanza) y modelado dimensional para analítica de negocio (esquema estrella, enfoque Kimball).

Índice
- [I. 🏥 InfoHealth: Arquitectura de Datos para Recuperar la Confianza](#i--infohealth-arquitectura-de-datos-para-recuperar-la-confianza)
  - [1. Diagnóstico y Arquitectura Propuesta](#1-diagnóstico-y-arquitectura-propuesta)
  - [2. Almacenamiento y Gestión de Datos](#2-almacenamiento-y-gestión-de-datos)
  - [3. Calidad de los Datos](#3-calidad-de-los-datos)
- [II. 🛒 Mercato: Del OLTP al Modelo Dimensional para BI](#ii--mercato-del-oltp-al-modelo-dimensional-para-bi)
- [Análisis Transversal](#análisis-transversal)
- [Conclusión General](#conclusión-general)

---

# I. 🏥 InfoHealth: Arquitectura de Datos para Recuperar la Confianza

## 1. Diagnóstico y Arquitectura Propuesta

### Introducción

Este caso aborda el desafío de la gestión de datos en la empresa InfoHealth. Desde el rol de Arquitecto de Datos, se propone un análisis de la situación actual, identificando las deficiencias clave que impiden la optimización de los procesos.

A partir de este diagnóstico, se presenta una propuesta de arquitectura de datos y un plan de mejora, diseñados para garantizar escalabilidad, seguridad y accesibilidad — pilares fundamentales para el crecimiento sostenido de la organización en el sector salud.

### Diagnóstico

La gran abundancia y diversidad de fuentes de datos, junto a la falta de una arquitectura de datos, ha generado casos de duplicación sin trazabilidad, lo que compromete la confianza en la información. Además, se han detectado riesgos de seguridad en el acceso a los datos.

Esta situación ha afectado al equipo de analistas, cuyo tiempo de preprocesamiento ha crecido exponencialmente. La dirección y el staff médico han perdido confianza en los reportes, por lo que se requiere un sistema ágil y preciso que soporte la operación del negocio.

### Propuesta

Se propone una arquitectura basada en capas que separa responsabilidades:

- **Fuentes de datos:** datos clínicos, IoT, formularios, correos.
- **Almacenamiento:** Data Lake (no estructurados), Data Warehouse (estructurados).
- **Procesamiento:** ETL/ELT.
- **Acceso:** dashboards de BI.
- **Seguridad:** cifrado de datos, control de acceso.

Diagrama de fuentes de datos, ingesta, integración y almacenamiento:
```mermaid
graph LR;
  DatosClínicos-->ETL;
  formularios-->ETL;
  IoT-->DataLake_RAW;
  correos-->DataLake_RAW;
  DataLake_RAW-->DataLake_TRUSTED;
  DataLake_TRUSTED-->DataLake_CURATED;
  ETL-->DataWarehouse;
  DataLake_CURATED-->DataWarehouse;
  DataWarehouse-->DataMart_Medicina;
  DataWarehouse-->DataMart_RRHH;
  DataWarehouse-->DataMart_Administracion;
```

### Gobernanza

Se aplican los principios del marco DAMA-DMBOK, destacando:

- **Calidad de datos:** garantizar precisión y consistencia.
- **Arquitectura de datos:** permite escalabilidad, promueve el reuso de componentes y facilita la trazabilidad.
- **Modelado y diseño de datos:** organiza los datos para una mejor comprensión y uso.
- **Seguridad de datos:** protege la información sensible en reposo, en tránsito y en uso.
- **Integración e interoperabilidad:** unifica datos de diversas fuentes para que sistemas distintos se comuniquen de forma fluida.
- **Data warehousing & business intelligence:** entrega una vista consolidada para análisis y toma de decisiones estratégicas.

### Justificación de diseño

La arquitectura propuesta separa almacenamiento y procesamiento, permitiendo manejar datos estructurados y no estructurados de forma escalable. Esto mejora la calidad de los datos, la seguridad y la velocidad de los reportes — factores críticos en el sector salud.

[Volver al índice](#índice)

## 2. Almacenamiento y Gestión de Datos

### Tecnologías sugeridas

Herramientas y tecnologías sugeridas por capa:

- **Almacenamiento:** Amazon S3 (Data Lake), Amazon Redshift (Data Warehouse).
- **Procesamiento:** Apache Spark y AWS Glue.
- **Acceso:** Power BI, Tableau.
- **Seguridad:** cifrado en reposo y en tránsito mediante AWS, control de acceso con IAM.

### Gobernanza

Prácticas recomendadas bajo DAMA-DMBOK:

- **Gestión de metadatos:** catálogo de datos centralizado; documentación de fuentes, transformaciones y linaje para trazabilidad.
- **Master Data Management:** datos maestros únicos para pacientes, médicos, historial y tratamientos, evitando duplicaciones.
- **Ciclo de vida de los datos:** retención de 3 años, luego archivo seguro de información médica.
- **Operaciones de datos:** monitoreo y backup diario; plan de recuperación ante desastres con RTO menor a 1 hora, garantizando disponibilidad y continuidad del servicio.

[Volver al índice](#índice)

## 3. Calidad de los Datos

**Objetivo:** diseñar un plan de aseguramiento de calidad de datos, integrado a la arquitectura definida.

### Controles por zona del Data Lake

- **Ingesta (RAW):** validación de formatos, detección de archivos corruptos, verificación de esquemas.
- **Procesamiento (TRUSTED):** reglas de limpieza, detección de duplicados, validación de rangos y dominios.
- **Curación (CURATED):** consistencia referencial, completitud de datos, exactitud de métricas calculadas.

### Métricas e indicadores

Completitud (% datos faltantes), exactitud (% errores), consistencia (duplicados), puntualidad (latencia de carga).

### Monitoreo y calidad

Data quality dashboards con alertas automáticas cuando las métricas superan umbrales críticos (ej: >5% datos faltantes), con plan de remediación y escalamiento automático al equipo de datos para re-procesar los lotes afectados.

### Integración en arquitectura

Los controles de calidad se ejecutan en cada zona del Data Lake usando AWS Glue DataBrew y Apache Griffin, con resultados almacenados en tablas de auditoría para trazabilidad completa del linaje de datos.

Diagrama de proceso de monitoreo y remediación:
```mermaid
graph LR;
  RAW-->|Validación básica|TRUSTED;
  TRUSTED-->|Limpieza + reglas|CURATED;
  CURATED-->|Métricas finales|DataWarehouse;
  RAW-->DQ_Dashboard;
  TRUSTED-->DQ_Dashboard;
  CURATED-->DQ_Dashboard;
```

[Volver al índice](#índice)

# II. 🛒 Mercato: Del OLTP al Modelo Dimensional para BI

## Introducción

La empresa Mercato, del sector retail, presenta problemas de ralentización del sistema causados por la carga que generan los procesos analíticos del departamento de business intelligence. La situación impacta a toda la organización y requiere resolución prioritaria.

## Diagnóstico

El análisis identificó que la lentitud reportada la genera el sistema realtime de analítica, que ejecuta sus consultas directamente contra la base de datos transaccional OLTP. Esto añade carga de procesamiento y complejidad operativa por las consultas con relaciones complejas.

## Propuesta

Se define, a nivel de Gobernanza de Datos, la implementación de una solución de Data Warehouse con enfoque bottom-up de Ralph Kimball, permitiendo centrar el diseño en el modelado multidimensional del Data Mart del área de inteligencia de negocios.

Se desarrolla una propuesta de modelado multidimensional para los hechos de ventas y sus dimensiones relevantes, mediante un cubo OLAP para análisis de hechos de ventas.

**Tabla de hechos "Ventas":** id_p, id_c, id_s, id_t, cantidad, importe_unitario, importe_total.

**Tabla de dimensiones:** Producto, Cliente, Sucursal, Tiempo.

Diagrama mermaid:
```mermaid
erDiagram
    FACT_VENTAS {
        int id_p PK, FK
        int id_c PK, FK
        int id_s PK, FK
        int id_t PK, FK
        int cantidad
        decimal importe_unitario
        decimal importe_total
    }
    DIM_PRODUCTO {
        int id_p PK
        string producto
        string sku
        string categoria
    }
    DIM_CLIENTE {
        int id_c PK
        string cliente
        string segmento
    }
    DIM_SUCURSAL {
        int id_s PK
        string sucursal
        string region
    }
    DIM_TIEMPO {
        int id_t PK
        date fecha
        string anio
    }
    FACT_VENTAS ||--o{ DIM_PRODUCTO : "es de"
    FACT_VENTAS ||--o{ DIM_CLIENTE : "lo realiza"
    FACT_VENTAS ||--o{ DIM_SUCURSAL : "ocurre en"
    FACT_VENTAS ||--o{ DIM_TIEMPO : "sucede en"
```

**Jerarquías y atributos de las dimensiones:**

- **Producto:** SKU → subcategoría → categoría (nombre_producto, marca, modelo)
- **Cliente:** nicho → segmento → tipo (nombre_cliente, edad, email)
- **Sucursal:** ciudad → región → país (nombre_sucursal, dirección, comuna)
- **Tiempo:** día → mes → año (nombre_dia, dia_habil, descuento)

Esta solución, optimizada para lectura, entrega escalabilidad, rendimiento analítico y facilidad de consulta para la toma de decisiones estratégicas.

## Justificación de diseño

Se optó por un esquema estrella en lugar de un esquema copo de nieve, ya que requiere menos tablas y reduce la complejidad tanto del modelo como de las consultas.

El esquema, orientado a consultas simples, aplica una desnormalización controlada que gana en simplicidad y eficiencia, y es evolutivo: facilita agregar nuevas dimensiones y métricas cuando se requiera.

Se complementa con Slowly Changing Dimensions tipo 4, usando una tabla separada para datos históricos (más de 3 años).

[Volver al índice](#índice)

---

## Análisis Transversal

**Transformación digital sectorial:** ambos casos (InfoHealth y Mercato) muestran cómo la falta de arquitectura de datos impacta directamente la operación. En salud, la pérdida de confianza en los reportes compromete decisiones críticas; en retail, la lentitud del sistema afecta la competitividad comercial.

**Gobernanza como factor crítico:** la implementación exitosa de las soluciones técnicas (Data Lake multicapa, Data Warehouse multidimensional) depende de una gobernanza sólida basada en DAMA-DMBOK que asegure calidad, seguridad y cumplimiento normativo.

**Escalabilidad y flexibilidad:** ambas arquitecturas priorizan la separación de responsabilidades y el diseño evolutivo. El enfoque bottom-up de Kimball en Mercato y la arquitectura por capas en InfoHealth permiten crecimiento incremental sin comprometer la estabilidad del sistema.

**Calidad como pilar transversal:** los controles de calidad en cada etapa del flujo de datos (RAW → TRUSTED → CURATED) garantizan la confiabilidad necesaria para la toma de decisiones estratégicas en ambos sectores.

[Volver al índice](#índice)

## Conclusión General

La tecnología actúa como habilitador de la transformación organizacional hacia decisiones basadas en datos. Los casos analizados muestran que las arquitecturas implementadas (Data Lake y Data Warehouse) resuelven problemas técnicos inmediatos mientras construyen capacidades analíticas sostenibles.

El éxito depende de integrar estas herramientas tecnológicas con procesos organizacionales efectivos, donde la gobernanza facilita la adopción gradual y el impacto medible en el desempeño empresarial.

[Volver al índice](#índice)[Volver](#m5)






