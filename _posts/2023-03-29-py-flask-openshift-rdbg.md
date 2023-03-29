---
layout: post
title: "Remote debug Python Flask app on openshift"
date: 2023-03-29 10:00:01
categories: [Python, Flask, openshift]
permalink: posts/2023-03-29/remote_debug_py_flask_app_on_openshift/
published: true
---

<!-- TOC BEGIN -->

<!-- TOC END -->

## <a name="p-1">Чому знадобилося вміння включати remote debug  для Node.js applications</a>

При розробці сучасних багатосервісних систем частно виникає необхідність remote debug. Чому виникає, та тому що запит з третьої сторони заходить на ваше середовище тільки з корпоративної мережі, і не так просто його перенаправити на вашу машину розробки. А  чи потрібна  така переадресація взагалі? У вас є специфічні бібліотеки чи залежні сервіси, що не доступні з вашого laptop. Тому, по перше, потрібно вміти і знати як включити reome debug. По друге, потріно збирати контейнери так, щоб вони дозволяли його включити. Цей блог, як раз і написаний як інструкція, як включити remote debug в контейнерах для Node.js express applications. В цьому блозі розгядається, як налаштувати Visual Studio Code  для remote debug контейнерів, що задеплоєні в Red Hat Openshift.


## <a name="p-2">Лінки на документацію  по debug</a>

