# arellan-security-governance — Políticas de Seguridad del Ecosistema

Políticas de seguridad de la información para la Clínica Automotriz Arellan Hnos. Define los controles, responsabilidades y estándares de seguridad que aplican a toda la organización digital.

## Política de Control de Acceso

### Principio de Mínimo Privilegio
Cada usuario tiene acceso solo a los datos y funciones estrictamente necesarias para su función. Ningún empleado tiene acceso a módulos que no correspondan a su rol.

### Gestión de Credenciales
- Contraseñas: mínimo 12 caracteres, combinación de mayúsculas, minúsculas, números y símbolos
- MFA obligatorio para roles: `OWNER`, `ADMIN`, `FINANCE`
- Contraseñas no reutilizables (últimas 5 contraseñas bloqueadas)
- Expiración de contraseñas: 90 días para roles críticos
- Prohibición absoluta de compartir credenciales entre empleados
- Las credenciales del sistema (API keys, secrets) se gestionan en AWS Secrets Manager

### Cuentas de Usuario
- Cada persona tiene una cuenta individual única, identificada por su email institucional
- Cuentas de ex-empleados: NUNCA se eliminan (se mantiene historial) — se desactivan (`status: TERMINATED`) dentro de las 2 horas de finalizar la relación laboral
- Al desvincular a alguien: rotación inmediata de TODOS los secretos del sistema

## Política de Tokens JWT y Sesiones

```
Access Token:
- Algoritmo: RS256 (asimétrico — las claves se guardan en Secrets Manager)
- Duración: 1 hora (roles admin/finance/owner)
- Duración: 12 horas (mechanic — tablet compartida)
- Rotación de claves: cada 90 días

Refresh Token:
- Duración: 7 días
- Almacenamiento: hash bcrypt en BD, nunca en texto plano
- Revocación: inmediata al logout, al cambiar contraseña, o forzado por owner
- Sesión única: configurable por rol (nuevo login invalida sesiones anteriores)
```

## Política de Protección de Datos Sensibles

Los siguientes campos se almacenan **cifrados con AES-256** en la base de datos:
- `mfa_secret` — secreto TOTP del usuario
- `account_number_encrypted` — cuentas bancarias del taller
- Datos de tarjetas: solo últimos 4 dígitos + token de pasarela (nunca número completo)

Campos prohibidos de aparecer en logs:
- Contraseñas (ni hasheadas)
- Tokens de sesión
- Números de tarjeta de crédito
- Credenciales de proveedores de pago

## Política de Red y Comunicaciones

- HTTPS obligatorio en todos los endpoints (TLS 1.2 mínimo, TLS 1.3 preferido)
- Cloudflare SSL en capa de CDN + certificados ACM en AWS
- HSTS habilitado con `max-age=31536000; includeSubDomains; preload`
- CORS restrictivo: solo orígenes `*.arellan.pe` explícitamente listados
- No se aceptan requests HTTP (solo HTTPS) — redirección automática 301

## Política de Código y Repositorios

- Código fuente en repositorios privados bajo la organización `arellan-tech`
- 2FA obligatorio para todos los miembros de la organización GitHub
- Branch protection en `main` y `develop` en todos los repos
- Prohibido commit de secretos, credenciales o datos de producción en código
- Secret scanning habilitado (GitHub detecta y alerta automáticamente)
- Code review obligatorio antes de merge a `main` (mínimo 1 aprobación)
- Los freelancers externos trabajan en ramas dedicadas, nunca tienen acceso a producción

## Política de Auditoría

La tabla `audit_logs` registra de forma automática e inmutable:
- Todos los logins y logouts (exitosos y fallidos)
- Toda acción que muta datos (INSERT, UPDATE, DELETE en tablas de negocio)
- Exportaciones y descargas de datos
- Cambios de roles y permisos
- Accesos a módulos sensibles (finanzas, audit)
- Uso de vehículos del taller

**Los audit logs son inmutables.** No existe endpoint de DELETE ni UPDATE sobre `audit_logs`. Los owners pueden ver pero nunca modificar.

## Política de Dispositivos

- La tablet del taller (`arellan-mechanic-ui`) es un dispositivo corporativo, no personal
- La tablet tiene acceso restringido exclusivamente a `taller.arellan.pe`
- Pin de bloqueo en la tablet; sesión de aplicación expira a los 30 minutos de inactividad
- Prohibido instalar aplicaciones no autorizadas en la tablet del taller
- Al detectar pérdida o robo de la tablet: revocar sesión remotamente y cambiar PIN de todos los usuarios del rol mechanic
