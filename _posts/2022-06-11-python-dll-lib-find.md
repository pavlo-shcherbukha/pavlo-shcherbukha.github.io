---
layout: post
title: "Python, як підключити .dll, .so бібліотеки"
date: 2022-06-11 10:00:01
categories: [python]
permalink: posts/2022-06-11/python-dll-lib-find/
published: true
---

## Підключення .so бібліотек на платформі windows 64 з python 3.9.

Написав невеликий тестовий скрипт на платформі windows, який  повинен  підключати динамічні DLL бібліотеки. Ніякі спроби вказати - де шукати бібіліотеки, по path або в поточній директорії успіху не мали. 
Як виявилося, Якщо запускати python без повного шляху  або запускати через Visual Studio Code, як показано :

```py
python.exe -B test.py

```
, то pyton шукає бібліотеки в каталозі windows\system32.


Якщо ж тест запукати python з повним шляїом, як показагл тут:

```py
C:\Python39_64\python.exe -B test.py

```
То "побачить" бібліотеки в поточному каталозі.


Зважаючи на викладене - робота на платформі windows має деякі скланощі та  не очевидні особливості, що можуть перешкоджати роботі інших програм (продукам), тому, маючи аналогічні бібліотеки для Linux, вирішив це все запускати на linux контейнерах. 


## Підключення .so бібліотек на платформі linux 64 з python 3.9.

Зважаючи на проблеми з підключенням бібліотек на платфрмі windows  вирішив запустити їх в linux контейнерах. Для вивчення, як це запустити, спершу використав [стандартнмй контейнер python з DockerHub](https://hub.docker.com/_/python). 
В даному випадку всі бібліотеки покладені в один каталог з .py скриптом. Але, потрібно мати на увазі, що linux .so бібліотеки підключаються з допомогою установки env-змінної **LD_LIBRARY_PATH**. В internet можна знайти варіант задавання env-змінних в коді, викоистовуючи пакет oc

```py
#  так
os.environ['LD_LIBRARY_PATH'] = "./x64"

# або так, коли бібліотеки в одному каталозі з py скриптами
os.environ['LD_LIBRARY_PATH'] = os.getcwd()
```
, та для контейнера це не працювало. Але, спрацювало явне задання змінної при старті контейнрера: 

```bash
docker run -e LD_LIBRARY_PATH=/usr/src/app myrepo/sh-test-py-app_s 
```

Сам Dockefile має такий вигляд:

```bash

FROM python:3.9
WORKDIR /usr/src/app
# COPY requirements.txt ./
# RUN pip install --no-cache-dir -r requirements.txt
RUN pip install py-cpuinfo
COPY ./app-src .
CMD [ "python", "-B", "test.py" ]

```
Побудова контейнера виглядає так:

```bash

echo створити образ
docker build -t myrepo/sh-test-py-app_s .

echo tag
docker tag myrepo/sh-test-py-app_s:latest myrepo/sh-test-py-app_s:1.0.0

echo push to docker
rem docker push myrepo/sh-test-py-app:1.0.0

```
. Звичайно, потрібно замінити **myrepo/** на ваш Docker репозиторій. 

## Підключення python бібліотекна платформі linux 64 з python 3.9 в стандартному UBI-8 контейнері від RedHat.

Так як передбаається deployment на OpenShift то логічно використовувати сертифіковані Redhat контейнери. За звичай використовуємо [Ubi-8 with python 9.3](https://catalog.redhat.com/software/containers/ubi8/python-39/6065b24eb92fbda3a4c65d8f). Образ уже адаптований під всі security правила та під python і гарантовано запуститься під openhsift.

Основна відмінність у запуску цього контейнера - тільки каталог, куди копіювати програмний код і, відповідно задання змінної **LD_LIBRARY_PATH**:

```bash
# LD_LIBRARY_PATH=/opt/app-root/src
docker run -e LD_LIBRARY_PATH=/opt/app-root/src myrepo/sh-test-py-app_s 
```
Все інше майже без змін.

## Можливості debug в контейнері

З огляду на те, що працюємо з контейнерами, бажано б освоїти інструменти debug  коду в контейнерах та очистки машини від непотрібних образів та контейнерів. 

Про Remote Debug в контейнерах  для VSC можна почитати по лінку: https://code.visualstudio.com/docs/containers/debug-python.

Про очистку від контейнернго сміття:

Видалити всі контейнри, що зупинені: 

```bash
    docker rm $(docker ps -q -f status=exited)
```

Видалити всі образи, що не мають мітки tag (що не зібралися успішно):

```bash
    docker rmi $(docker images -f “dangling=true” -q)

```
