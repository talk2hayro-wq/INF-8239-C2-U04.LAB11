# Incident simulation

* detected\_at: 2026-10-30T15:59:58.923377+00:00
* signal: recall gap exceeded
* containment: REVIEW and safe baseline
* evidence: preserved report and model version
* \- communication: interna en 24 h (ingeniero de ML → Coordinación de revisión territorial, líder del equipo de datos, comité de ética y asesoría legal); a personas afectadas en 5 días hábiles por el supervisor de operadores, con aviso de apelación y revisión manual de su caso
* \- corrective actions, owners and dates: (1) revisar umbral y datos de G1, empezando por 45-59 — líder del equipo de datos — 2026-11-13; (2) agregar una puerta de recall por grupo a governance.yml — Coordinación de revisión territorial — 2026-11-20; (3) incluir versión de modelo y política en deployment\_decision.json — ingeniero de ML — 2026-11-20
* 
* \## Simulacro (registro)
* \- Detección: monitoreo mensual; la brecha de recall supera 0.16 (señal simulada; en la ejecución real es 0.138).
* \- Contención: el supervisor de operadores suspende el uso de la puntuación y pasa la cola a orden de llegada; la Coordinación confirma STOP en 24 h.
* \- Evidencia preservada: reports/lab12 y reports/lab13, models/model.joblib, configs/governance.yml, commit y etiqueta Git; lista de casos afectados solo por case\_id.
* \- Rollback: línea base segura (cola por orden de llegada); reactivación solo con todas las puertas en verde, recall ≥ 0.68 por grupo y aprobación registrada.
* \- Postmortem: responsable el ingeniero de ML, Accountable la Coordinación de revisión territorial; entrega en 10 días hábiles (2026-11-13) al comité de ética de datos.