Flask template розроблений таким чином, що  його можна запустити в **development mode**, як описано в [Створення найпростішого скрипта для flask app та запуск applicaiton з індивідуальною назвою в DEVELOPENT mode](https://pavlo-shcherbukha.github.io/posts/2022-09-02/python-flask-1/). А бо ж flask application запускається в **productive mode** за WSGI  сервером типу gunicorn, як описано в [Python - flask запуск в контейнері від RadHat UBI8](https://pavlo-shcherbukha.github.io/posts/2022-09-02/python-flask-2/).

Збірка запуск та debug контейнера з сервісом python описана  за цими лінками: 

- [Visual Studio Code:  Python in a container](https://code.visualstudio.com/docs/containers/quickstart-python);
- [Visual Studio Code: Debug Python within a container](https://code.visualstudio.com/docs/containers/debug-python).

Але, воно мене не дуже влаштовує, так як запускає окремий контейнер. А мені потрібно задеплоїти додаток в openshift а вже потім підключати debug. При цьому, переключення між debug mode  та звичайним режимом не повинно передбачати переудови образу - тільки рестарт контейнера можливий.

Раптом мені попалася бібліотека від Microsoft, що забезепечує взаємодію debug  протоколу  python додатків та Visual Studio Code: [ptvsd](https://pypi.org/project/ptvsd/) або ж лінк на github [Python Tools for Visual Studio debug server](https://github.com/microsoft/ptvsd). Також, зандобилася бібліотека [debugpy](https://pypi.org/project/debugpy/) , або на github [debugpy - a debugger for Python](https://github.com/microsoft/debugpy).

Таким чином, мені для запуску debug  потрібно:

- для вбудовування можливості debug в образ встановити ці бібліотеки **pip install ptvsd debugpy**
- в контейнері  відкрити порт для remote debug 5678
- змінитин команду запуска так, щоб замість gunicorn запускався **python flask run**  з додатковими ключами
- налаштувати правильно файл  ".vscode/launch.json" для підключення до контейнера по порту 5678, або мною визначний
- Підготувати правильно Dockerfile та deployment-config.yaml

Для вбудовування програмного коду вирішив використовувати базовий [ubi8:python3.9](https://catalog.redhat.com/software/containers/ubi8/python-39/6065b24eb92fbda3a4c65d8f). Тут я процитую з опису до контейнера призначення основних env-змінних, 


> **Environment variables**
>
> To set these environment variables, you can place them as a key value pair into a .s2i/environment file inside your source code repository.
>
>    APP_SCRIPT
>
>    Used to run the application from a script file. This should be a path to a script file (defaults to app.sh unless set to null) that will be run to start the application.
>
>    APP_FILE
>
>    Used to run the application from a Python script. This should be a path to a Python file (defaults to app.py unless set to null) that will be passed to the Python interpreter to start the application.
>
>    APP_MODULE
>
>    Used to run the application with Gunicorn, as documented here. This variable specifies a WSGI callable with the pattern MODULE_NAME:VARIABLE_NAME, where MODULE_NAME is the full dotted path of a module, and VARIABLE_NAME refers to a WSGI callable inside the specified module. Gunicorn will look for a WSGI callable named application if not specified.
>
>    If APP_MODULE is not provided, the run script will look for a wsgi.py file in your project and use it if it exists.
>
>    If using setup.py for installing the application, the MODULE_NAME part can be read from there. For an example, see setup-test-app.
>
>    APP_HOME
>
>    This variable can be used to specify a sub-directory in which the application to be run is contained. The directory pointed to by this variable needs to contain wsgi.py (for Gunicorn) or manage.py (for Django).
>
>    If APP_HOME is not provided, the assemble and run scripts will use the application's root directory.
>
>    APP_CONFIG

>    Path to a valid Python file with a Gunicorn configuration file.
>
>
> **Gunicorn**
>
> The Gunicorn WSGI HTTP server is used to serve your application in the case that it is installed. It can be installed by listing it either in the requirements.txt file or in the install_requires section of the setup.py file.
>
> If a file named wsgi.py is present in your repository, it will be used as the entry point to your application. This can be overridden with the environment variable APP_MODULE. This file is present in Django projects by default.
>
> If you have both Django and Gunicorn in your requirements, your Django project will automatically be served using Gunicorn.
>
> **Application script file**
>
> This is the most general way of executing your application. It will be used in the case where you specify a path to an executable script file via the APP_SCRIPT environment variable, defaulting to a file named app.sh if it exists. The script is executed directly to launch your application.


##  <a name="p-4">Опис демо для включення remote debug</a>

Демонстрація, як включити remote debug в openhsift  для Python Flask application наведена в github  за лінком [py-flask-rdbg Remote debug flask application on openshift](https://github.com/pavlo-shcherbukha/py-flask-rdbg.git). 

В readme.md   описано як розгорнути та запустити локально, або в openshift.

В файлі **requirements.txt**  додані для встановлення бібліотеки debug, що згадувалися в попереніх розділах: 
- ptvsd 
- debugpy

Для локального debug в **.vscode/launch.json** налаштовано такий розділ  json:

```json
        {
            "name": "sh_app: Win Flask",
            "type": "python",
            "request": "launch",
            "module": "flask",
            "env": {
                "FLASK_APP": "sh_app.webapp",
                "FLASK_ENV": "development",
                "FLASK_DEBUG": "0",
                "DB_HOST": "pfu-couchdb",
                "DB_PORT": "80",
                "DB_NAME": "shdb"
            },
            "args": [
                "run",
                "--no-debugger",
                "--no-reload",
                "--port", "5010",

            ],
            "jinja": true
        },        

```
Як видно з прикладу, в ключі **env**  задаться різні (прикладні та службові) env змінні. А в ключі **args** задаються параметри запуску  додатку під flask. Ну, як мінімум можна поміняти порт, на відмінний від 5000, що призначається за замовчуванням. Також потрібно звернути увагу на змінну **"FLASK_APP"** в якій задається модуль, що потріно запустити.

Для віддаленого debug в **.vscode/launch.json** налаштовано інший розділ  json:

```json
        {
            "name": "sh_app : Remote Attach",
            "type": "python",
            "request": "attach",
            "port": 5680,
            "host": "0.0.0.0",
            "pathMappings": [
                {
                "localRoot": "${workspaceFolder}",
                "remoteRoot": "/opt/app-root/src"
                }
            ]
        },        

```

Тут потрібно звернути увагу на ключ **port**  що показує який порт на вашому laptop буде задіяно для debug. Якщо у вас 2 додакти, то ці порти повинні розрізнятися. Ключ **pathMappings** будую співставлення для відображення програмного коду на вашому laptop (**localRoot**) та програмного коду на файловій  системі контейнера (**remoteRoot**).

### Підготовка Dockerfile та DeploymentConfig

Для перемикання між робочим режимом та debug використана така властивість образу ubi8:

>**APP_SCRIPT**
>
>Used to run the application from a script file. This should be a path to a script file (defaults to app.sh unless set to null) that will be run to start the application.

Тобто, посилання на sh-скрипт в цій змінній аналізується першим і спрацює раніше інших. Тому якраз сюди і підставимо sh скрипт, що запустить наш одаток в режимі DEBUG. Для цього в Dockerfile додані рядки, що створюють sh скрипт в /opt/app-root/etc/xapp.sh  та дають права на запуск не root користувачу та відкриваємо debug порт 5678:

```bash
RUN echo "PYTHONUNBUFFERED=1" >> /opt/app-root/etc/xapp.sh
RUN echo "PYTHONDONTWRITEBYTECODE=1" >> /opt/app-root/etc/xapp.sh
RUN echo "python -m debugpy --listen 0.0.0.0:5678 --wait-for-client -m flask run -h 0.0.0.0 -p 8080" >> /opt/app-root/etc/xapp.sh
RUN chmod 777 /opt/app-root/etc/xapp.sh

EXPOSE 5678
```

Ну а далі в DeploymentConfig (**py-ubi8docker-srvc-templ.yaml**) потрібно додати env змінну **APP_SCRIPT** в якій прописати повний шлях до цього файлу: "/opt/app-root/etc/xapp.sh".  Якщо ж цю змінну не задати, то додаток запутиться в  робочому режимі за допомогою gunicorn в конфігурації, що описана в env-змінній: **GUNICORN_CMD_ARGS** 

```yaml
        spec:
          containers:
          - env:
            - name: APP_SCRIPT
              value: /opt/app-root/etc/xapp.sh
            - name: APP_DEBUG
              value: 'DEBUG_BRK'  
            - name: GUNICORN_CMD_ARGS
              value: -k gevent --workers=1 --worker-connections=2000  --bind=0.0.0.0:8080 --access-logfile=-
```

В DeploymentConfig (**py-ubi8docker-srvc-templ.yaml**) навмисно додана змінна, щоб програмно  задати точку зупинити в скрипті, там, де мені потрібно. Ось як це виглядає у **views.py**

```py
if os.environ.get("APP_DEBUG") == 'DEBUG_BRK':
    import debugpy
    print("===========1-DEBUG-BREAK======")
    breakpoint() 
    print("===========2-DEBUG-BREAK======")

application = Flask(__name__)

```

Тобто зупинка відбудеться зразу після імпорту бібліотечних пакетів. Це не обов'язково, але мені так зручніше. Зразу відкривається потрібний source code.
Далі breakpoints  можна ставити просто з допомогою Visual Studio Code.

### Підклченя POD  до Laptop

Для того, щоб підключити Debug Port вашого POD  до Laptop,  потрібно виконати команду  OpenShift **port-forward**:

oc port-forward  podname  laptopPort:podPort

В даному випадку це виглядає так:

```bash
PS > oc port-forward  py-ubi8docker-srvc-1-2tbbs 5680:5678
Forwarding from 127.0.0.1:5680 -> 5678
Forwarding from [::1]:5680 -> 5678
Handling connection for 5680

```

Далі запускаємо конфыгурацію debug з назвою: **sh_app : Remote Attach** і дебажимо.

## <a name="p-5">Супутні цікаві технології, що використані в цій demo</a>

- При розробці deplyment використовувалися технології S2i, тобто deployment прямо з програмного коду в github. Досить швидко і зручно. Можливо не дуже підходить на production.

- Використовувася deployment з Dockerfile, що знаходиться в github. При цьому була одна особливість, мені порібно було перенсти програмний код на образ не командою **COPY**  з мого laptop, а використовуючи git clone в контейнері. Враховуючи, що використовувався приватний репозиторій, потрібно було придумати, як  передати і підставити в URL параметри доступу до github. Це вирішено за допомогою  **git config --global url..insteadOf**:

```bash
RUN git config --global url."https://${GIT_USER}:${GIT_PSW}@github.com/".insteadOf "https://github.com/"

RUN git clone ${GIT_URL} /tmp/src  -b ${GIT_BRANCH}

```

На приклад ось за лінком: [Openshift/ubi8_docker_deployment/Dockerfile](https://github.com/pavlo-shcherbukha/py-flask-rdbg/blob/main/openshift/ubi8_docker_deployment/Dockerfile)


- Deployment ведестья за допомогою шаблонів OpenShift


Лінки на документацію з приводу шаблонів:
- [Openshift 4.5 Using templates](https://docs.openshift.com/container-platform/4.5/openshift_images/using-templates.html)
- [OpenShift Templates Development Guide](http://v1.uncontained.io/playbooks/fundamentals/template_development_guide.html)

Використовувати шаблони виявилося досить зурчно, тому, що можна винести якусь частину  налаштувань в параметризацію. На приклад як в цьому файлі [py-ubi8docker-srvc-templ.yaml](https://github.com/pavlo-shcherbukha/py-flask-rdbg/blob/main/openshift/ubi8_docker_deployment/py-ubi8docker-srvc-templ.yaml), і deployment легко перелаштовується для розгортання дублюючого сервісу в тому ж namespace або перенесення взагалі на інший кластер 

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
Або параметризація роутера: [py-ubi8docker-route-temp.yaml](https://github.com/pavlo-shcherbukha/py-flask-rdbg/blob/main/openshift/ubi8_docker_deployment/py-ubi8docker-route-temp.yaml).

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

При цьому deployment складається з 3 кроків [srvc-process.cmd](https://github.com/pavlo-shcherbukha/py-flask-rdbg/blob/main/openshift/ubi8_docker_deployment/srvc-process.cmd):

