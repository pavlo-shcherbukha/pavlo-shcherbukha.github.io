---
layout: post
title: "Node-RED in OpenShift"
date: 2023-08-17 10:00:01
categories: [openshift,node-RED]
permalink: posts/2023-08-17/node-red-in-openshift/
published: true
---

<!-- TOC BEGIN -->
- [1. Про що цей блог](#p-1)
- [2. Розгортання середовища розробки](#p-2)
- [3. Коротко про сутності черг Redis](#p-3)
- [4. Постановка задачі для прототипа](#p-4)
- [5. Автоматичний обробник, який буде запускатися та зупинятися користувачем](#p-5)

<!-- TOC END -->

## <a name="p-1">Про що цей блог</a>

Перший раз я познайомився з Node-RED  десь так 2016-2017 роках коли вивчав IBM-Cloud і,  сприйняв цей інструмент за якусь іграшку. І, це при тому, що сам на той час працюва з подібною, тільки великою іграшкою, що називається IBM Integration BUS. Але, ж IBM Integration BUS коштує дорого, а Node-RED - open source. І, до речі, у відповідності до [wiki](https://uk.wikipedia.org/wiki/Node-RED), народилася також в надрах IBM як інтеграційна шина, для інтеграції різних пристроїв. Потім, в 2016 році вона була віддана в opensource. І в хмарі IBM з'явилося 100-500 прикладів інтеграції різних IOT платформ, watson сервісів і все було зроблене на Node-RED. І, десь, в 2018 я почав розглядати Node-Red як легку альтернативу IBM Integration BUS  для не великих інтеграційних проектів. Ну, був у мене один робочий проект в хмарі IBM на Node-RED який мирно пропрацював до сьогодні. І, тепер настав час розширити та поглибити його. Тому, в цьому блозі, я зібрав свої ідеї з приводу організації колективної розробки під Node-RED  та deployment runtime  на Kubernetes based платформу OpenShift. Пройшлося трохи побігати по документації, трохи пошукати, потестувати, щоб вибрати найбільш прийнятний варіант роботи. І, щоб більше не шукати потрібну інформацію, все сконцентрував в цьому блозі. Звичайно, почнемо з розгортання середовища розробки.   Приклад можна знайти в моєму [github repo](https://github.com/pavlo-shcherbukha/node-red-in-openshift.git)


### <a name="p-2">2. Розгортання середовища розробки та його використання</a>

Якщо виконати розгортання  на windows машині так, як написано тут: [Running Node-RED locally](https://nodered.org/docs/getting-started/local), то у вас все встановиться в каталог **.node-red** у обіковому записі користувача. Мене не влаштовує, що все складається в каталозі під моєю обліковкою. При цьому кожний проект (кожний flows.json) буде в своєму каталозі, під управління окремого локального git  репозиторія ( в діалозі можна склонувати уже існуючий remote git repo). Але, всі встановлені пакети будуть  записуватися в **package.json**, який об'єднує всі проекти і цей файл  не під source control. І, якщо фінально здавати проект для Deployment в контейнер, то всі залежності треба переносити в ручну з загального **package.json** в аналогічний файл в каталозі проекту [pic-02](#pic-02).

<kbd><img src="/assets/img/posts/2023-08-18-node-red-and-openshift/doc/pic-02.png" /></kbd>
<p style="text-align: center;"><a name="pic-02">pic-02</a></p>

 Крім того з  remote репозиторіями git він працює по ssh  ключам, а я звик по http і токену. А якщо так, то пуш треба робити руками з командного рядку. Ну, і на додаток якщо виникає merge conflict,  то розбирати його прийдеться теж руками. Я вирішив покищо  виключити опцію **project**. І використати властивості Node.js. Для цього спроектував такий процес, в якому явно задаєтья опція з яким файлом потоків працювати. Іх в кореневому каталозі може бути багато, але  при розробці та deployment треба явно вказати який файл цікавить. Ну і викристав можливість задати settings.js такий, що відповідає окремо взятому середовищу: розробка, тестування, продуктивного


1. Створюю новий каталог проекту.
2. Стврюю пустий локальний git repo використовуючи 

```bash
    git init
```
ну, або ж можна зробити 

```bash
    git clone
```
якщо вже є remote репозторій на github та вирати бажаний branch.

3. Так як і для любого Node.js додатку створюю пустий npm репозиторій

```bash
    npm init

```
Після вводу  даних, що запитується під час виконання команди  отримуємо такий package.json

```json
{
  "name": "node-red-test01",
  "version": "1.0.0",
  "description": "testing nodre red development and deployment on openshift",
  "main": "index.js",
  "scripts": {
    "test": "node.exe red"
  },
  "author": "psh",
  "license": "ISC"
}

3.Інсталюю основний пакет Node-RED

```bash

npm install node-red

```
 І в результаті перший пакет  залежностей встановлено, див ключ **"dependencies"** в **package.json**.

 ```json
 {
  "name": "node-red-test01",
  "version": "1.0.0",
  "description": "testing nodre red development and deployment on openshift",
  "main": "index.js",
  "scripts": {
    "test": "node.exe red"
  },
  "author": "psh",
  "license": "ISC",
  "dependencies": {
    "node-red": "^3.0.2"
  }
}
```
4. Конфігурую ключ **"scripts"** в **package.json**  так, щоб  розробнику зручно було розробляти, запустити розроблений проект локально для тестування та щоб був рядок запуску при deployment. Для цього додаю в ключ **"scripts"**  три рядки

```json
 {
  "name": "node-red-test01",
  "version": "1.0.0",
  "description": "testing nodre red development and deployment on openshift",
  "main": "index.js",
  "scripts": {
    "dev": "node  node_modules/node-red/red.js --settings ./dev/settings.js --userDir .  --port 1880 --verbose user-registration.json",
    "test": "node  node_modules/node-red/red.js --settings ./test/settings.js --userDir .  --port 8080 --verbose user-registration.json",
    "start": "node  node_modules/node-red/red.js --settings ./prod/settings.js --userDir . --verbose --port 8080 $FLOW_NAME"
  },
  "author": "psh",
  "license": "ISC",
  "dependencies": {
    "node-red": "^3.0.2"
  }
}

```

Параметри командного рядка для запуску Node-RED  описані за лінком: [Command-line Usage](https://nodered.org/docs/getting-started/local#override-individual-settings) та процитовані на [pic-01](#pic-01)

<kbd><img src="/assets/img/posts/2023-08-18-node-red-and-openshift/doc/pic-01.png" /></kbd>
<p style="text-align: center;"><a name="pic-01">pic-01</a></p>


Ціль цих рядків полягає в тому, щоб:
- При розробці розробник запускав своє середовишще розробки командою 

```bash
    npm run dev
```
і при цьому на стандартному порту Node-RED,  розробник вказує flow, тому що в  налаштуваннями в файлі **settings.js** ведення проектів вимкнено.  
```js
        projects: {
            /** To enable the Projects feature, set this value to true */
            enabled: false,
            workflow: {
                /** Set the default projects workflow mode.
                 *  - manual - you must manually commit changes
                 *  - auto - changes are automatically committed
                 * This can be overridden per-user from the 'Git config'
                 * section of 'User Settings' within the editor
                 */
                mode: "manual"
            }
        },

```
Для тестування, трошки інша конфігурація, а саме:
    - вказано, інший порт, а саме 8080, що є так би мовити улюбленим для використання в openshift для  Node.js  додатків і запускається 

```bash
    npm run test

```    
При цьому з'являється можливість на локальній  машині одноасно запустити окілька проектів. Один, що розробляється на порту 1880, а залежні на порту 8080 в режимі npm  run test, на приклад. 


Для продуктиву ж там найменування flows  щадається через env змінну, що в майбутньому буде використано при deployment.
І зпускається 

```bash
    npm start

```   

Тобто  розробник запускає середвоище розробки і розробляє в звичному режимі. На [pic-03](#pic-03) показано запуск середовища розробки і спеціально акцентовано увагу на тому, який файл потоків запустився:

<kbd><img src="/assets/img/posts/2023-08-18-node-red-and-openshift/doc/pic-03.png" /></kbd>
<p style="text-align: center;"><a name="pic-03">pic-03</a></p>

Ну і для прикладу розроблено 4 потоки що ємулюють сервіси реєстрації користувача. З git я працюю з команного рядка, ну або якщо дуже захочеться, то використаю Visual Studio Code. По суті в процесі розробки  треба коммітити два файли:
- Файл потоків обробки
- package.json, якщо доставляли пакети.

Для прикладу в файлі **user-registration.json**  розробив 4 потоки - http методи, що емулюють реєстрацію користувача  
<kbd><img src="/assets/img/posts/2023-08-18-node-red-and-openshift/doc/pic-04.png" /></kbd>
<p style="text-align: center;"><a name="pic-04">pic-04</a></p>


Тепер в цьому ж проекті я зроблю ще отдин набір потоків, що бути викликати вже розроблені сервіси. Назву його: **loader.json**, що буде викликати по таймеру через задані інтервали  методи, що розроблені в  файлі **user-registration.json**.
Відповідно в package.json додам рядки для  запску розробки та тестування:

```json
{
  "name": "node-red-test01",
  "version": "1.0.0",
  "description": "testing nodre red development and deployment on openshift",
  "main": "index.js",
  "scripts": {
    "dev": "node  node_modules/node-red/red.js --settings ./dev/settings.js --userDir .  --port 1880 --verbose user-registration.json",
    "test": "node  node_modules/node-red/red.js --settings ./test/settings.js --userDir .  --port 8080 --verbose user-registration.json",
    "devloader": "node  node_modules/node-red/red.js --settings ./dev/settings.js --userDir .  --port 1880 --verbose loader.json",
    "testloader": "node  node_modules/node-red/red.js --settings ./test/settings.js --userDir .  --port 8080 --verbose loader.json",

    "start": "node  node_modules/node-red/red.js --settings ./prod/settings.js --userDir . --verbose --port 8080 $FLOW_NAME"
  },
  "author": "psh",
  "license": "ISC",
  "dependencies": {
    "node-red": "^3.0.2"
  }
}
```
Після цього **user-registration.json** запускаємо в режимі **npm run test**  а loader.json в режимі розробки: **npm run devloader**.

Тепер переходимо до deployment.





## <a name="p-3">3. Deployment</a>

Розгортати додатки Node-RED я буду в OpenShift.  Для цього використаю [пісочницю RedHat](https://developers.redhat.com/developer-sandbox) з доступним 2-тижневим екземпляром OpenShift. Для deployment я буду використовувати попередньо підготований  Docker файл, що лежть в кореневому каталозі:

```bash
FROM ubi8/nodejs-16

USER root
COPY . /tmp/src
RUN yum update -y && \
    yum install -y python39 gcc-c++ make
RUN alternatives --set python /usr/bin/python3
RUN chown -R 1001:0 /tmp/src
USER 1001
RUN /usr/libexec/s2i/assemble
EXPOSE 1880
EXPOSE 8080
CMD /usr/libexec/s2i/run

```

Як видно, використовуємо ертифікований RedHat контейнер [ubi8 Node.js](https://catalog.redhat.com/software/containers/ubi8/nodejs-16/615aee9fc739c0a4123a87e1), що локалізовано під Node.js. І додаток зпускається стандратною командою **npm start**. Фактично ж виконується команда, що прописана у відповідному ключі package.json:

```text
  "scripts": {
    "start": "node  node_modules/node-red/red.js --settings ./prod/settings.js --userDir . --verbose --port 8080 $FLOW_NAME"
  },

```

Саме розгортання в OpenShift побудовано на основі шаблонів, тобто  задаються почтакові параметри, і використовуючи їх з одного шаблону генеруэться кілька deployemnts. Приклад deployments знаходиться в каталозі **openshift\node-red**.
В файлі **app-srvc-templ.yaml** описаний шаблон розгортання сервсісу. Параметри винесені в кінці файлу:

```yaml
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
  - name: NODE_RED_FLOW
    displayName: FlowName 
    description: Full path to the Node Red Flow default flows.json
    required: true 

```

А в файлах **1-user-reg_srvc-process.cmd**,  **2-loader_srvc-process.cmd** з цього шаблону генеруються deployments для двох сервісів:
user-reg-srvc та loader-srvc. Для прикладу, наведені cmd файли:

- 1-user-reg_srvc-process.cmd 
```bash
@echo off
call ..\login.cmd
oc project %APP_PROJ%
pause

set fltempl=app-srvc-templ.yaml 
set fldepl=user-reg-srvc-depl.yaml 


set APP_SERVICE_NAME=user-reg-srvc
set APP_NAME=node-red-test
set GIT_BRANCH=master
set GIT_URL=https://github.com/pavlo-shcherbukha/node-red-in-openshift.git
set DOCKER_PTH=./Dockerfile
set NODE_RED_FLOW=./user-registration.json 

oc delete -f %fldepl%
pause
oc process -f %fltempl%  --param=NAMESPACE=%APP_PROJ%  --param=APP_SERVICE_NAME=%APP_SERVICE_NAME% --param=APP_NAME=%APP_NAME% --param=GIT_BRANCH=%GIT_BRANCH% --param=GIT_URL=%GIT_URL% --param=DOCKER_PTH=%DOCKER_PTH% --param=NODE_RED_FLOW=%NODE_RED_FLOW% -o yaml > %fldepl% 
pause
oc create -f %fldepl%
pause
 
```
- 2-loader_srvc-process.cmd 
```bash
@echo off
call ..\login.cmd
oc project %APP_PROJ%
pause

set fltempl=app-srvc-templ.yaml 
set fldepl=loader-srvc-depl.yaml 


set APP_SERVICE_NAME=loader-srvc
set APP_NAME=node-red-test
set GIT_BRANCH=master
set GIT_URL=https://github.com/pavlo-shcherbukha/node-red-in-openshift.git
set DOCKER_PTH=./Dockerfile
set NODE_RED_FLOW=./loader.json

oc delete -f %fldepl%
pause
oc process -f %fltempl%  --param=NAMESPACE=%APP_PROJ%  --param=APP_SERVICE_NAME=%APP_SERVICE_NAME% --param=APP_NAME=%APP_NAME% --param=GIT_BRANCH=%GIT_BRANCH% --param=GIT_URL=%GIT_URL% --param=DOCKER_PTH=%DOCKER_PTH% --param=NODE_RED_FLOW=%NODE_RED_FLOW% -o yaml > %fldepl% 
pause
oc create -f %fldepl%
pause
 
```

А  шаблон **app-srvc-route-templ.yaml**  призначений для  створення роутерів. Самі  параметри роутерів та генерація deployment описані в командних файлах 1-route-user-reg-process.cmd  та 2-route-loader-process.cmd.

І далі, все  управління додатком після deployment виконується як для стандартного Node.js додатка.


<kbd><img src="/assets/img/posts/2023-08-18-node-red-and-openshift/doc/pic-05.png" /></kbd>
<p style="text-align: center;"><a name="pic-05">pic-05</a></p>


Деталі можна подивтися за лінком:   [github repo](https://github.com/pavlo-shcherbukha/node-red-in-openshift.git)



