# Docker-проект

Магазин пельменей: Go-бэкенд + Vue-фронтенд, три контейнера через Docker Compose.

## Сервисы

- `backend` — Go API, слушает 8081
- `frontend` — Vue SPA, nginx-unprivileged на 8080
- `nginx` — reverse-proxy и балансировщик, наружу на 80

## Запуск

Локально с билдом:

```bash
docker compose --profile dev up --build
```

С готовыми образами из Docker Hub:

```bash
DOCKER_USER=<your_user> docker compose --profile prod up -d
```

Открыть http://localhost/

## Масштабирование

```bash
docker compose --profile dev up --build --scale backend=3
```

nginx балансирует через `upstream backend_upstream` (round-robin).

## Образы

Все образы собраны через multi-stage:

- `backend` — `golang:1.22-alpine` → `alpine:3.20` (~20 МБ)
- `frontend` — `node:18-alpine` → `nginxinc/nginx-unprivileged:alpine` (~50 МБ)
- `nginx` — `nginxinc/nginx-unprivileged:alpine` (~50 МБ)

## Безопасность

- multi-stage сборки, минимальные базовые образы
- non-root пользователи во всех контейнерах
- `read_only: true`, `cap_drop: ALL`, `no-new-privileges:true`
- `tmpfs` для временных каталогов
- лимиты CPU и памяти
- изолированные сети `frontend_net` и `backend_net`
- `.dockerignore` исключает мусор из контекста сборки
- секреты через Docker Secrets и GitHub Secrets

## Volumes

- `backend_data` — данные бэкенда
- `nginx_logs` — логи nginx-роутера

## Secrets

В compose используется Docker Secret `docker_password`, файл `./secrets/docker_password.txt` подгружается в контейнер `nginx` как `/run/secrets/docker_password`. Файл закоммичен с плейсхолдером, реальные значения в репо не лежат.

В CI/CD секреты `DOCKER_USER` и `DOCKER_PASSWORD` хранятся в GitHub Secrets.

## Trivy

В CI после пуша образов запускается job `trivy-scan`. Локально:

```bash
docker run --rm aquasec/trivy:latest image <user>/docker-project-backend:latest
docker run --rm aquasec/trivy:latest image <user>/docker-project-frontend:latest
docker run --rm aquasec/trivy:latest image <user>/docker-project-nginx:latest
```

## Переменные

| Переменная        | По умолчанию | Описание                              |
|-------------------|--------------|---------------------------------------|
| `DOCKER_USER`     | `sladkyi`    | Префикс имени образа в Docker Hub     |
| `NGINX_HOST_PORT` | `80`         | Порт хоста для nginx                  |
| `VUE_APP_API_URL` | `/api`       | URL API, запекается в SPA при сборке  |
