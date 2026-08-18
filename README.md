# Pet Project — Voting App на Kubernetes

Учебный pet-проект: multi-tier приложение для голосования, полностью описанное как инфраструктура-как-код и развёрнутое в Kubernetes.

## Архитектура

Vote (Flask) → Redis (очередь) → Worker (.NET) → Postgres (StatefulSet) → Result (Node.js)


Vote и Result доступны снаружи через общий Ingress. Подробное описание зависимостей между манифестами — в [`docs/architecture.md`](docs/architecture.md).

## Стек

- Kubernetes (minikube, Docker driver)
- Готовые образы: `dockersamples/examplevotingapp_vote`, `_worker`, `_result`
- Redis, PostgreSQL 15 (Alpine)
- Ingress (маршрутизация по хостам `vote.local` / `result.local`)

## Структура репозитория

\`\`\`
manifests/
├── vote/
├── redis/
├── worker/
├── postgres/
├── result/
└── ingress.yaml
docs/
└── architecture.md
\`\`\`

## Запуск

```bash
minikube addons enable ingress
kubectl apply -f manifests/ -R
```

Добавить в `hosts` (на Windows-хосте):

<IP minikube> vote.local
<IP minikube> result.local


## Статус

В разработке — все манифесты написаны, идёт тестовый деплой.
