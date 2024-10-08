# Django Site

Докеризированный сайт на Django для экспериментов с Kubernetes.

Внутри контейнера Django приложение запускается с помощью Nginx Unit, не путать с Nginx. Сервер Nginx Unit выполняет сразу две функции: как веб-сервер он раздаёт файлы статики и медиа, а в роли сервера-приложений он запускает Python и Django. Таким образом Nginx Unit заменяет собой связку из двух сервисов Nginx и Gunicorn/uWSGI. [Подробнее про Nginx Unit](https://unit.nginx.org/).
## Оглавление
- [Как подготовить окружение к локальной разработке](#как-подготовить-окружение-к-локальной-разработке)
    - [Как запустить сайт для локальной разработки](#как-запустить сайт-для-локальной-разработки)
    - [Как вести разработку](#как-вести-разработку)
    - [Переменные окружения](#переменные-окружения)
    - [Как загрузить образ в Doker Hub](#как-загрузить-образ-в-docker-hub)
- [Запуск в локальном кластере minikube](#запуск-в-локальном-кластере-minikube)
    - [Запуск 1 POD в minikube для локального использования](#запуск-1-pod-в-minikube)
    - [Как запустить Базу Данных в кластере](#как-запустить-базу-данных-в-кластере)
    - [Где и как хранить переменные окружения](#где-и-как-хранить-переменные-окружения)
    - [Как запустить Deploy файл в кластере](#как-запустить-deploy-файл-в-кластере)
    - [Как запустить службу в кластере](#как-запустить-службу-в-кластере)
    - [Как запустить Ingress](#как-запустить-ingress)
    - [Запуск миграции](#запуск-миграции)
    - [Запуск Очистки сессии](#запуск-очистки-сессии)
- [Разработка в Yandex Cloud](#разработка-в-yandex-cloud)
    - [Как запустить Deploy файл в Yandex Cloud](#как-запустить-deploy-файл-в-yandex-cloud)
    - [Как запустить Службу в Yandex Cloud](#как-запустить-службу-в-yandex-cloud)
    - [Как собрать секреты в Yandex Cloud](#как-собрать-секреты-в-yandex-cloud)
    - [Запуск проекта в Yandex Cloud](#запуск-проекта-в-yandex-cloud)
## Как подготовить окружение к локальной разработке

Код в репозитории полностью докеризирован, поэтому для запуска приложения вам понадобится Docker. Инструкции по его установке ищите на официальных сайтах:

- [Get Started with Docker](https://www.docker.com/get-started/)

Вместе со свежей версией Docker к вам на компьютер автоматически будет установлен Docker Compose. Дальнейшие инструкции будут его активно использовать.

### Как запустить сайт для локальной разработки

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

### Как вести разработку

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
### Переменные окружения

Образ с Django считывает настройки из переменных окружения:

`SECRET_KEY` -- обязательная секретная настройка Django. Это соль для генерации хэшей. Значение может быть любым, важно лишь, чтобы оно никому не было известно. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#secret-key).

`DEBUG` -- настройка Django для включения отладочного режима. Принимает значения `TRUE` или `FALSE`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#std:setting-DEBUG).

`ALLOWED_HOSTS` -- настройка Django со списком разрешённых адресов. Если запрос прилетит на другой адрес, то сайт ответит ошибкой 400. Можно перечислить несколько адресов через запятую, например `127.0.0.1,192.168.0.1,site.test`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#allowed-hosts).

`DATABASE_URL` -- адрес для подключения к базе данных PostgreSQL. Другие СУБД сайт не поддерживает. [Формат записи](https://github.com/jacobian/dj-database-url#url-schema).
### Как загрузить образ в Doker Hub
Зарегистрируетесь на [Docker Hub](https://hub.docker.com/)
<br>
Создайте токен для подключения к docker hub и подключите локальное устройство к docker hub аккаунту.(https://docs.docker.com/security/for-developers/access-tokens/)
создайте образ с помощью команд:
```bash
docker build --file Dockerfile --tag <docker_hub_username>/django_app:<tag> ./backend_main_django
docker push <docker_hub_username>/django_app:<tag>
```
Где

`docker_hub_username` - имя пользователя в docker hub

`tag` - версия образа. Если не указывать, то версия по умолчанию будет `latest`.



## Запуск в локальном кластере minikube
Скачайте [Minikube](https://minikube.sigs.k8s.io/)<br>
Скачайте [Kubernetes](https://kubernetes.io/)<br>
Запустите minikube:
```shell
minikube start --driver=docker
```
### Запуск 1 POD в minikube
Примечание:
 * Воспользуйтесь ранее [загруженным образом](#как-загрузить-образ-в-doker-hub)
 * Базу данных можно поднять на локальном устройстве с помощью docker команды.
```bash
docker run -d -p 5432:5432 --env="POSTGRES_PASSWORD=REPLACE_ME" --env="POSTGRES_DB=REPLACE_ME" --env="POSTGRES_USER=REPLACE_ME" postgres:12.0-alpine
```
  * [База данных можно запутить и в кластере](#как-запустить-базу-данных-в-кластере) 
Запустите POD:
```shell
kubectl run django-app-pod --image=elzig1999/django_app --port=8080 --env="SECRET_KEY=REPLACE_ME" --env="DATABASE_URL=postgres://..."
```
У вас запустится 1 POD с названием `django-app-pod` в локальном кластере minikube.
Определите свои переменные среды 

`SECRET_KEY` -- обязательная секретная настройка Django. Это соль для генерации хэшей. Значение может быть любым, важно лишь, чтобы оно никому не было известно. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#secret-key).

`DEBUG` -- настройка Django для включения отладочного режима. Принимает значения `TRUE` или `FALSE`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#std:setting-DEBUG).

`ALLOWED_HOSTS` -- настройка Django со списком разрешённых адресов. Если запрос прилетит на другой адрес, то сайт ответит ошибкой 400. Можно перечислить несколько адресов через запятую, например `127.0.0.1,192.168.0.1,site.test`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#allowed-hosts).

`DATABASE_URL` -- адрес для подключения к базе данных PostgreSQL. Другие СУБД сайт не поддерживает. [Формат записи](https://github.com/jacobian/dj-database-url#url-schema).

Пробросьте порты для получения доступа к приложению с локального устройства.
```shell
kubectl port-forward pod/django-app-pod 8080:80
```
### Как запустить Базу Данных в кластере
Для создания базы данных вам нужно установить [helm](https://helm.sh/ru/)<br>
Далее с помощью [helm устанавливаем базу данных Postgres](https://artifacthub.io/packages/helm/bitnami/postgresql)

```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```
Установка Postgres в кластере
```shell
helm install pg-django bitnami/postgresql --version <VERSION> --set auth.postgresPassword=<YOUR_ROOT_PASSWORD>   --set auth.password=<YOUR_PASSWORD>   --set auth.username=<YOUR_USERNAME>  --set auth.database=<YOUR_DATABASE>  
```
Где:

`YOUR_ROOT_PASSWORD` - пароль для root пользователя

`YOUR_USERNAME` - имя пользователя

`YOUR_PASSWORD` - пароль для пользователя

`YOUR_DATABASE` - название базы данных которую нужно создать

Что бы узнать имя хоста базы данных в кластере выполните команду
```shell
kubectl get svc | grep pg-django
```
Ответ должен иметь вид
```text
NAME                      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
pg-django-postgresql      ClusterIP   10.103.84.37   <none>        5432/TCP   2m40s
pg-django-postgresql-hl   ClusterIP   None           <none>        5432/TCP   2m40s
```
Хостом будет `pg-django-postgresql`. Хост имя неизменно.
Также рекомендуется проверить успешность установки базы данных:
```shell
kubectl get pvc | grep data-pg-django
```
Результат должен иметь вид
```text
NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
data-pg-django-postgresql-0   Bound    pvc-f5c800d7-58ae-498d-ba0f-6314cc94bef4   8Gi        RWO            standard       <unset>                 7m50s
``` 

далее нужно изменить в secret-файле строку `DATABASE_URL`

### Где и как хранить переменные окружения
Создайте файл yaml файл. (Желательно в directory `k8s`-для сборки prod версии или `k8s_dev` для введение разработки)<br>
Заполните его следующим образом:
```yaml
apiVersion: v1
kind: Secret
metadata:
    name: django-app-secret # Имя секрета НЕ МЕНЯИТЬ!
type: Opaque
data:
    DEBUG: ZmFsc2U= # true dHJ1ZQ== # false ZmFsc2U=
    DATABASE_URL: YOUR_DJANGO_DATABASE_URL_IN_BASE64
    SECRET_KEY: YOUR_SECRET_KEY_IN_BASE64
    ALLOWED_HOSTS: MTI3LjAuMC4xLGxvY2FsaG9zdCwxOTIuMTY4LjQ5LjIsc3Rhci1idXJnZXIudGVzdA==
```
`SECRET_KEY` -- обязательная секретная настройка Django. Это соль для генерации хэшей. Значение может быть любым, важно лишь, чтобы оно никому не было известно. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#secret-key).

`DEBUG` -- настройка Django для включения отладочного режима. Принимает значения `TRUE` или `FALSE`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#std:setting-DEBUG).

`ALLOWED_HOSTS` -- настройка Django со списком разрешённых адресов. Если запрос прилетит на другой адрес, то сайт ответит ошибкой 400. Можно перечислить несколько адресов через запятую, например `127.0.0.1,192.168.0.1,site.test`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#allowed-hosts).

`DATABASE_URL` -- адрес для подключения к базе данных PostgreSQL. Другие СУБД сайт не поддерживает. [Формат записи](https://github.com/jacobian/dj-database-url#url-schema).

**ALLOWED_HOSTS** -- это список разрешённых адресов. Рекомендуется использовать `127.0.0.1`, `localhost`, ip address класстера и DNS имя сайта(далее в проекте будем использовать `star-burger.test`).<br>
Обратите внимание на то что значения для `DEBUG` и `ALLOWED_HOSTS`, `SECRET_KEY`, `DATABASE_URL` должны быть в base64 формате. Для кодирования в base 64 используйте `base64` команду и вставьте в `data`:
```bash
echo -n "SECRET" | base64
```

Запустите secret-файл в кластере

```bash
kubectl apply -f <file_name>
```
### Как запустить Deploy файл в кластере
***Примечание***:
* [запустите базу данных](#как-запустить-базу-данных-в-кластере)
* [создает secret-файл и запустите его в кластере](#где-и-как-хранить-переменные-окружения).

Соберите Deploy-файл (для prod версии используете `k8s/deployment.yaml`):
```bash
kubectl apply -f k8s/deployment.yaml 
```
Или (для dev версии используете `k8s_dev/deployment.yaml`):
```bash
kubectl apply -f k8s_dev/deployment.yaml
```
### Как запустить службу в кластере
Соберите службу (для prod версии используете `k8s/service.yaml` или `k8s_dev/service.yaml`):
```bash
kubectl apply -f k8s/service.yaml
```
ИЛИ
```bash
kubectl apply -f k8s_dev/service.yaml
```
### Как запустить Ingress
***Примечание***:
* [запустите базу данных](#как-запустить-базу-данных-в-кластере)
* [создает secret-файл и запустите его в кластере](#где-и-как-хранить-переменные-окружения).
* [запустите deployment](#как-запустить-deploy-файл-в-кластере)
* [запустите службу](#как-запустить-службу-в-кластере)
* Приложение будет доступно только локально на вашем устройстве.

Для запуска Ingress вам нужно добавить расширение для работы с Ingress:
```bash
minikube addons enable ingress
```
Далее нужно собрать Ingress-файл(для prod версии используете `k8s/ingress.yaml` или `k8s_dev/ingress.yaml`):
```bash
kubectl apply -f k8s/ingress.yaml
```
ИЛИ
```bash
kubectl apply -f k8s_dev/ingress.yaml
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
### Запуск миграции
Для запуска миграции вам нужно выполнить следующую команду:
```shell
kubectl apply -f k8s/migrat-job.yaml
```
ИЛИ
```shell
kubectl apply -f k8s_dev/migrat-job.yaml
```
### Запуск Очистки сессии
Для запуска очистки сессии вам нужно выполнить следующую команду:
```shell
kubectl delete -f k8s/clearsessions-cronjob.yaml
```
ИЛИ
```shell
kubectl delete -f k8s_dev/clearsessions-cronjob.yaml
```
Таким образом сессия будет чиститься каждый 1 час

## Разработка в Yandex Cloud
***Примечание***:
* В качестве UI для k8s [Lens Desktop](https://k8slens.dev/)
* Конфигурационный файл для kubectl, будут храниться в `./yc_dev/`
Для запуска сервиса в yandex cloud необходимо:
1. Подключиться к кластеру Yandex cloud
   1. [Установите и инициализируйте интерфейс командной строки Yandex Cloud](https://yandex.cloud/ru/docs/cli/quickstart#install)
   2. [Добавьте учетные данные](https://yandex.cloud/ru/docs/managed-kubernetes/operations/connect#kubectl-connect) кластера Kubernetes в конфигурационный файл kubectl:
     ```
     yc managed-kubernetes cluster get-credentials --id <cluster-id> --external
     ```
2. Используйте утилиту kubectl для работы с кластером Kubernetes:
    ```
      kubectl get cluster-info
      kubectl get pods --namespace=<your-namespace>
    ```
### Как запустить Deploy файл в Yandex Cloud
Для запуска deploy-файла в yandex cloud необходимо:
1. [Подключиться к кластеру Yandex cloud](#разработка-в-yandex-cloud)
2. Используйте утилиту kubectl для работы с кластером Kubernetes
    ```shell
   kubectl apply -f yc_dev/deployment.yaml
    ```
Тем самым вы создаете Deploy с nginx в кластере.
### Как запустить Службу в Yandex Cloud
Для запуска службы в yandex cloud необходимо:
1. [Подключиться к кластеру Yandex cloud](#разработка-в-yandex-cloud)
2. [Запустить Deploy в кластере(Nginx)](#как-запустить-deploy-файл-в-yandex-cloud)
3. Узнать свой порт для работы с NodePort (в моем случае 30281). Поменять его в `yc_dev/service.yaml`:
    ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: nginx-service
     labels:
        app.kubernetes.io/component: server
        app.kubernetes.io/instance: nginx-service
        app.kubernetes.io/name: nginx
     spec:
      ports:
        - nodePort: 30281 # Порт для работы с NodePort
          port: 80
          protocol: TCP
          targetPort: 80
      selector:
        app.kubernetes.io/name: nginx
      type: NodePort
   ```
4. Используйте утилиту kubectl для работы с кластером Kubernetes
    ```shell
   kubectl apply -f yc_dev/service.yaml
    ```
Теперь ваша приложение будет доступна по ранее выданному домену (в моем случае https://edu-elated-rosalind.sirius-k8s.dvmn.org/)
### Как собрать секреты в Yandex Cloud
1. Вам необходимо получить SSL ключ. [Инструкция для Yandex Cloud](https://yandex.cloud/ru/docs/managed-postgresql/operations/connect)
2. Для создания секретов с SSL сертификатами вам необходимо его закодировать в base64.
   Для этого используйте утилиту `base64`:
    ```shell
    base64 -w 0 ~/.postgresql/root.crt
    ```
    Далее полученный результат запишите в секрет
    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: pg-root-cert # Имя секрета Для дальнейшего использования
    data:
        root.crt: <base_64_encoded_data>
    ```
    Замените `<base_64_encoded_data>` на полученный результат

    Далее вы можете использовать этот секрет в pode или deploy файле. Следующим образом:
    ```yaml
    # ...
    spec:
        volumes:
        - name: secret-volume
          secret:
            secretName: pg-root-cert # Необходимо указать имя секрета
        containers:
          - image: # ...
            volumeMounts:
              - name: secret-volume
                readOnly: true
                mountPath: "/root/.postgresql"
    ```
    Таким образом он записывает в папку /root/.postgresql файль с SSL сертификатами (название файла будет `root.crt`).
3. Запустите POD и подключитесь к POD. 
    ```shell
    kubectl apply -f yc_dev/ubuntu_test_pg.yaml
   kubectl exec -it ubuntu -- /bin/bash
    ```
4. Подключитесь к базе данных.
    ```shell
   psql "host=c-c9qash3nb1v9********.rw.mdb.yandexcloud.net \
      port=6432 \
      sslmode=verify-full \
      dbname=<имя_БД> \
      user=<имя_пользователя> \
      target_session_attrs=read-write"
    ```
   
### Запуск проекта в Yandex Cloud
1. [Подключиться к кластеру Yandex cloud](#разработка-в-yandex-cloud)
2. [Создайте секрет](#где-и-как-хранить-переменные-окружения)
3. [Соберите службу для себя](#как-запустить-службу-в-yandex-cloud)

Соберите проект
```shell
kubectl apply -f yc_dev/deployment-django.yaml
kubectl apply -f yc_dev/service-django.yaml
```


## Дополнительно
### Как Удалить СУБД?
Для удаления базы данных вам нужно выполнить следующую команду
```shell
helm uninstall pg-django
```
Но если вы хотите удалить и данные в базе данных, то вам нужно выполнить следующую команду
```shell
kubectl delete pvc data-pg-django-postgresql-0
```
### Удалить проект из minikube
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







