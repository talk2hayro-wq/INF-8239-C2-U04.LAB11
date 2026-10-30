# Incident Runbook — complete after LAB13

Territorial Review DSS - Synthetic. Roles según `student_inputs/raci.csv`. Simulacro: `uv run python scripts/simulate_incident.py` → `reports/lab13/INCIDENT_POSTMORTEM.md`.

1. **Detection and classification:**
   - **Señales:**
     - una puerta falla en `reports/lab13/deployment_decision.json` (recall < 0.68, brecha de recall o FPR > 0.16, latencia > 500 ms, modelo > 1024 KB);
     - el recall de un grupo baja de 0.68 en el monitoreo mensual;
     - una apelación muestra un caso que requería atención y quedó al final de la cola;
     - se detecta que la puntuación se usó para elegibilidad, intervención o declaración de emergencia;
     - aparece un identificador directo en los datos.
   - **Clasificación:**
     - *Alta:* uso fuera de alcance, identificadores directos o daño confirmado a una persona.
     - *Media:* puerta de equidad o desempeño incumplida.
     - *Baja:* latencia o falla técnica sin daño.
   - **Quién detecta:** el ingeniero de ML (monitoreo) o el supervisor de operadores (apelaciones).
2. **Containment authority:**
   - El **supervisor de operadores** detiene de inmediato el uso de la puntuación y pasa la cola al orden de llegada. No necesita aprobación previa para contener.
   - La **Coordinación de revisión territorial** (Accountable) confirma la detención (STOP) en un máximo de 24 horas y es la única que autoriza reactivar el sistema.
   - El código no devuelve STOP: la detención es una decisión humana.
3. **Evidence preservation:**
   - Antes de cambiar nada, se conservan:
     - `reports/lab13/deployment_decision.json` y `fairness_by_group.csv`;
     - los reportes de `reports/lab12/`;
     - el commit y la etiqueta Git del modelo en uso;
     - `models/model.joblib`;
     - `configs/governance.yml`;
     - la lista de casos afectados (solo `case_id`, sin datos personales);
     - el registro de decisiones humanas.
   - Responsable: el ingeniero de ML.
   - Los archivos no se sobrescriben; se copian a una carpeta fechada del incidente.
4. **Internal and affected-party communication:**
   - **Interna:** el ingeniero de ML informa a la Coordinación, al líder del equipo de datos, al comité de ética y a la asesoría legal en un máximo de 24 horas.
   - **Personas afectadas:** el supervisor de operadores avisa a las personas cuyos casos pudieron quedar mal priorizados, con asesoría legal, en un máximo de 5 días hábiles. Explica qué ocurrió y cómo apelar, y ofrece la revisión manual de su caso.
   - No se comunican datos de otras personas.
5. **Rollback or safe baseline:**
   - **Línea base segura:** cola por orden de llegada con la lista de verificación pública (alternativa no predictiva). No requiere modelo.
   - **Reversión de versión:** `git checkout u04-lab13` (u otra etiqueta aprobada) y `uv sync --frozen`.
   - Responsable: el ingeniero de ML. Aprueba: la Coordinación.
   - Antes de reactivar se exige una nueva ejecución de `lab13_governance.py` con todas las puertas en verde, recall ≥ 0.68 por grupo y aprobación registrada.
6. **Postmortem, owner and due date:**
   - Responsable del postmortem: el ingeniero de ML (R), con la Coordinación de revisión territorial como Accountable.
   - Plazo: 10 días hábiles desde la contención.
   - Contenido: cronología, causa, grupos afectados, decisión de contención, acciones correctivas con responsable y fecha, y cambios al registro de riesgos y al Model Card.
   - Se presenta al comité de ética de datos.
