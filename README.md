# Infrastructure Configuration

Local infrastructure for TASQALENT services using Docker Compose:

- Caches & databases: Redis, MongoDB, MySQL, PostgreSQL
- Messaging: RabbitMQ
- Observability: Elasticsearch + Kibana
- Workers: Notification service Celery worker

This stack is meant for **local development**, not production.

## Services

Defined in `docker-compose.yaml`:

- `redis` → cache
- `mongodb` → document store
- `mysql` → for auth service
- `postgres` → for review service
- `rabbitmq` → message broker (`amqp://tasqalent:tq_password@rabbitmq:5672/`)
- `elasticsearch` → log & data indexing (with security enabled)
- `kibana` → UI for Elasticsearch
- `notification-worker` → Celery worker for `tq-notification-service`

## Quick start

From `tq-infra/`:

```bash
# Start everything (recommended when working on notifications)
docker-compose up -d rabbitmq elasticsearch kibana notification-worker
```

Check containers:

```bash
docker ps
```

## Access points

- **RabbitMQ UI**: `http://localhost:15672`
  - User/pass from `docker-compose.yaml` (`RABBITMQ_DEFAULT_USER`, `RABBITMQ_DEFAULT_PASS`)
- **Elasticsearch**: `http://localhost:9200`
  - User: `elastic`, password: `elastic_password` (see `docker-compose.yaml`)
- **Kibana**: `http://localhost:5601`
  - Connects to Elasticsearch inside Docker using `kibana.yml` config.

## Working on the notification service

Typical workflow:

1. Start infra:

   ```bash
   cd tq-infra
   docker-compose up -d rabbitmq elasticsearch kibana notification-worker
   ```

2. In `tq-notification-service`, run tests and send a test notification:

   ```bash
   cd ../tq-notification-service
   pytest -q
   python -m scripts.send_test_notification
   ```

3. Observe:
   - **Worker logs**: `docker logs -f notification_worker`
   - **Elasticsearch docs** (e.g., index `notifications`):
     ```bash
     curl -u elastic:elastic_password "http://localhost:9200/notifications/_search?pretty"
     ```
   - **Kibana**: build visualizations/dashboards over the `notifications` index.

This setup gives you a complete local loop: publish a notification, process it in the worker, index into Elasticsearch, and inspect via Kibana.
