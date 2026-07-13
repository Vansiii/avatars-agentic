# Feature Spec: Catálogo de Estilos de Avatar

> **ID:** FEAT-003  
> **Versión:** 1.0  
> **Estado:** En desarrollo  
> **Prioridad:** P0 — Crítico (Core MVP)

---

## Descripción

Gestión y presentación del catálogo de estilos de avatar disponibles en la plataforma. Cada estilo define la estética visual del avatar generado y está asociado a uno o más casos de uso del negocio.

---

## Categorías de Estilo

| Categoría | Slug | Descripción | Plan mínimo |
|---|---|---|---|
| **Profesional** | `professional` | Retratos formales para LinkedIn y CV | Free |
| **Gaming / Streamer** | `gaming` | Estilos para Twitch, Discord, YouTube | Free |
| **Corporativo** | `corporate` | Avatares con identidad empresarial | Pro |
| **Redes Sociales** | `social` | Optimizados para Instagram, TikTok, X | Free |
| **Educación** | `education` | Personajes para cursos virtuales | Free |
| **Marketing / Mascota** | `marketing` | Personajes de marca y mascotas | Pro |
| **Videojuegos** | `gaming-character` | Personajes con lore (fantasía, sci-fi) | Pro |
| **Atención al Cliente** | `customer-service` | Asistentes virtuales para webs y apps | Enterprise |

---

## Estilos Iniciales del Catálogo (MVP)

### Categoría: Profesional
| Nombre | Descripción |
|---|---|
| Corporate Portrait | Fondo neutro, iluminación profesional, traje de negocios |
| Creative Professional | Estilo artístico moderno, look casual-elegante |
| Executive Headshot | Pose formal, fondo degradado oscuro, ultra-realista |

### Categoría: Gaming / Streamer
| Nombre | Descripción |
|---|---|
| Neon Gamer | Estilo cyberpunk, luces de neón, fondo oscuro |
| Anime Streamer | Arte estilo anime japonés, colores vibrantes |
| Pixel Art | Retrato en arte pixelado retro |
| Cartoon Avatar | Estilo cartoon occidental, proporciones exageradas |

### Categoría: Redes Sociales
| Nombre | Descripción |
|---|---|
| Gradient Pop | Fondo con gradiente de colores, moderno y llamativo |
| Minimalist | Fondo blanco/negro, minimalismo extremo |
| Sticker Style | Estilo pegatina con borde blanco grueso |

### Categoría: Videojuegos (Pro)
| Nombre | Descripción |
|---|---|
| Fantasy Hero | Héroe de RPG, armadura medieval, iluminación épica |
| Sci-Fi Soldier | Personaje futurista, traje de combate espacial |
| Dark Fantasy | Estilo oscuro, tenebroso, inspiración Dark Souls |

---

## Criterios de Aceptación

### AC-001: Visualización del catálogo
- [ ] Los estilos se muestran en una galería tipo grid (2 cols mobile, 3 cols tablet, 4 cols desktop).
- [ ] Cada estilo tiene: imagen de preview, nombre, categoría y badge de plan requerido.
- [ ] El usuario puede filtrar por categoría mediante tabs o chips.
- [ ] Los estilos de plan superior están visibles pero con overlay de bloqueo.

### AC-002: Gestión de estilos (Admin)
- [ ] Los administradores pueden crear, editar y desactivar estilos desde el panel admin.
- [ ] Al desactivar un estilo, desaparece del catálogo público pero no afecta generaciones previas.
- [ ] El `base_prompt` de cada estilo está encriptado en la base de datos.
- [ ] Se puede definir el tier mínimo requerido por estilo.

### AC-003: Preview de estilo
- [ ] Al hacer hover/tap en un estilo, se muestra una animación de zoom con detalle.
- [ ] Existe un botón "Ver ejemplo" que abre un modal con 3 ejemplos de avatares generados con ese estilo.

---

## Modelo de Datos

```json
{
  "id": "uuid",
  "name": "Corporate Portrait",
  "slug": "corporate-portrait",
  "category": "professional",
  "description": "Fondo neutro, iluminación profesional...",
  "preview_url": "https://cdn.example.com/styles/corporate-portrait.jpg",
  "example_urls": ["...", "...", "..."],
  "base_prompt": "<encriptado>",
  "is_active": true,
  "tier_required": "free",
  "tags": ["linkedin", "cv", "formal"],
  "sort_order": 1
}
```

---

## Endpoints API

```
GET  /api/v1/styles                  → Lista de estilos activos (filtros: category, tier)
GET  /api/v1/styles/{slug}           → Detalle de un estilo específico
POST /api/v1/admin/styles            → Crear nuevo estilo (ADMIN+)
PUT  /api/v1/admin/styles/{id}       → Actualizar estilo (ADMIN+)
DEL  /api/v1/admin/styles/{id}       → Desactivar estilo (ADMIN+)
```
