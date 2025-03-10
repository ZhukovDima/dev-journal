---
sidebar_position: 3
---

# Ветки
Ветка - хронологическая последовательность связанных между собой коммитов. Сразу после создания проекта, появляется основная ветка main (ранее master):
```shell
git branch                               # Выводит список доступных веток, помечает * текущую
git branch <ветка>                   # Создание новой ветки
git branch master <коммит>               # Создание ветки от коммита

git checkout <ветка>                 # Переключение на ветку
git checkout -b <ветка>              # Создание и переключение на ветку
git checkout -f <ветка>              # Принудительно переключится на ветку (все незакомиченные изменения пропадут)            

git branch <ветка> <новое_ветка> # Переименование ветки

git branch -d <ветка>                # Удаление ветки
git branch -D <ветка>                # Удаление ветки
```
Предположим мы по-ошибке сделали несколько коммитов в ветку master и хотим переместить эти коммиты в другую ветку (По сути надо передвинуть ветку master на несколько коммитов назад):
```shell
git checkout -b fix                     
git branch master <коммит>               # Создание ветки от коммита
git branch --force master <коммит>       # Создание (перемещение, если она существует)  ветки от коммита

git checkout -B master <коммит>
```
#### Перенос коммита из другой ветки
Cherry-pick - Перенос нескольких коммитов из одной ветки в другую:
```shell
git cherry-pick <commit_hash>
# При возникновении конфликта, после исправлений конфликтов:
git add .
git cherry-pick --continue 
```

### Слияние веток
Слияние веток - объединение коммитов одной ветки в другую. В момент слияния мы должны находиться в целевой ветке

Перед merge git найдёт коммит, где ветки разделились.  
```shell
git merge-base <ветка> <ветка>  # Коммит, где ветки разделились
```
Процесс слияния изменений: base + (our changes) + (their changes) = merge
```shell
git merge <ветка>        # Слияние (Находясь в ветке, в которую хотим слить изменения)
git merge <ветка> --log  # Включить в описание описания коммитов из ветки слияния
```
В результате появится - коммит слияния. Особенность этого коммита, что у него 2 родителя. 
```shell
git diff HEAD^1      # Просмотр изменений 1 родителя
git diff HEAD^2      # Просмотр изменений 2 родителя

git branch --merged  # Просмотр веток объединенных с текущей
git log --first-parent # Просмотр истории без коммитов из веток слияния
```
#### Метод fast-forward
Если в ветке нет своих коммитов в момент отхождения другой ветке, при слиянии может происходить слияние перемоткой. 
Коммиты из одной ветки перемещаются в ветку простым смещением указателя HEAD.

В результате мы получаем ровную историю коммитов, без коммитов слияния, и из которой не понятно, что именно раньше было в мастере и где начинаются коммиты, которые были в ветке слияния. 
Это затрудняет чтение истории разработки. 
```shell
git merge --no-ff
git merge --ff
```
#### Слияние коммита из ветки (merge --squash):
Применяется, когда результат работы какой-то ветки, мы хотели бы интегрировать в ветку, но историю коммитов нет.
```shell
git merge --squash <ветка> # Применяет изменения из ветки к текущей ветке
```

### Перебазирование веток (Rebase):
Перемещение ветки в Git истории, таким образом, чтобы она снова начиналась с начала ветки с которой делаем rebase: 
Происходит перемещение HEAD на начало ветки и после копирование по каждому коммиту. 
```shell
git rebase <ветка>  # Находясь в ветке, в которую хотите перебазировать изменения
```
> В случае конфликта rebase останавливается для разрешения конфликтов в состоянии DETACHED HEAD.
```shell
git reset --hard  # Очистит незакомиченные изменения, удалит конфликт, но ссылку HEAD не передвинет!

git rebase --abort     # Вернет HEAD обратно на ветку
git rebase --skip      # Проигнорирует конфликтный коммит и продолжит rebase
git rebase --quit      # Не возвращает HEAD обратно на ветку

# После исправления конфликта
git add . # Проиндексировать изменения
git rebase --continue  # Продолжить перебазирование
```
#### Перебазирование с запуском тестов (rebase -x)
Запустит тест после каждого перебазированного коммита
```shell
git rebase -x "тест" <ветка>

# Если произошла ошибка при перебазировании коммита, после исправления, его следует заменить
git add <исправленный_файл>
git commit --ammend --no-edit
git rebase --continue
```
#### Перенос части ветки (--onto)
```shell
git rebase --onto master feature # Перебазирует все коммиты начиная с feature в ветку master 
```
> При переносе коммитов удобнее использовать cherry-pick, так как rebase --onto перенесет не просто коммиты, но и ветку.

#### Интерактивное перебазирование, rebase -i

Объединение коммитов:
```shell
git cherry -v <ветка>  # Просмотреть на сколько коммитов одна ветка отличается от другой
```

Перебор всех комментов ветки:
```shell
git show HEAD
git show HEAD~1
```
Пр.: Если разница между ветками 3 коммита, то начальный коммит - HEAD~3, чтобы объединить все коммиты до этого коммита:
```shell
git rebase -i HEAD~3
```
Если на удалённом сервере находится версия ветки до слияния, то при попытке отправить ветку на сервер, произойдет ошибка:
```shell
git push origin <ветка>                # Ошибка
git push origin <ветка> --force [-f]   # Принудительная отправка на сервер
```

### Откат слияния веток
```shell
git reset --hard @~
```

### Разрешение конфликтов при слиянии
Конфликт возникает при слиянии, когда ветки модифицируют одно и то же место в файлах.  При этом возникает состояние - прерванное слияние.

Git запоминает коммит в котором мы осуществляем слияние в файле:
```shell
cat .git/MERGE_HEAD
```
Конфликты изображаются угловыми скобками, чтобы их было лучше видно в файле:
```txt
<?php
<<<<<<< HEAD
	echo "Hello, world!!!";     # Текст в текущей ветке master
=======
	session_start();            # Текст в ветке, которая сливается
	echo "Hello, world";
>>>>>>> test
?>
```
Чтобы разрешить конфликт, нужно выбрать правильный вариант, который учтёт изменения в обеих частях:
```shell
git checkout --ours <путь>                       # Просмотр нашей версии файла
git checkout --theirs <путь>                     # Просмотр версии файла из сливаемой ветке
git checkout --merge <путь>                      # Продолжить слияние
git checkout --conflict=diff3 --merge <путь>     # Просмотр что было в коммите до пересечения двух веток

git reset --hard                                 # Очистить процесс слияния
git reset --merge                                # Очистить процесс слияния, но сохранить незафиксированные изменения не участвующие в слиянии
git merge --abort                                # Альтернатива git reset --merge
```
Либо сделать это вручную:
```txt
<?php
    session_start();           
    echo "Hello, world";
?>
```
И зафиксировать изменения:
```shell
git merge --continue
git commit -am "Разрешён конфликт слияния"    
```
