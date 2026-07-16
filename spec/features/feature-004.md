# Feature Spec: Gestión de Usuarios

> **ID:** FEAT-004  
> **Versión:** 2.0  
> **Estado:** Planificado  
> **Prioridad:** P0 — Crítico  
> **Fase:** 001

---

## Descripción

Gestión de usuarios del sistema. Solo el admin puede crear cuentas. No hay registro público.

---

## Criterios de Aceptación

### AC-001: Login
- [ ] El usuario puede loguearse con email y contraseña
- [ ] Recibe JWT token (access + refresh)
- [ ] Token expira en 30 minutos

### AC-002: Crear usuario (solo admin)
- [ ] Solo el admin puede crear nuevos usuarios
- [ ] El admin define: email, nombre, rol (user)
- [ ] Se genera contraseña temporal que el usuario debe cambiar

### AC-003: Listar usuarios (solo admin)
- [ ] El admin ve todos los usuarios del sistema
- [ ] Puede activar/desactivar usuarios

### AC-004: Eliminar usuario (solo admin)
- [ ] Eliminar usuario elimina todos sus personajes y spots
- [ ] Confirmación antes de eliminar

### AC-005: Perfil de usuario
- [ ] El usuario puede ver su perfil
- [ ] Puede cambiar su contraseña
- [ ] Ve sus límites de uso semanales

---

## Especificaciones Técnicas

### Endpoints

```
POST   /api/v1/auth/login            → Login
POST   /api/v1/auth/refresh          → Refresh token

POST   /api/v1/users                 → Crear usuario (solo admin)
GET    /api/v1/users                 → Listar usuarios (solo admin)
PUT    /api/v1/users/{id}            → Actualizar usuario (solo admin)
DELETE /api/v1/users/{id}            → Eliminar usuario (solo admin)

GET    /api/v1/users/me              → Mi perfil
PUT    /api/v1/users/me/password     → Cambiar contraseña
GET    /api/v1/users/me/limits       → Mis límites semanales
```

---

## Modelo de Datos

```sql
users
  id            UUID PK
  email         VARCHAR(255) UNIQUE NOT NULL
  display_name  VARCHAR(100)
  password_hash VARCHAR(255) NOT NULL
  role          ENUM('admin', 'user') DEFAULT 'user'
  is_active     BOOLEAN DEFAULT TRUE
  created_at    TIMESTAMP
  updated_at    TIMESTAMP
```
