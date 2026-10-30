# Model Card — complete after LAB13

Territorial Review DSS - Synthetic · regresión logística (`models/model.joblib`, generado por `scripts/lab13_governance.py`) · Unidad 04, LAB11–13.

## Intended use
- Ordenar la cola de casos territoriales **sintéticos** para que un operador revise primero los de mayor riesgo estimado. El modelo prioriza la revisión humana; no decide ni interviene.
- Usuarios: operadores de revisión territorial y su supervisor. El equipo de datos lo mantiene y lo audita.
- Contexto: demostración académica del curso INF-8239 sobre gobernanza, equidad y Green AI.

## Out-of-scope use
- Intervención automática, decisión de elegibilidad o declaración de emergencia (`configs/governance.yml`).
- Declarar culpabilidad o tomar cualquier decisión sobre una persona sin revisión humana registrada.
- Perfilar personas o poblaciones reales: los datos son sintéticos y no representan territorios ni tasas reales.
- Usar el modelo con datos que incluyan identificadores directos (`name`, `document_number`, `email`, `phone`); el contrato de datos los rechaza.

## Data and split
- `data/territorial_risk_synthetic.csv`: 640 casos sintéticos de 2026, 9 columnas, `target` binario (55 % positivos). Ver `docs/DATASET_CARD.md`.
- Variables del modelo: `zone`, `age_band`, `previous_incidents`, `infrastructure_score` y `exposure`. Excluidas: `case_id`, `received_date`, `protected_group` (solo para auditar) y `target`.
- Partición temporal 75/25: 480 casos de entrenamiento hasta el 24 de septiembre y 160 de prueba desde el 25 de septiembre. Semilla 42.
- Modelo: `OneHotEncoder` + `StandardScaler` + `LogisticRegression(max_iter=1200, class_weight="balanced")`.

## Global and subgroup evaluation
| Métrica (160 casos de prueba) | Global | G1 (n = 101) | G2 (n = 59) | Brecha | Puerta |
| --- | --- | --- | --- | --- | --- |
| Accuracy | 0.719 | 0.703 | 0.746 | 0.043 | — |
| Recall | 0.717 | **0.653** | 0.791 | 0.138 | ≥ 0.68 global; brecha ≤ 0.16 |
| FPR | 0.279 | 0.250 | 0.375 | 0.125 | brecha ≤ 0.16 |
| Tasa de selección | 0.531 | 0.446 | 0.678 | 0.232 | — |

- Pasa todas las puertas de `governance.yml`, pero **el recall de G1 (0.653) está por debajo de 0.68**: 17 de 49 casos positivos de G1 quedan sin prioridad, frente a 9 de 43 en G2.
- Intersecciones grupo × edad con un soporte mínimo de 30 casos de prueba: solo G1 · 45-59 (35 casos) es interpretable, con recall 0.588. Las otras siete (8 a 26 casos) se reportan en `reports/lab12/logistic/fairness_intersectional.csv` sin interpretar.
- G2 tiene 16 negativos en la prueba: cada falso positivo cambia su FPR 6 puntos.
- La brecha de tasa de selección refleja sobre todo las tasas base distintas (0.49 en G1 y 0.73 en G2). No es la métrica pertinente para el daño priorizado, que es el falso negativo.
- Una brecha no prueba discriminación causal: indica dónde investigar (datos, umbral, variables sustitutas y más observaciones).

## Green AI evidence
| Alternativa | Recall | Brecha recall / FPR | Entrenamiento | Tamaño | Decisión |
| --- | --- | --- | --- | --- | --- |
| Regresión logística | 0.717 | 0.138 / 0.125 | 0.11 s | 3.6 KB | **Seleccionada**: suficiente y la más pequeña |
| Árbol (`max_depth=5`) | 0.717 | 0.181 / 0.168 | 0.16 s | 7.9 KB | Descartada: no mejora y supera las puertas de brecha |
| Cola por orden de llegada | — | — | 0 | 0 | Línea base segura y alternativa no predictiva |

- Los tiempos son de una ejecución en CPU y varían entre corridas; no se usaron para elegir el modelo.
- Pipeline gobernado (`reports/lab13/deployment_decision.json`): 0.29 s de entrenamiento, 3.56 KB y una latencia de 0.076 ms por caso, frente a las puertas de 500 ms y 1024 KB. El tiempo de entrenamiento varía entre corridas; el tamaño y las métricas se reproducen exactamente.
- No se justifica un modelo más complejo: el árbol ya no aporta utilidad.

## Limitations, human oversight and appeal
- **Decisión gobernada: REVIEW.** El pipeline devuelve GO porque sus puertas miden el recall global y las brechas, no el recall por grupo. Un responsable humano revisa la evidencia y registra su decisión; mientras tanto se usa la cola por orden de llegada.
- **El código nunca devuelve STOP.** La detención depende de la condición de retiro: recall < 0.68, brecha de recall o FPR > 0.16, uso fuera de alcance o identificadores directos. La activa el supervisor y la aprueba la Coordinación de revisión territorial (`student_inputs/raci.csv`).
- **Apelación:** la persona afectada, o el operador en su nombre, puede pedir que el supervisor revise el orden de su caso sin la puntuación del modelo en 5 días hábiles. La respuesta se registra.
- **Limitaciones:**
  - 160 casos de prueba, una partición y una semilla, sin intervalos de confianza.
  - `age_band` es una variable sensible y aun así entra al modelo; la decisión sobre ella está pendiente.
  - La deriva temporal no se midió.
  - La salida de `deployment_decision.json` no incluye la versión del modelo ni la de la política.
  - Los datos son sintéticos.
- **Condiciones para GO:** recall ≥ 0.68 global y por grupo, brechas ≤ 0.16, decisión registrada sobre `age_band`, política de uso firmada y apelación operativa.

## Owner and review date
- **Propietario (Accountable):** Coordinación de revisión territorial.
- **Responsables técnicos:** líder del equipo de datos (métricas y auditoría) e ingeniero de ML (despliegue, monitoreo y reversión).
- **Versión de esta ficha:** 30 de octubre de 2026 (evidencia de LAB11, LAB12 y LAB13).
- **Próxima revisión:** 30 de enero de 2027, o antes si ocurre un incidente o una puerta falla.
