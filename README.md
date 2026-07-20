# IEN — Despliegue en Northflank

Plataforma IEN (Inteligencia Emocional).

## Arquitectura

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Frontend        │────▶│  Backend API    │────▶│  MongoDB Atlas  │
│  (Static Site)   │     │  (Docker)       │     │  (Free 512MB)   │
│  Northflank      │     │  Northflank     │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

| Servicio   | Plataforma     | Costo  |
|------------|----------------|--------|
| Frontend   | Northflank Static | Gratis |
| Backend    | Northflank Docker | Gratis |
| MongoDB    | MongoDB Atlas  | Gratis (M0, 512MB) |
| Seeder     | CLI manual     | -      |

## Paso 1 — MongoDB Atlas (base de datos)

1. Ir a [cloud.mongodb.com](https://cloud.mongodb.com)
2. Crear cuenta gratuita
3. **Build a Database** → **Shared** (M0, gratis)
4. Elegir region mas cercana
5. Crear usuario y contrasena de base de datos
6. En **Network Access** → **Add IP Address** → **Allow Access from Anywhere** (`0.0.0.0/0`)
7. En **Database** → **Connect** → **Connect your application**
8. Copiar la URI de conexion. Queda algo asi:
   ```
   mongodb+srv://usuario:contrasena@cluster0.xxxxx.mongodb.net/ien?retryWrites=true&w=majority
   ```
9. Reemplazar `xxxxxxxx` con el nombre de la DB: `/ien`

## Paso 2 — Northflank (backend + frontend)

1. Ir a [northflank.com](https://northflank.com) → crear cuenta con GitHub
2. Conectar el repositorio
3. Crear proyecto: **New Project** → nombre `ien`

### Backend (Docker Service)

En el proyecto → **New Service** → **Deploy from GitHub**:

| Campo | Valor |
|-------|-------|
| **Service Type** | Docker |
| **Repository** | Tu fork/repo |
| **Build Context** | `/back` |
| **Dockerfile Path** | `Dockerfile` |
| **Port** | `3000` |
| **Health Check Path** | `/health` |

Variables de entorno:

| Variable | Valor |
|----------|-------|
| `MONGO_URI` | URI de MongoDB Atlas |
| `NODE_ENV` | `production` |
| `JWT_SECRET` | `openssl rand -hex 32` |
| `CRON_API_KEY` | `openssl rand -hex 32` |
| `RESEND_API_KEY` | API key de resend.com |
| `EMAIL_FROM` | Tu email de envio |
| `FRONTEND_URL` | URL del frontend (proximo paso) |
| `PORT` | `3000` |

### Frontend (Static Site)

En el proyecto → **New Service** → **Deploy from GitHub**:

| Campo | Valor |
|-------|-------|
| **Service Type** | Static |
| **Repository** | Tu fork/repo |
| **Build Path** | `/frontend` |
| **Build Command** | `npm ci && npm run build` |
| **Publish Directory** | `dist` |

Variables de entorno (build):

| Variable | Valor |
|----------|-------|
| `VITE_API_URL` | `https://TU_BACKEND_URL.northflank.app/api` |

La URL del backend la obtienes del dashboard de Northflank una vez desplegado.

Una vez desplegado el frontend, volver al backend y actualizar `FRONTEND_URL` con la URL del frontend.

## Paso 3 — Ejecutar el Seeder

1. En Northflank → ir al servicio **backend**
2. Pestaña **Logs** o **Terminal**
3. Ejecutar:
   ```bash
   node src/seed.js
   ```
4. Verificar que diga "Seed completado" o similar
5. **Importante**: Esto solo se ejecuta una vez

## Paso 4 — Verificar

1. Abrir la URL del frontend
2. Ir a `/login`
3. El seeder crea un admin por defecto — credenciales en `back/src/seed.js`

## Notas importantes

- **Free tier**: Northflank da 2 servicios gratis + static sites. Este proyecto usa 1 Docker (backend) + 1 Static (frontend)
- **MongoDB Atlas free**: Se borra tras 30 dias sin uso. Mantenerlo activo
- **Auto-deploy**: Northflank puede configurarse para desplegar automaticamente en cada push

## Testing local

```bash
cp .env.example .env
# Editar .env con valores reales
docker compose up --build
```

- Frontend: `http://localhost:80`
- Backend: `http://localhost:3000/api`
- MongoDB: `mongodb://localhost:27017/ien`
