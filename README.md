# Docker-проект

Магазин пельменей: Go-бэкенд + Vue-фронтенд, три контейнера через Docker Compose.

## Сервисы

- `backend` — Go API на порту 8081 (distroless, non-root)
- `frontend` — Vue SPA, nginx-unprivileged на 8080
- `nginx` — reverse-proxy и балансировщик на 80

## Запуск

Локально:

```bash
docker compose --profile dev up --build
```

С образами из Docker Hub:

```bash
DOCKER_USER=<your_user> docker compose --profile prod up -d
```

Открыть http://localhost/

## Масштабирование

```bash
docker compose --profile dev up --build --scale backend=3
```

nginx балансирует запросы между репликами через upstream.

## Безопасность

- multi-stage сборки, минимальные базовые образы (distroless, nginx-unprivileged)
- non-root пользователи во всех контейнерах
- `read_only`, `cap_drop: ALL`, `no-new-privileges`
- `tmpfs` для временных каталогов
- лимиты CPU и памяти
- изолированные сети (frontend и backend)
- секреты в GitHub Secrets

## Trivy

В CI запускается `trivy-scan` после пуша в Docker Hub. Локально:

```bash
docker run --rm aquasec/trivy:latest image <user>/docker-project-backend:latest
```

## Переменные

| Переменная        | Значение по умолчанию |
|-------------------|-----------------------|
| `DOCKER_USER`     | `sladkyi`             |
| `NGINX_HOST_PORT` | `80`                  |
| `VUE_APP_API_URL` | `/api`                |
