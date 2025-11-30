## Project setup and environment

### 1. Overview

This repository contains Kubernetes manifests for a pet **microservice architecture (MSA)** project deployed locally on **Minikube**. The goal is to simulate a realistic multi-service environment and gain hands-on experience with Kubernetes concepts such as namespaces, ingress, storage, observability, and CI/CD automation.

The application consists of four core services:

* **web** – frontend (UI)
* **auth** – authentication service (PHP-based)
* **crm** – core business logic
* **infra** – infrastructure stack (Kafka, Loki, Grafana, Prometheus, etc.)

---

### 2. Local environment requirements

* Linux-based system
* Docker Engine **v27+**
* Minikube **v1.37+**
* kubectl **v1.34+**

---

### 3. Minikube setup

#### Start Minikube

```bash
   minikube start \
     --driver=docker \
     --cpus=6 \
     --memory=8192 \
     --addons=ingress,metrics-server,dashboard \
     --kubernetes-version=v1.30.0
```

This ensures a stable and reproducible environment with Nginx ingress, metrics collection, and the Kubernetes dashboard enabled by default.

#### Access the dashboard

```bash
   minikube dashboard
```

The dashboard will automatically open in your browser. Leave the terminal process running.

#### Basic management commands

```
minikube stop       # safely stop the cluster (data preserved)
minikube start      # restart the cluster
minikube delete     # remove cluster completely (data lost)
```

For full cleanup, including local Docker images and volumes:

```bash
   minikube delete --all --purge
   docker system prune -af
```

---

### 4. Project structure

```
k8s/
 ├─ web/
 ├─ auth/
 ├─ crm/
 └─ infra/
     ├─ kafka/
     ├─ loki/
     └─ grafana/
```

Each namespace contains its own manifests, including:

* `namespace.yaml` – namespace definition
* `deployment.yaml` – main deployment spec
* `service.yaml` – internal service definition
* `ingress.yaml` – external access via Nginx ingress
* `secret/` and `config/` – configuration and secret management. No sealed secrets since this is a pet project, yes 
* `kustomization.yaml` – Kustomize aggregation

---

### 5. Deployment workflow

1. **Initialize Minikube** and ensure required addons are active.

2. **Deploy namespaces and components** using Kustomize:

   ```bash
   kubectl apply -k auth/
   ```

3. **Add local host mappings** for ingress access:

   ```bash
   sudo sed -i '/\.local/d' /etc/hosts
   echo "$(minikube ip)  web.local auth.local crm.local infra.local" | sudo tee -a /etc/hosts
   ```

   Run this command after each system reboot if Minikube IP changes.

4. **Access the services via HTTPS** (ingress configured):

   * [https://web.local](https://web.local)
   * [https://auth.local](https://auth.local)
   * [https://crm.local](https://crm.local)
   * [https://infra.local](https://infra.local)

5. **Monitor resources**:

   ```bash
   kubectl top pods -A
   kubectl get all -n auth
   kubectl logs -n auth <pod-name>
   ```

---

### 6. Future extensions

Planned enhancements include:

* Observability stack (Prometheus, Grafana, Loki).
* CI/CD integration with GitHub Actions and ArgoCD.
* Sealed secrets for GitOps-safe credentials (only if I have really nothing to do)

---

### 7. Notes

* This setup is intended for **local development and experimentation**, not production
* All manifests are compatible with Kubernetes **v1.30+**
* Images are pulled from Docker Hub (`alslob/msa-sandbox:*`) (public registry)
* TLS certificates are self-signed for local domains (`*.local`)

---

**Author:** alslob  <br />
**Purpose:** MSA Kubernetes sandbox for educational and infrastructure prototyping.
