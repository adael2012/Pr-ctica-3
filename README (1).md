# Lista de Tareas DevOps — Práctica 3 (ITLA, Electiva 1: Introducción a DevOps)

[![CI/CD](https://github.com/adael2012/Pr-ctica-3/actions/workflows/ci.yml/badge.svg)](https://github.com/adael2012/Pr-ctica-3/actions/workflows/ci.yml)

Aplicación web mínima (lista de tareas) con un pipeline **CI/CD** completo en **GitHub Actions**: revisión estática → pruebas con cobertura → build de imagen Docker → despliegue a GitHub Pages.

- **Autor:** Adael Araujo Pineda (matrícula 2025-1127)
- **Repositorio:** https://github.com/adael2012/Pr-ctica-3
- **Sitio publicado:** https://adael2012.github.io/Pr-ctica-3/

## Estructura

| Ruta | Descripción |
|---|---|
| `src/app.js` | API Express: tareas, `/health`, `/metrics` |
| `src/server.js` | Arranque del servidor |
| `public/index.html` | Frontend estático (se publica en Pages) |
| `tests/app.test.js` | Pruebas Jest + Supertest |
| `Dockerfile` | Imagen de producción con *healthcheck* |
| `.github/workflows/ci.yml` | Pipeline CI/CD |
| `docs/INFORME.md` | Informe de implementación |
| `CONTRIBUTING.md` | Guía de contribución |

## Ejecutar localmente

```bash
npm install
npm run lint      # revisión estática
npm test          # pruebas + cobertura (umbral mínimo 80 %)
npm start         # http://localhost:3000
```

### Endpoints

| Método | Ruta | Descripción |
|---|---|---|
| GET | `/health` | Estado del servicio |
| GET | `/metrics` | Solicitudes, errores, tareas, segundos activo |
| GET | `/api/tareas` | Lista de tareas |
| POST | `/api/tareas` | Crea tarea (`{"titulo": "..."}`, máx. 100 caracteres) |
| PATCH | `/api/tareas/:id` | Alterna completada |
| DELETE | `/api/tareas/:id` | **Nuevo:** elimina una tarea |

### Docker

```bash
docker build -t tareas-devops .
docker run -p 3000:3000 tareas-devops
```

## Documentación del pipeline

El workflow `.github/workflows/ci.yml` se dispara en cada **push** y **pull request** hacia `main`.

```
push / PR ──► [build-test] ──► [deploy-pages]
               │                 (solo push a main)
               ├─ checkout
               ├─ setup Node 20
               ├─ npm install
               ├─ lint
               ├─ pruebas + cobertura
               ├─ guardar reporte de cobertura
               └─ docker build
```

### Job `build-test` (Build y pruebas)

| Etapa | Qué hace | Si falla |
|---|---|---|
| Checkout | Descarga el código del repositorio | El job se detiene |
| Setup Node 20 | Instala Node.js 20 en el runner | El job se detiene |
| `npm install` | Instala Express, Jest y Supertest | El job se detiene |
| Revisión estática | `node --check` valida la sintaxis de `src/` | Pipeline en rojo |
| Pruebas con cobertura | `jest --coverage`; exige ≥ 80 % de cobertura | Pipeline en rojo, **no se despliega** |
| Guardar cobertura | Sube la carpeta `coverage` como artefacto descargable | — |
| Construir imagen Docker | Verifica que el `Dockerfile` construya | Pipeline en rojo |

### Job `deploy-pages` (Despliegue a GitHub Pages)

- Depende de `build-test` (`needs`): solo corre si todo lo anterior pasó.
- Solo se ejecuta en **push a `main`** (no en pull requests).
- Publica la carpeta `public/` con `configure-pages`, `upload-pages-artifact` y `deploy-pages`.
- La URL queda en el entorno `github-pages` del job.

> En GitHub Pages solo se sirve el frontend estático; la API requiere `npm start` o Docker.

## Instrucciones de despliegue

1. Crear un repositorio **público** vacío en GitHub.
2. Subir el proyecto:
   ```bash
   git init
   git add .
   git commit -m "Práctica 3: pipeline CI/CD"
   git branch -M main
   git remote add origin https://github.com/adael2012/Pr-ctica-3.git
   git push -u origin main
   ```
3. En **Settings → Pages → Source** elegir **GitHub Actions**.
4. En la pestaña **Actions** verificar que el workflow termine en verde.
5. Abrir la URL publicada (aparece en el job *Despliegue a GitHub Pages*).

## Contribuir

Ver [CONTRIBUTING.md](CONTRIBUTING.md).

## Licencia

MIT
