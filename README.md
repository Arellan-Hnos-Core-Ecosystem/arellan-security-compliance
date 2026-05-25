# arellan-security-compliance

Repositorio de seguridad, cumplimiento legal y gestión de riesgos de la Clínica Automotriz Arellan Hnos. Contiene políticas de acceso, matrices de riesgo, playbooks de respuesta a incidentes y documentación de cumplimiento normativo peruano.

## Descripción

`arellan-security-compliance` es el repositorio de mayor confidencialidad del ecosistema. Solo los owners tienen acceso completo. Centraliza todo lo relacionado con la postura de seguridad de la organización y su cumplimiento legal.

## Estructura

```
arellan-security-compliance/
├── arellan-security-governance/    # Políticas de seguridad, ITSEC
├── arellan-risk-management/        # Matrices de riesgo, mitigaciones
├── arellan-incident-response/      # Playbooks de respuesta a incidentes
├── arellan-legal-compliance/       # Cumplimiento legal peruano (SUNAT, MINJUS)
└── arellan-compliance-as-code/     # Controles automatizados en código
```

## Principios de Seguridad

1. **Zero Trust** — verificación explícita, mínimo privilegio, asumir breach
2. **Defensa en profundidad** — múltiples capas de control, ningún punto único de fallo
3. **Security by Design** — seguridad integrada desde el diseño, no añadida después
4. **Evidencia auditable** — todo control tiene trazabilidad en `audit_logs`

## Acceso a este Repo

Solo personas con rol `owner` en la organización `arellan-tech` tienen acceso de lectura. Este repo nunca debe tener colaboradores con acceso de escritura externo al equipo core.

Los archivos de este repo **nunca deben subirse públicamente** ni compartirse fuera de la organización.

## Revisión Periódica

| Documento | Revisión | Responsable |
|-----------|---------|-------------|
| Políticas de seguridad | Anual | Owners |
| Matrices de riesgo | Semestral | Owners |
| Playbooks de incidentes | Semestral | Equipo técnico |
| Certificaciones de cumplimiento | Anual | Owners + asesor legal |

## Repos Relacionados

- `arellan-infrastructure` — implementa los controles técnicos definidos aquí
- `arellan-platform` — aplica las políticas de autenticación y autorización definidas aquí
- `arellan-docs-governance` — documentación técnica derivada de estas políticas

## Licencia

Privado — Acceso muy restringido. © 2026 Arellan Hnos. Todos los derechos reservados.
