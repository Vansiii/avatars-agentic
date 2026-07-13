# Feature Spec: Avatares Corporativos para Empresas

> **ID:** FEAT-005  
> **Versión:** 1.0  
> **Estado:** Planificado  
> **Prioridad:** P1 — Alta (Ciclo 2)

---

## Descripción

Módulo B2B que permite a una empresa crear una "organización" dentro de la plataforma, definir su identidad visual corporativa y generar avatares estandarizados para todos sus empleados, manteniendo coherencia de marca.

---

## Propuesta de Valor

Las empresas pueden:
- Crear avatares para todos sus empleados con un estilo y paleta de colores unificados.
- Gestionar los avatares de su equipo desde un panel centralizado.
- Asignar créditos de generación a cada empleado de forma controlada.
- Descargar todos los avatares en un ZIP listo para usar en el directorio corporativo.

---

## Criterios de Aceptación

### AC-001: Creación de Organización
- [ ] Un usuario Enterprise puede crear una organización con: nombre, logo, paleta de colores (hex) y dominio de email corporativo.
- [ ] El dominio corporativo se valida para auto-aprobar nuevos miembros con ese email.
- [ ] La organización tiene un plan de créditos compartido gestionado por el admin.

### AC-002: Gestión de Miembros
- [ ] El admin puede invitar miembros por email o compartir un link de invitación.
- [ ] Los miembros invitados acceden a la plataforma con su cuenta (o crean una nueva).
- [ ] El admin puede asignar roles: `org_admin`, `org_member`.
- [ ] El admin puede ver el historial de generaciones de cada miembro.
- [ ] El admin puede revocar el acceso de un miembro.

### AC-003: Estilo Corporativo Personalizado
- [ ] El admin puede definir un "estilo corporativo" basado en:
  - Logo de la empresa (se integra en el avatar como elemento decorativo).
  - Paleta de colores primaria y secundaria.
  - Fondos predefinidos con los colores corporativos.
  - Tipografía del nombre (si se incluye texto en el avatar).
- [ ] Los miembros solo pueden usar el estilo corporativo de su organización (por defecto) o los estilos públicos del plan.

### AC-004: Generación Masiva
- [ ] El admin puede importar una lista de empleados (CSV: nombre, email, cargo, foto_url).
- [ ] El sistema genera avatares para todos los empleados de la lista de forma asíncrona.
- [ ] El admin recibe una notificación cuando el batch completa.
- [ ] El admin puede descargar todos los avatares generados en un archivo ZIP organizado por empleado.

### AC-005: Panel de Organización
- [ ] Panel `/org/dashboard` con: total de miembros, créditos usados/disponibles, generaciones recientes.
- [ ] Tabla de miembros con: nombre, email, cantidad de generaciones, último acceso.
- [ ] Gráfico de uso de créditos mensual.

---

## Modelo de Datos Adicional

```sql
organizations
  id              UUID PK
  name            VARCHAR(200)
  slug            VARCHAR(200) UNIQUE
  logo_url        TEXT
  primary_color   VARCHAR(7)   -- hex
  secondary_color VARCHAR(7)
  domain          VARCHAR(255) -- para auto-aprobación
  owner_id        UUID FK → users.id
  plan            ENUM('enterprise')
  created_at      TIMESTAMP

org_members
  id              UUID PK
  org_id          UUID FK → organizations.id
  user_id         UUID FK → users.id
  role            ENUM('org_admin','org_member')
  credits_allocated INTEGER DEFAULT 10
  credits_used    INTEGER DEFAULT 0
  joined_at       TIMESTAMP

org_styles
  id              UUID PK
  org_id          UUID FK → organizations.id
  name            VARCHAR(100)
  base_prompt     TEXT  -- prompt con identidad corporativa
  is_default      BOOLEAN DEFAULT FALSE
  created_at      TIMESTAMP
```

---

## Endpoints API

```
POST /api/v1/organizations                      → Crear organización (Enterprise)
GET  /api/v1/organizations/{id}                 → Detalle de organización
PUT  /api/v1/organizations/{id}                 → Actualizar branding corporativo
POST /api/v1/organizations/{id}/invite          → Invitar miembro
GET  /api/v1/organizations/{id}/members         → Lista de miembros
DEL  /api/v1/organizations/{id}/members/{uid}   → Revocar acceso
POST /api/v1/organizations/{id}/batch-generate  → Generar avatares masivos (CSV upload)
GET  /api/v1/organizations/{id}/export          → Descargar ZIP de todos los avatares
```

---

## Dependencias

- **FEAT-004:** Gestión de Usuarios (roles y autenticación)
- **FEAT-003:** Catálogo de Estilos (extensión para estilos corporativos)
- **FEAT-001:** Generación de Avatares (pipeline base)
- Infraestructura de batch processing para generación masiva

---

## Métricas de Éxito

| Métrica | Target |
|---|---|
| Empresas que generan avatares para todo su equipo en 1 sesión | > 60% |
| NPS de usuarios Enterprise | > 50 |
| Tiempo promedio para generar 20 avatares corporativos | < 15 minutos |
