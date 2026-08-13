# Avatars Service

Сервис загрузки и обработки аватаров на Go. Обеспечивает загрузку изображений, хранение в S3-совместимом хранилище, асинхронное генерирование превью через RabbitMQ, и REST API для управления аватарами пользователей.

## Архитектура

- **Server** (`cmd/server`) — HTTP-сервер на Echo: API, здоровье, веб-страницы
- **Worker** (`cmd/worker`) — консьюмер очей RabbitMQ: генерация превью изображений
- **PostgreSQL** — метаданные аватаров (id, user_id, статусы, S3-ключи)
- **S3 / MinIO** — хранение оригиналов и превью
- **RabbitMQ** — асинхронная очередь задач на обработку

### API endpoints

| Метод   | Путь                                     | Описание                |
| ------- | ---------------------------------------- | ----------------------- |
| GET     | `/health`                                | Здоровье сервиса        |
| POST    | `/api/v1/avatars`                       | Загрузить аватар        |
| GET     | `/api/v1/avatars/:avatar_id`            | Получить аватар         |
| DELETE  | `/api/v1/avatars/:avatar_id`            | Удалить аватар          |
| GET     | `/api/v1/avatars/:avatar_id/metadata`   | Метаданные аватара      |
| GET     | `/api/v1/users/:user_id/avatar`         | Текущий аватар          |
| DELETE  | `/api/v1/users/:user_id/avatar`         | Удалить аватар          |
| GET     | `/api/v1/users/:user_id/avatars`        | Список аватаров         |
| GET     | `/web/upload`                            | Страница загрузки       |
| GET     | `/web/gallery/:user_id`                 | Галерея пользователя    |

Запросы к `/api/v1/avatars` (POST/DELETE) требуют заголовок `X-User-Id`.

## Локальный запуск

### Требования

- Go 1.26
- Docker и Docker Compose

### 1. Запуск инфраструктуры

```bash
docker compose up -d postgres rabbitmq minio
```

### 2. Настройка переменных окружения

```bash
export S3_ACCESS_KEY=minioadmin
export S3_SECRET_KEY=minioadmin
export CORS_ALLOWED_ORIGINS="http://localhost:3000"
```

Остальные переменные имеют значения по умолчанию, соответствующие локальной инфраструктуре (`DB_HOST=localhost`, `S3_ENDPOINT=localhost:9000`, `RMQ_URL=amqp://guest:guest@localhost:5672`).

### 3. Запуск сервера и воркера

```bash
go run ./cmd/server
go run ./cmd/worker
```

Сервер стартует на `:8080`, автоматически применяет миграции и подключается к S3 и RabbitMQ.

### Альтернатива: запуск всего через Docker Compose

```bash
docker compose up --build
```

Это поднимет все сервисы (PostgreSQL, RabbitMQ, MinIO, server, worker) одной командой. После первого запуска создайте бакет в MinIO:

```bash
mc alias set local http://localhost:9000 minioadmin minioadmin
mb local/avatars
```

## Тесты

```bash
go test ./...
```
