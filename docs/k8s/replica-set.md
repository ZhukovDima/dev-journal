---
sidebar_position: 3
---
# ReplicaSet

ReplicaSet гарантирует, что определенное количество экземпляров подов (Pods) будет запущено в кластере Kubernetes в любой момент времени.  
Для чего нужна репликация:
- надежности. C несколькими экземплярами приложения вы избавляетесь от проблем в случае отказа одного (или нескольких) из них;   
- балансировки. Наличие нескольких экземпляров приложения позволяет распределять траффик между ними, предотвращая перегрузку одного инстанса (под / контейнер) или ноды;  
- масштабирования. При возрастающей нагрузке на уже существующие экземпляры приложения Kubernetes позволяет легко увеличивать количество реплик по необходимости.  
### Манифест ReplicaSet 
Оборачивает Pod внутри template и задает поле replicas количество экземпляров подов. 
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: order
spec:
  selector: # Эта ReplicaSet будет управлять подом с лейбелом указанным в matchLabels
    matchLabels:
      app: order
  replicas: 1
  template: # Шаблон для подов
    metadata:
      labels:
        app: order
    spec:
      containers:
        - name: order
          image: order-service:0.0.1
          imagePullPolicy: IfNotPresent
```