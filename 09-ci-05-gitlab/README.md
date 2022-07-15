# Домашнее задание к занятию "09.05 Gitlab"

## Проценко Анастасия

## Подготовка к выполнению

1. Необходимо [зарегистрироваться](https://about.gitlab.com/free-trial/)
2. Создайте свой новый проект
3. Создайте новый репозиторий в gitlab, наполните его [файлами](./src)
4. Проект должен быть публичным, остальные настройки по желанию

## Основная часть

### DevOps

В репозитории содержится код проекта на python. Проект - RESTful API сервис. Ваша задача автоматизировать сборку образа с выполнением python-скрипта:
1. Образ собирается на основе [centos:7](https://hub.docker.com/_/centos?tab=tags&page=1&ordering=last_updated)
2. Python версии не ниже 3.7
3. Установлены зависимости: `flask` `flask-jsonpify` `flask-restful`
4. Создана директория `/python_api`
5. Скрипт из репозитория размещён в /python_api
6. Точка вызова: запуск скрипта
7. Если сборка происходит на ветке `master`: Образ должен пушится в docker registry вашего gitlab `python-api:latest`, иначе этот шаг нужно пропустить

### Product Owner

Вашему проекту нужна бизнесовая доработка: необходимо поменять JSON ответа на вызов метода GET `/rest/api/get_info`, необходимо создать Issue в котором указать:
1. Какой метод необходимо исправить
2. Текст с `{ "message": "Already started" }` на `{ "message": "Running"}`
3. Issue поставить label: feature

### Developer

Вам пришел новый Issue на доработку, вам необходимо:
1. Создать отдельную ветку, связанную с этим issue
2. Внести изменения по тексту из задания
3. Подготовить Merge Requst, влить необходимые изменения в `master`, проверить, что сборка прошла успешно


### Tester

Разработчики выполнили новый Issue, необходимо проверить валидность изменений:
1. Поднять докер-контейнер с образом `python-api:latest` и проверить возврат метода на корректность
2. Закрыть Issue с комментарием об успешности прохождения, указав желаемый результат и фактически достигнутый

### Решение

[gitlab](https://gitlab.com/procenko-anastasia/09-ci-06-gitlab)

```
vagrant@vagrant:~$ docker pull registry.gitlab.com/procenko-anastasia/09-ci-06-gitlab/python-api.py:feature
feature: Pulling from procenko-anastasia/09-ci-06-gitlab/python-api.py
Digest: sha256:8d252d1fd0af3ab339f64aac7972634fa46583f0a41219f82f943537391dd43c
Status: Image is up to date for registry.gitlab.com/procenko-anastasia/09-ci-06-gitlab/python-api.py:feature
registry.gitlab.com/procenko-anastasia/09-ci-06-gitlab/python-api.py:feature
vagrant@vagrant:~$ docker run -p 5290:5290 -d registry.gitlab.com/procenko-anastasia/09-ci-06-gitlab/python-api.py:feature
4cccf5b9e8be598ab0133c83acb93ab6b36f19c826045b7db98d7570a718f106
vagrant@vagrant:~$ curl localhost:5290/get_info
{"version": 3, "method": "GET", "message": "Running"}
```
```
vagrant@vagrant:~$ docker run -p 5290:5290 -d registry.gitlab.com/procenko-anastasia/09-ci-06-gitlab/python-api.py:latest
vagrant@vagrant:~$ curl localhost:5290/get_info
{"version": 3, "method": "GET", "message": "Running"}
```