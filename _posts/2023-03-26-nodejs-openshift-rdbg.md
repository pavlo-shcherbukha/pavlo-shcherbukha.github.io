---
layout: post
title: "Remote debug node.js app on openshift"
date: 2023-03-26 10:00:01
categories: [node.js, openshift]
permalink: posts/2023-03-26/remote_debug_nodejs_app_on_openshift/
published: true
---

<!-- TOC BEGIN -->

<!-- TOC END -->

## <a name="p-1">Чому знадобилося вміння включати remote debug  для Node.js applications</a>

При розробці сучасних багатосервісних систем частно виникає необхідність remote debug. Чому виникає, та тому що запит з третьої сторони заходить на ваше середовище тільки з корпоративної мережі, і не так просто його перенаправити на вашу машину розробки. А  чи потрібна  така переадресація взагалі? У вас є специфічні бібліотеки чи залежні сервіси, що не доступні з вашого laptop. Тому, по перше, потрібно вміти і знати як включити reome debug. По друге, потріно збирати контейнери так, щоб вони дозволяли його включити. Цей блог, як раз і написаний як інструкція, як включити remote debug в контейнерах для Node.js express applications. В цьому блозі розгядається, як налаштувати Visual Studio Code  для remote debug контейнерів, що задеплоєні в Red Hat Openshift.


## <a name="p-2">Лінки на документацію  по debug</a>

