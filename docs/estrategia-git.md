# Estrategia de Git

## Ramas

- `main`: solo recibe el proyecto terminado y estable. Nunca se trabaja directo aquí.
- `development`: aquí vivieron todos los notebooks y cambios, con commits pequeños por fase.

## Flujo de trabajo

1. Todo el trabajo se hizo en `development` con un commit por fase (preprocesamiento, entrenamiento, interpretabilidad y monitoreo, documentación).
2. Antes de fusionar, revisé que el proyecto corriera de punta a punta en `development`.
3. Al final abrí un Pull Request de `development` hacia `main` y lo fusioné desde la interfaz de GitHub.
4. El proyecto se cerró con la etiqueta `v1.0.0` y sus notas de release.

## Convenciones de mensajes

- `feat:` para funcionalidad nueva (notebooks, modelos, reportes).
- `docs:` para documentación (README, esta estrategia, requirements).
- Mensajes cortos y en español, describiendo qué cambió.

## Lo que nunca se sube al repo

- El CSV de PaySim pesa 470 MB y no entra a Git (límite de GitHub de 100 MB). Se descarga de Kaggle según el README.
- Los datos procesados (`Datos/procesado/`), el entorno virtual, los tokens y los registros de MLflow local quedan en `.gitignore`.
