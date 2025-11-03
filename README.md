# 🧩 Name Service — Kubernetes IaC (Dev + Prod)

This repository contains **Kubernetes manifests** for deploying the **Name Service** application and its supporting **PostgreSQL database**.
The setup demonstrates a production-ready **Infrastructure-as-Code** approach using **Kustomize** to separate configurations for *development* and *production* environments.

---

## ✅ Acceptance Criteria

* ConfigMap mounted as a file inside the container ✅
* Secrets provided as environment variables ✅
* Environment-specific configuration via **Kustomize overlays** ✅
* Correct health probes on `/status` endpoint ✅

---

## 📂 Project Structure

```
k8s/
├── base/
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── postgres-pvc.yaml
│   ├── postgres-service.yaml
│   ├── postgres-statefulset.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/
    │   └── kustomization.yaml
    └── prod/
        ├── kustomization.yaml
        └── configmap-prod.yaml
```

---

## 🧱 Base Configuration (`k8s/base`)

The base directory defines shared infrastructure resources:

| File                          | Description                                                                                       |
| ----------------------------- | ------------------------------------------------------------------------------------------------- |
| **configmap.yaml**            | Provides application configuration (`config.yaml`), mounted into the container                    |
| **secret.yaml**               | Stores credentials and API keys (`stringData`, automatically base64-encoded)                      |
| **postgres-statefulset.yaml** | Deploys PostgreSQL with persistent storage and probes                                             |
| **postgres-service.yaml**     | Internal ClusterIP service exposing PostgreSQL on port 5432                                       |
| **deployment.yaml**           | Main application deployment (includes probes, volume mounts, init container to wait for Postgres) |
| **service.yaml**              | ClusterIP service exposing the application on port 8081                                           |
| **kustomization.yaml**        | Aggregates all resources into a single Kustomize layer                                            |

---

## 🎛 Overlays

### Development (`k8s/overlays/dev`)

* Uses base configuration as is
* Logger level set to `debug`
* Ideal for local clusters such as Minikube or kind

### Production (`k8s/overlays/prod`)

* Adds a `configmap-prod.yaml` patch that changes only one key parameter:

  * Logger level → `info`
* Uses `patchesStrategicMerge` (deprecated but still supported; can migrate to `patches` / JSON6902 later)

---

## ▶️ How to Deploy

### 🧪 Development Environment

```bash
kubectl apply -k k8s/overlays/dev
kubectl get pods
kubectl port-forward svc/name-service 8081:8081
curl -i http://127.0.0.1:8081/status  # expect 200 OK
```

### 🚀 Production Environment

```bash
kubectl apply -k k8s/overlays/prod
kubectl get pods
kubectl port-forward svc/name-service 8081:8081
curl -i http://127.0.0.1:8081/status  # expect 200 OK, logs show level=info
```

---

## 🔐 Secrets and Configs

* **Secrets (`stringData`)**: stored in plain text and automatically converted to base64 by Kubernetes — secure and readable.
* **ConfigMap as Volume**: used for structured YAML/JSON configs that apps load as files.
* **Env Vars**: best for small scalar values like credentials or flags.

---

## 🧠 Why Kustomize?

Kustomize enables maintaining **one base configuration** and applying lightweight environment-specific differences.
This avoids duplication, ensures consistency, and keeps production-grade YAML clean and reusable.

---

## 📜 Example Snippet — Deployment Highlights

```yaml
containers:
  - name: app
    image: name-service:dev
    ports:
      - name: http
        containerPort: 8081
    volumeMounts:
      - name: config-volume
        mountPath: /app/config.yaml
        subPath: config.yaml
    readinessProbe:
      httpGet:
        path: /status
        port: 8081
      initialDelaySeconds: 5
      periodSeconds: 5
```

---

## 🧩 Key Features

* PostgreSQL with persistent storage (PVC)
* Application health checks (readiness/liveness on `/status`)
* Init container waiting for PostgreSQL availability
* Secure handling of environment variables and secrets
* Configurable through Kustomize overlays

---
