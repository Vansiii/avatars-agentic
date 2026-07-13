# Feature Spec: Sistema de Suscripciones y Créditos

> **ID:** FEAT-002  
> **Versión:** 1.0  
> **Estado:** Planificado  
> **Prioridad:** P1 — Alta (Post-MVP inmediato)

---

## Descripción

Sistema de monetización basado en suscripción mensual con tres niveles de plan. Cada plan otorga una cuota de créditos mensuales para generaciones de avatar, acceso a estilos premium y resoluciones de descarga más altas.

---

## Planes y Pricing

| Plan | Precio/mes | Créditos/mes | Resolución máx. | Estilos premium | Soporte |
|---|---|---|---|---|---|
| **Free** | $0 | 5 | 512×512 px | ❌ | Comunidad |
| **Pro** | $9.99 | 100 | 1024×1024 px | ✅ | Email (48h) |
| **Enterprise** | $49.99 | Ilimitado | 2048×2048 px | ✅ + exclusivos | Dedicado |

### Reglas de Créditos
- Los créditos se renuevan el día 1 de cada mes (no se acumulan).
- Un crédito = 1 solicitud de generación (sin importar el número de variaciones).
- Los créditos no utilizados caducan al finalizar el período.
- Se pueden comprar créditos adicionales: $1.99 por 10 créditos (usuarios free/pro).

---

## Criterios de Aceptación

### AC-001: Visualización de planes
- [ ] La página `/app/billing` muestra los 3 planes con comparativa clara.
- [ ] El plan actual del usuario está resaltado visualmente.
- [ ] El precio mensual y anual (con descuento del 20%) se pueden alternar.

### AC-002: Suscripción
- [ ] El flujo de pago redirige a Stripe Checkout.
- [ ] Al completar el pago, el plan se activa en menos de 30 segundos.
- [ ] El usuario recibe un email de confirmación con factura.
- [ ] Si el pago falla, el usuario ve un mensaje claro y puede reintentar.

### AC-003: Gestión de suscripción activa
- [ ] El usuario puede ver la fecha del próximo cobro y el monto.
- [ ] El usuario puede cancelar la suscripción (permanece activa hasta fin del período).
- [ ] El usuario puede actualizar (upgrade/downgrade) el plan en cualquier momento.
- [ ] Un downgrade aplica al próximo ciclo; un upgrade aplica inmediatamente (prorrateado).

### AC-004: Webhooks de Stripe
- [ ] El sistema procesa `invoice.payment_succeeded` → activa/renueva plan.
- [ ] El sistema procesa `invoice.payment_failed` → notifica al usuario y pone plan en `past_due`.
- [ ] El sistema procesa `customer.subscription.deleted` → revierte a plan free.

### AC-005: Cuota y límites
- [ ] El sistema muestra el saldo de créditos disponibles en el dashboard.
- [ ] Al llegar a 0 créditos, el botón de generar se deshabilita con un CTA de upgrade.
- [ ] El usuario recibe un email al llegar al 20% de créditos restantes.

---

## Flujo de Suscripción

```
[Dashboard] → [Ver planes] → [Seleccionar Pro/Enterprise]
    ↓
[Stripe Checkout] → [Ingresa datos de tarjeta]
    ↓
[Stripe Webhook: payment_succeeded]
    ↓
[Backend: actualiza plan en DB + restablece créditos]
    ↓
[Email de bienvenida al plan] → [Redirect a dashboard]
```

---

## Endpoints API

```
GET  /api/v1/billing/plans          → Lista de planes disponibles
GET  /api/v1/billing/subscription   → Suscripción activa del usuario
POST /api/v1/billing/checkout       → Crea sesión de Stripe Checkout
POST /api/v1/billing/portal         → Acceso al portal de cliente de Stripe
POST /api/v1/billing/webhook        → Receptor de webhooks de Stripe (público, firmado)
POST /api/v1/billing/credits/buy    → Compra de créditos adicionales
```

---

## Dependencias

- Integración con **Stripe** (Checkout + Webhooks + Customer Portal)
- **FEAT-004:** Gestión de Usuarios (relación user → subscription)
- Sistema de email transaccional (SendGrid / Resend)
