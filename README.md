# TodoApp - API REST

API REST para gestión de tareas con Node.js, Express y MongoDB.

## Características

- CRUD completo de tareas
- Tests con Jest
- Cobertura de código > 80%
- CI/CD con GitHub Actions

## Instalación

```bash
npm install
```

## Tests

```bash
# Ejecutar tests
npm test

# Tests con coverage
npm run test:coverage
```

## Cobertura de Código

El proyecto mantiene una cobertura de código superior al 80%:

- Statements: 97.95%
- Branches: 100%
- Functions: 100%
- Lines: 97.87%

## CI/CD

El proyecto incluye un workflow de GitHub Actions que:
- Ejecuta los tests automáticamente
- Verifica que la cobertura sea >= 80%
- Se ejecuta en cada push y pull request

## Endpoints

- `GET /api/tareas` - Obtener todas las tareas
- `GET /api/tareas/:id` - Obtener una tarea
- `POST /api/tareas` - Crear una tarea
- `PUT /api/tareas/:id` - Actualizar una tarea
- `DELETE /api/tareas/:id` - Eliminar una tarea
