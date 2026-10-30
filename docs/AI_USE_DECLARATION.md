# Declaración de uso de IA

Proyecto acumulativo de la Unidad 04 (LAB11–13). Cada laboratorio agrega su sección.

## LAB11 · Evaluación de impacto ético

### Herramienta

* **Claude (Anthropic)**, en la aplicación de Claude; identificador del modelo configurado: `claude-opus-5-5`.
* No se usaron otras herramientas de IA generativa.

### Finalidad

* Revisar el proyecto base y el caso sintético (`configs/governance.yml`, datos, `impact.py`, scripts y pruebas) antes de completar las entradas.
* Proponer el contenido de los campos marcados como REPLACE en las tres entradas del laboratorio.
* Diagnosticar el error de sintaxis de `scripts/lab11\_impact.py`.

### Fragmentos afectados

|Archivo|Intervención de la IA|
|-|-|
|`student\_inputs/lab11\_impact.yml`|Redacción de los nueve campos|
|`student\_inputs/stakeholders.csv`|Beneficio, daño y participación de los cuatro actores; nombres y nivel de poder sin cambios|
|`student\_inputs/risk\_register.csv`|Escenario, afectado, control, responsable y evidencia de R01–R08; probabilidad, impacto y residual sin cambios, tal como venían en la plantilla|
|`scripts/lab11\_impact.py`|Corrección de una línea: los saltos de línea literales de la cadena de `DECISION\_RECORD.md` se reemplazaron por `\\n`; la lógica no cambió|
|`reports/lab11/DECISION\_RECORD.md`|Redacción completa|
|`reports/lab11/risk\_register\_scored.csv`, `stakeholders.csv`, `rmf\_summary.json`|Ninguna: los generó el script|

### Verificaciones humanas

* Los riesgos se basan en datos verificables del proyecto: la tasa de `target` por grupo (0.64 en G2 frente a 0.50 en G1), la fecha de corte de la partición temporal (24 de septiembre), el uso de `age\_band` como entrada aunque está declarada sensible y la ausencia de STOP en `deployment\_decision`.
* El estudiante ejecutó localmente `uv run python scripts/lab11\_impact.py` (prioridades R02 9, R01 8, R07 8) y `uv run pytest -q tests/test\_impact.py tests/test\_privacy.py` (3 passed).
* Se comprobó que la condición de retiro usa los mismos umbrales que las puertas de `governance.yml`.
* `DECISION\_RECORD.md` se guardó después de la última ejecución del script, que lo sobrescribe con la plantilla.

### Errores detectados y correcciones

* **Proyecto base:** `scripts/lab11\_impact.py` fallaba con `SyntaxError: unterminated string literal`; se corrigió la cadena. `scripts/simulate\_incident.py` (LAB13) tiene el mismo defecto.
* **Fecha de corte:** la primera redacción de R02 decía «enero a septiembre»; se corrigió a «hasta el 24 de septiembre» tras verificar la partición temporal.
* **Lectura de la prioridad:** el orden del script no se aceptó tal cual. R02 encabeza por probabilidad, pero R01 y R07 (impacto 4, residual alto) son los que justifican no desplegar.
* **Lectura del resumen NIST AI RMF:** `rmf\_summary.json` es texto fijo del script (solo MAP depende de los datos), así que se interpretó como evidencia de documentación, no de controles eficaces.

### Responsabilidad

El estudiante revisó cada valor propuesto, ejecutó los comandos y verificó las cifras antes de incluirlas. Asume la responsabilidad por el contenido entregado. Las probabilidades, los impactos y los responsables son juicios sobre un caso sintético y no describen una institución real.

## LAB12 · Auditoría de equidad con Fairlearn

LAB11 y LAB12 forman un solo ejercicio (Ejercicio 07).

### Herramienta

* **Claude (Anthropic)**, el mismo asistente que en LAB11 (`claude-opus-5-5`).

### Finalidad

* Interpretar las salidas de `describe\_dataset.py` y de `lab12\_audit.py` (logístico y árbol): métricas globales, por grupo e interseccionales.
* Proponer un soporte mínimo para las intersecciones y redactar `student\_inputs/lab12\_decision.yml`.
* Contrastar la auditoría con el registro de riesgos del LAB11 y explicar por qué una brecha no prueba causalidad.

### Fragmentos afectados

|Archivo|Intervención de la IA|
|-|-|
|`student\_inputs/lab12\_decision.yml`|Redacción de los seis campos|
|`README.md`|Secciones del LAB12|
|`reports/lab12/`|Ninguna: los generó `lab12\_audit.py`, sin modificaciones|
|Scripts y código de `src/`|Ninguna|

