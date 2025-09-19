# To-Do App — (Docker)

Aplicación **Full-Stack** para gestionar tareas (To-Do).  
Backend en **FastAPI** + **MongoDB**, frontend en **React (Vite)** con **Material UI**.  
Incluye **Docker Compose**, **tests del backend** y manejo de **CORS**, **errores** y **validación**.

## Demo local (2 comandos)

```bash
# 1) Variables de entorno
cp .env.example .env

# 2) Levantar todo (API, Mongo, Front)
docker compose up --build
```

- Frontend: http://localhost:5173  
- API (Swagger): http://localhost:8000/docs

---

## Estructura

```
.
├─ docker-compose.yml
├─ .env.example           # variables comunes (API + Front)
├─ PruebaTecnica/         # Backend (FastAPI)
│  ├─ Dockerfile
│  ├─ requirements.txt
│  └─ app/
│     ├─ main.py
│     ├─ api/v1/tasks.py
│     ├─ db/...
│     ├─ services/...
│     ├─ schemas/task.py
│     └─ tests/test_tasks.py
└─ PruebaFront/           # Frontend (React + Vite + MUI)
   ├─ Dockerfile
   ├─ nginx.conf
   └─ src/
      ├─ api/client.js
      ├─ App.jsx
      ├─ components/...
      └─ ...
```

---

## Variables de entorno

Crea `.env` a partir de `.env.example` en la **raíz** del proyecto:

```dotenv
# Backend
MONGO_URI=mongodb://mongo:27017
MONGO_DB=todo_db
CORS_ORIGINS=http://localhost:5173,http://127.0.0.1:5173

# Frontend (se inyecta en build)
VITE_API_URL=http://localhost:8000/api/v1
```

> En Docker, la API se conecta a Mongo por el **nombre del servicio** `mongo:27017` (red interna).

---

## Cómo correr **tests del backend**

```bash
docker compose run --rm api pytest -q
```

---

## Decisiones técnicas (resumen)

- **FastAPI + Pydantic**: tipado fuerte, validaciones, OpenAPI/Swagger gratis y rendimiento excelente.
- **MongoDB (motor)**: persistencia flexible para items simples (To-Do), índices y `_id` de alto desempeño.
- **Arquitectura por capas**:
  - `schemas/` (Pydantic) → contrato de entrada/salida
  - `services/` → reglas de negocio
  - `db/repositories/` → puertos/adaptadores (Mongo / in-memory para tests)
  - `api/` → routing y dependencias
- **Calidad**: tests de rutas/validación (Happy & Unhappy path), handler de errores, CORS configurado, idempotencia en DELETE, paginado `skip/limit`.
- **Frontend**:
  - **React + Vite** por DX y HMR rápido.
  - **Material UI** para UI consistente (Tabs, Dialogs, Snackbar).
  - Estado local con hooks (`useState`, `useEffect`) y separación por componentes.
  - **Snackbars** de éxito/error (verde/rojo) y edición en modal.

---

## Desarrollo local sin Docker (opcional)

### Backend
```bash
cd PruebaTecnica
python -m venv .venv
.\.venv\Scriptsctivate
pip install -r requirements.txt
uvicorn app.main:app --reload
```

### Frontend
```bash
cd PruebaFront
npm i
# crea PruebaFront/.env con:
# VITE_API_URL=http://localhost:8000/api/v1
npm run dev
```

---

## Troubleshooting

- **El puerto 27017 ya está en uso (Mongo):**  
  - Opción rápida: no publiques el puerto en `mongo` (quita `ports:` del compose)  
  - O cambia el host-port: `27018:27017`  
  - La API **no cambia**: `MONGO_URI=mongodb://mongo:27017`.

- **CORS en el navegador:**  
  Asegúrate de incluir `http://localhost:5173` en `CORS_ORIGINS` y reinicia la API.

---

**¡Gracias por revisar!**  
Ante cualquier duda:  
1) `cp .env.example .env`  
2) `docker compose up --build`  
Front → http://localhost:5173 · API → http://localhost:8000/docs
