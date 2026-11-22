# Стек мониторинга сервера и контейнеров (Prometheus + Grafana + Alertmanager)

Готовая к продакшену система мониторинга и оповещений, развёрнутая на облачном VPS (Ubuntu 24.04) через Docker Compose, закрытая за Nginx Reverse Proxy с HTTPS и интегрированная с уведомлениями в Telegram.

## Архитектура

```text
[ Хост-система и Docker Engine ]
       │
       ├── Node Exporter (:9100) ──┐
       └── cAdvisor (:8080) ───────┼──▶ [ Prometheus (:9090) ] ──▶ [ Alertmanager (:9093) ] ──▶ Telegram-бот
                                   │             │
                                   │             ▼
[ Браузер (HTTPS) ] ──▶ [ Nginx Reverse Proxy ] ──▶ [ Grafana (127.0.0.1:3000) ]
```

## Компоненты стека

- **Prometheus (`v2.55.1`)**: Собирает метрики каждые 15 секунд по pull-модели и хранит историю показателей за последние 15 дней (`--storage.tsdb.retention.time=15d`).
- **Node Exporter (`v1.8.2`)**: Снимает аппаратные и системные метрики сервера (загрузка CPU, использование RAM, свободное место на NVMe-диске, сетевой трафик) через системные директории, примонтированные только для чтения (`/proc`, `/sys`, `/`).
- **cAdvisor (`v0.49.1`)**: Отслеживает потребление CPU, оперативной памяти и дискового ввода-вывода каждым запущенным Docker-контейнером по отдельности.
- **Grafana (`v11.3.0`)**: Визуализирует метрики на дашбордах в реальном времени (шаблон *Node Exporter Full*, ID `1860`), опубликована по пути `/grafana/` за Nginx с поддержкой WebSockets и TLS.
- **Alertmanager (`v0.27.0`)**: Обрабатывает правила тревог (`TargetDown`, `HighDiskUsage`, `HighMemoryUsage`) и отправляет уведомления о сбоях (`Firing`) и восстановлении (`Resolved`) в Telegram.

## Безопасность и стандарты

- **Сетевая изоляция**: Веб-интерфейсы (`3000`, `9090`, `9093`) привязаны строго к локальному интерфейсу `127.0.0.1`; экспортеры отдают метрики только внутри изолированной сети Docker.
- **Минимальные привилегии**: Системные директории хоста (`/proc`, `/sys`, `/var/lib/docker`) проброшены в контейнеры с флагом `:ro` (только чтение).
- **Защита секретов**: Файлы с паролями и токенами (`.env`, `alertmanager/alertmanager.yml`) исключены через `.gitignore`; в репозитории хранятся только безопасные шаблоны `.example`.

## Быстрый старт

1. Клонируйте репозиторий и создайте рабочие файлы конфигурации из шаблонов:
   ```bash
   git clone [https://github.com/yyaauura7/monitoring-prometheus-grafana.git](https://github.com/yyaauura7/monitoring-prometheus-grafana.git)
   cd monitoring-prometheus-grafana
   cp .env.example .env
   cp alertmanager/alertmanager.yml.example alertmanager/alertmanager.yml
   ```
2. Укажите надёжный пароль администратора `GF_SECURITY_ADMIN_PASSWORD` в `.env`, а также `bot_token` и `chat_id` в `alertmanager/alertmanager.yml`.
3. Проверьте синтаксис конфигурации и запустите стек:
   ```bash
   docker compose config --quiet
   docker compose up -d
   ```
