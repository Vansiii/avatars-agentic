# Misión del Sistema — Sistema de Creación de Identidades Visuales Digitales

> **Versión:** 1.0  
> **Fecha:** 2026-07-08  
> **Estado:** Aprobado

---

## Definición del MVP

Desarrollo de una plataforma web de generación de avatares profesionales e identidades visuales digitales impulsada por inteligencia artificial, que permita a cualquier usuario subir una fotografía o escribir una descripción textual, seleccionar un estilo visual y recibir múltiples variaciones de avatar de alta calidad listas para descargar.

El núcleo funcional del MVP abarca:

1. **Ingesta de entrada:** Carga de imagen (JPEG/PNG/WEBP, ≤ 10 MB) o descripción textual libre.
2. **Selección de estilo:** Catálogo de estilos predefinidos (profesional, streamer/gamer, corporativo, redes sociales).
3. **Generación con IA:** Pipeline de generación de imágenes vía modelos de difusión (Stable Diffusion / DALL-E API).
4. **Entrega de resultados:** Presentación de 3–6 variaciones por solicitud.
5. **Descarga básica:** Exportación en resolución estándar (512 × 512 px, formato PNG).
6. **Sistema de autenticación:** Registro/login para persistencia de historial y cuota de uso.

---

## Público Objetivo

| Segmento | Descripción | Caso de Uso Principal |
|---|---|---|
| **Profesionales individuales** | Trabajadores, freelancers y ejecutivos | Avatar para LinkedIn, CV y correo corporativo |
| **Creadores de contenido** | Streamers, youtubers, tiktokers | Branding visual para Twitch, Discord, YouTube |
| **Empresas y startups** | Equipos de RRHH, marketing y comunicación | Avatares corporativos unificados para empleados |
| **Negocios con atención digital** | E-commerce, SaaS, servicios en línea | Asistentes virtuales con imagen personalizada |
| **Desarrolladores de videojuegos** | Estudios indie y AAA | Creación de personajes y NPCs estilizados |
| **Instituciones educativas** | Universidades y plataformas e-learning | Avatares para docentes y alumnos virtuales |
| **Agencias de marketing** | Agencias creativas y de publicidad | Mascotas, personajes y material publicitario |

---

## Exclusiones de Alcance (Out of Scope)

El sistema **no** abarca en esta versión inicial:

- **Procesamiento de video:** No se generan avatares animados ni clips de video en el MVP; queda reservado para la versión 2.0.
- **Edición avanzada en tiempo real:** No hay un editor de imagen tipo Photoshop integrado; las modificaciones se solicitan mediante nuevas generaciones.
- **Integración con hardware:** El sistema no gestiona cámaras, dispositivos de captura ni periféricos físicos.
- **Pasarela de pagos directa en MVP:** La monetización (suscripción) se habilitará en la fase post-MVP; el MVP usa créditos gratuitos con límite de uso.
- **Generación de audio/voz:** El sistema no produce voces ni sonidos para los avatares.
- **Moderación humana activa:** El MVP usa filtros automáticos de contenido; no existe equipo de revisión manual en tiempo real.
- **Aplicación móvil nativa:** El producto inicial es exclusivamente una Progressive Web App (PWA); las apps iOS/Android son roadmap futuro.
- **Almacenamiento ilimitado:** El historial de avatares generados se retiene 30 días para cuentas gratuitas.

---

## Arquitectura de Datos

### Artefactos Principales

| Artefacto | Tipo | Descripción |
|---|---|---|
| `user_image_input` | Blob (JPEG/PNG/WEBP) | Foto original subida por el usuario |
| `generation_request` | JSON | Parámetros de la solicitud: estilo, prompt, seed, modelo |
| `avatar_output` | Blob (PNG, 512 × 512 / 1024 × 1024) | Imagen generada por el modelo de IA |
| `user_profile` | JSON/DB Row | Datos de cuenta, plan activo, cuota de créditos |
| `generation_history` | JSON/DB Table | Log de todas las generaciones por usuario |
| `style_catalog` | JSON | Catálogo de estilos disponibles con prompts base |
| `subscription_record` | JSON/DB Row | Estado de suscripción, fechas, plan seleccionado |

### Flujo de Datos (Resumen)

```
[Usuario] → (Foto / Descripción)
         → [API Gateway]
         → [Servicio de Generación IA]
              → [Modelo de Difusión / LLM]
              → [Post-procesado & Upscaling]
         → [Storage de Blobs (S3 / GCS)]
         → [CDN]
         → [Usuario] ← (Avatares generados)
```

---

## Propuesta de Valor Central

> *"Convierte cualquier foto o idea en una identidad visual profesional en menos de 60 segundos, impulsada por inteligencia artificial."*

La plataforma democratiza el acceso a identidades visuales de alta calidad que antes requerían diseñadores gráficos profesionales, entregando resultados consistentes, personalizables y listos para usar en cualquier plataforma digital.
