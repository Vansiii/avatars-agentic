# SOUL — Reglas Inmutables del Sistema de Personajes para Spots de TV

> **Este archivo NO se modifica durante el desarrollo.** Es el contrato de negocio del Alpha.
> Todo agente lo lee al iniciar. Una tarea que exija violar una de estas reglas se rechaza,
> y el conflicto se anota en `MEMORY.md` para que un humano lo resuelva.

---

## 1. Producto

**Qué es:** una plataforma web que genera **personajes hiperrealistas** para spots publicitarios de televisión. Cada personaje se crea una vez y aparece de forma **consistente** en todos los spots de su categoría (deportes, noticias, entretenimiento, etc.).

**Qué NO es en el Alpha:** no hay edición de video manual, ni efectos especiales complejos, ni streaming en vivo, ni app móvil nativa, ni marketplace de personajes de terceros.

**Entrada válida:** imagen JPEG/PNG/WEBP de ≤ 10 MB, **o** un prompt de texto de 10 a 500 caracteres, **o** ambos. Al menos uno de los dos es obligatorio.

**Salida:** imagen hiperrealista del personaje + video del spot (corto 3-5s o largo 15-30s).

**Restricción de estilo:** Solo se genera contenido **hiperrealista**. No hay otros estilos.

---

## 2. Roles del Sistema

| Rol | Quién lo tiene | Qué puede hacer | Qué NO puede hacer |
|-----|----------------|------------------|---------------------|
| **admin** | Administrador del sistema (creado en setup inicial) | Crear/editar/eliminar usuarios, ver métricas, modificar límites de usuarios, gestionar categorías | **NO** puede crear personajes ni generar spots — es rol de gestión, no de contenido |
| **user** | Usuario final (creado por el admin) | Crear/editar personajes, generar spots, ver sus propios límites | No puede gestionar usuarios ni ver métricas de otros |

**No hay registro público.** Solo el admin puede crear cuentas de usuario.

**Límites aplican solo a usuarios `user`.** El admin no tiene límites de generación porque no genera contenido.

**Métricas visibles para admin:**
- Total de usuarios activos/inactivos
- Personajes creados por usuario (esta semana)
- Spots generados por usuario (esta semana)
- Límites de cada usuario (y capacidad de modificarlos)

---

## 3. Límites del Plan Gratuito

| Recurso | Límite |
|---------|--------|
| Crear/editar personajes | **2 por semana** |
| Generar spots (videos) | **5 por semana** |
| Rehacer generación | **3 veces gratis** (no consume crédito) |

**Reglas duras de límites:**
- Al alcanzar el límite semanal de personajes, el endpoint responde **403** con código `CHAR_001`.
- Al alcanzar el límite semanal de spots, el endpoint responde **403** con código `VID_001`.
- Los límites se resetean cada lunes a las 00:00 UTC.
- Un usuario `user` no puede crear roles de `admin`. Nunca se degrada silenciosamente.

---

## 4. Flujo de Creación de Personaje

```
1. Usuario envía entrada (imagen +/o texto)
2. Sistema valida la entrada (formato, tamaño, que la imagen no sea basura)
3. Sistema genera imagen hiperrealista del personaje (3 variaciones)
4. Usuario elige la que más le gusta
5. Si ninguna le gusta → "Rehacer" (gratis, máximo 3 veces)
6. Después de 3 rehacers, DEBE elegir una
7. Solo al ELEGIR se decrementa el contador de personajes de la semana
```

**Consistencia del personaje:** Una vez creado, el personaje tiene una imagen de referencia que se usa en TODAS las generaciones de video posteriores. El mismo personaje se ve igual en todos los spots de su categoría.

---

## 5. Flujo de Generación de Video (Spot)

```
1. Usuario selecciona un personaje existente
2. Usuario selecciona tipo de video (corto 3-5s / largo 15-30s)
3. Usuario describe el spot ("Pepito presentando los resultados del partido")
4. Sistema genera el video usando la imagen de referencia del personaje
5. Sistema genera 3 variaciones del video
6. Usuario elige la que más le gusta
7. Si ninguna le gusta → "Rehacer" (gratis, máximo 3 veces)
8. Solo al ELEGIR se decrementa el contador de spots de la semana
```

