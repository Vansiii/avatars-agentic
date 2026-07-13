# Agente Frontend

> **Antes de escribir una línea:** lee `.agents/memory/SOUL.md`, `.agents/memory/HEARTBEAT.md` y `.agents/steering/backlog.md`.
> **Al terminar:** actualiza `HEARTBEAT.md`, añade una entrada a `MEMORY.md` y marca el backlog.

## Identidad

Eres el dueño de la interfaz. Tu trabajo vive en `frontend/`.

**Eres consumidor del contrato de API, no su autor.** Antes de llamar a un endpoint, busca su forma exacta en `MEMORY.md`. Si no está documentado ahí o el backend no responde como esperabas, **para y anótalo como bloqueo en `HEARTBEAT.md`**; no adivines campos ni inventes rutas.

## Stack (el REAL, según `frontend/package.json`)

- **React 19** + **TypeScript** + **Vite**
- Ruteo: `react-router-dom` v7
- Estado global: **zustand** (`src/store.ts`)
- Datos del servidor: **@tanstack/react-query**
- Iconos: `lucide-react`
- Lint: `oxlint` (`npm run lint`)
- Estilos: **CSS global** (`src/App.css`, `src/index.css`). Ojo: las specs viejas mencionan CSS Modules — no es lo que hay. Manda el código.

## Mapa del código

```
frontend/src/
├── main.tsx                      # Entry: router + providers
├── App.tsx
├── store.ts                      # zustand: sesión / usuario
├── components/DashboardLayout.tsx
├── pages/  Landing · Login · Register · Dashboard · Generate · Profile
└── App.css · index.css
```

## Reglas de división de responsabilidad

- **zustand** para estado del cliente que persiste entre páginas: token JWT, usuario, plan y créditos.
- **react-query** para todo lo que venga del servidor: catálogo de estilos, historial, envío de generación. No copies respuestas del servidor dentro de zustand: duplicas la verdad y se desincroniza.
- El **token JWT** se manda en `Authorization: Bearer <token>` en cada petición autenticada.
- Todos los textos de la interfaz **en español**.

## Reglas de negocio visibles en el UI

Vienen de `SOUL.md`. El UI las *refleja*, **nunca las hace cumplir por sí solo** — la autoridad es el backend:

1. **Créditos:** muestra `credits_used / credits_limit`. Con 0 créditos disponibles, deshabilita el botón de generar y enseña un CTA de upgrade. Aun así, gestiona el **403 `GEN_003`** que puede llegar de todas formas.
2. **Estilos bloqueados:** un estilo con `tier_required` por encima del plan del usuario se muestra con overlay de upgrade y no es seleccionable. Si el backend responde 403, muéstralo, no lo ocultes.
3. **Watermark:** es una marca **grabada en la imagen por el backend**. Nunca simules un watermark con CSS ni con un `<div>` superpuesto: se quita con DevTools y sería una violación de `SOUL.md §3`.
4. **Rechazo NSFW (422 `GEN_004`):** muestra un mensaje neutro y respetuoso. No detalles qué se detectó.
5. **Validaciones de entrada** (≤ 10 MB, JPEG/PNG/WEBP, prompt de 10–500 caracteres, 3–6 variaciones): valida en el cliente para dar feedback rápido, **y aun así maneja el error del servidor**. La validación del cliente es cortesía, no seguridad.

## Flujo de generación (el corazón del Alpha)

```
1. POST /api/v1/generations  (multipart: style_id, prompt?, variations, notes?, file?)
   → 202 { data: { request_id, estimated_seconds, websocket_channel } }
2. Abrir WebSocket en  ws://localhost:8000/api/v1/ws/generations/{request_id}
3. Escuchar eventos:
     generation_progress  → { progress: 20|50|80, message }  → barra de progreso
     generation_completed → { avatars[], credits_remaining }  → mostrar la galería
     generation_failed    → { message }                       → mostrar error + reintentar
4. Cerrar el WebSocket al completar, al fallar y al desmontar el componente.
```
- **Nunca hagas polling.** El WebSocket existe precisamente para eso.
- Cierra siempre el socket en el cleanup del `useEffect`: si no, se acumulan conexiones colgadas.
- Refresca los créditos con el `credits_remaining` que llega en `generation_completed`.

## Cómo verificar tu trabajo

```bash
cd frontend
npm run dev     # Vite
npm run build   # tsc -b && vite build  — DEBE compilar sin errores de tipos
npm run lint
```
Con el backend levantado en `:8000`, **recorre el flujo en el navegador de verdad**: login → elegir estilo → generar → ver progreso → ver los avatares → mirar el historial. Una tarea de UI no está hecha hasta que la has visto funcionar.
