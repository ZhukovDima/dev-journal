---
sidebar_position: 7
---
# Metrics Profiling

### Metrics Server
> Недоступен по умолчанию в K8S и является дополнением Add-on

Установка Metrics Server:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.4.2/components.yaml
```
Patch the metrisc server to work with insecure TLS
```bash
kubectl patch deployment metrics-server -n kube-system --type 'json' -p '[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'
```

#### Просмотр статистики Metrics Server:
```bash
kubectl top pod
kubectl top node
```

### Kubernetes Dashboard
> Недоступен по умолчанию в K8S и является дополнением Add-on

Install Kubernetes Dashboard:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml
```
Patch the dashboard to allow skipping login:
```bash
kubectl patch deployment kubernetes-dashboard -n kubernetes-dashboard --type 'json' -p '[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--enable-skip-login"}]'
```

Run the Kubectl proxy to allow accessing the dashboard
```bash
kubectl proxy
```
