# Ejercicio en clase 1
### Data Mining
Leandro Coral, Juan Pablo Bautista, Ryan de la Torre, Emilio Soria, Esteban Silva 
# Snowflake Mini Data Warehouse (TPC-H) — Gold Layer

## Objetivo
Este repo contiene la **capa Gold** modelada como **Star Schema** (esquema estrella) en Snowflake, más un set de **5 queries** de negocio para análisis.

Entregables solicitados:
- Un diagrama en draw.io (exportado a PNG o PDF) con:
  - Capa: Gold
  - Star schema
  - Granularidad (grain) de la tabla de hechos
- Carpeta `sql/` con scripts por capa (raw → gold)
- Carpeta con un archivo `.sql` que responda 5 queries:
  1) Ingresos por región y mes (cliente)
  2) Top clientes (año específico, ej. 1996) + región
  3) Mix de producto (brands y manufacturers) + unidades
  4) Efecto del descuento (descuento promedio vs revenue neto total)
  5) Performance logístico (dispatch days por ship mode + volumen)
 
## Organizacion de carpetas
Este repositorio cuenta con las siguientes carpetas:
* Queries: En esta carpeta se encuentran las consultas usadas para responder las preguntas del ejercicio en clase.
* sql: En esta carpeta se encuentra un archivo ``.sql `` que contiene las queries utilizadas para transformar la capa RAW a GOLD.
* diagrama: En esta carpeta se encuentra la imagen del diagrama utilizado.

## Diagrama


## Capa Gold (Star Schema)

### Tablas en Gold
Dimensiones:
- `DM_PROJECT.GOLD.DIM_CUSTOMER`
- `DM_PROJECT.GOLD.DIM_DATE`
- `DM_PROJECT.GOLD.DIM_NATION`
- `DM_PROJECT.GOLD.DIM_PART`
- `DM_PROJECT.GOLD.DIM_REGION`
- `DM_PROJECT.GOLD.DIM_SHIP_MODE`
- `DM_PROJECT.GOLD.DIM_SUPPLIER`

Hechos:
- `DM_PROJECT.GOLD.FACT_SALES`

### Grain (granularidad) de FACT_SALES
La tabla `FACT_SALES` está a nivel de **línea de orden** (line item).  
Grain sugerido (según columnas vistas):
- Una fila por combinación de:
  - `L_ORDERKEY` + `L_LINENUMBER`
  - y asociada a fechas: `ORDER_DATE`, `SHIP_DATE`
  - y llaves: `C_CUSTKEY`, `S_SUPPKEY`, `P_PARTKEY`
  - y atributo logístico: `SHIP_MODE`

Medidas principales:
- `GROSS_SALES`
- `NET_SALES` (revenue neto)
- `L_DISCOUNT`
- `L_TAX`
- `DISPATCH_DAYS` (días entre ORDER_DATE y SHIP_DATE)

### 1) Ingresos por región y mes (cliente)

**Pregunta:** ¿Cuáles son los ingresos (revenue neto) por **región del cliente** y **mes**?

