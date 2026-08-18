# Архитектура проекта

## Поток данных

Vote (Flask) → Redis (очередь) → Worker (.NET) → Postgres (StatefulSet) → Result (Node.js)

Vote и Result — фронтенды без своего состояния, доступны снаружи через Ingress (`vote.local`, `result.local`).

## Зависимости между манифестами

### Postgres
- StatefulSet ссылается на ConfigMap `postgres-config-map` и Secret `postgres-secret` через `envFrom`
- StatefulSet.spec.serviceName должен совпадать с metadata.name headless Service (`postgres-service`)
- Service — headless (`clusterIP: None`), так как StatefulSet требует стабильного DNS-имени для Pod'а

### Почему у компонентов разное число реплик
- Vote, Result — stateless, реплик может быть несколько (2)
- Worker — конкурирует за сообщения в очереди Redis, поэтому реплика одна (1)
- Postgres — хранит данные, без настроенной репликации реплика одна (1)
- Redis — без кластеризации реплика одна (1)

## Требования перед деплоем

- `minikube addons enable ingress`
- Добавить `vote.local` и `result.local` в файл hosts на Windows-хосте
