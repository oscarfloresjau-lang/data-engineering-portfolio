# data-engineering-portfolio
This Repository is to upload data from data enginner 

Pipeline de datos en Databricks usando arquitectura medallón (Bronze → Silver → Gold)
para procesar ~180K registros operativos de supply chain (dataset DataCo) y calcular
KPIs de cumplimiento de entrega y rentabilidad.

## Arquitectura

- **Bronze**: ingesta incremental con Autoloader (Delta, checkpoint, schema evolution)
- **Silver**: tipado de 40+ columnas, validación de calidad (0 nulos introducidos),
  eliminación de PII, regla de negocio `is_late_delivery`
- **Gold**: modelo dimensional (star schema) + tablas de KPIs agregados

## Modelo dimensional (Gold)

- `dim_customer` (20,652 clientes)
- `dim_product` (118 productos)
- `dim_location` (3,800 ubicaciones, llave hash estable — ver decisiones técnicas)
- `dim_date` (1,133 fechas, 2015–2018)
- `fact_shipments` (180,519 líneas de pedido, grano: order item)

## Hallazgos de negocio

- **OTIF global: 42.72%** — más de la mitad de los envíos no llegan a tiempo/completos.
- **First Class: 0% OTIF** — el modo de envío "premium" tiene el peor cumplimiento de
  todos, señal de una posible desalineación entre el compromiso de entrega prometido
  y la capacidad logística real.
- Standard Class es el modo más confiable (60.23% OTIF).
- Márgenes de rentabilidad consistentes ~10-12% en la mayoría de categorías, con
  algunas excepciones notables (Golf Bags & Carts 17.46%, Men's Clothing 4.57%).

## Decisiones técnicas relevantes

- `dim_location` usa una llave hash (SHA-256) en vez de un join multi-columna directo,
  porque ~86% de los pedidos internacionales tienen `Order_Zipcode` nulo, y un join
  estándar no resuelve `NULL = NULL` como coincidencia.
- `Customer_Password` se descarta en Silver por gobierno de datos (PII sensible que
  no debe promoverse entre capas).
- Bronze usa Autoloader con `trigger(availableNow=True)` — patrón batch incremental,
  no streaming continuo, apropiado para un Job programado.

## Progreso
- [x] Bronze
- [x] Silver
- [x] Gold
- [ ] Orquestación con Databricks Workflows
- [ ] Conexión a Power BI
- [ ] Tests con pytest + CI básico
