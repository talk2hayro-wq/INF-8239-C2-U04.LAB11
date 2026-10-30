# INF-8239 · Unidad 04 · Gobernanza y DSS responsable

Estudiante: Hairo Fermin – AG4949 · LAB11 · 29 de octubre de 2026

Proyecto base acumulativo para LAB11–13. El estudiante completa archivos pequeños en `student\\\\\\\_inputs/`, ejecuta scripts y concentra su esfuerzo en interpretar y justificar decisiones.

Los datos son sintéticos y didácticos. El sistema no autoriza decisiones automáticas sobre personas.

## Caso

**Territorial Review DSS - Synthetic** (`configs/governance.yml`): un sistema de apoyo a decisiones que ordena casos territoriales sintéticos para revisión humana.

* Usos prohibidos: intervención automática, decisión de elegibilidad y declaración de emergencia.
* Variables sensibles: `protected\\\\\\\_group` y `age\\\\\\\_band`. Identificadores directos prohibidos: `name`, `document\\\\\\\_number`, `email` y `phone`.
* Puertas de calidad: recall ≥ 0.68, brechas de recall y FPR ≤ 0.16, latencia ≤ 500 ms y modelo ≤ 1024 KB.

## Requisitos

* [uv](https://docs.astral.sh/uv/) y Python 3.12 (`requires-python = ">=3.12,<3.13"`).
* CPU es suficiente. No necesita internet después de `uv sync`: el dataset viene incluido.

## Inicio

```bash
uv sync --frozen
uv run pytest -q
uv run python scripts/check\\\\\\\_environment.py
```

Resultado esperado: `10 passed` y `{'python\\\\\\\_project': 'ok', 'dataset\\\\\\\_status': 'valid', 'rows': 640, 'columns': 9}`. `dataset\\\\\\\_status: valid` indica que el archivo cumple el contrato técnico, no que sea adecuado para una decisión real.

## Ruta

```bash
uv run python scripts/lab11\\\\\\\_impact.py
uv run python scripts/lab12\\\\\\\_audit.py --model logistic
uv run python scripts/lab13\\\\\\\_governance.py
uv run streamlit run app.py
```

## LAB11 · Evaluación de impacto ético

1. Completar `student\\\\\\\_inputs/lab11\\\\\\\_impact.yml`, `stakeholders.csv` y `risk\\\\\\\_register.csv` (solo los valores REPLACE).
2. `uv run python scripts/lab11\\\\\\\_impact.py`: valida los campos, calcula la prioridad (probabilidad × impacto) y genera `reports/lab11/`.
3. `uv run pytest -q tests/test\\\\\\\_impact.py tests/test\\\\\\\_privacy.py`.
4. Escribir `reports/lab11/DECISION\\\\\\\_RECORD.md` **después** de la última ejecución del script, que lo sobrescribe con la plantilla.

|Salida (`reports/lab11/`)|Contenido|
|-|-|
|`risk\\\\\\\_register\\\\\\\_scored.csv`|8 riesgos ordenados por prioridad|
|`stakeholders.csv`|Copia validada de las partes interesadas|
|`rmf\\\\\\\_summary.json`|Resumen NIST AI RMF (GOVERN, MAP, MEASURE, MANAGE)|
|`DECISION\\\\\\\_RECORD.md`|Evidencia, incertidumbre, responsables y decisión|

## Estructura

```
configs/governance.yml   propósito, usos prohibidos, variables sensibles y puertas
data/                    territorial\\\\\\\_risk\\\\\\\_synthetic.csv (640 casos, 9 columnas)
src/inf8239\\\\\\\_u04/         data, impact, model, fairness, governance, pipeline
scripts/                 check\\\\\\\_environment, lab11\\\\\\\_impact, lab12\\\\\\\_audit, lab13\\\\\\\_governance, simulate\\\\\\\_incident
student\\\\\\\_inputs/          entradas del estudiante (LAB11–13)
reports/lab11..lab13/    evidencia generada por laboratorio
docs/                    DATASET\\\\\\\_CARD, MODEL\\\\\\\_CARD, RUNBOOK, AI\\\\\\\_USE\\\\\\\_DECLARATION
tests/                   datos, privacidad, impacto, equidad, modelo, gobernanza y app
app.py                   aplicación Streamlit
```

## Cambios respecto del proyecto base

* `scripts/lab11\\\\\\\_impact.py`: el texto de `DECISION\\\\\\\_RECORD.md` tenía saltos de línea literales dentro de la cadena y el script fallaba con `SyntaxError: unterminated string literal`. Se reemplazaron por `\\\\\\\\n`; la lógica no cambió.
* `scripts/simulate\\\\\\\_incident.py` (LAB13) tiene el mismo defecto; se corregirá en ese laboratorio.
* Las demás modificaciones son las entradas de `student\\\\\\\_inputs/`, los reportes de `reports/lab11/` y `docs/AI\\\\\\\_USE\\\\\\\_DECLARATION.md`.

## Hallazgos LAB11

* **Decisión: REVISAR (REVIEW).** El sistema puede pasar al entrenamiento y la auditoría del LAB12, pero no puede ordenar una cola real. Mientras tanto, la cola funciona por orden de llegada con una lista de verificación pública.
* **Riesgos prioritarios:** R02 deriva temporal (9), R01 más falsos negativos en G2 (8) y R07 uso fuera de alcance (8). R02 encabeza por probabilidad; R01 y R07, con impacto 4 y riesgo residual alto, son los que impiden desplegar.
* **Datos que respaldan los riesgos:**

  * G2 tiene una tasa de `target` de 0.64 frente a 0.50 en G1.
  * `age\\\\\\\_band` está declarada sensible y aun así es una entrada del modelo.
  * La partición temporal entrena hasta el 24 de septiembre.
* **Límite de los controles automáticos:** `deployment\\\\\\\_decision` solo devuelve GO o REVIEW, nunca STOP. Retirar el sistema depende de la condición de retiro escrita y de su propietario.
* **`rmf\\\\\\\_summary.json` es texto fijo:** solo MAP depende de los datos y MEASURE queda pendiente hasta el LAB12. Un archivo existente no prueba un control eficaz.
* **Pruebas:** 10 passed (proyecto completo) y 3 passed (impacto y privacidad).

|Ficha de propósito|Valor|
|-|-|
|Decisión asistida|Orden de la cola de revisión humana|
|Usuarios|Operadores de revisión territorial y su supervisor|
|Propietario|Coordinación de revisión territorial|
|Alternativa no predictiva|Cola por orden de llegada con lista de verificación pública|
|Condición de retiro|Recall < 0.68, brecha de recall o FPR > 0.16, uso fuera de alcance o identificadores directos|
|Apelación|Revisión manual por el supervisor, sin la puntuación del modelo, en 5 días hábiles|

## Uso previsto y límites

* Solo para practicar contratos de datos, evaluación temporal, auditoría por subgrupos y gobernanza.
* No usar para intervención automática, elegibilidad, declaración de emergencia ni para perfilar personas o poblaciones reales.
* Las probabilidades e impactos del registro son juicios sobre un caso sintético, no frecuencias observadas.

Detalle en `reports/lab11/DECISION\\\\\\\_RECORD.md` y `docs/AI\\\\\\\_USE\\\\\\\_DECLARATION.md`.

## Dataset

`data/territorial\\\\\\\_risk\\\\\\\_synthetic.csv`: 640 casos sintéticos de 2026 generados con una semilla educativa fija. No contiene personas, lugares, tasas ni estadísticas oficiales reales de la República Dominicana (`docs/DATASET\\\\\\\_CARD.md`).

