
# 📊 AST in Kubernetes (XC AppStack) Deployment Guide  
*Prometheus, OTEL Collector, and Grafana*

---

## 🧭 Overview

This guide provides end-to-end steps to deploy a monitoring stack in Kubernetes using:

- Prometheus (metrics collection)
- OpenTelemetry Collector (data pipeline)
- Grafana (visualization)
- ConfigMaps, Secrets, PVCs, and DockerHub Auth

---

## 1️⃣ Prerequisites and Setup

### 🔹 Clone Repositories

```bash
git clone https://github.com/f5devcentral/application-study-tool.git
cd application-study-tool
# Follow instructions up until the `docker run` step

git clone https://github.com/michelangelodorado/ast-k8s.git
cd ast-k8s/k8s-manifest
```

### 🔹 Apply Secrets

```bash
kubectl apply -f grafana-secret.yaml
kubectl apply -f otel-env-secret.yaml
```

### 🔹 DockerHub Pull Secret

```bash
kubectl create secret docker-registry dockerhub-auth \
  --docker-username=<your_username> \
  --docker-password=<your_password> \
  --docker-email=<your_email> \
  --namespace=m-dorado
```

---

## 2️⃣ Persistent Storage

### 🔸 Prometheus PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: prometheus-pvc
  namespace: m-dorado
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

### 🔸 Grafana PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-pvc
  namespace: m-dorado
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

```bash
kubectl apply -f prometheus-pvc.yaml
kubectl apply -f grafana-pvc.yaml
```

---

## 3️⃣ Prometheus Setup

### 🔸 Create ConfigMap

```bash
kubectl create configmap prometheus-config \
  --from-file=prometheus.yml=services/prometheus/prometheus.yml \
  --namespace=m-dorado
```

### 🔸 Deploy Prometheus Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: m-dorado
spec:
  selector:
    app: prometheus
  ports:
    - port: 9090
      targetPort: 9090
```

```bash
kubectl apply -f prometheus-service.yaml
```

---

## 4️⃣ OpenTelemetry Collector

### 🔸 Create ConfigMap

```bash
kubectl create configmap otel-config \
  --from-file=../services/otel_collector/defaults/bigip-scraper-config.yaml \
  --from-file=../services/otel_collector/pipelines.yaml \
  --from-file=../services/otel_collector/receivers.yaml \
  --namespace=m-dorado \
  --dry-run=client -o yaml > otel-configmap.yaml

kubectl apply -f otel-configmap.yaml
```

---

## 5️⃣ Grafana Setup

### 🔸 Secret (`grafana-static`)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: grafana-static
  namespace: m-dorado
type: Opaque
stringData:
  GF_SECURITY_ADMIN_USER: admin
  GF_SECURITY_ADMIN_PASSWORD: admin
  GF_SECURITY_ALLOW_EMBEDDING: "true"
  GF_SECURITY_ALLOW_ORIGIN: "*"
  GF_SECURITY_COOKIE_SAMESITE: none
  GF_SECURITY_COOKIE_SECURE: "true"
```

```bash
kubectl apply -f grafana-static-secret.yaml
```

### 🔸 Create ConfigMaps for Dashboards & Datasources

```bash
kubectl create configmap grafana-device-dashboards \
  --from-file=../services/grafana/provisioning/dashboards/bigip/device \
  --namespace=m-dorado

kubectl create configmap grafana-fleet-dashboards \
  --from-file=../services/grafana/provisioning/dashboards/bigip/fleet \
  --namespace=m-dorado

kubectl create configmap grafana-profile-dashboards \
  --from-file=../services/grafana/provisioning/dashboards/bigip/profile \
  --namespace=m-dorado

kubectl create configmap grafana-otel-dashboards \
  --from-file=../services/grafana/provisioning/dashboards/otel-collector \
  --namespace=m-dorado

kubectl create configmap grafana-dashboards-yaml \
  --from-file=../services/grafana/provisioning/dashboards \
  --namespace=m-dorado

kubectl create configmap grafana-datasources \
  --from-file=../services/grafana/provisioning/datasources \
  --namespace=m-dorado
```

---

## ✅ Final Verification

```bash
kubectl get pods -n m-dorado
kubectl get svc -n m-dorado
```
