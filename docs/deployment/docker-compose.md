# Docker Compose - Lokale Ontwikkelomgeving

Snel opzetten van volledige development stack.

```bash
git clone https://github.com/gemeentearnhem/schulddienstverlening-register.git
cd schulddienstverlening-register
docker-compose up
```

**Beschikbaar na opstart**:
- API: http://localhost:8000
- PostgreSQL: localhost:5432
- Keycloak (Auth): http://localhost:8080 (admin/password)
- Adminer (DB UI): http://localhost:8081

---

## docker-compose.yml

```yaml
version: '3.9'

services:
  # PostgreSQL Database
  db:
    image: postgres:15-alpine
    container_name: schulddienstverlening-db
    environment:
      POSTGRES_USER: schulddienstverlening
      POSTGRES_PASSWORD: dev-password-change-me
      POSTGRES_DB: schulddienstverlening
      POSTGRES_INITDB_ARGS: "--encoding=UTF8"
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data
      - ./docs/database/init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U schulddienstverlening"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis Cache
  redis:
    image: redis:7-alpine
    container_name: schulddienstverlening-redis
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Keycloak (OAuth2/OIDC Provider)
  keycloak:
    image: keycloak/keycloak:latest
    container_name: schulddienstverlening-auth
    environment:
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: password
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://db:5432/keycloak
      KC_DB_USERNAME: schulddienstverlening
      KC_DB_PASSWORD: dev-password-change-me
    ports:
      - "8080:8080"
    command: start-dev
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/"]
      interval: 30s
      timeout: 10s
      retries: 3

  # FastAPI Backend
  api:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: schulddienstverlening-api
    environment:
      DATABASE_URL: postgresql://schulddienstverlening:dev-password-change-me@db:5432/schulddienstverlening
      REDIS_URL: redis://redis:6379
      KEYCLOAK_URL: http://keycloak:8080
      JWT_SECRET: your-secret-key-change-in-prod
      LOG_LEVEL: DEBUG
      ENVIRONMENT: development
    ports:
      - "8000:8000"
    volumes:
      - ./backend:/app
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
      keycloak:
        condition: service_healthy
    command: uvicorn main:app --host 0.0.0.0 --port 8000 --reload

  # Frontend (React/Vue)
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    container_name: schulddienstverlening-frontend
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
    depends_on:
      - api
    environment:
      REACT_APP_API_URL: http://localhost:8000
      REACT_APP_AUTH_URL: http://localhost:8080

  # Adminer (Database UI)
  adminer:
    image: adminer:latest
    container_name: schulddienstverlening-adminer
    ports:
      - "8081:8080"
    depends_on:
      - db

volumes:
  db_data:

networks:
  default:
    name: schulddienstverlening-network
```

---

## Environment Variables

Create `.env.local`:

```env
# Database
DATABASE_URL=postgresql://schulddienstverlening:password@db:5432/schulddienstverlening
DATABASE_POOL_SIZE=10

# Redis
REDIS_URL=redis://redis:6379/0

# Auth (Keycloak)
KEYCLOAK_URL=http://localhost:8080
KEYCLOAK_REALM=schulddienstverlening
KEYCLOAK_CLIENT_ID=schulddienstverlening-api
JWT_SECRET=my-secret-key-min-32-chars

# Logging
LOG_LEVEL=DEBUG
LOG_FORMAT=json

# Features
FEATURE_AUDIT_LOGGING=true
FEATURE_GDPR_EXPORT=true
FEATURE_EMAIL_NOTIFICATIONS=false

# Performance
API_RATE_LIMIT=1000
DB_QUERY_TIMEOUT=30
```

---

## Initial Setup

### 1. Keycloak Configuration

After Keycloak starts (http://localhost:8080):

1. Login: admin / password
2. Create Realm: "schulddienstverlening"
3. Create Clients:
   - `schulddienstverlening-api` (bearer-only)
   - `web-app` (public, for frontend)
4. Create Roles:
   - burger
   - schuldhulpverlener
   - gemeente
   - admin
5. Create Test User:
   - Username: testburger
   - Email: burger@test.nl
   - Roles: burger

### 2. Database Initialization

SQL init script in `docs/database/init.sql`:

```sql
-- Tables created automatically from migrations
-- Run Alembic:
docker-compose exec api alembic upgrade head

-- Load test data (optional):
docker-compose exec db psql -U schulddienstverlening < tests/data/fixtures.sql
```

### 3. API Startup

Check logs:

```bash
docker-compose logs -f api
```

Expected: "Application startup complete" on port 8000

---

## Usage

### Test API

```bash
# Get list of dossiers (no auth required in dev)
curl -H "accept: application/json" \
  http://localhost:8000/v1/schulddossiers

# With authentication
curl -H "Authorization: Bearer <token>" \
  http://localhost:8000/v1/schulddossiers/{id}
```

### Database Access

**Via Adminer**: http://localhost:8081
- Server: db
- User: schulddienstverlening
- Password: dev-password-change-me
- Database: schulddienstverlening

**Via psql CLI**:

```bash
docker-compose exec db psql -U schulddienstverlening -d schulddienstverlening
```

---

## Development Workflow

### Making Code Changes

```bash
# Backend changes auto-reload (uvicorn --reload)
# Frontend changes auto-refresh (React hot reload)

# After DB schema changes:
docker-compose exec api alembic revision --autogenerate -m "Change description"
docker-compose exec api alembic upgrade head

# After dependency changes:
docker-compose build
docker-compose up
```

### Viewing Logs

```bash
docker-compose logs -f api          # API logs
docker-compose logs -f db           # Database logs
docker-compose logs -f frontend     # Frontend logs
docker-compose logs                 # All services
```

### Running Tests

```bash
docker-compose exec api pytest tests/
docker-compose exec api pytest --cov=app tests/  # with coverage
```

---

## Cleanup

```bash
# Stop services
docker-compose stop

# Remove containers
docker-compose down

# Remove data volumes
docker-compose down -v

# Rebuild images
docker-compose build --no-cache
```

---

## Production Deployment

For production, use:
- Kubernetes (with Helm charts)
- Azure App Service
- Docker Swarm

See `deployment/kubernetes-hints.md` for details.

---

**Document version**: 1.0 | **Last update**: Mei 2026
