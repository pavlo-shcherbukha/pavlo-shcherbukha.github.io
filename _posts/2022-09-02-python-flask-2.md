---
layout: post
title: "Python - flask запуск в контейнері від RadHat UBI8"
date: 2022-09-02 10:00:01
categories: [python, flask]
permalink: posts/2022-09-02/python-flask-2/
published: true
---

## Зміст

<!-- TOC BEGIN -->
1. [Про що цей блог](#p-1)
2. [Розгортання Flask  applicatin  для  продуктивної ксплуатації](#p-2)
3. [Побудова образу локально](#p-3)
4. [Побудова образу з використанням аргументів](#p-3)
5. [Висновки](#p-5)


<!-- TOC END -->



## <a name="p-1">Про що цей блог</a>

Це продовження записок початківця про python flask. В цьому блозі:

- створено більш розширене application, що  має кілька методів web servce, які показують як прцювати з різними типами заптиів (POST, PUT, GET, DELEETE)

- Як передавати параметри в тілі запиту (POST m PUT)

- Як передавати параметри в URL, як частина URL (GET, DELETE, PUT)

- Як передавати параметри в URL, якпарамери (GET)

Такж показано, як  можна обробляти помилки Rest API та давати коректну відповідь в JSON

приклад цього FLASK application знаходиться в репозиторії [py-flask-app-smpl-restapi-for-ubi8](https://github.com/pavlo-shcherbukha/py-flask-app-smpl-restapi-for-ubi8) та всеб апі описано в README.MD до цього репозиторію

Основна ж  ціль цього блогу запустити цей сервіс в контейнері.  Базовим образом для цього контейнеру вибрано UBI8 з Python 3.9 від RedHat [UBI8 з адаптацією під Python 3.9,  ubi8/python-39 ](https://catalog.redhat.com/software/containers/ubi8/python-39/6065b24eb92fbda3a4c65d8f), тому що мені його ще в RedHat openshift запускати. Та і взагалі, як на мене, RedHat більш надійна платформа  для enterprise сегменту (хоча це питання дискусійне).


## <a name="p-2">Розгортання Flask  applicatin  для  продуктивної експлуатації</a>


Після розробки web service на базі Flask портібно зробити його загальнодоступним для інших користувачів. Коли ми розробляли і запсукали його лоакльно  командою

```bash
python -m flask run

```
, то використовували вбудований сервер розробки. Це не можна використовувати для продуктивної експлуатації. Замість цього слід використовувати спеціальний сервер WSGI або платформу хостингу. Flaskявляє собою  WSGI application. А WSGI server використовується для зауску application. Є багато WSGI серверів. Ось наведено їх не великий список:

    Gunicorn
    Waitress
    mod_wsgi
    uWSGI
    gevent
    eventlet
    ASGI

За звичай  дуже популярним є [Gunicorn](https://flask.palletsprojects.com/en/2.2.x/deploying/gunicorn/), але, на стільки я зрозумів, він під windows не працює, а працює під Linux. Тому під нми і будемо запускати додаток в контейнері. В допонення можу послатися на статтю 2021 року [Deploy multiple Flask Applications using Nginx and Gunicorn](https://towardsdatascience.com/deploy-multiple-flask-applications-using-nginx-and-gunicorn-16f8f7865497), де описано дость життэвий приклад.

Тому спробуємо зібрати і запустит контейнер локально





## <a name="p-3">Побудова образу локально</a>

Для побудови образу локально достатьньо скопіювати програмний код в контейнер. Про це детально написано в описі до  [UBI8 з адаптацією під Python 3.9,  ubi8/python-39 ](https://catalog.redhat.com/software/containers/ubi8/python-39/6065b24eb92fbda3a4c65d8f).
Пркиклад Dockerfile наведено в корені [py-flask-app-smpl-restapi-for-ubi8](https://github.com/pavlo-shcherbukha/py-flask-app-smpl-restapi-for-ubi8)  в файлі **[py-flask-app-smpl-restapi-for-ubi8](https://github.com/pavlo-shcherbukha/py-flask-app-smpl-restapi-for-ubi8)**

```text

FROM registry.fedoraproject.org/f33/python3
# Add application sources to a directory that the assemble script expects them
# and set permissions so that the container runs without root access
USER 0
COPY . /tmp/src
RUN /usr/bin/fix-permissions /tmp/src
USER 1001

# Install the dependencies
RUN python3.9 -m pip install --upgrade pip
RUN /usr/libexec/s2i/assemble
EXPOSE 8080
#EXPOSE 5000
# Set the default command for the resulting image
CMD /usr/libexec/s2i/run

```
По суті, командою 

```text
COPY . /tmp/src
```
ми копіюємо з локальнго каталога програмний код в контейнер.
Команда 

```text
RUN python3.9 -m pip install --upgrade pip
RUN /usr/libexec/s2i/assemble
```
інсталює необхідні бібліотеки з файла **requiremenst.txt**. Web service буде запускатися в контейнері на порту 8080, що є типовим для ReadHat:

```text
EXPOSE 8080
```

Для запуску побудови контейнера використовуємо команду:

```bash
docker build  --file ./Dockerfile_local   -t pshkxml/smplapp-srvc-ubi8 .

```

Побудова образу записана в cmd файлі: **sh-build-local.cmd** в репозиторії програмнго коду. На [pic-01](#pic-01) показноа результат.

<kbd><img src="/assets/img/posts/2022-09-02-python-flask-2/doc/pic-01.png" /></kbd>
<p style="text-align: center;"><a name="pic-01">pic-01</a></p> 

Переглянути наявність побудованих образів можна командою

```bash

docker images pshkxml/smplapp-srvc-ubi8

```

<kbd><img src="/assets/img/posts/2022-09-02-python-flask-2/doc/pic-02.png" /></kbd>
<p style="text-align: center;"><a name="pic-02">pic-02</a></p> 

Тепер спробуємо запустит цей образ на локальному компютері , та подивитися, що відбувається в середені контейнеру.
Запуск контейнеру виконуться командою:

```bash
docker run -e GUNICORN_CMD_ARGS="--workers=1 --bind=0.0.0.0:8080 --access-logfile=-" -p 8081:8080 pshkxml/smplapp-srvc-ubi8 

````
,що записана в **sh-build-local.cmd** в корені репозиторію

- -e GUNICORN_CMD_ARGS="--workers=1 --bind=0.0.0.0:8080 --access-logfile=-"
описує аргументи для запуску gunicorn.  --workers=1 - означає що запускаео тільки 1 поток. --bind=0.0.0.0:8080 - означанє, що запити надходять з любих ip на порт 8080.

- -p 8081:8080 вказує, що порт контейнера 8080  відображається на порт 8081 на ваш localhost  для підключення.

При спробы запустити отримали помилку:

<kbd><img src="/assets/img/posts/2022-09-02-python-flask-2/doc/pic-03.png" /></kbd>
<p style="text-align: center;"><a name="pic-03">pic-03</a></p> 

, що вимагає передати env-змінну, який модуль потрібно запускати. Про що ясно написно в документації на контейнер:

<kbd><img src="/assets/img/posts/2022-09-02-python-flask-2/doc/pic-04.png" /></kbd>
<p style="text-align: center;"><a name="pic-04">pic-04</a></p> 

Таким чином модифікуємо нашу команду для запуску контейнера:

```

docker run -e APP_MODULE="hello_app.webapp" -e GUNICORN_CMD_ARGS="--workers=1 --bind=0.0.0.0:8080 --access-logfile=-" -p 8081:8080 pshkxml/smplapp-srvc-ubi8 

```

І Вуаля - контейнер запустився!

<kbd><img src="/assets/img/posts/2022-09-02-python-flask-2/doc/pic-05.png" /></kbd>
<p style="text-align: center;"><a name="pic-05">pic-05</a></p> 

А по http://localhost:8081/ выдкрлась головна сторінка і /api/health працює! 

<kbd><img src="/assets/img/posts/2022-09-02-python-flask-2/doc/pic-05.png" /></kbd>
<p style="text-align: center;"><a name="pic-05">pic-05</a></p> 



Тепер перевіримо, а скільки ж потоків gunicorn в контейнері працює?

Для цього перевіримо в іншому сеансі CMD, які контейнри запущені командою

```json
dcoker ps

```

<kbd><img src="/assets/img/posts/2022-09-02-python-flask-2/doc/pic-07.png" /></kbd>
<p style="text-align: center;"><a name="pic-07">pic-07</a></p> 

Та підкоючимося до цього контейнера з допомогою bash

```bash
docker exec  -u root -it 949f2ac21f27   /bin/bash
```
 
<kbd><img src="/assets/img/posts/2022-09-02-python-flask-2/doc/pic-08.png" /></kbd>
<p style="text-align: center;"><a name="pic-08">pic-08</a></p> 

Я к бачимо, підключидися під рутом і отримали список файлі в контейнері. Тепер командою bash отримаємо спиок процесів в контейнері:

```bash
ps -aux

```
<kbd><img src="/assets/img/posts/2022-09-02-python-flask-2/doc/pic-09.png" /></kbd>
<p style="text-align: center;"><a name="pic-09">pic-09</a></p> 


як видно один основний і один worker.  Спробємо тепер запустити контейнер без вказування кількості workers і знову подивимося кількість процесів. Для цього виконаємо таку послідовність коман:

```bash
# Stop container
docker container stop 949f2ac21f27

# delete container, not image
docker container rm 949f2ac21f27

# Start  new container
docker run -e APP_MODULE="hello_app.webapp" -e GUNICORN_CMD_ARGS="--bind=0.0.0.0:8080 --access-logfile=-" -p 8081:8080 pshkxml/smplapp-srvc-ubi8

# get new continer id
docker ps

# enter into container using bash
docker exec  -u root -it d0a76de21477   /bin/bash

# get pid processses
ps -aux

# the list of pid's is shown on pic-11

# then stop and delete
# Stop container
docker container stop d0a76de21477

# delete container, not image
docker container rm d0a76de21477

```
<kbd><img src="/assets/img/posts/2022-09-02-python-flask-2/doc/pic-11.png" /></kbd>
<p style="text-align: center;"><a name="pic-11">pic-11</a></p> 

Таким чином видно, як можна масштабувати нас сервіс в одному контейнері. А якщо уявити, що це запустимо на кількох контейнерах в kubernetes/openshift - то взагалі виглядає досить заворожуюче.



## <a name="p-4">Побудова образу з використанням аргументів</a>

Тепер інша практична проблема. Я будував контейнер з модулів програмного коду,  що лежали  локально на робочій станції. Але на практиці програмний код потрібно клонувати з git репозиторію. Але ж не буду я прописувати параметри git праямо в докер-файлі. Та, інколи ще якісь credentials треба передати в контейнер. Для цих цілей використовується побудова образа з використанням аргументів.

Для передачі аргументів в Dockerfile  під час побудови використовується команда ARG **Dockerfile_git**. Тут показано, як посати аргументи, та як їх використаті в Docker File

```text
FROM registry.fedoraproject.org/f33/python3
# ARGS for dockerfile

ARG SH_GIT_URL
ARG SH_GIT_BRANCH

#
#
# Add application sources to a directory that the assemble script expects them
# and set permissions so that the container runs without root access
USER 0
#COPY . /tmp/src
RUN git clone ${SH_GIT_URL} /tmp/src -b ${SH_GIT_BRANCH}


RUN /usr/bin/fix-permissions /tmp/src
USER 1001

# Install the dependencies
RUN python3.9 -m pip install --upgrade pip
RUN /usr/libexec/s2i/assemble
EXPOSE 8080
# EXPOSE 5000
# Set the default command for the resulting image
CMD /usr/libexec/s2i/run


```

Побудувати образ можна командою, яка передбачає задання аргуменітв Dockerfile  **sh-build-git.cmd**:

```bash
docker build --build-arg SH_GIT_URL=https://github.com/pavlo-shcherbukha/py-flask-app-smpl-restapi-for-ubi8.git --build-arg SH_GIT_BRANCH=tz-000001-init --file ./Dockerfile_git -t pshkxml/smplapp-srvc-ubi8 .

```
Аргументи задаються ключем --build-arg


## <a name="p-5">Висновки</a>

Це елементарні приклади для початківця щоб  запустити python appliction в docker контейнері та навчитися з ним працювати. Ну а  в репозиторії програмного коду показані типові приклади побудови api/
Сам репозиторій з прикладом знаходиться за лінком: [py-flask-app-smpl-restapi-for-ubi8](https://github.com/pavlo-shcherbukha/py-flask-app-smpl-restapi-for-ubi8/tree/main)

