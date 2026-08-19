## Pet Project — Voting App на Kubernetes

> Проект полностью локальный: разворачивается на minikube внутри Ubuntu VM (VirtualBox), не на облачном кластере.

Учебный pet-проект: multi-tier приложение для голосования, полностью описанное как инфраструктура-как-код и развёрнутое в Kubernetes.

## Архитектура

Vote (Flask) → Redis (очередь) → Worker (.NET) → Postgres (StatefulSet) → Result (Node.js)

Vote и Result доступны снаружи через общий Ingress. Подробное описание зависимостей между манифестами — в [`docs/architecture.md`](docs/architecture.md).

## Стек

- Kubernetes (minikube, Docker driver)
- Готовые образы: `dockersamples/examplevotingapp_vote`, `_worker`, `_result`
- Redis, PostgreSQL 15 (Alpine)
- Ingress (маршрутизация по хостам `vote.local` / `result.local`)
- Все объекты изолированы в namespace `voting-app`
- Readiness/liveness probes и resource requests/limits для всех компонентов

## Структура репозитория

- manifests/namespace.yaml — создаёт namespace `voting-app`
- manifests/vote/ — Deployment + Service для Vote
- manifests/redis/ — Deployment + Service для Redis
- manifests/worker/ — Deployment для Worker
- manifests/postgres/ — ConfigMap, Secret, StatefulSet, Service
- manifests/result/ — Deployment + Service для Result
- manifests/ingress.yaml — маршрутизация Vote/Result
- docs/architecture.md — описание зависимостей между манифестами

## Запуск

```bash
minikube addons enable ingress
kubectl apply -f manifests/namespace.yaml
kubectl apply -f manifests/ -R
```

Добавить записи в hosts-файл — протестировано и подтверждено рабочим внутри самой Ubuntu VM:

echo "$(minikube ip) vote.local result.local" | sudo tee -a /etc/hosts

Доступ с браузера на Windows-хосте отдельно не проверялся — зависит от режима сети VirtualBox (NAT/Bridged/Host-only), может потребовать дополнительной настройки.

## Статус

v1.1.0 — полностью рабочий деплой с readiness/liveness probes и resource limits, протестирован через Ingress внутри Ubuntu VM (minikube). Проходит полный цикл: голос → очередь → обработка → база → результат.

## Дальнейшие шаги

- CI/CD (GitHub Actions) не внедрён — можно сделать автоматическую проверку манифестов на каждый push, дублирование репозитория в GitLab
- Helm chart — сознательно не делаем: для одного локального окружения не даёт практической пользы, добавляет сложность без причины
