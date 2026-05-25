# arellan-incident-response — Playbooks de Respuesta a Incidentes

Procedimientos de respuesta ante incidentes de seguridad, fraude operativo y fallos críticos para el ecosistema de la Clínica Automotriz Arellan Hnos.

## Clasificación de Incidentes

| Severidad | Descripción | Tiempo de Respuesta |
|-----------|-------------|-------------------|
| **P0 — Crítico** | Sistema caído, fraude activo confirmado, fuga de datos | Respuesta inmediata (< 15 min) |
| **P1 — Alto** | Acceso no autorizado, descuadre de caja significativo, vehículo fuera del perímetro | < 1 hora |
| **P2 — Medio** | Error en módulo específico, intento de intrusión bloqueado, alerta de anomalía | < 4 horas |
| **P3 — Bajo** | Degradación de performance, alertas de stock crítico | < 24 horas |

## Playbook P0 — Fuga de Datos o Acceso No Autorizado

**PRESERVAR EVIDENCIA ANTES DE CUALQUIER ACCIÓN**

```
PASO 1: Identificar el alcance (< 5 minutos)
  → ¿Qué datos fueron accedidos? ¿Por quién? ¿Desde dónde?
  → Revisar audit_logs: SELECT * FROM audit_logs WHERE created_at > NOW() - INTERVAL '2 hours' ORDER BY created_at DESC
  → Revisar access logs en CloudWatch / Axiom

PASO 2: Contener el incidente (< 10 minutos)
  → Revocar TODAS las sesiones activas:
    UPDATE refresh_tokens SET revoked_at = NOW() WHERE revoked_at IS NULL;
  → Si el origen es un usuario específico: desactivar cuenta inmediatamente
    UPDATE accounts SET status = 'TERMINATED' WHERE id = '<user_id>';
  → Si el origen es externo: activar Cloudflare Under Attack Mode

PASO 3: Preservar evidencia (< 30 minutos)
  → Export de audit_logs del período del incidente (no modificar nada)
  → Snapshot de base de datos en ese momento
  → Screenshots de CloudWatch / Axiom con los logs del incidente
  → Guardar todo en carpeta fechada en S3 privado

PASO 4: Evaluar impacto y notificar
  → ¿Se expusieron datos de clientes? → Obligación legal de notificar a MINJUS (72h según Ley 29733)
  → ¿Se expusieron datos financieros? → Notificar a la familia directa y considerar asesor legal
  → Actualizar status.arellan.pe con mensaje de incidente

PASO 5: Erradicar y recuperar
  → Rotar TODOS los secretos (JWT, API keys, DB password)
  → Aplicar parche si hay una vulnerabilidad técnica identificada
  → Verificar que el vector de ataque está cerrado antes de reabrir el sistema

PASO 6: Post-mortem (dentro de 48 horas)
  → Documentar: qué pasó, por qué pasó, qué se hizo, qué se cambia
  → Actualizar este playbook si se identificaron gaps
  → Actualizar la matriz de riesgos (arellan-risk-management)
```

## Playbook P1 — Fraude Financiero Activo

```
Indicadores de activación:
  - Descuadre de caja > S/.50 sin justificación
  - Transacción financiera sin OT vinculada
  - Gasto autorizado con evidencia de proveedor falso (SUNAT no lo valida)
  - Modificación de audit_log detectada (intento)

PASO 1: No alertar al sospechoso
  → El sistema envía push silencioso a owners únicamente
  → No confrontar al empleado hasta tener evidencia completa

PASO 2: Recopilar evidencia digital
  → Export de audit_logs del período → PDF firmado digitalmente
  → Transacciones financieras involucradas
  → Comparar con registros de cámara de seguridad (fechas y horas del audit_log)

PASO 3: Consultar asesor legal
  → La Ley 728 (Perú) regula el despido justificado
  → La evidencia del sistema es admisible, pero debe estar acompañada de declaraciones

PASO 4: Desactivar cuenta cuando se decida actuar
  → UPDATE accounts SET status = 'TERMINATED' WHERE id = '<employee_id>'
  → La cuenta queda inactiva pero el historial se conserva íntegramente

PASO 5: Rotar accesos que el empleado conocía
```

## Playbook P1 — Vehículo Fuera del Perímetro (Joyride)

```
El sistema genera automáticamente: estado ALERT_JOYRIDE + push a owners + bloqueo app mecánico

PASO 1: Verificar en tiempo real
  → arellan-mobile-app muestra ubicación GPS del vehículo en tiempo real

PASO 2: Contactar al responsable asignado
  → El sistema ya tiene el teléfono del empleado asignado

PASO 3: Si no responde en 10 minutos
  → Llamar directamente
  → Si no hay respuesta: considerar denuncia policial por uso no autorizado de bien ajeno

PASO 4: Al regreso del vehículo
  → Registrar incidencia en módulo de personal
  → Fotografiar estado del vehículo (km, combustible, daños)
  → Evidencia queda en audit_log automáticamente
```

## Playbook P0 — Backend Completamente Caído

```
PASO 1: Verificar causa
  → Railway / AWS dashboard
  → Logs en CloudWatch / Axiom
  → Sentry para errores recientes

PASO 2: Si fue un deploy reciente → rollback inmediato
  → ./scripts/rollback.sh production

PASO 3: Comunicar a usuarios
  → Actualizar status.arellan.pe con estado "Sistema en mantenimiento"
  → WhatsApp a owners con ETA de recuperación

PASO 4: Modo de operación manual de emergencia
  → El taller puede operar con registro manual en papel durante máx. 4 horas
  → Al restaurar el sistema: ingresar los registros manuales como OTs retroactivas

PASO 5: Post-mortem obligatorio
```

## Contactos de Emergencia

| Rol | Responsabilidad en incidente |
|-----|--------------------------|
| Edgar (Owner) | Decisiones de negocio, autorización de acciones críticas |
| Juan (Owner) | Aprobación de respuesta a incidentes de seguridad |
| Equipo técnico | Análisis técnico, contención, remediación |
| Asesor legal externo | Incidentes con implicaciones legales o laborales |
