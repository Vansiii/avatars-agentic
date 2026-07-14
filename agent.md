# ⚠️ Archivo obsoleto — no es el sistema de agentes vigente

Este archivo (v3.0, 2026-07-13) describía un sistema de **diez roles**
(`orchestrator`, `frontend`, `backend`, `ai`, `database`, `security`, `testing`,
`devops`, `documentation`, `reviewer`) con memoria compartida en `docs/estado.md`
y `docs/decisiones.md`.

**Ninguno de los dos existe.** `docs/` nunca se creó, y el sistema que sí opera
desde el mismo 2026-07-13 es otro: **[.agents/AGENTS.md](.agents/AGENTS.md)**, con
tres roles (Frontend, Backend, IA) y memoria en `.agents/memory/{SOUL,HEARTBEAT,MEMORY}.md`.

Diez roles para un Alpha monolítico es además la sobreingeniería que `SOUL.md §6`
prohíbe explícitamente — siete de esos roles (`database`, `security`, `testing`,
`devops`, `documentation`, `reviewer`) nunca tuvieron una tarea propia en el
backlog; sus responsabilidades ya las cubren Backend e IA.

**Usa `.agents/AGENTS.md` como punto de entrada.** Este archivo se conserva solo
como referencia histórica de por qué existió una segunda definición contradictoria
— no lo edites para añadir roles nuevos; si un dominio de verdad necesita un rol
propio (evidencia: tareas repetidas de ese dominio bloqueadas por falta de dueño),
añádelo en `.agents/AGENTS.md` y crea su subagente en `.claude/agents/`.
