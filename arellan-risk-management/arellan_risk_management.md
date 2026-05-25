# arellan-risk-management — Gestión de Riesgos

Registro de riesgos identificados, su probabilidad, impacto y planes de mitigación para el ecosistema digital de la Clínica Automotriz Arellan Hnos.

## Metodología

Clasificación de riesgos por impacto (1-5) × probabilidad (1-5) = nivel de riesgo (1-25).

| Nivel | Rango | Acción requerida |
|-------|-------|-----------------|
| Crítico | 20–25 | Mitigación inmediata, monitoreo continuo |
| Alto | 12–19 | Plan de mitigación en 30 días |
| Medio | 6–11 | Seguimiento mensual |
| Bajo | 1–5 | Revisión trimestral |

## Registro de Riesgos

### Riesgos de Seguridad

| ID | Riesgo | Impacto | Prob. | Nivel | Estado |
|----|--------|---------|-------|-------|--------|
| R-SEC-01 | Credenciales de empleado comprometidas | 5 | 3 | 15 (Alto) | Mitigado: MFA obligatorio + audit log |
| R-SEC-02 | Acceso no autorizado a módulo financiero | 5 | 2 | 10 (Medio) | Mitigado: RBAC + rate limiting + MFA |
| R-SEC-03 | Fuga de datos de clientes (DNI, teléfono) | 4 | 2 | 8 (Medio) | Mitigado: data masking para mecánicos |
| R-SEC-04 | Inyección SQL o XSS en la aplicación | 5 | 2 | 10 (Medio) | Mitigado: Prisma ORM + class-validator |
| R-SEC-05 | Credenciales en código (git leak) | 4 | 2 | 8 (Medio) | Mitigado: secret scanning + .gitignore |

### Riesgos Operativos

| ID | Riesgo | Impacto | Prob. | Nivel | Plan de Mitigación |
|----|--------|---------|-------|-------|--------------------|
| R-OPS-01 | Caída del backend en horario pico | 4 | 2 | 8 | Railway/AWS autoscaling + health checks |
| R-OPS-02 | Pérdida de datos por fallo de BD | 5 | 1 | 5 | Backups diarios + replication |
| R-OPS-03 | Tablet del taller sin conexión | 3 | 4 | 12 (Alto) | Modo offline con queue local |
| R-OPS-04 | Fallo del servicio de notificaciones | 3 | 2 | 6 | Queue con retry + fallback email |
| R-OPS-05 | Pérdida de fotos de vehículos (evidencia) | 4 | 1 | 4 | S3 con replicación + hash SHA-256 |

### Riesgos de Negocio

| ID | Riesgo | Impacto | Prob. | Nivel | Plan de Mitigación |
|----|--------|---------|-------|-------|--------------------|
| R-BIZ-01 | Cobros fuera del sistema (Yape personal) | 5 | 3 | 15 (Alto) | QR dinámico obligatorio + audit de ingresos |
| R-BIZ-02 | Comisiones no declaradas en importaciones | 4 | 3 | 12 (Alto) | Módulo de importaciones con validación de margen |
| R-BIZ-03 | Uso no autorizado de vehículos del taller | 3 | 3 | 9 (Medio) | Control de vehículos + geofencing |
| R-BIZ-04 | Manipulación de registros de inventario | 4 | 2 | 8 (Medio) | Audit log inmutable + ajustes solo por owner/admin |
| R-BIZ-05 | Represalia de personal desvinculado | 3 | 3 | 9 (Medio) | Desactivación inmediata de cuenta + rotación de secretos |

### Riesgos Legales/Laborales

| ID | Riesgo | Impacto | Prob. | Nivel | Plan de Mitigación |
|----|--------|---------|-------|-------|--------------------|
| R-LEG-01 | Demanda laboral sin evidencia documentada | 4 | 3 | 12 (Alto) | Audit log de incidencias disciplinarias trazables |
| R-LEG-02 | Incumplimiento Ley 29733 (datos personales) | 3 | 2 | 6 | Registro ante MINJUS + políticas de dato |
| R-LEG-03 | Evasión tributaria no intencional | 4 | 2 | 8 | Integración SUNAT Fase 2 + trazabilidad de ingresos |
| R-LEG-04 | Accidente con vehículo de cliente sin autorización | 5 | 1 | 5 | Evidencia fotográfica inmutable + geofencing |

## Riesgos Críticos Activos — Seguimiento

### R-BIZ-01 — Cobros fuera del sistema
- **Estado actual:** En mitigación activa (MVP Fase 1)
- **Control implementado:** Flujo QR dinámico desde tablet — el mecánico no puede mostrar QR personal
- **Verificación:** Audit log de cada transacción comparado con cierre de caja diario
- **Próxima revisión:** Post-despliegue MVP

### R-OPS-03 — Tablet sin conexión
- **Estado:** Mitigado en Fase 1
- **Control:** Service Worker con cola IndexedDB — acciones se encolan y sincronizan al reconectar
- **Límite:** La cola tiene capacidad para 8 horas de trabajo offline