![Ejercicio1_region](https://github.com/user-attachments/assets/517ec84d-a856-4c9f-a6a6-4d9a00e6d8ea)


El análisis muestra la evolución mensual del revenue neto (`NET_SALES`) por región del cliente utilizando la tabla de hechos `FACT_SALES` y las dimensiones de cliente y fecha.

**Hallazgos principales:**

- Todas las regiones (AFRICA, AMERICA, ASIA, EUROPE y MIDDLE EAST) presentan niveles de revenue muy similares dentro de cada mes.
- En los primeros meses observados (por ejemplo 1992-01), los ingresos por región se concentran alrededor de valores cercanos entre sí, sin que una región domine claramente a las demás.
- A lo largo del tiempo, el patrón se mantiene estable, con variaciones mensuales pero sin cambios drásticos en la distribución regional.
- Esto sugiere que las ventas están relativamente balanceadas entre regiones y que el crecimiento o disminución del revenue ocurre de forma paralela en todas ellas.

El negocio mantiene una distribución geográfica equilibrada en términos de revenue neto mensual, sin una región claramente dominante durante el período analizado.

#### Evolución del revenue neto por mes y región del cliente

![Ejercicio1_mes](https://github.com/user-attachments/assets/08533311-f7df-48bd-98b6-347485db4058)

Esta consulta analiza cómo cambian los ingresos (`NET_SALES`) a lo largo del tiempo, desagregados por **mes** y **región del cliente**, usando la tabla de hechos `FACT_SALES` y las dimensiones de fecha y cliente.

**Hallazgos principales:**

- El revenue muestra una evolución temporal consistente entre regiones, con patrones similares mes a mes.
- Durante los primeros años observados (como 1992), AFRICA, AMERICA, ASIA, EUROPE y MIDDLE EAST presentan niveles muy cercanos de ingresos dentro de cada mes.
- No se identifica una región que se despegue de forma sostenida a lo largo del tiempo; las variaciones mensuales afectan a todas de manera paralela.
- El comportamiento sugiere que los ciclos de negocio impactan globalmente y no están concentrados en una sola zona geográfica.

El revenue neto evoluciona de manera homogénea entre regiones, lo que indica un portafolio de ventas diversificado y una exposición balanceada entre mercados.
### 2) Top clientes (año específico)

**Pregunta:** ¿Quiénes son los 10 clientes con mayor revenue neto en un año específico (ej. 1996) y cuál es su región?

![Ejercicio2](https://github.com/user-attachments/assets/c1b8e819-a44d-4079-aeb1-e25d963de0f9)

**Hallazgos principales:**
- En 1996, el cliente #1 pertenece a EUROPE con 1,642,610.8027 de revenue neto.
- Distribución regional dentro del Top 10:
  * MIDDLE EAST: 4 clientes
  * EUROPE: 3 clientes
  * ASIA: 2 clientes
  * AMERICA: 1 cliente
- Aunque el #1 es de Europa, Middle East aparece más veces en el Top 10, lo que sugiere alta concentración de clientes grandes en esa región.


### 3) Mix de producto: brands y manufacturers

![Ejercicio3](https://github.com/user-attachments/assets/52ef5972-0b63-4f69-9480-08bc124e442d)

Esta consulta evalúa qué **brands** y **manufacturers** generan mayor revenue neto y cuántas **unidades vendidas** concentran, utilizando la tabla de hechos `FACT_SALES` y la dimensión de producto `DIM_PART`.

**Hallazgos principales:**

- El revenue neto no está distribuido uniformemente entre todas las marcas, sino que se concentra en un subconjunto reducido de brands con alto volumen de ventas.
- Los manufacturers con mayor revenue también son los que mueven más unidades, lo que indica que el desempeño está impulsado principalmente por volumen y no únicamente por precios unitarios altos.
- Existe una correlación clara entre unidades vendidas y revenue neto total: los productos más comercializados tienden a liderar en ingresos.
- El mix de producto está dominado por ciertos fabricantes estratégicos, que representan una parte relevante del total del negocio.

El análisis del mix de producto muestra que una fracción limitada de brands y manufacturers explica una porción significativa del revenue total. Esto sugiere oportunidades para:

- Priorizar relaciones comerciales con los fabricantes líderes.
- Analizar la rentabilidad individual de los productos top.
- Diseñar estrategias para diversificar el portafolio en categorías con menor participación.

### 4) Efecto del descuento por región

![Ejercicio4](https://github.com/user-attachments/assets/ed8cb19d-78b7-47e1-bfb2-912fbecc4c2e)

Esta consulta analiza cómo varía el **descuento promedio** aplicado a las ventas en cada región y cómo se relaciona con el **revenue neto total**, utilizando la tabla de hechos `FACT_SALES` y la dimensión de cliente `DIM_CUSTOMER`.

**Hallazgos principales:**

- El descuento promedio es prácticamente idéntico en todas las regiones, con valores cercanos a 0.05.
- MIDDLE EAST presenta el descuento promedio ligeramente más alto, seguido muy de cerca por ASIA, AFRICA, AMERICA y EUROPE.
- A pesar de estas diferencias mínimas en descuentos, el revenue neto total por región se mantiene en rangos similares, sin una región que sobresalga de forma clara.
- Esto indica que la política de descuentos es homogénea a nivel global y no está siendo usada como un diferenciador fuerte entre mercados.

Las regiones aplican niveles de descuento casi iguales, y el revenue neto total no muestra una dependencia fuerte del descuento promedio. El negocio parece operar con una estrategia de pricing consistente a escala internacional.

### 5) Performance logístico por ship mode

![Ejercicio5](https://github.com/user-attachments/assets/b174e15c-e4c7-474a-9cde-245467afb7e1)

Esta consulta analiza el desempeño operativo del proceso de despacho, midiendo el **tiempo promedio de envío** (días entre `ORDER_DATE` y `SHIP_DATE`) por **SHIP_MODE**, junto con el volumen total movido por cada modalidad.

**Hallazgos principales:**

- Los distintos ship modes presentan diferencias claras en los tiempos promedio de despacho, lo que refleja distintos niveles de servicio logístico.
- Los modos de envío con menor tiempo promedio tienden a concentrar mayor volumen de órdenes, lo que sugiere que el negocio prioriza las modalidades más rápidas.
- Los ship modes con tiempos más altos podrían estar asociados a envíos económicos o rutas más largas.
- El análisis permite comparar eficiencia operativa versus carga de trabajo en cada modalidad.

El desempeño logístico varía significativamente según el tipo de envío. Identificar qué ship modes combinan alta velocidad y alto volumen es clave para optimizar costos, renegociar contratos de transporte y mejorar los niveles de servicio al cliente.



