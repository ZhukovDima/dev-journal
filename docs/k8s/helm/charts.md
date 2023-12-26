---
sidebar_position: 1
---
# Charts

[Helm](https://helm.sh/) — это менеджер пакетов с открытым исходным кодом для Kubernetes, который упрощает развертывание, управление и обновление приложений в кластере Kubernetes.

> Пример: Вы работаете в кластере K8S (Minikube, AWS) и Вам необходимо развернуть сервис MySQL в кластере. Если не существует менеджера пакетов,
> то необходимо найти Docker Image для MySQL (например на Docker Hub), написать yaml манифест (Deployment, StatefulSet, DaemonSet), так же
> написать service yaml манифест для портов и прочее. Достаточно сложно описать наилучшую конфигурацию (Requests, Limits, Ports).  
> Используя Helm - helm install mysql.

### Где найти пакеты (charts)?

Deprecated репозиторий - https://charts.helm.sh/stable/, чарты храились на гитхабе https://github.com/helm/charts, 
но из-за проблем поддержки stable версий, было принято решение чтобы само community взяло на себя поддержку - https://artifacthub.io/

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

helm install mysql bitnami/mysql

helm list

helm uninstall mysql
```

### Chart Values

Предположим необходимо установить мониторинг в кластер k8s - https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack  
После установки стека мониторинга, мы хотим задать логин и пароль входа в графический интерфейс Grafana. Все параметры заданы в Values

```bash
helm show values prometheus-community/kube-prometheus-stack
helm show values prometheus-community/kube-prometheus-stack > values.yml
helm show values prometheus-community/kube-prometheus-stack | grep -i password
```
Задать Value можно при установке:
```bash
helm install monitoring prometheus-community/kube-prometheus-stack --set adminPassword='admin
```

### Кастомизация Charts Values

```bash
helm upgrade monitoring prometheus-community/kube-prometheus-stack --set grafana.adminPassword=admin
```
> Частая ошибка команда - --set adminPassword=admin
> добавит новый пункт adminPassword='admin', но не изменит нужный adminPassword, в связи с тем что adminPassword вложен в grafana

> Команды helm upgrade сбросит все изменения на дефолтные values.yml

Задать изменения из values.yml
```bash
helm upgrade monitoring prometheus-community/kube-prometheus-stack --values=values.yml
```
Возможно задать в yaml файл только те свойства, которые меняются:
```yaml
## Using default values from https://github.com/grafana/helm-charts/blob/main/charts/grafana/values.yaml
grafana:
  adminPassword: admin

  service:
    portName: http-web
    type: NodePort
    nodePort: 30001
```

### Контроль за Chart?

[SnowflakeServer](https://martinfowler.com/bliki/SnowflakeServer.html) - (всегда уникальна) это сервер на который со временем свободно устанавливается большое количество программ.
(Появляется ситуация что мы не помним что устанавливалось на сервер, какой версии, конфигурации, кастомные команды и др) нужен какой-то лог что делалось с сервером. 
Очень сложно делать реплику на такой сервер и поднять еще один сервер со всем необходимым.   
[PhoenixServer](https://martinfowler.com/bliki/PhoenixServer.html) - это сервер который полностью сконфигурирован скриптовым механизмом (пр.: Ansible). Для добавлений изменений придется править скрипт.
И сам скрипт хранится в системе управления версиями.

> Использование helm install/upgrade [remote chart] приводит к Snowflake Cluster. Пересобрать такой кластер будет очень сложно (что-то устанавливалось/правилось и не было задокументировано, remote chart может быть удален и др.)

```bash
helm pull [chart url | repo/chartname]

helm pull prometheus-community/kube-prometheus-stack
helm pull prometheus-community/kube-prometheus-stack --untar=true
```
Команда скачает исходники файлов chart .tgz (например в вашу директорию для создания кластера my-cluster и теперь чтобы пересобрать кластер достаточно пробежать и установить всем необходимые chart)

```bash
helm install monitoring ./kube-prometheus-stack

helm upgrade monitoring --values=myvalues.yaml .
```
Возможно создать шаблон k8s сущностей из chart, на выходе получим yaml файл с сущностями k8s, который можно установить через kubectl apply вместо helm install

```bash
helm template monitoring ./kube-prometheus-stack/ --values=./kube-prometheus-stack/myvalues.yaml > monitoring-stack.yml
```