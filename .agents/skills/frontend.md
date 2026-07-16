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
- Estilos: **CSS global** (`src/App.css`, `src/index.css`)

## Mapa del código

```
frontend/src/
├── main.tsx                      # Entry: router + providers
├── App.tsx
├── store.ts                      # zustand: sesión / usuario
├── components/DashboardLayout.tsx
├── pages/  Landing · Login · Dashboard · Characters · GenerateSpot · Profile
└── App.css · index.css
```

## Reglas de división de responsabilidad

- **zustand** para estado del cliente que persiste entre páginas: token JWT, usuario, rol.
- **react-query** para todo lo que venga del servidor: personajes, spots, generación.
- El **token JWT** se manda en `Authorization: Bearer <token>` en cada petición autenticada.
- Todos los textos de la interfaz **en español**.

## Reglas de negocio visibles en el UI

Vienen de `SOUL.md`. El UI las *refleja*, **nunca las hace cumplir por sí solo**:

1. **Límites semanales:** muestra cuántos personajes y spots quedan esta semana. Con 0 disponibles, deshabilita el botón de crear/generar.
2. **Rehacer:** cuando hay 3 variaciones, muestra botón "Rehacer" (máx 3 veces) y botón "Elegir". Solo al elegir se decrementa el límite.
3. **Filtro NSFW (422 `GEN_004`):** muestra un mensaje neutro y respetuoso. No detalles qué se detectó.
4. **Validaciones de entrada** (≤ 10 MB, JPEG/PNG/WEBP, prompt de 10–500 caracteres): valida en el cliente para dar feedback rápido, **y aun así maneja el error del servidor**.
5. **Roles:** el admin ve gestión de usuarios; el usuario final ve personajes y spots.

## Flujo de creación de personaje

```
1. POST /api/v1/characters  (multipart: prompt?, file?, name, category)
   → 202 { data: { character_id, estimated_seconds } }
2. Escuchar WebSocket o polling para progreso
3. Recibir 3 variaciones de imagen del personaje
4. Usuario elige una → POST /api/v1/characters/{id}/select  (variation_index)
   → 200 { data: { character_id, selected: true } }
5. Si no le gusta → POST /api/v1/characters/{id}/redo
   → 202 { data: { variations[], redos_remaining } }
```

## Flujo de generación de spot (video)

```
1. POST /api/v1/spots  (character_id, type: short|long, script)
   → 202 { data: { spot_id, estimated_seconds } }
2. Escuchar WebSocket para progreso
3. Recibir 3 variaciones de video
4. Usuario elige una → POST /api/v1/spots/{id}/select  (variation_index)
   → 200 { data: { spot_id, selected: true } }
5. Si no le gusta → POST /api/v1/spots/{id}/redo
   → 202 { data: { variations[], redos_remaining } }
```

## Cómo verificar tu trabajo

```bash
cd frontend
npm run dev     # Vite
npm run build   # tsc -b && vite build  — DEBE compilar sin errores de tipos
npm run lint
```
Con el backend levantado en `:8000`, **recorre el flujo en el navegador de verdad**: login → crear personaje → elegir variación → generar spot → ver video. Una tarea de UI no está hecha hasta que la has visto funcionar.
