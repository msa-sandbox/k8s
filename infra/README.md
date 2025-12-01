# Infra Monitoring Stack (Draft)

This setup provides a lightweight monitoring and logging stack inside Kubernetes using **Prometheus**, **Grafana**, and **Loki**.

---

## Prometheus

Collects metrics from cluster components and services.

* Configured via `prometheus-config` ConfigMap (`prometheus.yml`)
* Scrapes:

    * Itself (`localhost:9090`)
    * `auth` service (`app-service.auth.svc.cluster.local:9253`)
    * Node exporter
    * Kube-state-metrics
* Exposed through Ingress:
  `https://infra.local/prometheus`
* When adding to Grafana:
  Data source URL → `https://infra.local/prometheus`
  Enable **Skip TLS certificate validation**

---

## Loki

Centralized log storage (Grafana Loki).

* Exposed as `http://loki.infra.svc.cluster.local:3100`
* Logs collected by **Promtail** DaemonSet from `/var/log/containers`
* Grafana data source (when working):
  `http://loki.infra.svc.cluster.local:3100`

---

## Grafana

Visualization layer for metrics and logs.

* Access via: `https://infra.local/grafana`
  Default credentials: `admin / admin`
* Data sources:

    * Prometheus → `https://infra.local/prometheus`
    * Loki (optional) → `http://loki.infra.svc.cluster.local:3100`
* Persistent storage via PVC (`grafana-pvc`) — dashboards and settings survive restarts

---

## Notes

* TLS handled at Ingress level (`infra-tls`)
* All components deployed in `infra` namespace
* Loki/Promtail currently under testing
