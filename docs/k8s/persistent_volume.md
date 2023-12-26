---
sidebar_position: 5
---
# Persistent Volumes

Для работы с постоянными томами в Kubernetes имеются объекты API позволяющие на постоянной основе выделять ресурсы хранения и управлять ими.
Все данные работы хранятся внутри Пода. Когда Под умирает, все данные пропадают вместе с ним.   
Использование PV гарантирует, что хранилище останется постоянным.

- volumeMounts - Точка монтирования — в какой каталог внутри контейнера будет монтироваться постоянный том, а также имя тома.
- volumes - Реализация монтирования. Существует несколько типов хранения https://kubernetes.io/docs/concepts/storage/volumes/#volume-types

### Манифест Volume:
```yaml
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
            # Use secrets in real life
            - name: POSTGRES_DB
              value: topk8s
            - name: POSTGRES_USER
              value: 'user'
            - name: POSTGRES_PASSWORD
              value: 'password'
          # Точка монтирования — в какой каталог внутри контейнера будет монтироваться постоянный том, а также имя тома.
          volumeMounts:
            - name: postgres-persistent-storage
              mountPath: /var/lib/postgresql/data
      volumes:
        - name: postgres-persistent-storage
          # Конфигурация как реализовывать монтирование. Типы хранения:
          # https://kubernetes.io/docs/concepts/storage/volumes/#volume-types

#         hostPath:
#           path: /mnt/postgres-persistent-storage
#           type: DirectoryOrCreate

          # Указатель на конфигурацию
          persistentVolumeClaim:
            claimName: postgres-pvc
```
### PersistentVolumeClaim и PersistentVolume

```yaml
# Что мы хотим?
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  storageClassName: localstorage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi # Мне нужно 10Gi Пр.: Для 20Gi имплементация local-storage на 10Gi не сработает.
---
# Имплементация claim-а. Как мы хотим чтобы claim был реализован.
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-storage
spec:
  storageClassName: localstorage
  capacity:
    storage: 10Gi # физическое хранилище на 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/postgres-persistent-storage/
    type: DirectoryOrCreate
```
### Storage Classes и Binding
Связывание (binding) PersistentVolumeClaim с PersistentVolume происходит по полям:
- storageClassName
- accessModes
- storage

### Основные команды
```bash
kubectl get pv
kubectl get pvc
```