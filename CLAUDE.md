# Instrucciones del Proyecto — Sistema de Personajes para Spots de TV

> Este archivo es leído automáticamente por MiMoCode al iniciar una sesión en este proyecto.
> Contiene las instrucciones base para cualquier modelo de IA que trabaje aquí.

---

## Antes de hacer CUALQUIER cosa

Lee estos archivos, en este orden:

1. `.agents/memory/SOUL.md` — Reglas inmutables del sistema. NUNCA las violes.
2. `.agents/memory/HEARTBEAT.md` — Estado actual del proyecto.
3. `.agents/steering/backlog.md` — Qué tareas están pendientes.
4. `.agents/AGENTS.md` — Protocolo de trabajo y roles.

---

## Qué es este proyecto

Sistema de Personajes para Spots Publicitarios de TV. Genera personajes hiperrealistas que aparecen de forma consistente en múltiples videos publicitarios.

---

## Roles del sistema

- **Admin**: Crea usuarios, gestiona el sistema.
- **User**: Crea personajes, genera spots.

No hay registro público. Solo el admin crea cuentas.

---

## Protocolo de memoria (OBLIGATORIO)

### Al INICIAR cualquier tarea:
- Lee SOUL.md, HEARTBEAT.md y backlog.md.

### Al TERMINAR cualquier tarea:
- Actualiza HEARTBEAT.md (sobrescribe).
- Añade entrada a MEMORY.md (append-only).
- Marca la tarea en backlog.md: `[ ]` → `[x]`.

---

## Reglas duras

1. **Monolito FastAPI.** PROHIBIDO: Celery, Redis, microservicios, Kubernetes.
2. **Filtro NSFW obligatorio.** Entrada y salida. Fail-closed.
3. **Consistencia del personaje.** Usar imagen de referencia en cada generación.
4. **Límites semanales:** 2 personajes, 5 spots por usuario.
5. **Rehacer:** 3 veces gratis, no decrementa límite hasta selección final.
6. **Verificar ejecutando**, no leyendo. "Compila" no es verificación.
7. **Leer documentación oficial** antes de usar cualquier librería. No confíes en datos de entrenamiento.
8. **Buenas prácticas:** formato con ruff/oxlint, tests por endpoint, commits descriptivos.

---

## Stack tecnológico

- Backend: Python 3.11 + FastAPI + SQLAlchemy + PostgreSQL
- Frontend: React 19 + TypeScript + Vite + Zustand + React Query
- Auth: JWT (HS256)
- IA: Pollinations.ai (testing gratuito)

---

## Archivos clave

| Archivo | Qué contiene |
|---------|--------------|
| `.agents/memory/SOUL.md` | Reglas inmutables de negocio |
| `.agents/AGENTS.md` | Protocolo del orquestador |
| `.agents/skills/frontend.md` | Skill del agente Frontend |
| `.agents/skills/backend.md` | Skill del agente Backend |
| `.agents/skills/ai.md` | Skill del agente IA |
| `.agents/steering/backlog.md` | Tareas pendientes |
| `spec/sdd.md` | Arquitectura técnica |
| `spec/roadmap.md` | Fases de implementación |

---

## No hagas esto

- No marques tareas como hechas sin ejecutarlas.
- No inventes estado — verifica contra el código.
- No cruces fronteras de dominio sin coordinación.
- No añadas infraestructura que SOUL.md prohíbe.
- No comentes código obvio — comenta el POR QUÉ, no el QUÉ.
- No escribas código sin comentarios en español que expliquen decisiones de negocio.

---

## Requisitos previos (antes de empezar a programar)

El humano debe completar estos pasos ANTES de que los agentes trabajen:

### Configuración del entorno
- [ ] Python 3.11 instalado
- [ ] Node.js instalado
- [ ] PostgreSQL disponible (local o Supabase)
- [ ] Git configurado con usuario y email

### Cuentas y API keys (el agente PREGUNTA antes de crear)
- [ ] Crear cuenta en GitHub y repositorio del proyecto
- [ ] Crear cuenta en Supabase (o tener PostgreSQL local)
- [ ] Obtener API key de Pollinations.ai (si se necesita para testing)

### Archivos de configuración (el agente crea, el humano provee valores)
- [ ] `avatars-backend/.env` con:
  - `DATABASE_URL=postgresql://...`
  - `SECRET_KEY=<valor secreto>`
  - `POLLINATIONS_API_KEY=<si aplica>`

---

## Acciones delicadas — PREGUNTAR antes de ejecutar

Estas acciones **requieren confirmación del humano** antes de ejecutarlas. El agente debe preguntar si quiere automatizarlas o hacerlas manualmente:

| Acción | Por qué es delicada |
|--------|---------------------|
| `git init` | Inicializa un repositorio nuevo |
| `git add .` y `git commit` | Crea un commit con todos los cambios |
| `git push` | Envía código a GitHub (requiere credenciales) |
| `pip install` | Instala dependencias Python |
| `npm install` | Instala dependencias Node |
| `uvicorn` / `npm run dev` | Levanta servidores |
| Crear archivo `.env` | Contiene secrets |
| Ejecutar migraciones de DB | Modifica la base de datos |
| `CREATE TABLE` o `create_all()` | Crea tablas en la DB |

**Protocolo:** antes de ejecutar cualquiera de estas acciones, pregunta:
> "¿Quieres que ejecute `[comando]` o prefieres hacerlo manualmente?"

Si el humano dice "automatízalo", ejecuta el comando. Si dice "lo hago yo", espera a que termine.