---

## 6. Filtro NSFW — NO NEGOCIABLE

**Toda imagen y todo prompt pasa por filtro NSFW automático**, en dos puntos:

1. **Entrada:** la foto subida y el prompt de texto se validan ANTES de generar.
2. **Salida:** la imagen/video generado se valida ANTES de guardar y mostrar al usuario.

Reglas duras:
- Contenido rechazado → **422** con código `GEN_004` y mensaje neutro al usuario. **No** se describe qué se detectó ni se devuelve el contenido ofensivo.
- Un rechazo NSFW **no decrementa el límite semanal**.
- Todo rechazo se registra en un log de seguridad **inmutable** (append-only).
- **El fallo del filtro se resuelve rechazando.** Si el servicio de moderación no responde, la generación se rechaza; nunca se deja pasar contenido sin validar.

---

## 7. Privacidad de los Datos del Usuario

- La **imagen de entrada** subida por el usuario se elimina a las **24 horas**. Es material sensible.
- Se hace **strip de metadatos EXIF** a toda imagen subida (contiene geolocalización).
- El usuario puede eliminar su cuenta; eliminar la cuenta borra personajes, videos e imágenes del almacenamiento.
- **Nunca** se registran en logs: contraseñas, tokens JWT, ni el contenido binario de las imágenes.
- Las claves de API viven **solo** en variables de entorno. Jamás se escriben en el código ni se commitean.

---

## 8. Arquitectura del Alpha (Restricción Técnica Inmutable)

- **Monolito FastAPI.** Un solo servicio, un solo despliegue.
- **PROHIBIDO en el Alpha:** Celery, Redis, RabbitMQ, microservicios, PgBouncer, Kubernetes, service mesh.
- El trabajo asíncrono se hace con **`BackgroundTasks` de FastAPI**. Nada más.
- Stack fijado: **Python 3.11 + FastAPI + SQLAlchemy + PostgreSQL** / **React + TypeScript + Vite**.
- Si una solución "correcta a futuro" exige infraestructura nueva, se elige la simple y se anota la deuda técnica en `MEMORY.md`.

---

## 9. Buenas Prácticas de Código

### 9.1 Leer documentación antes de usar una librería

**ANTES de usar cualquier librería, framework o API, lee su documentación oficial.** No confíes en datos de entrenamiento del modelo — pueden estar desactualizados o ser incorrectos.

- Verifica la versión instalada (`pip list`, `npm list`) y busca la documentación de **esa versión exacta**.
- Si la documentación dice que algo funciona de una manera y el código actual lo hace distinto, **la documentación manda** — adapta el código.
- Si una función tiene parámetros nuevos o cambió su comportamiento, actualiza el código para usar la API correcta.
- No asumas que una librería hace algo sin verificar. Ejemplos:
  - `requests` no maneja retries por defecto — hay que configurarlos.
  - `FastAPI` no valida query params automáticamente sin `Query()`.
  - `SQLAlchemy 2.0` síncrono y async son APIs diferentes.

### 9.2 Formato y estilo

| Lenguaje | Herramienta | Configuración |
|----------|-------------|---------------|
| Python | `ruff` | Formato + lint en uno solo |
| TypeScript | `oxlint` + `prettier` | Ya configurado en el frontend |
| SQL | legible, indentado | Sin estilo automático |

- Ejecuta `ruff format` antes de commit en Python.
- Ejecuta `npm run lint` antes de commit en TypeScript.
- Si el linter corrige algo, revisa el cambio — no lo aceptes ciegamente.

**Código limpio y legible:**

- **Indentación consistente:** 4 espacios en Python, 2 espacios en TypeScript.
- **Líneas máximas:** 88 caracteres (Python con ruff), 100 (TypeScript).
- **Espacios en blanco:** no dejes líneas vacías dobles al final de funciones.
- **Orden de imports:** stdlib → terceros → locales, separados por línea vacía.
- **Nombres descriptivos:** `get_character_by_id()` no `gcbi()`. `user_id` no `uid`.
- **Funciones cortas:** máximo ~30 líneas. Si es más larga, divídela.
- **Archivos organizados:** una función/clase por archivo cuando sea posible.

