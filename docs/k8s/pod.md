---
sidebar_position: 1
---
# Pod
Имеется группа сервисов (контейнеров), упакованные в docker image.  
Можно управлять созданием, рестартом, удалением сервисов (контейнеров) вручную, пр.: 
```
docker container run <image_name>
``` 
Управлять такой системой вручную становится сложно.

Kubernetes используется для организации системы, где Kubernetes будет управлять системой.
Чтобы описать такую систему, Kubernetes имеет много концепций (ReplicaSet, Service, Deployment)

[Documentation](https://kubernetes.io/docs/concepts/workloads/pods/)  
Группа из одного или нескольких контейнеров с общей сетью. По-сути pod это оболочка для контейнера.  
В большинстве случаев Pod и Container имеют связь 1:1 (1 Pod включает 1 Container), но возможно иметь несколько контейнеров в поде.
Например, когда контейнеру нужен вспомогательный Helper контейнер для собирания логов, кеша.

### Манифест Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: order
spec:
  containers:
    - name: order
      image: order-service:0.0.1
      imagePullPolicy: IfNotPresent
```

Kubectl — это инструмент командной строки для управления кластерами Kubernetes. kubectl ищет файл config в директории $HOME/.kube.

Kubectl — это Kubernetes API, главного механизма взаимодействия человека с кластером. Kubernetes API — это HTTP REST API, а значит, любое действие с кластером можно выполнить при помощи стандартных HTTP-запросов.
apiVersion: v1 - это версия REST API.  

### Основные команды

Создание:
```bash
kubectl apply -f workload.yml
```
Получить информацию:
```bash
kubectl get all
kubectl describe pod order
kubectl logs order
```
Изменить конфигурацию:
```bash
kubectl edit pod order
```
Удалить Pod
```bash
kubectl delete pod [--all]
kubectl delete -f . 
```
Приконнектиться и исполнить команды внутри пода:
```bash
kubectl exec -it order -- sh

wget http://localhost:8085/api/orders
```
Сервис из пода недоступен из вне кластера. Команда, которая позволяет создать безопасный туннель между локальным компьютером и подом, работающим в кластере Kubernetes.
```bash
kubectl port-forward order 8085:8085
```