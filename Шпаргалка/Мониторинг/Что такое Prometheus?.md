> **Prometheus** — это система мониторинга и хранения метрик, ориентированная на **pull-модель сбора данных** и **time-series** формат.
> 
> Он периодически опрашивает сервисы по `/metrics` (в формате Prometheus exposition), сохраняет значения во внутреннюю базу и позволяет строить **алерты, графики и дашборды** через **PromQL** и **Grafana**.
> 
> В Т-Банке мы используем Prometheus как основной источник метрик — он собирает данные со всех микросервисов (FastAPI, Kafka, PostgreSQL, RabbitMQ, Kubernetes).
> 
> Метрики идут в **Alertmanager** (для алертов) и **Grafana** (для визуализации и SLO).

---

## 🔹 Основные принципы Prometheus

1. 📥 **Pull-модель** — Prometheus сам опрашивает таргеты по HTTP (`/metrics`), а не ждёт, пока сервисы что-то “пушнут”.
    
2. 🧩 **Экспортеры** — адаптеры для сбора метрик из систем вроде PostgreSQL, Kafka, Redis, RabbitMQ, Kubernetes.
    
3. 🧮 **PromQL** — язык запросов для агрегации и анализа метрик.
    
4. 📊 **Time-series база** — все данные хранятся как `(metric_name, labels, timestamp, value)`.
    
5. ⚡ **Alertmanager** — компонент для правил и уведомлений (Slack, PagerDuty).
    
6. 🌍 **Service discovery** — динамически находит поды и сервисы в Kubernetes.
    

---

## 🔹 Пример из твоего проекта (Т-Банк)

> У нас каждый микросервис (на FastAPI) отдаёт endpoint `/metrics` с данными через библиотеку `prometheus_client`.  
> Prometheus забирает эти данные раз в 15 секунд, а потом метрики агрегируются и визуализируются в Grafana.
> 
> Мы измеряем ключевые **SLI** — error rate, latency, throughput, consumer lag, queue depth и т.д.
> 
> Например:
> 
> - **Kafka consumer lag** — через `kafka_exporter`;
>     
> - **RabbitMQ queue depth** — через `rabbitmq_exporter`;
>     
> - **Postgres performance** — через `postgres_exporter`;
>     
> - **CPU, memory, network** — через `node_exporter` и `kube-state-metrics`.
>     

---

## 🔹 Пример метрик из FastAPI

```
from prometheus_client import Counter, Histogram, generate_latest, CONTENT_TYPE_LATEST
from fastapi import FastAPI, Request, Response
from time import perf_counter

app = FastAPI()

REQS = Counter("http_requests_total", "Total HTTP requests", ["method", "path", "status"])
LATENCY = Histogram("http_request_duration_seconds", "Request latency", ["method", "path"])

@app.middleware("http")
async def metrics_middleware(request: Request, call_next):
    start = perf_counter()
    response = await call_next(request)
    duration = perf_counter() - start
    REQS.labels(request.method, request.url.path, response.status_code).inc()
    LATENCY.labels(request.method, request.url.path).observe(duration)
    return response

@app.get("/metrics")
def metrics():
    return Response(generate_latest(), media_type=CONTENT_TYPE_LATEST)

```

> Теперь Prometheus может просто забрать эти данные по `GET /metrics`.

---

## 🔹 Пример запроса в PromQL

```
# Ошибки за последние 5 минут
rate(http_requests_total{status=~"5.."}[5m])

# 95-й перцентиль латентности
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))

```

---

## 🔹 Пример алерта

```
groups:
- name: api_alerts
  rules:
  - alert: HighErrorRate
    expr: rate(http_requests_total{status=~"5.."}[5m]) 
          / rate(http_requests_total[5m]) > 0.02
    for: 10m
    labels:
      severity: critical
    annotations:
      summary: "Ошибка API > 2%"

```
> Алерт попадёт в **Alertmanager**, оттуда — в **Slack** и **PagerDuty**.

---

## 🔹 Как Prometheus интегрирован в экосистему

```
[Service / App] → /metrics
     ↓
[Prometheus] — собирает, хранит
     ↓
[Alertmanager] — алерты
[Grafana] — визуализация
[Tempo/Jaeger] — трейсы
[Loki] — логи

```

> Всё это даёт полную **observability-тройку**: метрики, логи, трейсы.

---

## 💬 Как можно красиво завершить ответ

> В целом, Prometheus — это ядро системы мониторинга в современном DevOps-стеке.  
> Он обеспечивает метрики для SLO, алерты для инцидентов и данные для анализа производительности.
> 
> В Т-Банке мы используем его повсеместно — от API и брокеров до инфраструктуры Kubernetes и бизнес-метрик.