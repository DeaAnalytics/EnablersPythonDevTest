# Reto Técnico — LookML Generator

## Descripción general
Queremos evaluar tu capacidad para construir **scripts en Python** que trabajen con archivos de configuración, generen salidas estructuradas y mantengan buenas prácticas de ingeniería de software.

## Objetivo del reto
Desarrolla una **CLI en Python** que:
1. **Lea y valide** un archivo `input.yaml` (proporcionado en `sample_data/`).  
2. **Genere archivos de salida** centrados en un **catálogo de datos** (tablas, columnas, indicadores, tipos, etc.)y su **linaje** (relaciones entre tablas, explores y joins) derivados del YAML de entrada.  
   - La salida será **YAML o JSONL**, a tu elección (Se esperan al menos dos archivos (catalog.yaml o catalog.json y lineage.yaml o lineage.json). 
   - Debe modelar, como mínimo, **entidades de catálogo** (p. ej., business unit, tablas, columnas, indicadores) y su **linaje** (p. ej., relaciones/joins entre tablas, explores y dependencias).  
   - **Puedes definir los atributos** que consideres adecuados (nombres, tipos, descripciones, tags, calidad/nullable, owners, dominio, etc.), siempre que mantengas consistencia y lo documentes en el README.  
3. La salida debe conservar coherencia en nombres, estructura y tipos.
4. El código debe aplicar **un patrón de diseño** (por ejemplo: Builder, Strategy o Factory) y explicarlo brevemente en el README.
5. Incluir **pruebas unitarias** con **cobertura** (usando `pytest` o `unittest`).
6. (Deseable) Configurar **CI/CD** (por ejemplo, un workflow de GitHub Actions) que ejecute lint, pruebas y cobertura.

### Criterios de aceptación de la salida (catálogo/linaje)
- Representa **todas las tablas** y **columnas** definidas en el YAML.  
- Incluye **indicadores/medidas** asociadas a sus tablas.  
- Describe **relaciones** entre entidades (joins de explores, dependencias entre vistas/modelos).  
- Mantiene **referencialidad** (IDs/nombres que conecten entidades) y **consistencia de tipos**.  
- Está **documentada** en el README: formato elegido, atributos definidos y decisiones de modelado.

## Requisitos técnicos
- Lenguaje: **Python 3.10+**.
- El proyecto debe incluir:
  - `src/` con el código fuente.
  - `tests/` con las pruebas.
  - `README.md` explicando cómo ejecutar el proyecto, cómo correr los tests, y qué patrón de diseño se implementó.
  - `requirements.txt`

- **El archivo `README.md` debe incluir:**
   - Instrucciones para instalar dependencias y ejecutar la CLI.  
   - Ejemplo de uso (comando y salida esperada).  
   - Cómo correr las pruebas unitarias (`pytest`, `unittest`, etc.).  
   - Breve explicación del patrón de diseño implementado y las razones de su elección.    
 


## Evaluación
| Criterio | Descripción |
|-----------|--------------|
| Correctitud funcional | El script procesa correctamente el YAML y genera la salida esperada. |
| Diseño del código | Claridad, organización, uso de patrón de diseño. |
| Pruebas y cobertura | Pruebas unitarias significativas, cobertura ≥80%. |
| Documentación | README claro, explica patrón y pasos de ejecución. |
| CI/CD (bonus) | Lint + tests automáticos en GitHub Actions. |

## Entrega
1. Haz **fork** del repositorio.
2. Implementa tu solución dentro de las carpetas indicadas.


---

## Archivo de entrada (YAML)
Archivo: `sample_data/input.yaml`

```yaml
business_unit:
  name: bu_metales
  description: "Unidad demo para análisis comercial (datos sintéticos)."
  owner: "equipo_analitica@empresa.com"
  contact: "data-office@empresa.com"
  sla: "48h"

version: "1.1.0"
last_updated: "2025-11-04"
author: "Equipo Central de Analytics"
contact: "data-office@empresa.com"

tables:
  - name: ventas
    business_domain: comercial
    data_source:
      type: bigquery
      connection: conn_datahub-sintetico
      project: demo-mart
      dataset: mart_comercial
      table: ventas
    columns:
      - name: fecha
        type: date
      - name: pedido_id
        type: string
        is_primary_key: true
      - name: cliente_id
        type: string
      - name: producto_id
        type: string
      - name: cantidad
        type: number
      - name: monto
        type: number

  - name: clientes
    business_domain: comercial
    data_source:
      type: bigquery
      connection: conn_datahub-sintetico
      project: demo-mart
      dataset: dwh_comercial
      table: clientes
    columns:
      - name: cliente_id
        type: string
        is_primary_key: true
      - name: pais
        type: string

  - name: productos
    business_domain: comercial
    data_source:
      type: bigquery
      connection: conn_datahub-sintetico
      project: demo-mart
      dataset: dwh_producto
      table: productos
    columns:
      - name: producto_id
        type: string
        is_primary_key: true
      - name: familia
        type: string
      - name: precio_lista
        type: number

indicators:
  - name: total_ventas_mxn
    type: measure
    calculation: "sum(monto)"
    business_domain: comercial
    table: ventas
    aggregation_type: sum

  - name: total_pedidos
    type: measure
    calculation: "count_distinct(pedido_id)"
    business_domain: comercial
    table: ventas
    aggregation_type: count_distinct

  - name: avg_ticket_mxn
    type: measure
    calculation: "sum(monto) / nullif(count_distinct(pedido_id), 0)"
    business_domain: comercial
    table: ventas
    aggregation_type: derived

explores:
  - name: ventas
    base_table: ventas
    business_domain: comercial
    joins:
      - view: clientes
        type: left_outer
        sql_on: "${ventas.cliente_id} = ${clientes.cliente_id}"
      - view: productos
        type: left_outer
        sql_on: "${ventas.producto_id} = ${productos.producto_id}"
