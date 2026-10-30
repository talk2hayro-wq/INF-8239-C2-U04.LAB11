# Dataset Card

Synthetic territorial-risk dataset generated with a fixed educational seed. It contains no real persons, locations, rates or official Dominican Republic statistics. Intended only to practice contracts, temporal evaluation, subgroup auditing and governance. Prohibited: operational profiling or inference about real populations.

## Motivación y procedencia

* Archivo: `data/territorial\_risk\_synthetic.csv`, incluido en el proyecto base de la Unidad 04 (INF-8239). Generado con una semilla educativa fija; no se descarga ni se recolectó de personas.
* Propósito: practicar contratos de datos, evaluación temporal, auditoría por subgrupos y gobernanza de un sistema de apoyo a decisiones.

## Composición

|Columna|Tipo|Uso|
|-|-|-|
|`case\_id`|Identificador del caso (CASE-0001…)|Excluida del modelo|
|`received\_date`|Fecha, 2026-01-01 a 2026-12-27|Solo para el corte temporal|
|`zone`|Norte, Sur, Este, Oeste|Variable del modelo|
|`protected\_group`|G1 (396) y G2 (244)|**Solo auditoría**; no entra al modelo|
|`age\_band`|18-29, 30-44, 45-59, 60+|Variable del modelo; declarada sensible en `governance.yml`|
|`previous\_incidents`|Entero de 0 a 9|Variable del modelo|
|`infrastructure\_score`|Real de 0 a 1|Variable del modelo|
|`exposure`|Real de 0 a 1|Variable del modelo|
|`target`|0 o 1 (55 % positivos)|Etiqueta|

* 640 filas, sin valores faltantes ni identificadores directos.
* Tasa de `target` = 1: 0.50 en G1 y 0.64 en G2. Por zona, entre 0.52 (Norte) y 0.62 (Sur).

## Contrato y privacidad

* `src/inf8239\_u04/data.py` exige las 9 columnas, un `target` binario y rechaza `name`, `document\_number`, `email` y `phone` antes de predecir (`tests/test\_privacy.py`).
* Cumplir el contrato (`dataset\_status: valid`) no demuestra que los datos sean adecuados para decidir sobre personas.

## Uso y partición

* Partición temporal 75/25: 480 casos hasta el 24 de septiembre (entrenamiento) y 160 desde el 25 de septiembre (prueba).
* En la prueba: 101 casos de G1 y 59 de G2, con solo 16 negativos en G2.

## Limitaciones

* Las tasas, zonas y grupos son inventados; ninguna conclusión se traslada a poblaciones reales.
* `zone` y `age\_band` pueden funcionar como variables sustitutas del grupo.
* La etiqueta refleja el proceso de generación sintética, no una necesidad real verificada.
* Tamaños pequeños por intersección: ninguna intersección grupo × edad del conjunto de prueba supera 35 casos.

## Mantenimiento

* Responsable: líder del equipo de datos. Accountable: Coordinación de revisión territorial (`student\_inputs/raci.csv`).
* Cualquier cambio del archivo exige volver a ejecutar las pruebas y las auditorías de LAB12 y LAB13.