**Comentarios en español:**

- **Comenta el POR QUÉ, no el QUÉ.** El código ya dice qué hace; comenta por qué lo hace así.
  ```python
  # ❌ MAL: esto es obvio
  user = db.query(User).filter(User.id == user_id).first()

  # ✅ BIEN: explica la razón
  # Se busca el usuario completo porque necesitamos verificar el plan antes de permitir la generación
  user = db.query(User).filter(User.id == user_id).first()
  ```

- **Comenta decisiones de negocio,** no código autoexplicativo.
  ```python
  # ❌ MAL: comentario obvio
  # Incrementamos el contador de uso
  user.credits_used += 1

  # ✅ BIEN: contexto de negocio
  # Solo se decrementa al completar la generación (SOUL.md §3) — nunca en generaciones fallidas
  user.credits_used += 1
  ```

- **Comenta workarounds y hacks** con referencia al bug o issue.
  ```python
  # HACK: Pollinations devuelve 403 en vez de 422 para contenido NSFW
  # (ver issue #123). Tratamos cualquier 4xx como rechazo.
  if response.status_code in (400, 403, 422):
  ```

- **Docstrings en funciones públicas** (endpoints, servicios):
  ```python
  def create_character(name: str, file: UploadFile) -> Character:
      """
      Crea un personaje a partir de una imagen de referencia y/o descripción.
      
      Retorna 3 variaciones de imagen hiperrealista para que el usuario elija.
      El filtro NSFW se aplica a cada variación antes de retornar.
      """
  ```

- **NO comentes:** imports, variables obvias, closing braces, código auto-generado.

### 9.3 Refactoring

- **No refactorees código que funciona** a menos que tengas una razón concreta (bug, feature nueva que lo requiere, o deuda técnica documentada en el backlog).
- **KISS siempre.** Si hay dos formas de hacerlo, elige la más simple.
- **DRY solo cuando tiene sentido.** Tres líneas similares son mejor que una abstracción prematura. Si la abstracción no se usa en 3+ lugares, no la crees.
- **No renombres variables solo por estilo** — eso crea diffs ruidosos que dificultan el review.
- **Un cambio por commit.** No mezcles refactoring con features o bugs.

### 9.4 Tests

- **Mínimo:** un test por endpoint nuevo.
- **Flujos críticos:** test de integración para creación de personajes y generación de spots.
- **Filtro NSFW:** test con imagen que debería rechazarse (fail-closed verificado).
- **No tests = no completado.** Si una tarea no tiene test y es verificable, no la marques `[x]`.
- Ejecuta los tests antes de marcar una tarea como hecha: `pytest` (backend) o `npm test` (frontend).

### 9.5 Manejo de errores

- **Excepciones específicas**, no genéricas. En vez de `except Exception`, usa `except ValidationError`, `except FileNotFoundError`, etc.
- **Fail-closed en seguridad.** Si el filtro NSFW falla, rechaza — no dejes pasar.
- **Mensajes de error al usuario en español**, nombres de variables y funciones en inglés.
- **No silencies errores** con `pass` vacío. Si algo falla, regístralo o levántalo.
- **Logs con contexto:** incluye `request_id`, `user_id`, y qué operación falló.

### 9.6 Seguridad

- **Nunca hardcodees secrets.** TODO va a variables de entorno (`.env`).
- **Valida input en boundaries.** El frontend valida por UX; el backend valida por seguridad — nunca confíes solo en el frontend.
- **Strip de metadatos** en imágenes de usuario (EXIF contiene geolocalización).
- **Rate limiting** por usuario, no solo por IP.
- **JWT con expiración corta** (30 min access, 7 días refresh).

### 9.7 Commits

- **Formato:** `[componente] descripción corta en imperativo`
  - Ejemplo: `[backend]添加 endpoint de personajes`
  - Ejemplo: `[frontend] fix de selección de variación`
  - Ejemplo: `[docs] actualizar SOUL.md con buenas prácticas`
- **Un cambio por commit.** No mezcles features con fixes.
- **No commitees:** `.env`, `node_modules/`, `__pycache__/`, archivos generados.
- **Mensaje claro:** explica el "por qué", no solo el "qué".
