# Registro de decisión · LAB11

**Sistema:** Territorial Review DSS - Synthetic (`configs/governance.yml`)
**Decisión evaluada:** si el sistema puede pasar a entrenamiento y auditoría (LAB12) como apoyo para ordenar la cola de revisión humana.
**Propietario:** Coordinación de revisión territorial
**Fecha:** 29 de octubre de 2026 · **Próxima revisión:** al cerrar el LAB12, con las métricas por grupo

## Decisión: REVISAR (REVIEW)

No se recomienda desplegar (GO) ni detener (STOP). El sistema puede avanzar al entrenamiento y la auditoría del LAB12 con datos sintéticos, pero **no puede ordenar ninguna cola real** hasta cumplir las condiciones de este registro. Mientras tanto, la cola funciona por orden de llegada con la lista de verificación pública (alternativa no predictiva).

## Evidencia disponible

| Archivo | Qué muestra | Qué no demuestra |
| --- | --- | --- |
| `student_inputs/lab11_impact.yml` | Propósito, decisión asistida, usuarios, población afectada, propietario, alternativa no predictiva, usos prohibidos, condición de retiro y apelación | Que esas reglas se cumplan en la práctica |
| `reports/lab11/stakeholders.csv` | Beneficio, daño, poder y participación de cuatro actores | Que la consulta comunitaria se haya hecho |
| `reports/lab11/risk_register_scored.csv` | 8 riesgos priorizados por probabilidad × impacto | Que los controles sean eficaces; la puntuación ordena la atención, no mide el daño |
| `reports/lab11/rmf_summary.json` | GOVERN y MAP documentados; MEASURE pendiente (LAB12); MANAGE con retiro y apelación descritos | El script escribe ese texto fijo: confirma que los archivos existen, no que los controles funcionen |
| `tests/test_impact.py`, `tests/test_privacy.py` | 3 passed: la puntuación funciona, se rechazan valores REPLACE y columnas con identificadores directos | Que no exista reidentificación por combinación de zona, edad y fecha |

## Tres riesgos prioritarios

1. **R02 · Deriva temporal (3 × 3 = 9, residual medio).** Es el más probable: el modelo se entrena con casos hasta el 24 de septiembre y se evalúa con los posteriores. Si los patrones cambian, la cola pierde casos urgentes sin que nadie lo note. Encabeza la lista por probabilidad, no por gravedad. *Control:* partición temporal, monitoreo mensual del recall con umbral de 0.68 y reentrenamiento aprobado por el propietario. *Responsable:* líder del equipo de datos.
2. **R01 · Más falsos negativos en G2 (2 × 4 = 8, residual alto).** G2 tiene una tasa base de 0.64 frente a 0.50 en G1. Un modelo que no considera esa diferencia puede dejar al final de la cola más casos de G2 que sí requieren atención. Es un daño grave para el grupo, y el riesgo sigue siendo alto después de los controles porque la auditoría por grupo aún no existe. *Control:* puerta de brecha de recall y FPR ≤ 0.16 y revisión manual de los casos de G2 con baja prioridad. *Responsable:* líder del equipo de datos.
3. **R07 · Uso fuera de alcance (2 × 4 = 8, residual alto).** Usar la puntuación para decidir elegibilidad, intervenir o declarar emergencias convertiría un orden de revisión en una decisión sobre personas. Ningún control técnico lo impide: `deployment_decision` solo devuelve GO o REVIEW, y la advertencia de la aplicación no bloquea el uso. *Control:* política de uso firmada, advertencia visible y registro de cada decisión con su justificación humana. *Responsable:* Coordinación de revisión territorial.

R01 y R07 empatan en prioridad e impacto; el orden entre ambos no indica que uno sea más importante. El riesgo de mayor prioridad (R02) no es el más grave: R01 y R07 tienen impacto 4 y riesgo residual alto, y son los que justifican no desplegar todavía.

## Incertidumbre

- No hay métricas del modelo: recall, brechas por grupo, latencia y tamaño se medirán en el LAB12. La sección MEASURE del resumen NIST AI RMF está vacía.
- Las probabilidades e impactos son juicios del equipo sobre un caso sintético, no frecuencias observadas.
- `age_band` es una variable sensible que el modelo usa (R03); su efecto se desconoce hasta la auditoría interseccional.
- Los datos son sintéticos: ninguna conclusión se traslada a poblaciones reales.

## Condiciones para pasar a GO

1. Recall ≥ 0.68 en la partición temporal y brechas de recall y FPR entre G1 y G2 ≤ 0.16 (LAB12).
2. Decisión registrada sobre retirar o justificar `age_band`, respaldada por la auditoría interseccional.
3. Política de uso firmada por el propietario y registro de decisiones humanas activo.
4. Vía de apelación operativa: supervisor designado, plazo de 5 días hábiles e informe mensual.

## Condición de retiro (STOP)

Detener el sistema y volver a la cola por orden de llegada si ocurre cualquiera de estas situaciones:
- el recall baja de 0.68;
- la brecha de recall o de FPR entre grupos supera 0.16;
- se detecta que se usa para elegibilidad, intervención automática o declaración de emergencia;
- aparecen identificadores directos en los datos.

Solo se reactiva con una nueva evaluación aprobada por el propietario.

## Responsables

| Responsabilidad | Rol |
| --- | --- |
| Decisión final y firma | Coordinación de revisión territorial |
| Métricas, auditoría por grupo y monitoreo (R01, R02) | Líder del equipo de datos |
| Decisión sobre `age_band` (R03) | Comité de ética de datos |
| Privacidad (R04) | Oficial de protección de datos |
| Apelaciones y respaldo operativo (R05, R06) | Supervisor de operadores |
