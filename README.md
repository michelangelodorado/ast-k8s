
# 📊 Kubernetes Deployment for Prometheus, OTEL Collector, and Grafana

This guide provides complete setup instructions to deploy Prometheus, OpenTelemetry Collector, and Grafana on a Kubernetes cluster using ConfigMaps, Secrets, Persistent Volumes, and DockerHub authentication.

## 📦 Setup Steps

### 1. Clone Repositories

```bash
git clone https://github.com/f5devcentral/application-study-tool.git
cd application-study-tool
```

### 2. Follow Base Instructions

Perform the original instructions from the main repo up until the `docker run` step.

### 3. Clone AST K8s add-on

```bash
git clone https://github.com/michelangelodorado/ast-k8s.git
cd ast-k8s/k8s-manifest
```

### 4. Apply Secrets

```bash
kubectl apply -f grafana-secret.yaml
kubectl apply -f otel-env-secret.yaml
```

## 📦 PersistentVolumeClaims

### `prometheus-pvc.yaml`
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

### `grafana-pvc.yaml`
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

## 🔐 DockerHub Pull Secret

```bash
kubectl create secret docker-registry dockerhub-auth   --docker-username=<your_username>   --docker-password=<your_password>   --docker-email=<your_email>   --namespace=m-dorado
```

## 📈 Prometheus

### Create ConfigMap

```bash
kubectl create configmap prometheus-config   --from-file=prometheus.yml=services/prometheus/prometheus.yml   --namespace=m-dorado
```

### Service

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

## 📡 OTEL Collector

### OpenTelemetry ConfigMap

```bash
kubectl create configmap otel-config \
  --from-file=../services/otel_collector/defaults/bigip-scraper-config.yaml \
  --from-file=../services/otel_collector/pipelines.yaml \
  --from-file=../services/otel_collector/receivers.yaml \
  --namespace=m-dorado \
  --dry-run=client -o yaml > otel-configmap.yaml

kubectl apply -f otel-configmap.yaml
```

## 📡 Prometheus

### Prometheus ConfigMap

```bash
kubectl create configmap prometheus-config \
  --from-file=prometheus.yml=services/prometheus/prometheus.yml \
  --namespace=m-dorado
```

### Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: otel-env
  namespace: m-dorado
type: Opaque
stringData:
  GF_SECURITY_ADMIN_USER: admin
  GF_SECURITY_ADMIN_PASSWORD: admin
  SENSOR_SECRET_TOKEN: "YOUR_TOKEN"
  SENSOR_ID: "YOUR_ID"
```

```bash
kubectl apply -f otel-env-secret.yaml
```

## 📊 Grafana

### Secret: `grafana-static`

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

### ConfigMaps (Split by Directory)

```bash
kubectl create configmap grafana-device-dashboards   --from-file=provisioning/dashboards/bigip/device   --namespace=m-dorado

kubectl create configmap grafana-fleet-dashboards   --from-file=provisioning/dashboards/bigip/fleet   --namespace=m-dorado

kubectl create configmap grafana-profile-dashboards   --from-file=provisioning/dashboards/bigip/profile   --namespace=m-dorado

kubectl create configmap grafana-otel-dashboards   --from-file=provisioning/dashboards/otel-collector   --namespace=m-dorado

kubectl create configmap grafana-dashboards-yaml   --from-file=dashboards.yaml=provisioning/dashboards/dashboards.yaml   --namespace=m-dorado

kubectl create configmap grafana-datasources   --from-file=provisioning/datasources   --namespace=m-dorado
```

## ✅ Verification

```bash
kubectl get pods -n m-dorado
kubectl get svc -n m-dorado
```
