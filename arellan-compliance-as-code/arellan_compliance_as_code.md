# arellan-compliance-as-code — Controles de Cumplimiento Automatizados

Implementación técnica de controles de seguridad y cumplimiento como código. Automatiza la verificación de que las políticas definidas en `arellan-security-governance` se están aplicando correctamente.

## Controles Automatizados Activos

### Control 1: Verificación de MFA para Roles Críticos
```typescript
// Guard que verifica MFA al acceder a módulos sensibles
@Injectable()
export class MfaRequiredGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest()
    const user: AuthUser = request.user
    const requiredMfaRoles = [UserRole.OWNER, UserRole.ADMIN, UserRole.FINANCE]

    if (requiredMfaRoles.includes(user.role) && !user.mfaVerified) {
      throw new ForbiddenException(
        'Este módulo requiere verificación MFA activa. Por favor, inicia sesión nuevamente.'
      )
    }
    return true
  }
}
```

### Control 2: Rate Limiting Financiero
```typescript
// Rate limiting específico para módulo financiero: 10 req/min por usuario
@UseGuards(ThrottlerGuard)
@Throttle({ default: { ttl: 60_000, limit: 10 } })
@Controller('finance')
export class FinanceController { ... }
```

### Control 3: Verificación de Inmutabilidad de Audit Logs
```typescript
// Test E2E que verifica que el audit log no acepta DELETE ni UPDATE
describe('Audit Log Immutability', () => {
  it('should reject DELETE on audit_logs', async () => {
    await expect(
      prisma.$executeRaw`DELETE FROM audit_logs WHERE id = ${testLogId}`
    ).rejects.toThrow()
  })

  it('should reject UPDATE on audit_logs', async () => {
    await expect(
      prisma.$executeRaw`UPDATE audit_logs SET action = 'TAMPERED' WHERE id = ${testLogId}`
    ).rejects.toThrow()
  })
})
```

### Control 4: Verificación de Segregación de Funciones Financieras
```typescript
// Verificar que quien registra un gasto no puede aprobarlo
describe('Finance Segregation of Duties', () => {
  it('should prevent requester from approving their own expense', async () => {
    const expense = await createExpense(financeUser)  // Finance crea el gasto

    // Finance no puede aprobar su propio gasto
    await expect(
      api.finance.approveExpense(expense.id, { userId: financeUser.id })
    ).rejects.toMatchObject({ status: 403, code: 'SELF_APPROVAL_FORBIDDEN' })
  })
})
```

### Control 5: Verificación de Data Masking para Mecánicos
```typescript
describe('Data Masking for Mechanics', () => {
  it('should not expose client phone to mechanic role', async () => {
    const response = await api.orders.getById(orderId, { role: 'MECHANIC' })
    expect(response.client.phone).toBeUndefined()
    expect(response.client.email).toBeUndefined()
    expect(response.client.dni).toBeUndefined()
  })
})
```

## Reportes de Cumplimiento Automatizados

```typescript
// Reporte mensual de cumplimiento de seguridad
@Cron('0 8 1 * *')  // Primer día del mes a las 8 AM
async generateMonthlyComplianceReport() {
  const report = {
    period: format(subMonths(new Date(), 1), 'MMMM yyyy'),
    mfa_adoption_rate: await this.getMfaAdoptionByRole(),
    failed_login_attempts: await this.getFailedLogins(),
    off_hours_access: await this.getOffHoursAccess(),
    audit_log_integrity: await this.verifyAuditLogIntegrity(),
    secrets_rotation_status: await this.checkSecretsRotation(),
    dependency_vulnerabilities: await this.checkDependencies(),
  }

  await this.notificationsService.sendComplianceReport(report)
}
```

## Checklist de Seguridad — Revisión Mensual

El equipo técnico debe verificar mensualmente:

```markdown
## Checklist de Seguridad Mensual — [MES/AÑO]

### Accesos y Cuentas
- [ ] Revisar lista de usuarios activos — eliminar cuentas que no correspondan
- [ ] Verificar que todas las cuentas de ex-empleados tienen status TERMINATED
- [ ] Confirmar que MFA está activo en todos los roles críticos
- [ ] Revisar intentos de login fallidos del mes

### Secretos y Credenciales
- [ ] Verificar fecha de rotación de JWT keys (rotar si >90 días)
- [ ] Confirmar que .env no está en ningún repo (secret scanning OK)
- [ ] Revisar alertas de Dependabot y actualizar dependencias vulnerables

### Audit Log
- [ ] Verificar integridad del audit log (test de inmutabilidad)
- [ ] Revisar accesos fuera de horario laboral
- [ ] Revisar exportaciones de datos del período

### Infraestructura
- [ ] Verificar que los backups se están ejecutando correctamente
- [ ] Revisar uptime del mes (status.arellan.pe)
- [ ] Confirmar que Sentry no tiene errores críticos sin resolver

### Backups
- [ ] Probar restauración del backup más reciente en staging
```
