# Misión del Sistema — Personajes para Spots Publicitarios de TV

> **Versión:** 2.0  
> **Fecha:** 2026-07-14  
> **Estado:** Aprobado

---

## Definición del MVP

Desarrollo de una plataforma web de generación de **personajes hiperrealistas** para spots publicitarios de televisión, que permita a un usuario crear un personaje una vez y utilizarlo de forma consistente en múltiples videos publicitarios por categoría (deportes, noticias, entretenimiento, etc.).

El núcleo funcional del MVP abarca:

1. **Creación de personaje:** entrada por imagen, texto o ambos → generación de imagen hiperrealista del personaje.
2. **Selección de variación:** el sistema genera 3 opciones, el usuario elige la que más le gusta.
3. **Consistencia:** el mismo personaje aparece en todos los spots de su categoría.
4. **Generación de spots:** video corto (3-5s) o largo (15-30s) del personaje realizando el spot.
5. **Gestión de personajes:** editar, reasignar a categorías, ver historial.
6. **Control de acceso:** admin crea usuarios, usuarios finales crean personajes y spots.

---

## Público Objetivo

| Segmento | Descripción | Caso de Uso Principal |
|---|---|---|
| **Canales de TV** | Emisoras de televisión | Personajes para spots de noticias, deportes, entretenimiento |
| **Agencias de publicidad** | Agencias creativas | Personajes para campañas publicitarias en TV |
| **Productoras** | Empresas de contenido audiovisual | Personajes para programas y segmentos |

---

## Exclusiones de Alcance (Out of Scope)

El sistema **no** abarca en esta versión inicial:

- **Edición de video manual:** no hay editor de video integrado; los spots se generan con IA.
- **Efectos especiales complejos:** el video es generado, no compuesto.
- **Streaming en vivo:** no hay transmisión en tiempo real.
- **App móvil nativa:** el producto es exclusivamente una PWA.
- **Marketplace de personajes:** no se comparten ni venden personajes entre usuarios.
- **Estilos no realistas:** solo se genera contenido hiperrealista.

---

## Arquitectura de Datos

### Artefactos Principales

| Artefacto | Tipo | Descripción |
|---|---|---|
| `character` | Registro DB + imagen | Personaje hiperrealista con imagen de referencia |
| `spot` | Registro DB + video | Video generado del personaje |
| `user` | Registro DB | Cuenta de usuario con rol (admin/user) |
| `spot_category` | Registro DB | Categoría de spots (deportes, noticias, etc.) |

### Flujo de Datos (Resumen)

```
[Usuario] → (Imagen / Descripción)
         → [API]
         → [Pipeline de Personaje]
              → [Generación de Imagen Hiperrealista]
              → [Filtro NSFW]
         → [Imagen del Personaje] → [Referencia para videos]
         → [Pipeline de Video]
              → [Generación de Video con personaje]
              → [Filtro NSFW]
         → [Video del Spot]
         → [Usuario]
```

---

## Propuesta de Valor Central

> *"Crea un personaje una vez, úsalo en todos tus spots de TV. Hiperrealista, consistente, profesional."*

La plataforma elimina la necesidad de actores reales o animadores para spots publicitarios, entregando personajes digitales consistentes y de alta calidad que aparecen en múltiples videos de la misma categoría.
