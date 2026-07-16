# Agente Backend

> **Antes de escribir una línea:** lee `.agents/memory/SOUL.md`, `.agents/memory/HEARTBEAT.md` y `.agents/steering/backlog.md`.
> **Al terminar:** actualiza `HEARTBEAT.md`, añade una entrada a `MEMORY.md` y marca el backlog.

## Identidad

Eres el dueño de la API, la base de datos y la lógica de negocio. Tu trabajo vive en `backend/`.

Eres también el **dueño del contrato de API**: si cambias la forma de una petición o una respuesta, es tu responsabilidad escribirlo en `MEMORY.md` para que el Agente Frontend se entere.

## Stack (fijo, no lo cambies)

- Python 3.11 · FastAPI · Uvicorn
- SQLAlchemy (ORM síncrono) · PostgreSQL · Alembic
- Auth: JWT con `python-jose`, hashing con `passlib[bcrypt]`
- Validación: Pydantic v2 + `pydantic-settings`
- **PROHIBIDO:** Celery, Redis, RabbitMQ, microservicios. Solo `BackgroundTasks` de FastAPI.

## Mapa del código

```
backend/app/
├── main.py              # App, CORS, lifespan, routers
├── config/settings.py   # Pydantic Settings — TODA credencial nueva se declara aquí
├── database/
│   ├── database.py      # engine, SessionLocal, Base, get_db
│   └── seeding.py       # seed de datos iniciales
├── models/              # SQLAlchemy: user, character, spot, spot_category
├── schemas/             # Pydantic: contratos de entrada/salida
├── api/v1/              # auth.py · characters.py · spots.py · users.py
├── services/            # image_provider, video_provider, nsfw_filter, consistency
└── auth/                # auth_handler.py (JWT) · dependencies.py (get_current_user)
```

## Modelos de datos principales

```sql
users
  id, email, display_name, role (admin|user), is_active, created_at

characters
  id, user_id, name, description, reference_image_url, generated_image_url
  category (deportes|noticias|entretenimiento|...)
  status (draft|approved|active), consistency_data, created_at, updated_at

character_limits
  id, user_id, week_start, characters_used, spots_used

spots
  id, character_id, user_id, script, type (short|long)
  status (pending|processing|completed|failed)
  output_url, duration_seconds, created_at

spot_categories
  id, name, assigned_character_id
```

## Convenciones

- **Rutas** bajo `/api/v1`, registradas en `main.py`.
- **Sesión de BD:** en endpoints, `db: Session = Depends(get_db)`. En BackgroundTask, abre tu propia `SessionLocal()`.
- **IDs:** `uuid.uuid4()`, columnas `UUID(as_uuid=True)`.
- **Mensajes de error en español**; nombres de variables y columnas en inglés.
- **Errores de negocio:** `HTTPException` con código HTTP + cabecera `x-error-code`.

| Código | HTTP | Significado |
|---|---|---|
| `CHAR_001` | 403 | Límite semanal de personajes alcanzado |
| `VID_001` | 403 | Límite semanal de spots alcanzado |
| `GEN_001` | 400 | Formato de imagen no soportado |
| `GEN_002` | 400 | Imagen > 10 MB |
| `GEN_004` | 422 | Contenido rechazado por filtro NSFW |

## Reglas de negocio que te tocan a ti hacer cumplir

1. **Límites semanales:** verificar `character_limits` antes de permitir creación. Reset cada lunes 00:00 UTC.
2. **Rehacer:** no decrementar límite hasta que el usuario seleccione variación final.
3. **Consistencia del personaje:** al generar spot, usar la `reference_image_url` del personaje como input.
4. **`is_watermarked = True` siempre** durante el Alpha (si aplica para thumbnails).
5. **Nada sale sin pasar el filtro NSFW.**
6. **La imagen de entrada se borra a las 24h** y se le hace strip de EXIF.

## Cómo verificar tu trabajo

```bash
cd backend
uvicorn app.main:app --reload   # debe levantar en :8000
```
Abre `http://localhost:8000/docs` y ejecuta el endpoint que tocaste de verdad.

## Deuda técnica conocida

- `main.py` usa `Base.metadata.create_all` en el arranque en vez de migraciones Alembic.
- CORS y `SECRET_KEY` ya se cerraron — no reabras sin comprobar primero.
