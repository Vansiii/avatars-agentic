# Feature Spec: Gestión de Usuarios y Perfiles

> **ID:** FEAT-004  
> **Versión:** 1.0  
> **Estado:** En desarrollo  
> **Prioridad:** P0 — Crítico (Core MVP)

---

## Descripción

Ciclo de vida completo de la cuenta de usuario: registro, autenticación, gestión del perfil, historial de generaciones y cumplimiento de privacidad (exportación y eliminación de datos).

---

## Criterios de Aceptación

### AC-001: Registro
- [ ] El usuario puede registrarse con email y contraseña.
- [ ] El usuario puede registrarse con OAuth2 (Google, GitHub).
- [ ] El email de verificación se envía en menos de 60 segundos.
- [ ] La contraseña requiere mínimo 8 caracteres, 1 mayúscula y 1 número.
- [ ] Un email ya registrado muestra un error claro sin revelar si existe la cuenta (seguridad).

### AC-002: Inicio de sesión
- [ ] El usuario puede iniciar sesión con email/contraseña o OAuth2.
- [ ] Tras 5 intentos fallidos, se bloquea el acceso por 15 minutos.
- [ ] El usuario puede solicitar un "magic link" como alternativa a la contraseña.
- [ ] El token JWT expira en 15 minutos; el refresh token, en 7 días.

### AC-003: Perfil de usuario
- [ ] El usuario puede actualizar: nombre, foto de perfil, bio corta (200 chars).
- [ ] La foto de perfil se procesa y guarda como avatar 200×200 px.
- [ ] El usuario puede ver su plan actual, créditos usados y créditos restantes.

### AC-004: Historial de generaciones
- [ ] El usuario ve todas sus generaciones pasadas ordenadas por fecha (más reciente primero).
- [ ] Cada entrada muestra: fecha, estilo usado, thumbnails de avatares generados y estado.
- [ ] El usuario puede descargar nuevamente un avatar desde el historial (mientras esté vigente).
- [ ] El historial se pagina (20 items por página).
- [ ] Los avatares de free tier se marcan como "expiran en X días" cuando quedan menos de 7 días.

### AC-005: Privacidad y GDPR
- [ ] El usuario puede exportar todos sus datos (JSON + imágenes ZIP) desde la configuración.
- [ ] La exportación se prepara de forma asíncrona y se envía por email cuando está lista.
- [ ] El usuario puede eliminar su cuenta permanentemente.
- [ ] La eliminación de cuenta borra: datos personales, historial, imágenes de entrada y avatares generados.
- [ ] Tras eliminar la cuenta, el email queda en lista negra para prevenir re-registro involuntario.

### AC-006: Notificaciones
- [ ] El usuario puede configurar qué emails recibir: generaciones completadas, créditos bajos, novedades.
- [ ] Las preferencias de notificación se pueden actualizar en cualquier momento.

---

## Flujos Principales

### Registro con Email
```
[Formulario de registro]
    → Validación client-side
    → POST /api/v1/auth/register
    → Email de verificación enviado
    → [Pantalla: "Revisa tu email"]
    → Usuario hace clic en link
    → GET /api/v1/auth/verify?token=xxx
    → Cuenta activada → Redirect al dashboard
```

### Eliminación de Cuenta
```
[Configuración → Zona de peligro → Eliminar cuenta]
    → Modal de confirmación (escribe "ELIMINAR" para confirmar)
    → POST /api/v1/users/me/delete
    → Job asíncrono: elimina datos, imágenes, cancela suscripción
    → Email de confirmación de eliminación
    → Session invalidada → Redirect a landing
```

---

## Endpoints API

```
POST /api/v1/auth/register             → Registro con email
POST /api/v1/auth/login                → Login con email/contraseña
POST /api/v1/auth/oauth/{provider}     → OAuth2 (google, github)
POST /api/v1/auth/refresh              → Renovar access token
POST /api/v1/auth/logout               → Revocar refresh token
POST /api/v1/auth/verify               → Verificar email
POST /api/v1/auth/magic-link           → Solicitar magic link

GET  /api/v1/users/me                  → Perfil del usuario autenticado
PUT  /api/v1/users/me                  → Actualizar perfil
DEL  /api/v1/users/me                  → Eliminar cuenta
GET  /api/v1/users/me/history          → Historial de generaciones (paginado)
POST /api/v1/users/me/export           → Solicitar exportación de datos
PUT  /api/v1/users/me/notifications    → Actualizar preferencias de notificación
```

---

## Seguridad

- Contraseñas hasheadas con **bcrypt** (cost factor 12).
- Tokens de verificación y magic links de un solo uso con TTL de 1 hora.
- Los refresh tokens tienen rotación: al usarse, se emite uno nuevo e invalida el anterior.
- Reutilización de refresh token (token theft) → invalida todos los tokens de la sesión.
