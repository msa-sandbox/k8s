# Kafka module (infra/kafka)

## Overview

Deploys a single-node **Kafka (KRaft mode)** with **Kafdrop UI** inside the `infra` namespace. Intended for local Minikube setup and internal service-to-service messaging. Data is ephemeral and lost when the cluster is deleted.

## Components

* **Kafka broker** – main message broker (no persistence).
* **Init job** – creates topics after broker startup.
* **Kafdrop** – simple web UI to inspect topics and messages.
* **Ingress** – exposes Kafdrop at `https://infra.local/kafdrop`.

## Workflow

1. Apply module:

   ```bash
   kubectl apply -k infra/kafka/
   ```
2. Kafka starts first, then an init job waits for readiness and creates topics.
3. Kafdrop connects to the broker using in-cluster address (`kafka.infra.svc.cluster.local:9092`).

## TLS & Networking

* TLS is handled at the **Ingress** level via `infra-tls` secret.
* Internal Kafka communication remains **PLAINTEXT**.
* Services from other namespaces connect via DNS: `kafka.infra.svc.cluster.local:9092`.

## Maintenance

* Re-run topic creation:

  ```bash
  kubectl delete job kafka-init-job -n infra && kubectl apply -f infra/kafka/job-init.yaml
  ```
