---
sidebar_position: 1
---

# Git проект
Git использует эффективный механизм хранения данных: он не хранит полные копии состояний, а вместо этого реализует своеобразную файловую систему, где загружаются только изменившиеся файлы. Для файлов без изменений используются ссылки на более ранние версии.

Коммит представляет собой снимок файлов проекта, который сохраняется в системе контроля версий. Ветка является хронологической последовательностью коммитов и может разделяться, идти параллельно и сливаться.

Git работает как распределенная система, что означает следующее:

- Может использоваться с центральным репозиторием на выделенном сервере
- Участники команды обмениваются изменениями через этот сервер
- Каждый участник имеет локальную копию всех файлов, веток и истории правок

### Настройка Git
Перед началом работы с Git необходимо настроить информацию о пользователе:
```shell
git config --global user.name "Dmitrii Zhukov" 
git config --global user.email "zhukovdima@list.ru"
git config --global core.editor nano
git config --list
git config --list --show-origin
```
Файл конфигурации (.gitconfig) является текстовым и может быть отредактирован вручную.

### Git проект
Git отслеживает не всю файловую систему, а только выбранную папку на компьютере. Для начала работы её нужно пометить как git-проект. 
Существует два способа получить git-проект:
```shell
# Создание нового репозитория
git init

# Клонирование существующего проекта из удаленного репозитория
git clone [URL-репозитория]
```
Для добавления файла под контроль версии его необходимо проиндексировать:
```shell
git add .                    # индексировать изменения 
git add -p                   # индексировать изменения для каждого изменённого фрагмента в файле
git add --force              # принудительно индексировать изменения (например файл/каталог в gitignore)

git status                   # проверка статуса изменений
git status -s                # проверка статуса изменений (short)

git diff                     # проверка изменений вне индекса

# Флаг --cashed означает работу с индексом
git diff --cached            # проверка изменений в индексе
git rm <paths> --cached      # удаление файла из индекса
```
>Игнорирование файлов:   
>Конфигурационный файл - .gitignore. Внутри файла прописывается шаблон для игнорирования: *.log, tmp/*
>Пустые папки не попадают под отслеживание Git. Чтобы избежать этого можно добавить в папку файл .keep  

### Восстановление предыдущих версий файлов
- Не отслеживаемый git файл достаточно просто удалить.
```shell
git clean -dxf             
```
- Отслеживаемый (неиндексированный) файл (красный):
```shell
git checkout -- <paths>              # Откатить изменения (получить последнее сохранённое состояние)

git checkout <коммит> <paths>        # Откатить изменения файла из коммита
```
> Запись -- указывает на путь, если имя файла совпадает с именем ветки
- Индексированный файл (зеленый):
```shell
# Удалить файл/каталог из индекса (переведёт в неиндексированное состояние, сохранив изменения)
git reset HEAD <paths>  # Очищает индекс, по сути является обратным действием от  - git add .

# Принудительно перезаписать все файлы из той версии на которую переключаемся, без сохранения изменений                   
git reset --hard HEAD [git checkout -f HEAD]   
```

### Состояние отдалённый HEAD
Команда позволяет перейти на любой коммит и вернуться на предыдущее состояние проекта.
```shell
git checkout <коммит> 
```
Возникает состояние Detached HEAD. Все коммиты будут производиться от коммита на котором находится HEAD. После переключения
на начало ветки git checkout master, такие коммиты будут недостижимы.

### Просмотр истории изменений
```shell
git log
git log <ветка>
git log -2                            # Ограничить количество коммитов 
git log -p                            # Просмотр истории с изменениями в коммитах
git log --stat                        # Просмотр статистики: список измененных файлов, сколько строк было добавлено и удалено
git log --pretty=oneline              # Просмотр комментов в одну строку
git log --pretty=format:"%h - %an, %ar : %s"
git log --pretty=format:"%h -%d %s %cd"
git log --pretty=format:"%h -%d %s %cd" --graph
git log --since=2019-03-01 --until=2019-03-25
git log -SHello                       # Найти коммит в котором упоминается строка
git log --author="Dmitrii Zhukov"

git blame <paths>                  # Найти кто автор/дата последних изменений в файле

git show <tag_name | commit_hash>  # Просмотр коммита
git show <commit>:<путь_к_файлу>   # Просмотр файла в предыдущем коммите
git show :/<текст коммита>         # Просмотр коммита по словам в описании
```

### Псевдонимы команд
```shell
git config --global alias.gg 'log --pretty=format:"%C(Yellow)%h%Creset -%C(green)%d%Creset %s %C(bold blue)(%cn) %C(green)%cd%Creset" --graph --date=format:"%d.%m.%Y %H:%M"'
```