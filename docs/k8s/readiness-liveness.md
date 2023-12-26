---
sidebar_position: 9
---
# Readiness and Liveness Probes
Spring Boot приложению (как и любому http серверу) при запуске контейнера обычно необходимо какое-то время перед тем как оно будет готово обрабатывать запросы.
Получается ситуация, что Pod запущен, но программа внутри пода еще стартует. Запрос, который попадет в такой под, зависнет и отвалится по таймауту.

Простой способ увидеть проблему - увеличить количество replicas и проверить запросом к сервису:
```bash
while true; do curl http://localhost:30080/api; echo; done
```
### Readiness probe

- Readiness probes - Готов ли Pod принимать запросы
- Liveness probes - Если probe совершается с ошибкой, K8S перестартует контейнер 

Probes может быть httpGet, exec command, tcp 

#### Probes:
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
          livenessProbe:
            httpGet:
              scheme: HTTPS
              path: /actuator/health
              port: actuator
            initialDelaySeconds: 40
            periodSeconds: 60
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              scheme: HTTPS
              path: /actuator/health
              port: actuator
            initialDelaySeconds: 40
            periodSeconds: 5
            timeoutSeconds: 5
          resources:
            requests:
              memory: 300Mi
              cpu: 100m
            limits:
              memory: 500Mi
              cpu: 200m
```