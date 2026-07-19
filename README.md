# IEN — Despliegue en Render

Plataforma IEN (Inteligencia Emocional).

## Arquitectura

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Frontend        │────▶│  Backend API    │────▶│  MongoDB Atlas  │
│  (Static Site)   │     │  (Docker)       │     │  (Free 512MB)   │
│  Render Free     │     │  Render Free    │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

| Servicio   | Plataforma     | Costo  |
|------------|----------------|--------|
| Frontend   | Render Static  | Gratis |
| Backend    | Render Web     | Gratis |
| MongoDB    | MongoDB Atlas  | Gratis (M0, 512MB) |
| Seeder     | Render Shell   | Gratis |

## Paso 1 — MongoDB Atlas (base de datos)

1. Ir a [cloud.mongodb.com](https://cloud.mongodb.com)
2. Crear cuenta gratuita
3. **Build a Database** → **Shared** (M0, gratis)
4. Elegir región más cercana
5. Crear usuario y contraseña de base de datos
6. En **Network Access** → **Add IP Address** → **Allow Access from Anywhere** (`0.0.0.0/0`)
7. En **Database** → **Connect** → **Connect your application**
8. Copiar la URI de conexión. Queda algo así:
   ```
   mongodb+srv://usuario:contraseña@cluster0.xxxxx.mongodb.net/ien?retryWrites=true&w=majority
   ```
9. Reemplazar `xxxxxxxx` con el nombre de la DB: `/ien`

## Paso 2 — Render (backend + frontend)

1. Ir a [render.com](https://render.com) → crear cuenta con GitHub
2. **New** → **Blueprint** → conectar repo `alex43x/ien`
3. Render detecta el `render.yaml` y crea 2 servicios automáticamente
4. Configurar variables de entorno en cada servicio

### Backend (ien-backend)

En el dashboard de Render → **ien-backend** → **Environment**:

| Variable        | Valor                                                          |
|-----------------|----------------------------------------------------------------|
| MONGO_URI       | `mongodb+srv://user:pass@cluster.../ien?retryWrites=true&w=majority` |
| JWT_SECRET      | Generar con: `openssl rand -hex 32`                            |
| CRON_API_KEY    | Generar con: `openssl rand -hex 32`                            |
| RESEND_API_KEY  | Tu API key de [resend.com](https://resend.com)                 |
| EMAIL_FROM      | Tu email de envío                                              |
| FRONTEND_URL    | URL del frontend (paso siguiente)                              |

### Frontend (ien-frontend)

El frontend es un **Static Site** en Render. La URL se genera automáticamente.

Una vez desplegado, copiar la URL (algo como `https://ien-frontend.onrender.com`) y pegarla en `FRONTEND_URL` del backend.

## Paso 3 — Ejecutar el Seeder

1. En Render → ir al servicio **ien-backend**
2. Pestaña **Shell** (en el menú lateral)
3. Ejecutar:
   ```bash
   node src/seed.js
   ```
4. Verificar que diga "Seed completado" o similar
5. **Importante**: Esto solo se ejecuta una vez

## Paso 4 — Verificar

1. Abrir la URL del frontend
2. Ir a `/login`
3. El seeder crea un admin por defecto — verificar credenciales en `back/src/seed.js`

## Notas importantes

- **Spin-down**: El backend se duerme tras 15 min sin tráfico. Al recibir una request, tarda ~30-60s en despertar
- **MongoDB Atlas free**: Se borra tras 30 días sin uso. Mantenerlo activo
- **Variables de entorno**: El `render.yaml` tiene las variables marcadas como `sync: false` — hay que configurarlas manualmente en el dashboard
- **Auto-deploy**: Cada push a `main` despliega automáticamente backend y frontend

## Testing local

```bash
cp .env.example .env
# Editar .env con valores reales
docker compose up --build
```

- Frontend: `http://localhost:80`
- Backend: `http://localhost:3000/api`
- MongoDB: `mongodb://localhost:27017/ien`
