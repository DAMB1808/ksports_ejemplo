# KinalSports Stack

Orquestador Docker para la plataforma KinalSports. Este repositorio **no contiene el codigo** de los microservicios: los clona desde repos independientes con un script de bootstrap.

Repositorio: `https://github.com/<ORG>/kinalsports-stack`

> Reemplaza `<ORG>` por el nombre de tu organización o usuario de GitHub (por ejemplo, el que te haya asignado el docente).

## Configurar organizacion GitHub

Antes de ejecutar el bootstrap, verifica el campo `org` en [`repos.yaml`](repos.yaml). Debe coincidir con la organización donde están alojados los microservicios. Si clonas manualmente desde la terminal, usa el mismo valor en lugar de `<ORG>` en las URLs de esta documentación.

## Inicio rapido

```bash
git clone https://github.com/<ORG>/kinalsports-stack.git
cd kinalsports-stack

cp .env.docker.example .env.docker
# Edita .env.docker con tus secretos (JWT, Cloudinary, SMTP, etc.)

chmod +x scripts/bootstrap.sh
./scripts/bootstrap.sh

docker compose --env-file .env.docker up --build
```

### Workspace con repos hermanos

Si ya tienes los microservicios clonados **al mismo nivel** que `kinalsports-stack/` (sin volver a clonar con bootstrap):

```bash
chmod +x scripts/link-local.sh
./scripts/link-local.sh
docker compose --env-file .env.docker up --build
```

Postgres standalone opcional:

```bash
./scripts/bootstrap.sh --with-pg
```

## Arquitectura

```
                    +------------------+       +-------------------+
                    |   client-admin   |       |   client-user     |
                    | React + Vite     |       | React Native Expo |
                    | :5173            |       | :8081             |
                    +--------+---------+       +---------+---------+
                             |                           |
              auth-service   |                           | auth-node
              server-admin |                           | server-user
                             v                           v
              +--------------+--------------+  +--------+---------+
              | auth-service (.NET) :5156   |  | auth-node :3007  |
              | PostgreSQL                  |  | PostgreSQL       |
              +--------------+--------------+  +--------+---------+
                             |                           |
                             v                           v
              +--------------+--------------+  +--------+---------+
              | server-admin :3009          |  | server-user :3008|
              | MongoDB                     |  | MongoDB          |
              +-----------------------------+  +------------------+
```

| Stack   | Cliente      | Autenticacion       | API de negocio |
| ------- | ------------ | ------------------- | -------------- |
| Admin   | client-admin | auth-service (.NET) | server-admin   |
| Usuario | client-user  | auth-node (Node)    | server-user    |

## Estructura del workspace (despues del bootstrap)

```
kinalsports-stack/
├── docker-compose.yml
├── dockerfiles/
├── repos.yaml              # manifest de repos a clonar
├── scripts/
│   ├── bootstrap.sh        # clona microservicios dentro de kinalsports-stack/
│   └── link-local.sh       # enlaces simbolicos a repos hermanos (dev local)
├── .env.docker.example
├── client-admin/           # clonado (repo independiente)
├── client-user/
├── server-admin/
├── server-user/
├── authentication-service/
│   ├── auth-node/
│   └── auth-service/
└── pg/                     # opcional (--with-pg)
```

Las carpetas de microservicios estan en `.gitignore` de este repo.

## Repositorios de microservicios

| Servicio     | Repo GitHub                                      | Ruta local                             |
| ------------ | ------------------------------------------------ | -------------------------------------- |
| client-admin | `<ORG>/client-admin`                    | `client-admin/`                        |
| client-user  | `<ORG>/client-user`                     | `client-user/`                         |
| server-admin | `<ORG>/server-admin`                    | `server-admin/`                        |
| server-user  | `<ORG>/server-user`                     | `server-user/`                         |
| auth-node    | `<ORG>/auth-node`                         | `authentication-service/auth-node/`    |
| auth-service | `<ORG>/auth-service`                    | `authentication-service/auth-service/` |
| pg           | `<ORG>/pg` *(opcional)*                 | `pg/`                                  |

## Puertos Docker

| Servicio     | Puerto host       | URL base                                    |
| ------------ | ----------------- | ------------------------------------------- |
| postgres     | 5435              | `localhost:5435`                            |
| mongodb      | 27020             | `localhost:27020`                           |
| auth-service | 5156              | `http://localhost:5156/api/v1`              |
| auth-node    | 3007              | `http://localhost:3007/api/v1`              |
| server-admin | 3009              | `http://localhost:3009/kinalSportsAdmin/v1` |
| server-user  | 3008              | `http://localhost:3008/kinalSportsUser/v1`  |
| client-admin | 5173              | `http://localhost:5173`                     |
| client-user  | 8081, 19000-19002 | Metro bundler                               |

Health checks:

```bash
curl http://localhost:5156/health
curl http://localhost:3007/api/v1/health
curl http://localhost:3009/kinalSportsAdmin/v1/health
curl http://localhost:3008/kinalSportsUser/v1/health
```

## Variables de entorno

- **Stack Docker:** copia `.env.docker.example` a `.env.docker` en esta carpeta y usa `docker compose --env-file .env.docker`.
- **Cada microservicio:** tiene su propio `.env.example` (Node) o `appsettings.*.example.json` (.NET) para desarrollo **sin Docker**. Ver README del servicio.

`JWT_SECRET`, `JWT_ISSUER`, `JWT_AUDIENCE` e `INTERNAL_SERVICE_TOKEN` deben coincidir entre servicios.

## Detener y limpiar

```bash
docker compose down -v --rmi local --remove-orphans
```

Borra contenedores, volúmenes (Postgres/Mongo) e imágenes locales del proyecto `kinal-sports`.

## Desarrollo local (sin Docker)

Tras `./scripts/bootstrap.sh` (o `./scripts/link-local.sh`), entra a cada repo y configura su `.env` local:

```bash
cd client-admin && cp .env.example .env && pnpm install && pnpm dev
cd server-admin && cp .env.example .env && pnpm install && pnpm dev
# ... repetir por servicio
```

**Puertos al conectar desde el host** (con el stack Docker levantado):

| Servicio   | Puerto host |
| ---------- | ----------- |
| PostgreSQL | 5435        |
| MongoDB    | 27020       |

## Documentacion por servicio

Tras el bootstrap, consulta el README dentro de cada carpeta clonada:

- `authentication-service/auth-node/README.md`
- `authentication-service/auth-service/README.md`
- `server-admin/README.md`
- `server-user/README.md`
- `client-admin/README.md`
- `client-user/README.md`

## Autor y licencia

**Braulio Echeverría** — Fundación Kinal, Guatemala (2026)

Proyecto educativo desarrollado en el marco del plan de estudio **PESNUM** de la carrera de **Perito en Computación**, bajo supervisión del Catedrático (PEM).

Licencia **MIT** con fines educativos — texto completo en [LICENSE](LICENSE).
