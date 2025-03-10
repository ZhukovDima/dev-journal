---
sidebar_position: 2
---

# Git коммит

После индексации для фиксации отслеживаемых файлов создается снимок состояния (коммит):
```shell
git commit -m "Описание коммита"  # Сообщение коммита лучше всего сформулировать таким образом, чтобы перед его началом можно было поставить "Мною" - Добавлена новая статья
git commit -am "Описание коммита" # Автоматический перевод всех изменений в индексное состояние перед коммитом (только индексированных файлов)
git commit --author='John Smith <john@example.com>' --date=''  # Git различает автора изменений и коммитера
git commit -m '' <paths>       # Коммит отдельного файла
```

### Reset
#### Удаление коммитов (Жёсткий reset --hard)
Передвигает текущую ветку [HEAD] на указанный коммит, все коммиты до указанного коммита будут оторваны и со временем будут удалены. (Найти их идентификаторы можно в Reflog)
> Все изменения из коммитов до указанного коммита пропадут.
```shell
git reset --hard <коммит> 

# Вернуть изменения
git reflog <ветка>
git reset --hard <коммит>  
```

#### Замена и объединение коммитов (Мягкий reset --soft)
Передвигает текущую ветку [HEAD] на указанный коммит.
> Все изменения, из предыдущих указанному, коммитов сохранятся и перейдут в индексное состояние.
```shell
git reset --soft <коммит> 

# После изменений можно взять/поменять описание коммита из другого коммита
git commit -c <коммит>

# Вернуть изменения
git reflog <ветка>
git reset --soft <коммит> 
```

#### Смешанный reset (Мягкий reset --mixed, без флагов)
Передвигает текущую ветку [HEAD] на указанный коммит.
> Все изменения, из предыдущих указанному, коммитов сохранятся, как и reset --soft, но будут в неиндексированном состоянии. Удобно если мы хотим заново переиндексировать изменения.
```shell
git reset [--mixed] <коммит> 
```

### Правка последнего коммита (сommit --amend)
Применяется при необходимости исправления последнего коммита. После внесений исправлений, необходимо индексировать изменения и сделать commit
```shell
git .
git commit --amend [--no-edit|--reset-author]
```
Команда commit --amend сделает
```shell
git reset --soft @~
git commit -c ORIG_HEAD
```

### Правка коммита (revert)
У комманд отмены коммита (reset и rebase) есть общая проблема. Если коммит уже отправлен другим разработчикам, его так просто не отредактировать и не удалить. 
Локальную историю можно исправить, но если коммит уже получили разработчики и, возможно, на его основе сделали что-то еще, то отменить его у всех не получится.  

В git есть комманда revert, она смотрит какие изменения сделаны в указанном коммите и создаёт новый коммит с противоположными изменениями.

```shell
git revert @~                # Создаёт новый коммит с противоположными изменениями
git revert <коммит>          # Отменить можно произвольный коммит
```