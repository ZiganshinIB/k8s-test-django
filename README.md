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
## Полное развертывание
### 1. Необходимо создать secret-manifest c [secret-данными](https://kubernetes.io/docs/concepts/configuration/secret/)
1. Создайте файл yaml файл 
2. Запольните его следующим оброзом
```yaml
apiVersion: v1
kind: Secret
metadata:
    name: django-app-secret # Имя секрета
type: Opaque
data:
    DEBUG: ZmFsc2U= # true dHJ1ZQ== # false ZmFsc2U=
    DATABASE_URL: YOUR_DJANGO_DATABASE_URL_IN_BASE64
    SECRET_KEY: YOUR_SECRET_KEY_IN_BASE64
    ALLOWED_HOSTS: YOUR_ALLOWED_HOSTS_IN_BASE64
```
**ALLOWED_HOSTS** -- это список разрешённых адресов. Рекомендуется использовать `127.0.0.1`, `localhost`, ip address класстера и DNS имя сайта(далее в проекте будем использовать `star-burger.test`).
Вы можете определять свои допольнительные переменные окружения
3. Для кодирования в base 64 используйте `base64` команду и вставьте в `data`
```shell
echo -n "YOUR_DJANGO_DATABASE_URL" | base64
echo -n "YOUR_SECRET_KEY" | base64
# тут рекомендуем прописать  "127.0.0.1,localhost,192.168.49.2,star-burger.test"
echo -n "YOUR_ALLOWED_HOSTS" | base64
```


4. Запустите secret-файл
```shell
kubectl apply -f <file_name>
```
### 2. Запуск deploy-manifest
Для запуска Deploy-файла вам нужно выполнить следующую команду:
```shell
kubectl apply -f k8s/deployment.yaml
```
### 3. Запуск службы для Deploy-файла
Для запуска службы вам нужно выполнить следующую команду:
```shell
kubectl apply -f k8s/service.yaml
```
### 4. Запуск Ingress
Для запуска Ingress вам нужно добавить расширение для работы с Ingress:
```shell
minikube addons enable ingress
```
Далее нужно собрать Ingress-файл:
```shell
kubectl apply -f k8s/ingress.yaml
```
Веб приложение доступно с локального устройства
Что бы перейти на сайт вам нужно узнать ip-адрес кластера
```shell
minikube ip
```
Ответ будет иметь следующий вид
```text
192.168.49.2
```
Для запуска проекта в локальной сети также требуется добавить host в dns-запись. Для этого добвте запись в файле /etc/hosts:
```text 
# <minikube ip> star-burger.test
192.168.49.2 star-burger.test
```
После всего этого веб приложение будет доступна по адресу http://star-burger.test
### 5. Запуск миграции
Для запуска миграции вам нужно выполнить следующую команду:
```shell
kubectl apply -f k8s/migrat-job.yaml
```
### 6. Запуск Очистки сессии
Для запуска очистки сессии вам нужно выполнить следующую команду:
```shell
kubectl delete -f k8s/clearsessions-cronjob.yaml
```
Таким оброзом сессия будет чиститься по 1 раз за час.
## Допольнительно
### Создание базы данных
Для создания базы данных вам нужно установить [helm](https://helm.sh/ru/)
Далее с помощью [helm устанавливаем базу данных Postgres](https://artifacthub.io/packages/helm/bitnami/postgresql)
### Добавление репозитории
```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```
### Установка Postgres
```shell
helm install pg-django bitnami/postgresql --version <VERSION> --set auth.postgresPassword=<YOUR_ROOT_PASSWORD>   --set auth.password=<YOUR_PASSWORD>   --set auth.username=<YOUR_USERNAME>  --set auth.database=<YOUR_DATABASE>  
```
Где:

`VERSION` - версия postgresql

`YOUR_ROOT_PASSWORD` - пароль для root пользователя

`YOUR_USERNAME` - имя пользователя

`YOUR_PASSWORD` - пароль для пользователя

`YOUR_DATABASE` - название базы данных которую нужно создать

### Измениете secret-файл
Узнаем имя хоста у базы данных 
```shell
kubectl get svc | grep pg-django
```
Ответ должен иметь вид
```text
NAME                      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
pg-django-postgresql      ClusterIP   10.103.84.37   <none>        5432/TCP   2m40s
pg-django-postgresql-hl   ClusterIP   None           <none>        5432/TCP   2m40s
```
Хостом будет `pg-django-postgresql`. Хост имя не меняет 
Также рекомендуется проверить успешность установки базы данных:
```shell
kubectl get pvc | grep data-pg-django
```
Результат дожен иметь вид
```text
NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
data-pg-django-postgresql-0   Bound    pvc-f5c800d7-58ae-498d-ba0f-6314cc94bef4   8Gi        RWO            standard       <unset>                 7m50s
``` 

далее нужно изменить в secret-файле строку `DATABASE_URL`
На результат следующей комманды
```shell
echo -n "postgres://<YOUR_USERNAME>:<YOUR_PASSWORD>@pg-django-postgresql:5432/<YOUR_DATABASE>" | base64
```
где:

`YOUR_USERNAME` - имя пользователя. Которую вы ранеее указали

`YOUR_PASSWORD` - пароль. Который вы ранеее указали

`YOUR_DATABASE` - название базы данных. Которую вы ранеее указали

### Как Удалить СУБД?
Для удаления базы данных вам нужно выполнить следующую команду
```shell
helm uninstall pg-django
```
Но если вы хотите удалить и данные в базе данных, то вам нужно выполнить следующую команду
```shell
kubectl delete pvc data-pg-django-postgresql-0
```
## Удалить проект из minikube
Для удаления проекта вам нужно выполнить следующие команды
```shell
kubectl delete -f k8s/clearsessions-cronjob.yaml
kubectl delete -f k8s/ingress.yaml
kubectl delete -f k8s/service.yaml
kubectl delete -f k8s/deployment.yaml
```
Проверти состояние minikube
```shell
kubectl get pods
kubectl get svc
kubectl get pvc
kubectl get cronjobs
kubectl get ingress
```







