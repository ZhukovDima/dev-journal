---
sidebar_position: 11
---
# ConfigMaps

### Environment переменные
Одно из преимуществ docker image в том, что мы можем просто его загрузить и использовать. Image уже pre-packed со всеми системными переменными для запуска. 
Не нужно знать какой язык программирования использован (устанавливать jvm, конфигурировать системные переменные и прочее).   
Если есть необходимость что-то поменять в docker image, то это сделать нельзя, docker image не немутабелен. (Можно сделать новый от другого docker image). 
Но часто приходится передавать свой данные внутрь контейнера.   
Самый распространенный способ передать данные это использование системных переменных (environment variables):
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
          image: order-service:0.0.1
          imagePullPolicy: IfNotPresent
          env:
            - name: KAFKA_HOST
              value: "kafka-service"
```
Недостаток такого подхода - возможное дублирование env переменных в нескольких подах. 

### ConfigMap
Можно вынести общую информацию в отдельную конфигурацию ConfigMap:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: global-kafka-config
  namespace: default
data:
  KAFKA_HOST: "kafka-service"
```
Просмотр
```bash
kubectl get cm
kubectl describe cm global-kafka-config
```
Использование ConfigMap

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: notification-bot
spec:
  selector:
    matchLabels:
      app: notification-bot
  replicas: 1
  template:
    metadata:
      labels:
        app: notification-bot
    spec:
      containers:
        - name: notification-bot
          image: notification-bot:0.0.1
          imagePullPolicy: IfNotPresent
#          env:
#            - name: KAFKA_HOST
#              valueFrom:
#                configMapKeyRef:
#                  name: global-kafka-config
#                  key: KAFKA_HOST
          envFrom:
            - configMapRef:
                name: global-kafka-config
```
Проверить env переменные импортировались
```bash
kubectl exec -it <POD_NAME> -- sh
echo $KAFKA_HOST
```
> При применении изменении в yml файле ConfigMap, изменения не подхватываются в под автоматически. Необходимо рестартовать под.
> Либо как вариант добавлять новые версии ConfigMap global-kafka-config-v2, изменяя configMapName в конфигурации Deployment.

#### Загрузка ConfigMap через Volume
Существует возможность загружать параметры через файл:
В примере создастся файл KAFKA_HOST по пути /etc/any/directory/config (Для каждой переменной в ConfigMap создастся файл с именем и значением внутри файла).  
Удобнее разместить все переменные в один файл:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: global-kafka-config
  namespace: default
data:
  config.yml: |
    KAFKA_HOST: "kafka-service"
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: notification-bot
spec:
  selector:
    matchLabels:
      app: notification-bot
  replicas: 1
  template:
    metadata:
      labels:
        app: notification-bot
    spec:
      containers:
        - name: notification-bot
          image: notification-bot:0.0.1
          imagePullPolicy: IfNotPresent
          volumeMounts:
            - name: kafka-config-volume
              mountPath: /etc/any/directory/config # k8s создаст эту директорию
      volumes:
        - name: kafka-config-volume
          configMap:
            name: global-kafka-config
```