### Verificaciones humanas

* El estudiante ejecutó `describe\_dataset.py`, `lab12\_audit.py --model logistic` y `--model tree`, `validate\_lab12\_decision.py` (`decision\_status: documented`) y `pytest -q tests/test\_fairness.py tests/test\_model.py` (2 passed).
* Las métricas del documento se copiaron de `global\_metrics.json` y de los CSV de cada modelo.
* Los conteos de positivos y negativos por grupo se derivan de las métricas del reporte (por ejemplo, G1: 45 seleccionados × 0.711 = 32 aciertos; 32 / 0.653 = 49 positivos). Se confirmaron recalculándolos con los modelos guardados.

### Errores detectados y correcciones

* **Hipótesis R01 del LAB11:** se esperaban más falsos negativos en G2 por su tasa base; la auditoría mostró que el recall bajo está en G1. Se documenta como contraste y no se reescribe el registro del LAB11.
* **Soporte mínimo:** la guía pide no interpretar los grupos por debajo de un «mínimo documentado» que el proyecto no define. Se fijó en 30 casos de prueba y se declara como decisión propia.
* **Puertas frente a decisión:** las puertas de `governance.yml` darían GO al modelo logístico; la decisión contextual es REVIEW porque el recall de G1 (0.653) está por debajo de 0.68.
* **Tiempos:** varían entre ejecuciones (en una verificación previa el árbol fue el más rápido); se reportan los de la ejecución del estudiante y no se usan para elegir el modelo.

### Responsabilidad

El estudiante revisó la decisión propuesta, ejecutó los comandos y verificó las cifras. La recomendación es académica, sobre datos sintéticos, y no autoriza decisiones sobre personas.

## LAB13 · Gobernanza y DSS responsable con Streamlit

### Herramienta

* **Claude (Anthropic)**, el mismo asistente (`claude-opus-5-5`).

### Finalidad

* Revisar `governance.yml`, `lab13\_governance.py`, `app.py` y las pruebas.
* Reescribir la matriz RACI y completar el Model Card, el runbook y el Dataset Card con la evidencia de LAB11–12.
* Diagnosticar el error de `simulate\_incident.py`, redactar los campos del postmortem e interpretar la decisión de despliegue y el tablero.

### Fragmentos afectados

|Archivo|Intervención de la IA|
|-|-|
|`student\_inputs/raci.csv`|Reescrito: 9 actividades con los roles del LAB11|
|`docs/MODEL\_CARD.md`, `docs/RUNBOOK.md`|Redacción completa|
|`docs/DATASET\_CARD.md`|Se conservó el aviso original y se agregaron secciones|
|`scripts/simulate\_incident.py`|Una línea corregida (saltos de línea literales en el f-string); la lógica no cambió|
|`reports/lab13/INCIDENT\_POSTMORTEM.md`|Comunicación, acciones correctivas y registro del simulacro; las líneas generadas por el script se conservaron|
|`reports/lab13/deployment\_decision.json`, `fairness\_by\_group.csv`|Ninguna: los generó el script|
|`app.py`, `src/` y `configs/`|Ninguna|

### Verificaciones humanas

* El estudiante ejecutó `lab13\_governance.py` (GO), `simulate\_incident.py`, `pytest -q` (10 passed) y la aplicación Streamlit, y comprobó cada elemento del tablero con una captura.
* La latencia (0.076 ms), el tamaño (3.56 KB) y el tiempo (0.29 s) se tomaron de `deployment\_decision.json`.
* Se comprobó que cada actividad de la RACI tiene un solo Accountable.

### Errores detectados y correcciones

* **Proyecto base:** `simulate\_incident.py` fallaba con `SyntaxError`; se corrigió igual que `lab11\_impact.py`.
* **RACI:** venía completa con roles genéricos en inglés que no coincidían con los responsables del LAB11; se reescribió para que la responsabilidad sea coherente en los tres laboratorios.
* **GO frente a REVIEW:** el pipeline devuelve GO; se documentó que las puertas no miden el recall por grupo y que la decisión gobernada es REVIEW.
* **Limitaciones no corregidas en el código:** no se produce STOP, la decisión no incluye versión, el tablero no muestra la probabilidad por caso y `use\_container\_width` está obsoleto. Se documentan sin modificar el código de la guía.
* **Señal del simulacro:** «recall gap exceeded» es simulada; en la ejecución real la brecha es 0.138. Se aclara en el postmortem.

### Responsabilidad

El estudiante revisó los documentos propuestos, ejecutó los comandos y verificó las cifras y el tablero. El DSS es una demostración académica con datos sintéticos y no autoriza decisiones sobre personas.

