---
sidebar_position: 4
---
# Deployment

Контроллер развертывания (Deployment controller) предоставляет возможность декларативного обновления (rolling updates) для объектов типа поды (Pods) и наборы реплик (ReplicaSets).   

### Манифест Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order
spec:
  selector: 
    matchLabels:
      app: order
  replicas: 1
  template: 
    metadata:
      labels:
        app: order
    spec:
      containers:
        - name: order
          image: order-service:0.0.2
          imagePullPolicy: IfNotPresent
```
### Управление rollouts

Статус:
```bash
kubectl rollout status deployment order
```
История:
```bash
kubectl rollout history deployment order
```
Возврат:
```bash
kubectl rollout undo deployment order [--to-revision=]
```
Обновить поды
```bash
kubectl rollout restart deployment
```