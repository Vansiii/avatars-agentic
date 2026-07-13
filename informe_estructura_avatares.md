# Informe de Revisión y Plan de Acción: Sistema de Avatares

Estimado equipo,

He revisado a fondo su repositorio de especificaciones para la plataforma de Avatares (`agent.md`, `sdd.md`, `mision.md`). Han documentado un diseño de arquitectura ambicioso y el concepto de usar un **Sistema Multi-Agente (Agent System)** para el desarrollo es innovador. Sin embargo, para que puedan codificar su primera prueba Alpha con éxito usando IA, hay un problema técnico con la forma en que lo han estructurado y una severa sobreingeniería en el sistema base.

A continuación les detallo los errores y cómo vamos a ajustar su "Harness" para que su enfoque multi-agente realmente funcione.

---

## 1. Errores Detectados en su Enfoque Actual

1. **Sobreingeniería Extrema para el Alpha:**
   Su SDD define una arquitectura basada en microservicios, Celery, Redis, PgBouncer y WebSockets. Tratar de levantar todo esto solo para un MVP generador de avatares los va a retrasar semanas peleando con Docker. Para el Alpha, **deben usar un monolito en FastAPI**.
2. **El Sistema Multi-Agente Carece de "Memoria Compartida":**
   Ustedes han definido archivos para `orchestrator.md`, `frontend.md`, `backend.md`, etc. **Sí es posible usar un enfoque multi-agente para desarrollar**, pero si le lanzan estos 10 roles independientes a una IA sin conectarlos, los agentes perderán el contexto. Para que el Agente Frontend sepa qué hizo el Agente Backend ayer, necesitan un **Protocolo de Memoria Centralizado**.

## 2. Soluciones: Cómo hacer que su Multi-Agente funcione

Para el Alpha, aplicaremos el principio **KISS (Keep It Simple, Stupid)** en el software, pero estructuraremos bien a sus agentes:
* **Cero Celery / Redis por ahora.** El Alpha será un monolito en FastAPI con peticiones síncronas o `BackgroundTasks` a la API de IA (Replicate/OpenAI).
* **El "Harness" Multi-Agente (Plantilla `.agents`):**
Vamos a adoptar la estructura de la carpeta `.agents` pero adaptada para múltiples agentes. Esto significa que **todos sus agentes compartirán una misma carpeta de memoria dinámica**.

## 3. Implementación del Harness Multi-Agente (Plantilla Obligatoria)

Deben estructurar la raíz de su proyecto con una carpeta `.agents/` configurada de la siguiente manera:

### A. El Orquestador (Archivo Maestro): `.agents/AGENTS.md`
Este será el archivo que la IA leerá primero.
```markdown
# Orquestador del Sistema de Avatares

## 1. Identidad y Misión
Eres el Agente Orquestador. Tu trabajo es delegar tareas a tus sub-agentes (Frontend, Backend, DevOps) para construir un MVP monolítico en FastAPI y React que genere avatares usando IA.

## 2. Protocolo de Memoria (OBLIGATORIO PARA TODOS LOS AGENTES)
Todos los agentes (tú y tus sub-agentes) DEBEN leer `.agents/memory/SOUL.md` al iniciar.
Al cambiar de turno o terminar una tarea, DEBEN actualizar `.agents/memory/HEARTBEAT.md` y `MEMORY.md`.

## 3. Roles Disponibles (Modos)
- Si la tarea es de UI, asume el rol detallado en `.agents/skills/frontend.md`.
- Si la tarea es de BD/API, asume el rol detallado en `.agents/skills/backend.md`.
```

### B. Memoria Compartida (Carpeta `.agents/memory/`)
*(Esto es lo que evita que los agentes se desconecten entre sí)*

**1. `SOUL.md` (Reglas Inmutables):**
> Aquí explican sus roles de negocio (usuario free vs premium) y que todo avatar en el Alpha lleva marca de agua y filtro NSFW.

**2. `HEARTBEAT.md` (El Pulso del Proyecto):**
> El orquestador actualiza esto para que el Agente Frontend sepa en qué estado dejó las cosas el Agente Backend.
> - **Estado Backend:** Rutas FastAPI listas en puerto 8000.
> - **Foco Actual:** Agente Frontend debe conectar el UI de React a la ruta `/generate`.

**3. `MEMORY.md` (Historial Inmutable):**
> Un log donde todos los agentes escriben qué hicieron. Ej: *"El Agente Backend integró Replicate API"*.

### C. Sus Agentes (Carpeta `.agents/skills/`)
Aquí es donde pondrán los archivos de roles que ya tenían, para que el Orquestador los llame cuando sea necesario:
- `frontend.md` (React 18, Vite, CSS Modules)
- `backend.md` (FastAPI, Python 3.11, sin Celery)
- `ai.md` (Lógica de conexión a Replicate)

### D. Tareas: `.agents/steering/backlog.md`
> Listado de checklist de sus issues del Alpha para que el Orquestador marque progreso.

Migren su documentación a este formato de arnés (Harness) unificado. De esta forma, el desarrollo multi-agente que proponen **sí será viable y extremadamente eficiente**. Simplifiquen la arquitectura de software (cero Redis/Celery) y logremos el Alpha. Quedo a la espera de sus avances.
