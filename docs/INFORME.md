# Informe de implementación — Práctica 3: Pipeline CI/CD con GitHub Actions

**Estudiante:** Adael Araujo Pineda — Matrícula 2025-1127
**Asignatura:** Electiva 1: Introducción a DevOps — ITLA
**Repositorio:** https://github.com/adael2012/Pr-ctica-3

## 1. Objetivo

Implementar un pipeline CI/CD básico con GitHub Actions para una aplicación web sencilla (lista de tareas en Node.js/Express), automatizando revisión estática, pruebas, build de contenedor y despliegue a GitHub Pages.

## 2. Configuración inicial

- Repositorio público en GitHub con rama `main`.
- Estructura basada en la plantilla oficial: `src/`, `public/`, `tests/`, `Dockerfile`, `.github/workflows/`.
- Node.js 20 y Express 4 como base; Jest y Supertest como herramientas de prueba.

## 3. Implementación de pruebas

| Tipo | Cantidad | Descripción |
|---|---|---|
| Plantilla original | 6 | `/health`, crear tarea, validación sin título, alternar, 404 en PATCH, `/metrics` |
| **Nuevas** | 5 | DELETE existente, DELETE 404, título > 100 caracteres, ids sin repetir tras borrar, título solo espacios |
| **Total** | **11** | Todas pasan |

Las pruebas usan Supertest contra `createApp()`, por lo que ejercitan rutas, validaciones y middleware (pruebas de integración de la API) además de la lógica unitaria.

**Cobertura** (`jest --coverage`): 96 % sentencias, 87.5 % ramas, 100 % funciones, 100 % líneas. Umbral global configurado en `package.json`: **80 %**; si baja, el pipeline falla.

## 4. Cambios al código

1. **Nueva funcionalidad:** `DELETE /api/tareas/:id` (200 con la tarea eliminada, 404 si no existe).
2. **Corrección de un defecto de la plantilla:** los ids se calculaban con `tareas.length + 1`; al eliminar tareas se podían repetir ids. Se reemplazó por un contador incremental `siguienteId`, cubierto por una prueba.
3. **Validación:** títulos de más de 100 caracteres devuelven 400.
4. **Frontend:** clic derecho sobre una tarea la elimina.

## 5. Pipeline

Archivo: `.github/workflows/ci.yml`. Disparadores: push y pull request a `main`.

**Job `build-test`:** checkout → Node 20 → `npm install` → lint → pruebas con cobertura → subir reporte de cobertura (artefacto) → `docker build`.

**Job `deploy-pages`:** depende de `build-test`; solo en push a `main`; publica `public/` en GitHub Pages.

Mecanismo de seguridad del despliegue: si cualquier etapa de `build-test` falla (lint, pruebas, cobertura < 80 % o Docker), el despliegue **no se ejecuta**.

## 6. Demostración rojo → verde

> Completar con capturas de tu repositorio.

1. **Provocar el fallo:** en `tests/app.test.js` cambiar temporalmente `expect(r.status).toBe(201)` por `toBe(200)` en la prueba de crear tarea y hacer push.
   - Captura 1: pestaña *Actions* con el workflow en **rojo** y el paso *Pruebas con cobertura* fallido.
   - Captura 2: confirmación de que *deploy-pages* fue omitido.
2. **Corregir:** restaurar `toBe(201)` y hacer push.
   - Captura 3: workflow en **verde**.
3. **Resultado:** captura de la URL de GitHub Pages funcionando.

## 7. Documentación entregada

- `README.md`: descripción, estructura, endpoints, ejecución, documentación del pipeline y despliegue.
- `CONTRIBUTING.md`: guía de contribución.
- `docs/INFORME.md`: este informe.

## 8. Conclusiones y lecciones aprendidas

- Automatizar pruebas en cada push detecta errores antes de que lleguen a producción.
- Encadenar el despliegue a las pruebas (`needs`) evita publicar código roto.
- Un umbral de cobertura obliga a acompañar cada funcionalidad con sus pruebas.
- Revisar el código de la plantilla permitió encontrar un defecto real (ids repetidos).
- Mejoras futuras: ESLint real, persistencia con SQLite, publicar la imagen en `ghcr.io` y definir alertas con `/metrics`.

## 9. Guion sugerido para el video/PPT (step by step)

1. Presentar el proyecto y la estructura del repositorio.
2. Mostrar el código nuevo (`DELETE`) y las pruebas; ejecutar `npm test` local.
3. Recorrer `ci.yml` explicando cada etapa.
4. Hacer el push con prueba fallida → mostrar rojo.
5. Corregir → mostrar verde.
6. Abrir el sitio en GitHub Pages y explicar la documentación.
