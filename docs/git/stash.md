---
sidebar_position: 7
---

# Stash
### Откладывание изменений
Если в ходе работы над веткой, мы внесли изменения в файл, но вынужденны перейти в другую ветку, в лучшем случае эти незафиксированные изменения перейдут в другую ветку.
```shell
git add . 
git stash                     # Добавление изменений в stash список
git stash list                # Просмотр stash списка
git stash apply               # Вернуть данные из stash
git stash pop                 # Вернуть данные из stash
git stash apply stash@{0}     # Вернуть конкретный элемент из stash
git stash drop                # Удалить данные из stash
```

