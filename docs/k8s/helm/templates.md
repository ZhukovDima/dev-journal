---
sidebar_position: 2
---
# Templates

Создание helm chart
```bash
helm create <name-of-the-chart>
```
- .helmignore  -  Contains patterns for files to ignore when packaging Helm charts.
- Chart.yaml -  Метаданные о чартах, такие как: версия, имя, ключевые слова для поиска и так далее.
- values.yaml - Настройки чарта по умолчанию.
- charts/ - Каталог для управляемых вручную зависимостей чарта.
- templates/ - Написанные на языке Go файлы шаблонов, объединенные с конфигурационными значениями из файла values.yaml и предназначенные для генерации манифестов Kubernetes.

Go вставки в файлах-шаблонах описываются внутри двойных фигурных скобок: 

```bash
helm template .
```
Переменные для вставки находятся в файле values.yaml
```yaml
      containers:
      - name: appstore
        image: {{ .Values.dockerrepo }}/order-service:0.0.1
```
```yaml
javaopts: "-XX:MinRAMPercentage=30.0 -XX:MaxRAMPercentage=70.0 -XX:MetaspaceSize=96m -XX:MaxMetaspaceSize=256m"
dockerrepo: repo.int.zone
devmode:
  enabled: false
```
Значения можно переопределять
```bash
helm template . --set devmode.enabled=true
```

### Template Functions
https://helm.sh/docs/chart_template_guide/function_list/

Функции:
```yaml
      containers:
      - name: appstore
        image: {{ lower .Values.dockerrepo }}/order-service:0.0.1
       # image: {{ default "default.repo.int" .Values.dockerrepo }}/order-service:0.0.1
```

Pipeline:
```yaml
      containers:
      - name: appstore
        image: {{ .Values.dockerrepo | default "default.repo.int" | upper }}/order-service:0.0.1
```
### Flow Control
Возможно добавлять условия в шаблон, пр. если environment dev, то брать имидж с тэгом 0.0.1-dev:
```yaml
devmode: true # false yes no
environment: dev
```
```yaml
      containers:
      - name: appstore
        image: {{ .Values.dockerrepo }}/order-service:0.0.1{{if .Values.devmode }}-dev{{end}}
    #    image: {{ .Values.dockerrepo }}/order-service:0.0.1{{if eq .Values.environment "dev"}}-dev{{end}}
```

### Named Templates (partial, subtemplate)
Если необходимо переопределить yaml блок, возможно вынести yaml блок в отдельный файл templates, название файла должно начинаться с нижнего подчеркивания _.
Стандарт для расширения файлов .tpl.   
Пример: _helpers.tpl
```tpl
  {{ define "order-service-image" }}
    - name: appstore
      image: {{ .Values.dockerrepo }}/order-service:0.0.1{{if .Values.devmode }}-dev{{end}}
  {{ end }}
```

```yaml
  containers:
# {{ template "order-service-image" . }} 
{{ include "order-service-image" . }} # include можно использовать в Pipeline
```
Текст процессор Go сделает вставку после строчки кода и после удалит её. После этого останется пустая строка. Чтобы избавиться от этого
```yaml
  containers:
{{- template "order-service-image" . }} 
```
Так как шаблон может использоваться во многих файлах, точно не известно количество отступов.   
Задать количество отступов:
```yaml
  containers:
{{- include "order-service-image" . | indent 6 }} 
```