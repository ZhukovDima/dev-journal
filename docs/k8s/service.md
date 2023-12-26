---
sidebar_position: 2
---
# Service

В Kubernetes объекты Pod недолговечны - могут умирать, регулярно рестартуются.   
Service - это долгоживущий объект в Kubernetes, абстракция, определяющая логический набор подов и политику доступа к ним . 
Как правило, этот набор подов определяется на основе меток (присваиваются в момент создания подов) и селекторов.

Типы сервисов:
- ClusterIP - сервис будет доступен только внутри кластера
- NodePort - сервис будет доступен извне, необходимо указать порт (nodePort) и должен быть больше чем 30000. 
В клауд системах используют тип LoadBalancer, так же можно использовать Ingress сервис.

### Сервис kube-dns
Сервис kube-dns прослушивает события службы и события конечных точек через Kubernetes API и обновляет записи DNS по мере необходимости.  
Представляет собой таблицу пар key value.  
key - это лейбелы сервисов  
value - ip адреса сервисов   

Сервис kube-dns находится в неймспейсе kube-system  
Список namespaces
```bash
kubectl get namespaces
```
В фале resolv.conf содержатся адреса dns серверов, к которым имеет доступ данная система.
```bash
kubectl exec -it order -- sh
cat /etc/resolv.conf

nslookup order-service
```
### Манифест Service и Pod:  
Services:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: zookeeper-service
spec:
  # Определяет какие поды будут представлены этим сервисом
  # Сервис становится точкой доступа для других сервисов
  # или внешних пользователей (браузер)
  selector:
    app: zookeeper
  ports:
    - name: http
      port: 2181
  type: ClusterIP
---
apiVersion: v1
kind: Service
metadata:
  name: kafka-service
spec:
  selector:
    app: kafka
  ports:
    - name: http
      port: 9092
  type: ClusterIP
```

Pods:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: zookeeper
  # Лейбл для соединения с сервисом
  labels:
    app: zookeeper
spec:
  containers:
    - name: zookeeper
      image: confluentinc/cp-zookeeper:7.4.3
      env:
        - name: ZOOKEEPER_CLIENT_PORT
          value: '2181'
        - name: ZOOKEEPER_TICK_TIME
          value: '2000'
---
apiVersion: v1
kind: Pod
metadata:
  name: kafka
  # Лейбл для соединения с сервисом
  labels:
    app: kafka
spec:
  containers:
    - name: kafka
      image: confluentinc/cp-kafka:7.4.3
      env:
        - name: KAFKA_BROKER_ID
          value: '1'
        - name: KAFKA_ZOOKEEPER_CONNECT
          value: 'zookeeper-service'
        - name: KAFKA_LISTENER_SECURITY_PROTOCOL_MAP
          value: 'PLAINTEXT:PLAINTEXT'
        - name: KAFKA_ADVERTISED_LISTENERS
          value: 'PLAINTEXT://kafka-service:9092'
        - name: KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR
          value: '1'
        - name: KAFKA_TRANSACTION_STATE_LOG_MIN_ISR
          value: '1'
        - name: KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR
          value: '1'
```