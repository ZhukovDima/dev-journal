---
sidebar_position: 5
---

# Git Reflog
Каждый раз при операциях по изменению ссылок (при переключении с ветки на ветку, коммиты) Git записывает в Reflog (Reference log):
```shell
cat .git/logs/HEAD # Логи для ссылки HEAD   

git reflog [HEAD|master]
git log --oneline -g
```
С помощью Reflog возможно найти переключения на удалённую ветку (например, чтобы ее восстановить).
Также некоторые параметры будут искать совпадения в reflog.
```shell
git checkout @{-1}   # Последний checkout в reflog
git checkout -
```
