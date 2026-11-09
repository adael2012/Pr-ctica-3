# Guía de contribución

¡Gracias por ayudar a mejorar este proyecto! Sigue estos pasos para contribuir.

## Flujo de trabajo

1. Haz un **fork** del repositorio o crea una rama desde `main`.
2. Nombra la rama según el tipo de cambio: `feature/nombre`, `fix/nombre` o `docs/nombre`.
3. Haz cambios pequeños y enfocados.
4. Antes de subir, ejecuta localmente:
   ```bash
   npm run lint
   npm test
   ```
5. Haz commit con mensajes claros en español, en imperativo (ej.: `Agrega endpoint DELETE de tareas`).
6. Sube la rama y abre un **Pull Request** hacia `main`.
7. El pipeline de GitHub Actions debe estar en **verde** antes de fusionar.

## Reglas de código

- Todo cambio de comportamiento debe incluir **al menos una prueba** en `tests/`.
- La cobertura global no debe bajar del **80 %** (el pipeline lo exige).
- Mantén los mensajes de error de la API en español y con formato `{ "error": "..." }`.
- No subas `node_modules/`, `coverage/` ni archivos con secretos.

## Reportar problemas

Abre un *issue* indicando: pasos para reproducir, resultado esperado, resultado obtenido y versión de Node.js.

## Revisión

Los Pull Requests se revisan verificando: pipeline en verde, pruebas nuevas, documentación actualizada si aplica.