Відправною точнку для  пошуку відподвіної документації був сайт Node.js. За лінком [Debugging Guide](https://nodejs.org/en/docs/guides/debugging-getting-started) описані основні ключі.

Дуже крисною виявилася документація по Visual Studio Code [Node.js debugging in VS Code](https://code.visualstudio.com/docs/nodejs/nodejs-debugging). Особливо корисний розділ [Attaching to Node.js](https://code.visualstudio.com/docs/nodejs/nodejs-debugging#_attaching-to-nodejs).

Якщо вже говоримо про OpenShift  то в ньому за звичай використовуються базові образи, орієнтовані на спеціальні frameworks  чи мови програмування із серії UBI. В свїх дослідах використовував UBI8 для Node.JS. На приклад: [UBI8 for Node.js 16 ](https://catalog.redhat.com/software/containers/ubi8/nodejs-16/615aee9fc739c0a4123a87e1). Тут вже готовий налаштований образ з специфічним env змінними, що дозволяють управляти роботою контейнерезованого додатку і до нього за лінокм є опис з прикладами використання.

Ну, і як альтенативу, розглянув можливість використання образу типу Alpina [Nodejs Alpina](https://hub.docker.com/_/node). Основною перевагою цього образу є його малий розмір:

> This image is based on the popular Alpine Linux project, available in the alpine official image. Alpine Linux is much smaller than most distribution base images (~5MB), and thus leads to much slimmer images in general.

Але на притивагу alpina у RedHat є група  контейнерних образів із серії "mini": [ubi8/nodejs-16-minimal](https://catalog.redhat.com/software/containers/ubi8/nodejs-16-minimal/615aefd53f6014fa45ae1ae2?container-tabs=overview).

 Що цікаво, в UBI8, уже  передбачено все портіне для розробника.  Образ "допилювати" не потрібно, на відміну від Alpine.

З приводу контейнерізації Node.js app можна ще прочитати за лінками:

- https://raka.hashnode.dev/minimize-docker-container-for-nodejs

- https://www.youtube.com/watch?v=mA8wtTUCdgc


##  <a name="p-4">Опис демо для включення remote debug</a>

Демонстрація, як включити remote debug в openhsift  для Node.js application наведена в github  за лінком [ubi8-nodejs-rdbg Remote Debug using Node.js express app on openshift](https://github.com/pavlo-shcherbukha/ubi8-nodejs-rdbg.git). 

В readme.md  детально описано як розгорнути та запустити додаток в openshift та як налаштувати debug. На мій погляд aplina image  потребує доопрацювання для того щоб гнучко запускати його за різних умов, на відміну від ubi8 -  де порібно скопіювати тільки програмний код, а все інше виконується налаштуванням env змінних.  
## <a name="p-5">Супутні цікаві технології, що використані в цій demo</a>

- При розробці deplyment використовувалися технології S2i, тобто deployment прямо з програмного коду в github. Досить швидко і зручно. Можливо не дуже підходить на production.

- Використовувася deployment з Dockerfile, що знаходиться в github. При цьому була одна особливість, мені порібно було перенсти програмний код на образ не командою **COPY**  з мого laptop, а використовуючи git clone в контейнері. Враховуючи, що використовувався приватний репозиторій, потрібно було придумати, як  передати і підставити в URL параметри доступу до github. Це вирішено за допомогою  **git config --global url..insteadOf**:

```bash
RUN git config --global url."https://${GIT_USER}:${GIT_PSW}@github.com/".insteadOf "https://github.com/"

RUN git clone ${GIT_URL} /usr/appsrc -b ${GIT_BRANCH}

```

На приклад ось за лінком: [enshift/ubi8_docker_deployment/Dockerfile](https://github.com/pavlo-shcherbukha/ubi8-nodejs-rdbg/blob/main/openshift/ubi8_docker_deployment/Dockerfile)

- В alpina запус додатка ведеться через додавтковий sh  скрипт, що формує env  змінні для debug  та для запску з допомогою npm: **apprun.sh**

```bash
#!/bin/bash

run_node() {
  if [ -z "$DEBUG_PORT" ]; then
     export DEBUG_PORT=5858
  fi

  if [ -z "$NPM_RUN" ]; then
     export NPM_RUN=start
  fi
}
run_node
echo "Launching via npm..."
exec npm run -d $NPM_RUN

```

Приклад за лінком: [openshift/alpina_docker_deployment/Dockerfile](https://github.com/pavlo-shcherbukha/ubi8-nodejs-rdbg/blob/main/openshift/alpina_docker_deployment/Dockerfile)


- Deployment ведестья за допомогою шаблонів OpenShift


Лінки на документацію з приводу шаблонів:
- [Openshift 4.5 Using templates](https://docs.openshift.com/container-platform/4.5/openshift_images/using-templates.html)
- [OpenShift Templates Development Guide](http://v1.uncontained.io/playbooks/fundamentals/template_development_guide.html)

Використовувати шаблони виявилося досить зурчно, тому, що можна винести якусь частину  налаштувань в параметризацію. На приклад як в цьому файлі [nodejs-ubi8docker-srvc-templ.yaml](https://github.com/pavlo-shcherbukha/ubi8-nodejs-rdbg/blob/main/openshift/ubi8_docker_deployment/nodejs-ubi8docker-srvc-templ.yaml), і deployment легко перелаштовується для розгортання дублюючого сервісу в тому ж namespace або перенесення взагалі на інший кластер 

Ось що винесено в параметри:

```text
parameters:
  - name: NAMESPACE
    displayName: Namespace 
    description: The Namespace where service must be deployed. 
    required: true   
  - name: APP_SERVICE_NAME
    displayName: APP Service Name 
    description: The name of the OpenShift Service exposed for the APP.
    required: true   
  - name: APP_NAME
    displayName: Application Name 
    description: The name of the OpenShift Application  for Groupe of Servoces.
    required: true 
  - name: GIT_BRANCH
    displayName: Git branch
    description: Git branch where source code
    required: true 
  - name: GIT_URL
    displayName: Git URL 
    description: Git url where source code
    required: true 
  - name: DOCKER_PTH
    displayName: Dockerfile path 
    description: Path to Dockerfile
    required: true 
```
Або параметризація роутера: [nodejs-ubi8docker-route-temp.yaml](https://github.com/pavlo-shcherbukha/ubi8-nodejs-rdbg/blob/main/openshift/ubi8_docker_deployment/nodejs-ubi8docker-route-temp.yaml).

```text
parameters:
  - name: NAMESPACE
    displayName: Namespace 
    description: The Namespace where router must be deployed. 
    required: true   
  - name: APP_SERVICE_NAME
    displayName: APP Service Name 
    description: The name of the OpenShift service exposed for the APP.
    required: true   
  - name: APP_NAME
    displayName: Application Name 
    description: The name of the OpenShift Application  for Groupe of Servoces.
    required: true 
  - name: ROUTENAME
    displayName: ROUTE name 
    description: Input ROUTENAME name 
    required: true   
  - name: HOSTNAME
    displayName: HOST name 
    description: Input HOSTNAME name 
    required: true 
  - name: PORTNUMBER
    displayName: PORT NUMBER 
    description: Input PORT number 
    required: true 
```

При цьому deployment складається з 3 кроків [route-process.cmd](https://github.com/pavlo-shcherbukha/ubi8-nodejs-rdbg/blob/main/openshift/ubi8_docker_deployment/route-process.cmd):

```bash

@echo on

rem логін в openshift
call ..\login.cmd

rem  вибираємо потрібний проект (namespace)
oc project %APP_PROJ%

rem вказуємо файл шаблону
set fltempl=nodejs-ubi8docker-route-temp.yaml

rem вказуємо вихідний файл deployment, який потім і будемо деплоїти
set fldepl=nodejs-ubi8docker-route-depl.yaml
 
rem Вказую параметри окремо взятого deployemnt
rem та вивджу їх в консоль для візуального контролю
echo %APP_PROJ% .....
echo %APP_DNS%   .....

set APP_SERVICE_NAME=nodejs-ubi8docker-srvc
set APP_NAME=nodejs-remote-debug
set ROUTENAME=nodeubi8docker-srvc-%APP_PROJ%.%APP_DNS%
set SRVHOSTNAME=nodeubi8docker-srvc-%APP_PROJ%.%APP_DNS%
set PORTNUMBER=8080

echo APP_SERVICE_NAME=%APP_SERVICE_NAME%
echo APP_NAME=%APP_NAME%
echo ROUTENAME=%ROUTENAME%
echo SRVHOSTNAME=%SRVHOSTNAME%
echo PORTNUMBER=%PORTNUMBER%

rem видаляю попредній deployment
oc delete -f %fldepl%

rem Обробляю шаблон з урахуванням параметрів та генерую новий файл deployment
oc process -f %fltempl% --param=NAMESPACE=%APP_PROJ% --param=APP_SERVICE_NAME=%APP_SERVICE_NAME% --param=APP_NAME=%APP_NAME% --param=ROUTENAME=%ROUTENAME% --param=HOSTNAME=%SRVHOSTNAME% --param=PORTNUMBER=%PORTNUMBER% -o yaml > %fldepl% 

rem Виконуюю сам deployemnt
oc create -f %fldepl%

rem все
echo finish
pause
```