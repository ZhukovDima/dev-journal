---
sidebar_position: 14
---
# Jobs
Job (работа, задание) — это yaml-манифест, который создаёт под для выполнения разовой задачи. Если запуск задачи завершается с ошибкой, Job перезапускает поды до успешного выполнения или до истечения таймаутов. Когда задача выполнена, Job считается завершённым и больше никогда в кластере не запускается. Job — это сущность для разовых задач.

### Когда используют Job

- При установке и настройке окружения. Например, мы построили CI/CD, который при создании новой ветки автоматически создаёт для неё окружение для тестирования. Появилась ветка — в неё пошли коммиты — CI/CD создал в кластере отдельный namespace и запустил Job — тот, в свою очередь, создал базу данных, налил туда данные, все конфиги сохранил в Secret и ConfigMap. То есть Job подготовил цельное окружение, на котором можно тестировать и отлаживать новую функциональность.

- При выкатке helm chart. После развёртывания helm chart с помощью хуков (hook) запускается Job, чтобы проверить, как раскатилось приложение и работает ли оно.

### Batch Job
K8S перезапустит Pod который закончил свое выполнение.   
Можно задать Restart policy - Always, OnFailure, Never. 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-job
spec: # Pod
  containers:
    - name: long-job
      image: python:rc-slim
      command: ["python"]
      args: ["-c", "import time; print('starting'); time.sleep(30); print('done')"]
  restartPolicy: Never
```
Если Pod завершит работу в статусе ERROR перезапуска Pod не произойдет.
K8S Batch Job задает возможность перезапуска пода до успешного выполнения
#### Манифест Batch Job
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: test-job
spec: # Job
  template:
    spec: # Pod
      containers:
        - name: long-job
          image: python:rc-slim
          command: ["python"]
          args: ["-c", "import time; print('starting'); time.sleep(30); print('done')"]
      restartPolicy: Never
  backoffLimit: 2
```
### Cron Job
Cron – это планировщик задач. Утилита, позволяющая выполнять скрипты на сервере в назначенное время с заранее определенной периодичностью. 
К примеру, у вас есть скрипт, который собирает какие-либо статистические данные каждый день в 6 часов вечера. Такие скрипты называют «заданиями», а их логика описывается в специальных файлах под названием сrontab.

#### Cron Tab
crontab – это таблица с расписанием запуска скриптов и программ, оформленная в специальном формате, который умеет считывать компьютер. Для каждого пользователя системы создается отдельный crontab-файл со своим расписанием.
Эта встроенная в Linux утилита доступна на низком уровне в каждом дистрибутиве.   
Основан на - [Синтаксис Cron Tab](https://crontab.guru/)
#### Манифест Cron Job
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cron-job
spec: # CronJob
  schedule: "* * * * *"
  jobTemplate: 
    spec: # Job
      template:
        spec: # Pod
          containers:
            - name: long-job
              image: python:rc-slim
              command: ["python"]
              args: ["-c", "import time; print('starting'); time.sleep(30); print('done')"]
          restartPolicy: Never
      backoffLimit: 2
```
Команды
```shell
kubectl get cronjob
kubectl describe cronjob <NAME>
```