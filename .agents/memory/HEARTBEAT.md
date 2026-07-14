# HEARTBEAT — El Pulso del Proyecto

> **Foto del estado ACTUAL.** Este archivo se **sobrescribe** en cada cambio de turno.
> No es un historial: si buscas qué pasó antes, ve a `MEMORY.md`.

**Última actualización:** 2026-07-14 — por el **Agente Orquestador** (corrección tras auditoría independiente — la versión anterior de este archivo declaraba "96%, sin bloqueantes" con dos violaciones activas de SOUL.md sin detectar)

---

## 📊 Estado Global del Proyecto

**No uses el porcentaje anterior (96%) para decidir nada.** Se calculó marcando tareas `[x]` sin ejecutarlas. Los porcentajes de abajo son honestos pero aproximados — la única fuente de verdad es correr el código.

| Épica | Estado real | Nota |
|-------|------------|------|
| A - Fundaciones | ✅ Cerrada | Verificado, sin caveats |
| B - IA Real | 🔴 Reabierta parcialmente | B-03 (NSFW entrada) desactivado en el código pese a estar marcado `[x]`. B-04 (NSFW salida) tiene un riesgo real de categorías no verificado |
| C - Frontend | ✅ Cerrada (sin motivo para dudar de esta) | No se encontró discrepancia código/documento aquí |
| D - Seguridad | 🟡 Cerrada con un caveat | D-01 (strip EXIF) calcula el resultado limpio y lo descarta — no causa fuga hoy porque no se persiste la imagen de entrada, pero no hace lo que dice hacer |
| E - Cierre | ⏳ Pendiente + 2 tareas nuevas (E-04, E-05) | Cero tests siguen existiendo en el repo |
| F - Higiene de repo | 🔴 **Nueva, bloqueante** | Credenciales de producción comiteadas en `.agents/memory/`; `.gitignore` ignora todo el código fuente (0 archivos de `backend/`/`frontend/` están versionados) |

---

## 🚨 Bloqueos reales (no "sin bloqueantes")

1. **F-01/F-02 — Credenciales filtradas.** La contraseña de Supabase y el `SECRET_KEY` de JWT (el mismo valor para ambos) están en texto plano en dos archivos versionados (`.agents/memory/AUDIT_2026-07-13.md`, `.agents/memory/MEMORY.md`), presentes en el historial de commits. Requiere rotación humana antes de cualquier otra cosa.
2. **F-03 — Nada del código está en git.** El `.gitignore` raíz ignora `/frontend` y `/backend` completos. Cualquier `git clean` destruiría el proyecto sin posibilidad de recuperación.
3. **B-03 — Filtro NSFW de entrada apagado.** `safe=false` en producción. Incumple `SOUL.md §4` activamente, no en teoría.
4. **B-04 — Filtro NSFW de salida sin verificar.** Riesgo de que las categorías de NudeNet no coincidan con la versión instalada y el filtro no rechace nunca nada. No confirmado ni descartado — falta un test con fixture.

Ver `.agents/steering/backlog.md` Épica F para el detalle y el orden sugerido (F-01 → F-02 → F-03).

---

## Estado Backend — 🟡 Funcional, con dos brechas de seguridad activas

### ✅ Lo que sí está verificado
- Monolito FastAPI corriendo (puerto 8000), sin Celery ni Redis
- Rutas `/api/v1`: `auth`, `styles`, `generations` (202 + WebSocket + historial)
- Generación real con Pollinations.ai, paralela con `asyncio.gather()`
- Watermark real con Pillow, aplicado a toda imagen
- CORS restringido a localhost + `CORS_ORIGINS` de entorno (código en `main.py`, verificado leyendo el archivo)
- `SECRET_KEY` sin default inseguro, falla el arranque si falta (verificado en `settings.py`)
- Rate limiting activo, pero **el identificador "por usuario" en realidad es por IP** (ver `E-05`)
- Cleanup scheduler corriendo cada 6h, pero opera sobre una fila (`input_image_url`) que siempre tiene el mismo valor inventado (`uploads/mock_input.png`) — nunca borra nada real

### 🔴 No verificado o roto
- Filtro NSFW de entrada: desactivado (`B-03`)
- Filtro NSFW de salida: implementado pero sin test de fixture (`B-04`)
- Strip EXIF: calcula y descarta el resultado (`D-01`)
- Cero tests en `backend/tests/`

---

## Estado Frontend — 🟢 Sin discrepancias encontradas

No se auditó línea por línea en esta pasada, pero no hay evidencia de que el frontend tenga el mismo problema de "marcado hecho sin verificar" que el backend. Generate.tsx está cableado al backend real, sin simulación falsa. Si vuelves a auditar, prioriza el backend — ahí es donde aparecieron las tres discrepancias.

---

## Estado IA — 🟡 Pollinations.ai, con el filtro de entrada apagado

Ver `.agents/skills/ai.md`, sección "Filtro NSFW — estado real (corregido 2026-07-14)". No repitas la versión anterior de esta sección sin releer el código: decía "entrada cubierta" cuando no lo estaba.

---

## 🎯 Foco Actual

**Antes que cualquier feature nueva:**
1. `F-01` — rotar credenciales (humano)
2. `F-03` — arreglar `.gitignore` y comitear el código real (solo después de F-01)
3. `B-03` — decidir si `safe=true` vuelve o se documenta como decisión explícita
4. `E-04` — escribir el test de fixture NSFW, que responde de una vez si `B-04` funciona

Todo lo demás (E-01, E-02, E-03, E-05) puede esperar a que estas cuatro estén resueltas.

---

## 📝 Nota para el siguiente agente

Si vas a confiar en este archivo, no le des más crédito del que merece un documento que ya estuvo equivocado dos veces (ver `MEMORY.md`, entradas de auditoría). La regla vigente está en `.agents/AGENTS.md §6`: verifica ejecutando, no leyendo. Si corriges algo de esta lista, dilo en `MEMORY.md` con qué comando o prueba lo confirmaste — no solo "implementado".
