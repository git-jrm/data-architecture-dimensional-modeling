# Arquitectura de datos y Modelado dimensional

En esta página se desarrolla el proyecto del módulo 5 del bootcamp de ing de datos, llamado "**Proyecto Arquitectura Datos**". Donde abordaremos una solución integral para una empresa que presenta diversos desafíos los cuales se desarrollan por etapas cada una aborda aspectos diferentes.

## Índice:
- [Etapa 1: Arquitectura de Datos](#etapa-1-arquitectura-de-datos)
- [Etapa 2: Enfoques para el Almacenamiento y Gestión de los Datos](#etapa-2-enfoques-para-el-almacenamiento-y-gestión-de-los-datos)
- [Etapa 3: Calidad de los Datos](#etapa-3-calidad-de-los-datos)
- [Etapa 4: Modelamiento Multidimensional](#etapa-4-modelamiento-multidimensional)
- [Análisis transversal](#an%C3%A1lisis-transversal)
- [Conclusión](#conclusi%C3%B3n)

## Etapa 1: Arquitectura de Datos

### Introducción

Este documento tiene como objetivo principal abordar el desafío de la gestión de datos en la empresa InfoHealth. A través del rol de Arquitecto de Datos, se propone un análisis exhaustivo de la situación actual, identificando las deficiencias clave que impiden la optimización de los procesos. 

Basado en este diagnóstico, se presenta una propuesta de arquitectura de datos y un plan de mejora, diseñados para garantizar la escalabilidad, seguridad y accesibilidad, pilares fundamentales para el crecimiento sostenido de la organización en el sector salud.

### Diagnóstico

Debido a la gran abundancia y diversidad de fuentes de datos junto a la falta de arquitectura de datos ha provocado un escenario donde ya se han reportado varios casos de duplicación de datos que sin trazabilidad genera riesgos de confianza, además se han detectado riesgos de seguridad en el acceso a la información.

Esta situación ha afectado al equipo de analistas de datos ya que el tiempo empleado en preprocesar los datos ha crecido exponencialmente.

Además la dirección y el staff médico han perdido la confianza en los reportes, por lo que es clave priorizar el proyecto ya que se requiere un sistema ágil y preciso para apoyar la operación del negocio.

### Propuesta

Se propone una arquitectura basada en capas que separa responsabilidades:

- Fuentes de datos: Datos clínicos, IoT, formularios, correos.
- Almacenamiento: Data Lake (no estructurados), Data Warehouse (estructurados).
- Procesamiento: ETL/ELT.
- Acceso: Dashboards de BI.
- Seguridad: Cifrado de datos. Control de acceso.

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

Para la Gobernanza se aplicaron los principios del marco DAMA-DMBOK destacando los siguientes aspectos:

- Calidad de datos: Garantizar la precisión y consistencia.
- Arquitectura de datos: permite la escalabilidad del sistema, promueve el reuso de componentes y facilita la trazabilidad de los datos.
- Modelado y diseño de datos: organiza los datos para una mejor comprensión y uso, asegurando su consistencia y eficiencia en el acceso.
- Seguridad de datos: proteger la información sensible en reposo, en tránsito y en uso.
- Integración e interoperabilidad de datos: unifica datos de diversas fuentes, permitiendo que sistemas diferentes se comuniquen de manera fluida.
- Data warehousing & business intelligence: proporciona una vista consolidada de los datos para análisis, facilitando la toma de decisiones estratégicas.

### Justificación de diseño

La arquitectura propuesta separa el almacenamiento y procesamiento, permitiendo manejar datos estructurados y no estructurados de forma escalable. Esto mejora la calidad de los datos, la seguridad y la velocidad de los reportes, lo que es vital para el sector salud.

[Volver](#m5-arquitectura-y-modelamiento-de-datos)

## Etapa 2: Enfoques para el Almacenamiento y Gestión de los Datos

### Tecnologías sugeridas

Las herramientas y tecnologías sugeridas por capas son:

- Almacenamiento: Amazon S3 (Data Lake), Amazon Redshift (Data Warehouse).
- Procesamiento: Apache Spark y AWS Glue.
- Acceso: Power BI, Tableau.
- Seguridad: cifrado en reposo y en tránsito mediante AWS y acceso con IAM.

### Gobernanza

Para la Gobernanza se recomiendan las prácticas del DAMA-DMBOK destacando:

- Gestión de metadatos: se implementa catálogo de datos centralizado, documentar fuentes, transformaciones y linaje para trazabilidad.
- Master Data Management: se define utilizar datos maestros únicos para: pacientes, médicos, historial, tratamientos. Para evitar duplicaciones y mantener la consistencia.
- Gestión del ciclo de vida de los datos: se definir políticas para la retención de 3 años de datos, y luego pasa a archivo seguro de información médica.
- Operaciones de datos: se establece procedimiento de monitoreo y backup diario. Y plan de recuperación de catastrofes de menos de 1 horas, garantizando disponibilidad y continuidad del servicio.

[Volver](#m5-arquitectura-y-modelamiento-de-datos)

## Etapa 3: Calidad de los Datos

Objetivo: Diseñar un plan de aseguramiento de calidad de los datos, integrado a la arquitectura definida.

### Controles por etapa

- Ingesta (RAW): validación de formatos, detección de archivos corruptos, verificación de esquemas
- Procesamiento (TRUSTED): reglas de limpieza, detección de duplicados, validación de rangos y dominios.
- Curación (CURATED): consistencia referencial, completitud de datos, exactitud de métricas calculadas.

### Métricas e indicadores

Completitud (% datos faltantes), exactitud (% errores), consistencia (duplicados), puntualidad (latencia de carga).

### Monitoreo y calidad

Implementación de data quality dashboards con alertas automáticas cuando las métricas superen umbrales críticos (ej: >5% datos faltantes). Plan de remediación con escalamiento automático al equipo de datos y re-procesamiento de lotes afectados.

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

[Volver](#m5-arquitectura-y-modelamiento-de-datos)

## Etapa 4: Modelamiento Multidimensional

### Introducción

En la actualidad la empresa Mercato del sector retail está comenzando a tener problemas de ralentización en el sistema ocasionado por la carga que supone los procesos de analitica del departamento de business intelligence quienes requieren con urgencia mejorar su sistema de analitica. Esta situación impacta a toda la organización por lo que es prioritaria.

### Diagnóstico

Luego de un análisis de la situación se concluyó que la lentitud que se ha empezado a reportar este último periodo está siendo generada actualmente por el sistema realtime de analitica que realiza sus consultas directamente a la base de datos transaccional OLTP.

Esto además de carga de procesamiento ha comenzado a generar complejidad operativa en el sistema de consultas con relaciones complejas.

### Propuesta

Diseño modelo multidimensional:

Definimos a nivel de Gobernanza de Datos una implementación de una solución de Data Warehouse, con enfoque bottom-up de Ralph Kimball. Esto nos permitirá centrarnos en el modelado multidimensional del Data Mart del área inteligencia de negocios.

Como analista de datos vamos a realizar una propuesta desarrollando el modelado multidimensional para los hechos de ventas y sus dimensiones relevantes. Mediante un cubo OLAP para análisis de hechos de ventas.

Tabla de hechos “Ventas”: id_p, id_c, id_s, id_t, cantidad, importe_unitario, importe_total.
Tabla de dimensiones Producto, Cliente, Sucursal, Tiempo.

Jerarquías y atributos de las dimensiones:

Producto: SKU → subcategoría → categoría (nombre_producto, marca, modelo)
Cliente: nicho → segmento → tipo (nombre_cliente, edad, email)
Sucursal: ciudad → región →país (nombre_sucursal, dirección, comuna).
Tiempo: día → mes → año (nombre_dia, dia_habil, descuento)

Con esta solución optimizada para lectura obtenemos escalabilidad, rendimiento analítico, facilidad para las consultas y presión para la toma de decisiones estratégicas.

### Justificación de diseño

Se optó por modelar un esquema estrella más simple que el esquema copo de nieve, ya que tiene menos tablas que mantener la complejidad es más baja, también la complejidad de las consultas.

El esquema al estar orientado a consultas simples realiza una desnormalización controlada ganando simplicidad y eficiencia. Además es evolutivo porque facilita agregar nuevas dimensiones y métricas si se requiere.

Se complementa con la implementación de Slowly Changing Dimensions “Type 4”, para una tabla separada con data histórica (ej: más de 3 años).

[Volver](#m5-arquitectura-y-modelamiento-de-datos)

## Análisis transversal

- Transformación digital sectorial: Ambos casos (InfoHealth y Mercato) evidencian cómo la falta de arquitectura de datos impacta directamente en la operación empresarial. En el sector salud, la pérdida de confianza en los reportes compromete decisiones críticas, mientras que en retail, la lentitud del sistema afecta la competitividad comercial.
- Gobernanza como factor crítico: La implementación exitosa de las soluciones técnicas propuestas (Data Lake multicapa, Data Warehouse multidimensional) depende fundamentalmente de una gobernanza sólida basada en DAMA-DMBOK, que asegure calidad, seguridad y cumplimiento normativo.
- Escalabilidad y flexibilidad: Las arquitecturas propuestas priorizan la separación de responsabilidades y el diseño evolutivo. El enfoque bottom-up de Kimball en Mercato y la arquitectura por capas en InfoHealth permiten crecimiento incremental sin comprometer la estabilidad del sistema.
- Calidad como pilar transversal: La implementación de controles de calidad en cada etapa del flujo de datos (RAW→TRUSTED→CURATED) garantiza la confiabilidad necesaria para la toma de decisiones estratégicas en ambos sectores.

[Volver](#m5-arquitectura-y-modelamiento-de-datos)

## Conclusión

La tecnología actúa como habilitador fundamental en la transformación organizacional hacia decisiones basadas en datos. Los casos analizados demuestran que las arquitecturas implementadas (Data Lake y Data Warehouse) resuelven problemas técnicos inmediatos mientras construyen capacidades analíticas sostenibles.

El éxito depende de integrar estas herramientas tecnológicas con procesos organizacionales efectivos, donde la gobernanza facilita la adopción gradual y el impacto medible en el desempeño empresarial

[Volver](#m5-arquitectura-y-modelamiento-de-datos)
