# arellan-legal-compliance — Cumplimiento Legal (Perú)

Documentación de obligaciones legales del ecosistema digital de la Clínica Automotriz Arellan Hnos conforme a la legislación peruana vigente.

## Marco Legal Aplicable

| Norma | Ámbito | Obligación |
|-------|--------|-----------|
| **Ley N.° 29733** | Protección de Datos Personales | Registro de banco de datos ante MINJUS |
| **DS 003-2013-JUS** | Reglamento Ley 29733 | Política de privacidad publicada, procedimientos ARCO |
| **Ley N.° 30096** | Delitos Informáticos | Protección contra acceso no autorizado |
| **Ley N.° 728** | Productividad y Competitividad Laboral | Despido justificado con evidencia documentada |
| **DL 728** | Fomento del Empleo | Contratos, liquidaciones, beneficios sociales |
| **Ley de Comprobantes de Pago (DL 814)** | SUNAT | Emisión de comprobantes válidos |
| **RS 007-99/SUNAT** | Reglamento Comprobantes | Facturas electrónicas (Fase 2+) |

## Obligaciones con MINJUS — Ley 29733

### Registro de Banco de Datos
El taller almacena datos personales de clientes (nombre, DNI, teléfono, email, dirección). Antes del lanzamiento del portal de clientes, debe inscribirse el banco de datos ante la Autoridad Nacional de Protección de Datos Personales (MINJUS).

**Pasos:**
1. Acceder al Portal de Datos Personales: `datospersonales.minjus.gob.pe`
2. Registrar la empresa como titular del banco de datos
3. Describir: qué datos se recopilan, para qué, por cuánto tiempo, con quién se comparten
4. Obtener el certificado de inscripción

**Estado:** Pendiente — debe realizarse antes del lanzamiento del portal de clientes.

### Política de Privacidad
Debe publicarse en `cliente.arellan.pe` antes del lanzamiento:
- Qué datos se recopilan y para qué
- Cómo se protegen
- Cómo ejercer derechos ARCO
- Contacto del responsable de datos: `datos@arellan.pe`

## Obligaciones con SUNAT

### Emisión de Comprobantes Electrónicos (Fase 2)
Las empresas formales en Perú tienen obligación de emitir facturas electrónicas (FE) a través de un Operador de Servicios Electrónicos (OSE). Esta obligación aplica al taller cuando emite facturas a empresas o cuando el monto lo requiere.

**Proveedor recomendado para MVP:** Nubefact (OSE homologado SUNAT, desde S/.29/mes)
**Integración técnica:** Ver `arellan-docs-governance/arellan-technical-docs/integrations/sunat-integration.md`

### RUC y Tributos
- Verificar que el taller esté en régimen tributario correcto (MYPE Tributaria recomendado)
- Los ingresos registrados en el sistema deben coincidir con las declaraciones mensuales (PDT 621)

## Obligaciones Laborales — Ley 728

El módulo de personal del sistema es una herramienta de respaldo para la gestión laboral, **no reemplaza** al asesor laboral:

### Evidencia para Despido Justificado
Cuando se decide no renovar o terminar la relación laboral con un empleado, el sistema provee:
- **Historial de incidencias disciplinarias** — registradas en `audit_logs` y módulo de personal
- **Registros de asistencia** — integración ZKTeco con timestamps exactos
- **Registro de acciones no autorizadas** — cualquier acción fuera de rol queda en audit
- **Evidencia de cobros irregulares** — transacciones fuera del flujo autorizado

**IMPORTANTE:** El sistema provee evidencia técnica. El proceso legal de terminar una relación laboral **siempre requiere asesoría legal externa**. No actuar solo con el registro del sistema.

### Proceso Recomendado Antes de Terminar Contrato
1. Consultar con abogado laboral las formas de terminación válidas bajo la Ley 728
2. Preparar la liquidación correcta (CTS, vacaciones proporcionales, gratificaciones)
3. Documentar las causas con respaldo del sistema (si aplica falta grave)
4. Generar carta de no renovación con la anticipación que estipula la ley
5. Al término: desactivar cuenta en el sistema INMEDIATAMENTE

## Responsabilidad Civil — Vehículos de Clientes

Los vehículos de clientes ingresados al taller son activos bajo custodia del taller. El sistema:
- Registra el estado del vehículo al ingreso con **4 fotos + hash SHA-256** (evidencia inmutable)
- Registra cualquier movimiento autorizado o no autorizado del vehículo
- Genera geofencing de 200m alrededor del taller

Esta evidencia es fundamental si un cliente presenta un reclamo por daños o si ocurre un accidente. **Recomendación:** Contratar seguro de responsabilidad civil para talleres automotrices (póliza de RC Profesional).
