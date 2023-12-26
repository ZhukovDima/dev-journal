---
sidebar_position: 12
---
# Secrets
Секрет — это любые данные, которые мы бы не хотели показывать широкому кругу лиц: парольная фраза, ключевая пара, ключ шифрования, API-токен.  
Значения в поле data должны быть base64 encoded. (base64 - переводит бинарные данные в ASCII символы)
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres-creds
data:
  POSTGRES_DB: dG9wazhz
  POSTGRES_USER: dXNlcg==
  POSTGRES_PASSWORD: cGFzc3dvcmQ=
type: Opaque
```
Значения в поле stringData k8s переведет в base64 автоматически
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres-creds
stringData:
  POSTGRES_DB: 'topk8s'
  POSTGRES_USER: 'admin'
  POSTGRES_PASSWORD: 'admin'
type: Opaque
```
> Secrets не secure в k8s. Кодирование не является шифрованием.
>```bash
> kubectl get secret postgres-creds -o yaml
>```
Манифест Deployment с Secrets:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  selector:
    matchLabels:
      app: postgres
  replicas: 1
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:16.1
          env:
            - name: POSTGRES_DB
              valueFrom:
                secretKeyRef:
                  name: postgres-creds
                  key: POSTGRES_DB
            - name: POSTGRES_USER
              valueFrom:
                secretKeyRef:
                  name: postgres-creds
                  key: POSTGRES_USER
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-creds
                  key: POSTGRES_PASSWORD
          volumeMounts:
            - name: postgres-persistent-storage
              mountPath: /var/lib/postgresql/data
      volumes:
        - name: postgres-persistent-storage
          persistentVolumeClaim:
            claimName: postgres-pvc
```
