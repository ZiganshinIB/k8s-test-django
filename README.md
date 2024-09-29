# Django Site

Докеризированный сайт на Django для экспериментов с Kubernetes.

Внутри контейнера Django приложение запускается с помощью Nginx Unit, не путать с Nginx. Сервер Nginx Unit выполняет сразу две функции: как веб-сервер он раздаёт файлы статики и медиа, а в роли сервера-приложений он запускает Python и Django. Таким образом Nginx Unit заменяет собой связку из двух сервисов Nginx и Gunicorn/uWSGI. [Подробнее про Nginx Unit](https://unit.nginx.org/).

## Как подготовить окружение к локальной разработке

Код в репозитории полностью докеризирован, поэтому для запуска приложения вам понадобится Docker. Инструкции по его установке ищите на официальных сайтах:

- [Get Started with Docker](https://www.docker.com/get-started/)

Вместе со свежей версией Docker к вам на компьютер автоматически будет установлен Docker Compose. Дальнейшие инструкции будут его активно использовать.

## Как запустить сайт для локальной разработки

Запустите базу данных и сайт:

```shell
$ docker compose up
```

В новом терминале, не выключая сайт, запустите несколько команд:

```shell
$ docker compose run --rm web ./manage.py migrate  # создаём/обновляем таблицы в БД
$ docker compose run --rm web ./manage.py createsuperuser  # создаём в БД учётку суперпользователя
```

Готово. Сайт будет доступен по адресу [http://127.0.0.1:8080](http://127.0.0.1:8080). Вход в админку находится по адресу [http://127.0.0.1:8000/admin/](http://127.0.0.1:8000/admin/).

## Как вести разработку

Все файлы с кодом django смонтированы внутрь докер-контейнера, чтобы Nginx Unit сразу видел изменения в коде и не требовал постоянно пересборки докер-образа -- достаточно перезапустить сервисы Docker Compose.

### Как обновить приложение из основного репозитория

Чтобы обновить приложение до последней версии подтяните код из центрального окружения и пересоберите докер-образы:

``` shell
$ git pull
$ docker compose build
```

После обновлении кода из репозитория стоит также обновить и схему БД. Вместе с коммитом могли прилететь новые миграции схемы БД, и без них код не запустится.

Чтобы не гадать заведётся код или нет — запускайте при каждом обновлении команду `migrate`. Если найдутся свежие миграции, то команда их применит:

```shell
$ docker compose run --rm web ./manage.py migrate
…
Running migrations:
  No migrations to apply.
```

### Как добавить библиотеку в зависимости

В качестве менеджера пакетов для образа с Django используется pip с файлом requirements.txt. Для установки новой библиотеки достаточно прописать её в файл requirements.txt и запустить сборку докер-образа:

```sh
$ docker compose build web
```

Аналогичным образом можно удалять библиотеки из зависимостей.

<a name="env-variables"></a>
## Переменные окружения

Образ с Django считывает настройки из переменных окружения:

`SECRET_KEY` -- обязательная секретная настройка Django. Это соль для генерации хэшей. Значение может быть любым, важно лишь, чтобы оно никому не было известно. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#secret-key).

`DEBUG` -- настройка Django для включения отладочного режима. Принимает значения `TRUE` или `FALSE`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#std:setting-DEBUG).

`ALLOWED_HOSTS` -- настройка Django со списком разрешённых адресов. Если запрос прилетит на другой адрес, то сайт ответит ошибкой 400. Можно перечислить несколько адресов через запятую, например `127.0.0.1,192.168.0.1,site.test`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#allowed-hosts).

`DATABASE_URL` -- адрес для подключения к базе данных PostgreSQL. Другие СУБД сайт не поддерживает. [Формат записи](https://github.com/jacobian/dj-database-url#url-schema).
# Развертывание на Minikube

Для развертывания на Minikube необходимо установить `minikube` и `kubectl`

## Запуск minikube

```shell
minikube start --driver=docker
```
## Быстрый старт POD в Minikube (в ручную)
### Запуск POD
```shell
kubectl run django-app-pod --image=elzig1999/django_app --port=8080 --env="SECRET_KEY=REPLACE_ME" --env="DATABASE_URL=postgres://..."
```
Определите свои переменые среды 

`SECRET_KEY` -- обязательная секретная настройка Django. Это соль для генерации хэшей. Значение может быть любым, важно лишь, чтобы оно никому не было известно. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#secret-key).

`DEBUG` -- настройка Django для включения отладочного режима. Принимает значения `TRUE` или `FALSE`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#std:setting-DEBUG).

`ALLOWED_HOSTS` -- настройка Django со списком разрешённых адресов. Если запрос прилетит на другой адрес, то сайт ответит ошибкой 400. Можно перечислить несколько адресов через запятую, например `127.0.0.1,192.168.0.1,site.test`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#allowed-hosts).

`DATABASE_URL` -- адрес для подключения к базе данных PostgreSQL. Другие СУБД сайт не поддерживает. [Формат записи](https://github.com/jacobian/dj-database-url#url-schema).
### Mapping Ports
```shell
kubectl port-forward pod/django-app-pod 8080:80
```
## Запуск Deploy (с manifest-файлом)
### Необходимо создать manifest-файл c [configMap-данными](https://kubernetes.io/docs/concepts/configuration/secret/)
1. Создайте файл yaml файл
2. Запольните его следующим оброзом 
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: django-app-configmap
data:
    SECRET_KEY: 'REPLACE_ME'
    ALLOWED_HOSTS: '*'
    DEBUG: 'false'
    DATABASE_URL: 'postgres://YOUR_DJANGO_DATABASE_URL'
```
Вы можете определять свои допольнительные переменные окружения
3. Запустите configMap-файл
```shell
kubectl apply -f <file_name>
```
4. Запустите deploy-файл
```shell
kubectl apply -f k8s/k8s-django-app-deploy.yaml
```
5. Веб приложение доступно с локального устройства
Что бы перейти на сайт вам нужно узнать ip-адрес кластера
```shell
minikube ip
```
Ответ имеет следующий вид
```text
192.168.49.2
```
приложение будет доступно по адресу http://$(minikube ip):30080. (http://192.168.49.2:30080)

## Запуск Ingress (с manifest-файлом)
Необходимо добавить расширение для работы с Ingress
```shell
minikube addons enable ingress
```
Добавте в переменые среды следующие в ALLOWED_HOSTS `star-burger.test`
Для запуска проекта в локальной сети также требуется добавить host в dns-запись. Для этого добвте запись в файле /etc/hosts:
```text 
# <minikube ip> star-burger.test
192.168.49.2 star-burger.test
```
Запустите:
```shell
kubectl apply -f k8s/k8s-django-app-ingress.yaml
```
Приложение доступна по адресу http://star-burger.test
## Запуск Очистки сессии(с manifest-файлом)
```shell
kubectl delete -f k8s/k8s-django-app-clearsessions.yaml
```
Таким оброзом сессия будет чиститься по 1 раз за час.



