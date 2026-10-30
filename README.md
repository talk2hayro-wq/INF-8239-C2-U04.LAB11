# INF-8239 · Unidad 04 · Gobernanza y DSS responsable

Autor académico: Edwin Ramón José Nolasco
Estudiante: Hairo Fermin – AG4949 · Ejercicio 07 (LAB11 y LAB12) y LAB13 · 30 de octubre de 2026

Proyecto base acumulativo para LAB11–13. El estudiante completa archivos pequeños en `student_inputs/`, ejecuta scripts y concentra su esfuerzo en interpretar y justificar decisiones.

> Los datos son sintéticos y didácticos. El sistema no autoriza decisiones automáticas sobre personas.

## Caso
**Territorial Review DSS - Synthetic** (`configs/governance.yml`): un sistema de apoyo a decisiones que ordena casos territoriales sintéticos para revisión humana.
- Usos prohibidos: intervención automática, decisión de elegibilidad y declaración de emergencia.
- Variables sensibles: `protected_group` y `age_band`. Identificadores directos prohibidos: `name`, `document_number`, `email` y `phone`.
- Puertas de calidad: recall ≥ 0.68, brechas de recall y FPR ≤ 0.16, latencia ≤ 500 ms y modelo ≤ 1024 KB.

## Requisitos
- [uv](https://docs.astral.sh/uv/) y Python 3.12 (`requires-python = ">=3.12,<3.13"`).
- CPU es suficiente. No necesita internet después de `uv sync`: el dataset viene incluido.

## Inicio
```bash
uv sync --frozen
uv run pytest -q
uv run python scripts/check_environment.py
```
Resultado esperado: `10 passed` y `{'python_project': 'ok', 'dataset_status': 'valid', 'rows': 640, 'columns': 9}`. `dataset_status: valid` indica que el archivo cumple el contrato técnico, no que sea adecuado para una decisión real.

## Ruta
```bash
uv run python scripts/lab11_impact.py
uv run python scripts/lab12_audit.py --model logistic
uv run python scripts/lab13_governance.py
uv run streamlit run app.py
```

## LAB11 · Evaluación de impacto ético
1. Completar `student_inputs/lab11_impact.yml`, `stakeholders.csv` y `risk_register.csv` (solo los valores REPLACE).
2. `uv run python scripts/lab11_impact.py`: valida los campos, calcula la prioridad (probabilidad × impacto) y genera `reports/lab11/`.
3. `uv run pytest -q tests/test_impact.py tests/test_privacy.py`.
4. Escribir `reports/lab11/DECISION_RECORD.md` **después** de la última ejecución del script, que lo sobrescribe con la plantilla.

| Salida (`reports/lab11/`) | Contenido |
| --- | --- |
| `risk_register_scored.csv` | 8 riesgos ordenados por prioridad |
| `stakeholders.csv` | Copia validada de las partes interesadas |
| `rmf_summary.json` | Resumen NIST AI RMF (GOVERN, MAP, MEASURE, MANAGE) |
| `DECISION_RECORD.md` | Evidencia, incertidumbre, responsables y decisión |

## Estructura
```
configs/governance.yml   propósito, usos prohibidos, variables sensibles y puertas
data/                    territorial_risk_synthetic.csv (640 casos, 9 columnas)
src/inf8239_u04/         data, impact, model, fairness, governance, pipeline
scripts/                 check_environment, lab11_impact, lab12_audit, lab13_governance, simulate_incident
student_inputs/          entradas del estudiante (LAB11–13)
reports/lab11..lab13/    evidencia generada por laboratorio
docs/                    DATASET_CARD, MODEL_CARD, RUNBOOK, AI_USE_DECLARATION
tests/                   datos, privacidad, impacto, equidad, modelo, gobernanza y app
app.py                   aplicación Streamlit
```

## Cambios respecto del proyecto base
- `scripts/lab11_impact.py`: el texto de `DECISION_RECORD.md` tenía saltos de línea literales dentro de la cadena y el script fallaba con `SyntaxError: unterminated string literal`. Se reemplazaron por `\n`; la lógica no cambió.
- `scripts/simulate_incident.py` (LAB13) tenía el mismo defecto y se corrigió igual.
- `student_inputs/raci.csv` se reescribió con los roles del registro de riesgos y 9 actividades (venía completo con roles genéricos).
- Las demás modificaciones son las entradas de `student_inputs/`, los reportes de `reports/lab11/` y `reports/lab12/` y `docs/AI_USE_DECLARATION.md`. `lab12_audit.py` no se modificó.

## Hallazgos LAB11
- **Decisión: REVISAR (REVIEW).** El sistema puede pasar al entrenamiento y la auditoría del LAB12, pero no puede ordenar una cola real. Mientras tanto, la cola funciona por orden de llegada con una lista de verificación pública.
- **Riesgos prioritarios:** R02 deriva temporal (9), R01 más falsos negativos en G2 (8) y R07 uso fuera de alcance (8). R02 encabeza por probabilidad; R01 y R07, con impacto 4 y riesgo residual alto, son los que impiden desplegar.
- **Datos que respaldan los riesgos:**
  - G2 tiene una tasa de `target` de 0.64 frente a 0.50 en G1.
  - `age_band` está declarada sensible y aun así es una entrada del modelo.
  - La partición temporal entrena hasta el 24 de septiembre.
- **Límite de los controles automáticos:** `deployment_decision` solo devuelve GO o REVIEW, nunca STOP. Retirar el sistema depende de la condición de retiro escrita y de su propietario.
- **`rmf_summary.json` es texto fijo:** solo MAP depende de los datos y MEASURE queda pendiente hasta el LAB12. Un archivo existente no prueba un control eficaz.
- **Pruebas:** 10 passed (proyecto completo) y 3 passed (impacto y privacidad).

| Ficha de propósito | Valor |
| --- | --- |
| Decisión asistida | Orden de la cola de revisión humana |
| Usuarios | Operadores de revisión territorial y su supervisor |
| Propietario | Coordinación de revisión territorial |
| Alternativa no predictiva | Cola por orden de llegada con lista de verificación pública |
| Condición de retiro | Recall < 0.68, brecha de recall o FPR > 0.16, uso fuera de alcance o identificadores directos |
| Apelación | Revisión manual por el supervisor, sin la puntuación del modelo, en 5 días hábiles |

## LAB12 · Auditoría de equidad con Fairlearn
LAB11 y LAB12 forman un solo ejercicio (Ejercicio 07): la auditoría mide lo que la evaluación de impacto dejó pendiente (MEASURE).
```bash
uv run python scripts/describe_dataset.py
uv run python scripts/lab12_audit.py --model logistic
uv run python scripts/lab12_audit.py --model tree
uv run python scripts/validate_lab12_decision.py
uv run pytest -q tests/test_fairness.py tests/test_model.py
```
Cada modelo guarda `global_metrics.json`, `fairness_by_group.csv` y `fairness_intersectional.csv` en `reports/lab12/<modelo>/`. `protected_group` no entra al modelo; se conserva solo para auditar.

| Métrica (160 casos de prueba) | Logístico | Árbol | Puerta |
| --- | --- | --- | --- |
| Accuracy | 0.719 | 0.706 | — |
| Recall global | 0.717 | 0.717 | ≥ 0.68 |
| Recall G1 / G2 | 0.653 / 0.791 | 0.633 / 0.814 | — |
| FPR G1 / G2 | 0.250 / 0.375 | 0.269 / 0.438 | — |
| Brecha de recall | 0.138 | 0.181 | ≤ 0.16 |
| Brecha de FPR | 0.125 | 0.168 | ≤ 0.16 |
| Entrenamiento | 0.11 s | 0.16 s | — |
| Tamaño | 3.6 KB | 7.9 KB | ≤ 1024 KB |

## Hallazgos LAB12
- **El promedio oculta una falla en G1:** recall global de 0.717, pero 0.653 en G1 (17 de 49 positivos sin prioridad) y 0.588 en G1 · 45-59.
- **La hipótesis R01 del LAB11 se invirtió:** se esperaban más falsos negativos en G2, pero el recall bajo está en G1.
- **El árbol no aporta:** iguala el recall, empeora accuracy y FPR, supera las puertas de brecha y pesa el doble. Las diferencias son de uno o dos casos. Por Green AI se elige el logístico, la opción suficiente.
- **Soporte mínimo para las intersecciones: 30 casos de prueba** (el proyecto no lo define). Solo G1 · 45-59 (35) lo alcanza; las otras siete se reportan sin interpretar. G2 tiene solo 16 negativos, así que su FPR es inestable.
- **La tasa de selección no es la métrica pertinente:** su brecha (0.232) refleja sobre todo las tasas base distintas (0.49 en G1 y 0.73 en G2 en la prueba).
- **Decisión (`student_inputs/lab12_decision.yml`): REVIEW.** El logístico pasa las puertas, pero las puertas no miden el recall por grupo. Siguen pendientes la decisión sobre `age_band`, la política de uso y la apelación.
- **Brecha no es causalidad:** tasas base distintas, muestras pequeñas, posibles variables sustitutas (`zone`, `age_band`), etiqueta sintética y una sola partición. La brecha indica qué investigar; no prueba discriminación.
- **Pruebas:** `decision_status: documented` y 2 passed.

Detalle en `reports/lab11/DECISION_RECORD.md`, `student_inputs/lab12_decision.yml`, `docs/MODEL_CARD.md`, `docs/RUNBOOK.md` y `docs/AI_USE_DECLARATION.md`.

## LAB13 · Gobernanza y DSS responsable con Streamlit
```bash
uv run python scripts/lab13_governance.py
uv run python scripts/simulate_incident.py
uv run pytest -q
uv run streamlit run app.py
uv export --format requirements-txt --no-dev -o requirements.txt
```
- `lab13_governance.py` valida `student_inputs/raci.csv`, ejecuta el pipeline completo y escribe `reports/lab13/deployment_decision.json` y `fairness_by_group.csv`.
- `simulate_incident.py` escribe `reports/lab13/INCIDENT_POSTMORTEM.md`; los campos de comunicación y acciones correctivas se completan después, porque el script sobrescribe el archivo.
- Documentación completada: `docs/MODEL_CARD.md`, `docs/RUNBOOK.md` y `docs/DATASET_CARD.md`.

| Puerta (`governance.yml`) | Valor | Umbral |
| --- | --- | --- |
| Recall global | 0.717 | ≥ 0.68 |
| Brecha de recall / FPR | 0.138 / 0.125 | ≤ 0.16 |
| Latencia por caso | 0.076 ms | ≤ 500 ms |
| Tamaño del modelo | 3.56 KB | ≤ 1024 KB |
| Entrenamiento | 0.29 s | — |

## Hallazgos LAB13
- **GO automático, REVIEW gobernado:** el pipeline pasa las cinco puertas, pero no mide el recall por grupo (G1 0.653). La decisión humana registrada es REVIEW.
- **RACI:** 9 actividades con un solo Accountable cada una. El supervisor de operadores puede detener (STOP) sin aprobación previa y la Coordinación lo confirma en 24 h.
- **El código nunca devuelve STOP:** la detención y el rollback a la cola por orden de llegada están en `RUNBOOK.md` y dependen de personas.
- **Tablero:** muestra el propósito, los usos prohibidos, las métricas, los grupos, las puertas y la acción humana. No muestra la probabilidad ni la prioridad por caso, la versión del modelo ni la latencia, y el indicador más visible es «GO». Se usa `use_container_width`, que está obsoleto (solo es una advertencia).
- **Simulacro:** detección, contención, preservación de evidencia, comunicación, rollback y postmortem con responsables y fechas. Hay tres acciones correctivas: revisar G1, agregar una puerta de recall por grupo e incluir la versión en la decisión.
- **Pruebas:** 10 passed (datos, privacidad, impacto, equidad, modelo, gobernanza y Streamlit sin navegador).
- **Publicación:** `requirements.txt` sin dependencias de desarrollo. El repositorio no tiene datos personales ni secretos. Etiqueta `u04-lab13`.

## Uso previsto y límites
- Solo para practicar contratos de datos, evaluación temporal, auditoría por subgrupos y gobernanza.
- No usar para intervención automática, elegibilidad, declaración de emergencia ni para perfilar personas o poblaciones reales.
- Las probabilidades e impactos del registro son juicios sobre un caso sintético, no frecuencias observadas.
- Las métricas de equidad se calculan con 160 casos de prueba, una partición y una semilla: indican qué investigar, no declaran el modelo justo o injusto.

## Dataset
`data/territorial_risk_synthetic.csv`: 640 casos sintéticos de 2026 generados con una semilla educativa fija. No contiene personas, lugares, tasas ni estadísticas oficiales reales de la República Dominicana (`docs/DATASET_CARD.md`).
