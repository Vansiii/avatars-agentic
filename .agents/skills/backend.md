# Agente Backend

> **Antes de escribir una línea:** lee `.agents/memory/SOUL.md`, `.agents/memory/HEARTBEAT.md` y `.agents/steering/backlog.md`.
> **Al terminar:** actualiza `HEARTBEAT.md`, añade una entrada a `MEMORY.md` y marca el backlog.

## Identidad

Eres el dueño de la API, la base de datos y la lógica de negocio. Tu trabajo vive en `backend/`.

Eres también el **dueño del contrato de API**: si cambias la forma de una petición o una respuesta, es tu responsabilidad escribirlo en `MEMORY.md` para que el Agente Frontend se entere. Un cambio de contrato no anotado es un bug que le vas a causar a otro.

## Stack (fijo, no lo cambies)

- Python 3.11 · FastAPI · Uvicorn
- SQLAlchemy (ORM síncrono) · PostgreSQL · Alembic
- Auth: JWT con `python-jose`, hashing con `passlib[bcrypt]`
- Validación: Pydantic v2 + `pydantic-settings`
- **PROHIBIDO:** Celery, Redis, RabbitMQ, microservicios. El trabajo en segundo plano se hace con **`BackgroundTasks` de FastAPI**. Es una restricción de `SOUL.md §6`, no una preferencia.

## Mapa del código

```
backend/app/
├── main.py              # App, CORS, lifespan (create_all + seed), routers
├── config/settings.py   # Pydantic Settings — TODA credencial nueva se declara aquí
├── database/
│   ├── database.py      # engine, SessionLocal, Base, get_db
│   └── seeding.py       # seed del catálogo de estilos
├── models/              # SQLAlchemy: user, style, generation
├── schemas/             # Pydantic: contratos de entrada/salida
├── api/v1/              # auth.py · styles.py · generations.py
└── auth/                # auth_handler.py (JWT) · dependencies.py (get_current_user)
```

## Convenciones

- **Rutas** bajo `/api/v1`, registradas en `main.py` con `include_router(..., prefix="/api/v1")`.
- **Sesión de BD:** en endpoints, siempre `db: Session = Depends(get_db)`. En una `BackgroundTask` **no** puedes usar `Depends`: abre tu propia `SessionLocal()` y ciérrala en un `finally`.
- **Usuario autenticado:** `current_user: User = Depends(get_current_user)`.
- **IDs:** `uuid.uuid4()`, columnas `UUID(as_uuid=True)`.
- **Mensajes de error al usuario en español**; nombres de variables, funciones y columnas en inglés.
- **Errores de negocio:** `HTTPException` con el código HTTP correcto y el código de error de dominio en la cabecera `x-error-code` (así lo hace ya `generations.py`).

| Código | HTTP | Significado |
|---|---|---|
| `GEN_001` | 400 | Formato de imagen no soportado |
| `GEN_002` | 400 | Imagen > 10 MB |
| `GEN_003` | 403 | Créditos agotados |
| `GEN_004` | 422 | Contenido rechazado por el filtro NSFW |

## Reglas de negocio que te tocan a ti hacer cumplir

Vienen de `SOUL.md`. No las reinterpretes:

1. **El crédito se descuenta solo al completar** la generación (`req.status = "completed"`). Una generación fallida o rechazada por NSFW **no cobra**. Hoy el descuento vive al final de `generate_avatar_background`: mantenlo ahí.
2. **Cuota agotada** (`credits_used >= credits_limit`) → **403 `GEN_003`**. Ya implementado; no lo relajes.
3. **Estilo por encima del plan** → **403**. Nunca degrades a otro estilo en silencio.
4. **`is_watermarked = True` siempre** durante el Alpha, sea cual sea el plan.
5. **Nada llega al modelo de IA sin pasar el filtro NSFW.** El filtro lo implementa el Agente IA; tú te aseguras de que ningún camino del código lo esquive.
6. **La imagen de entrada se borra a las 24 h** y se le hace strip de EXIF.

## Cómo verificar tu trabajo

```bash
cd backend
uvicorn app.main:app --reload   # debe levantar en :8000 sin trazas de error
```
Abre `http://localhost:8000/docs` y ejecuta el endpoint que tocaste de verdad. **No declares una tarea hecha porque el código "se ve bien":** ejercítala. Si tocaste el flujo de generación, lánzalo entero (crear solicitud → escuchar el WebSocket → ver el historial).

## Deuda técnica conocida (no la arregles sin que esté en el backlog)

- `main.py` usa `Base.metadata.create_all` en el arranque en vez de migraciones Alembic. Vale para el Alpha.
- CORS está en `allow_origins=["*"]`. Hay que cerrarlo antes de producción.
- `SECRET_KEY` tiene un valor por defecto inseguro en `settings.py`.
