---
layout: post
title: "Python Flask logging"
date: 2024-09-15 10:00:01
categories: [Python, Flask]
permalink: posts/2024-09-15/py_flask_logging/
published: true
---

<!-- TOC BEGIN -->
- [1. Постановка проблеми](#p1)
- [2. Корисна документація](#p2)
- [3. Опис вимог до логування](#p3)
- [4. Особливості реалізації](#p4)
- [5. Приклад реалізації](#p5)
<!-- TOC END -->

## <a name="p1">1. Постановка проблеми</a>

При розробці Python Flask додатків сиандартний логер  працює в текстовому режимі. А це не завжди зручно, тому
виникла необхідність логувати їх роботу специфічним чином в JSON, щоб  унаслідувати вже існуючий формат. 
На Pypi репозиторії можна знайти багато логерів, але кожний з них вирішує своє, специфічне завдання, але однозначно не те, що потрібно мені.
Прийшлося трошки почитати документацію та розробити свій логер


## <a name="p2">2. Корисна документація</a>

- [Python JSON Logging](https://aminalaee.dev/posts/2022/python-json-logging/)
По суті це базовий ресурс, з якого взято більшість підходів.

- [logging — Logging facility for Python](https://docs.python.org/3/library/logging.html#logrecord-attributes)

Лінк на базову документацію по пакету [logging](https://pypi.org/project/logging/).
А ось і документація україгською [logging — Можливість журналювання для Python](https://docs.python.org/uk/3.9/library/logging.html).

- [Structured log files in Python using python-json-logger](http://web.archive.org/web/20201130054012/https://wtanaka.com/node/8201)
Тут трошки про структуру логування в  Python.
- [Injecting Request Information](https://flask.palletsprojects.com/en/2.3.x/logging/#injecting-request-information)
про те, як достукатися до інформації з http запиту flask

корисне, щось можна взяти з пакету json-logging

## <a name="p3">3. Опис вимог до логування</a>

Основними вимогами до  мого логування є такі:

- Запис логу повинно мати json-структуру з такими реквізитами:

  - timestamp - мітка часу
  - level - рівень логування (DEBUG, INFO, WARNING, ERROR, CRITICAL)
  - label - помітка участка кода, де виконується програмний код
  - ahostname - localhost або найменування host
  - message - log-повідомлення
  - ausername - - login користовича

Приклад json:

```json
[
 {"timestamp": "2024-10-31T16:24:38.1730384678Z", "level": "DEBUG", "label": "sh_app.views", "ahostname": "testhost", "message": "debug message", "ausername": null}
,{"timestamp": "2024-10-31T16:24:38.1730384678Z", "level": "INFO", "label": "sh_app.views", "ahostname": "testhost", "message": "INFO ...... ......MESSAGE", "ausername": null}
,{"timestamp": "2024-10-31T16:24:38.1730384678Z", "level": "DEBUG", "label": "sh_app.views", "ahostname": "testhost", "message": "START TEST ERROR", "ausername": null}
,{"timestamp": "2024-10-31T16:24:38.1730384678Z", "level": "ERROR", "label": "sh_app.views", "ahostname": "testhost", "message": "division by zero", "ausername": null, "stack_info": "/home/psh/psh_dev/github-io/github-io/tz-000026-flask-json-logging/FlaskLoggingJson/tz-000001-init/FlaskLoggingJson/sh_app/views.py", "filename": "views.py", "lineno": 103, "module": "views"}

]
```

Так як додаток являє собою webservice  то логер повинен логувати ще і параметри http запитів, а саме:
- method
- path
- request content type
- request size
- request host + ip
- request params
- request timestamp
- час обробки  запиту (duration)
- response content type 
- response size
Рівень логування: **INFO**

Приклад логування http показано нижче:

```JSON
{
  "timestamp": "2024-11-04T08:34:39.1730709279Z",
  "level": "INFO",
  "label": "http_api",
  "ahostname": "testhost",
  "message": "HTTP API REQ",
  "ausername": null,
  "http_api": {
    "method": "POST",
    "path": "/api/unpack",
    "request_content_type": "multipart/form-data; boundary=--------------------------925103508313841402258282",
    "request_size": 128386,
    "status": 200,
    "duration": 0.59,
    "time": 1730709279.269854,
    "ip": "10.3.10.10",
    "host": "testhost2.ua",
    "params": {},
    "response_content_type": "application/json",
    "response_size": 104071
  }
}

```

## <a name="p4">4. Особливості реалізації</a>

Перш за все потрібно відключити логування flask  за замовчуванням.
Для цього зразу після створення Flask applicaion видаляємо  log handler  за замовчуванням

```py
from flask.logging import default_handler

application = Flask(__name__)
# видаляємо  log handler  за замовчуванням
application.logger.removeHandler(default_handler)
logging.getLogger('werkzeug').disabled = True


```
Детальніше про логування в flask  можна почитати за лінком [Logging Removing the Default Handler](https://flask.palletsprojects.com/en/stable/logging/)

Другим  кроком - є використання **Flask** **decorators**: "@application.before_request", "@application.after_request" для логування  http- запитів. 

https://flask.palletsprojects.com/en/stable/api/
     

> before_request_funcs: dict[ft.AppOrBlueprintKey, list[ft.BeforeRequestCallable]]
>
>    A data structure of functions to call at the beginning of each request, in the format {scope: [functions]}. The scope key is the name of a blueprint the functions are active for, or None for all requests.
>
>    To register a function, use the before_request() decorator.
>
>    This data structure is internal. It should not be modified directly and its format may change at any time.
>
> after_request_funcs: dict[ft.AppOrBlueprintKey, list[ft.AfterRequestCallable[t.Any]]]
>
>    A data structure of functions to call at the end of each request, in the format {scope: [functions]}. The scope key is the name of a blueprint the functions are active for, or None for all requests.
>
>    To register a function, use the after_request() decorator.
>
>    This data structure is internal. It should not be modified directly and its format may change at any time.
>
> Werkzeug
> Werkzeug logs basic request/response information to the 'werkzeug' logger. 
> If the root logger has no handlers configured, Werkzeug adds a StreamHandler to its logger.



> Application Globals
> To share data that is valid for one request only from one function to another, a global variable is not good enough because it would break in threaded environments. Flask provides you with a special object that ensures it is only valid for the active request and that will return different values for each request. In a nutshell: it does the right thing, like it does for request and session.
> flask.g
>     A namespace object that can store data during an application context. This is an instance of Flask.app_ctx_globals_class, which defaults to ctx._AppCtxGlobals.



Ось тут, нижче, в фрагменті коду показані функції, що стоять за декораторами:

```py
from flask import  g


@application.before_request
def start_timer():
    g.start = time.time()

@application.after_request
def log_request(response):
    if request.path == '/favicon.ico':
        return response
    #elif request.path.startswith('/static'):
    #    return response

    now = time.time()
    duration = round(now - g.start, 2)
    
    ip = request.headers.get('X-Forwarded-For', request.remote_addr)
    host = request.host.split(':', 1)[0]
    args = dict(request.args)

    log_params = {
        'method': request.method, 
        'path': request.path,
        'request_content_type': request.headers.get('Content-Type'),
        'request_size': int(request.headers.get('Content-Length') or 0),
        'status': response.status_code, 
        'duration': duration, 
        'time': now,
        'ip': ip,
        'host': host,
        'params': args,
        'response_content_type': response.content_type,
        'response_size': response.calculate_content_length()

    }

    logger.info("HTTP API REQ", extra=log_params)

    return respons

```

І потрібно звернути увагу змінну **log_params**,  що при логуванні передається як ключ **extra** -  тобто як додаткові реквізити

```py
logger.info("HTTP API REQ", extra=log_params)
```

 З приводу декораторів  можна почита ще цікаві речі за лінком [Flask: Before and After request Decorators](https://medium.com/innovation-incubator/flask-before-and-after-request-decorators-e639b06c2128)

Третім кроком є використання стандартного логера python: [logging — Logging facility for Python](https://docs.python.org/3/library/logging.html)

Тобто, підключаємо стандартний логер та створюємо свій  log handler:

```py

ogger = logging.getLogger(__name__)
# тут читаємо env-змінну, що дзволяє встановити власну глибину логування
apploglevel=os.environ.get("LOGLEVEL")
if apploglevel==None:
    logger.setLevel(logging.DEBUG)
elif apploglevel=='DEBUG':
    logger.setLevel(logging.DEBUG)    
elif apploglevel=='INFO':
    logger.setLevel(logging.INFO)    
elif apploglevel=='WARNING':
    logger.setLevel(logging.WARNING)    
elif apploglevel=='ERROR':    
    logger.setLevel(logging.ERROR)    
elif apploglevel=='CRITICAL':
    logger.setLevel(logging.CRITICAL)    
else:
    logger.setLevel(logging.DEBUG)  

handler = logging.StreamHandler()
# в цьому рядку підключаємо свій форматер log-записів у вигляді json
handler.setFormatter( hello_app.shjsonformatter.JSONFormatter())
logger.addHandler(handler)

```


А сам форматер створений у вигляді кастомного класу **shjsonformatter.py**:

```py
import json
import logging
import time
import os
class JSONFormatter(logging.Formatter):
    def __init__(self) -> None:
        super().__init__()
        self.def_keys = ['name', 'msg', 'args', 'levelname', 'levelno',
            'pathname', 'filename', 'module', 'exc_info',
            'exc_text', 'stack_info', 'lineno', 'funcName',
            'created', 'msecs', 'relativeCreated', 'thread',
            'threadName', 'processName', 'process', 'message']

    def format(self, record: logging.LogRecord) -> str:
        #message = record.__dict__.copy()
        message={}
        message['timestamp']=self.fmttime( record.created )
        message['level'] = record.levelname
        message['label'] = record.name
        message['ahostname'] = os.environ.get("HOSTNAME")  
        message["message"] = record.getMessage() 
        message['ausername'] = None

        if record.levelname == "ERROR":
            if record.stack_info:
                message["stack_info"] = self.formatStack(record.stack_info)
            if record.pathname:
                message["stack_info"] = record.pathname
            if record.filename:
                message["filename"] = record.filename   
            if record.lineno:
                message["lineno"] = record.lineno
            if record.exc_text:
                message["exc_text"] = record.exc_text
            if record.exc_info:
                message["exc_info"] = record.exc_info
            if record.module:
                message["module"] = record.module


        extra = {k: v for k,v in record.__dict__.items()
             if k not in self.def_keys}

        if len(extra)>0:
            message['label'] = 'http_api'
            message['http_api'] = extra

        retrecord = json.dumps(message, ensure_ascii=False)
        return retrecord
    
    def fmttime( self,  logtime ):
        tmformat="%Y-%m-%dT%H:%M:%S.%sZ"
        tmformatv1="%Y-%m-%dT%H:%M:%S +0000"
         
        gmttimestamp=time.gmtime( logtime )
        strgmtime=time.strftime( tmformat, gmttimestamp)
        return strgmtime



```

І четвертий крок - це в кожній функції створюємо дочерній логер для подальшого логування.
Таким чином, в ключі **Label** завжди буде записуватися найменування модуля та функції, що є власником логуючого запису.


```py

@application.route("/about/")
def about():
    logger=logging.getLogger(__name__).getChild("about")
    logger.debug("render about")
    return render_template("about.html")

```

А в класі з якогось модуля, можемо  написати назву класа та назву функції:

```py
logger=logging.getLogger(self.plogname).getChild( f"{__name__}.Eds:SignData")

```


І додатково, потрібно розуміти, що при запуску [flask app](https://flask.palletsprojects.com/en/stable/deploying/gunicorn/)  під управліням [gunicorn](https://docs.gunicorn.org/en/latest/run.html#commonly-used-arguments), на приклад:

```bash
-k gevent --workers=1 --worker-connections=2000  --bind=0.0.0.0:8080 --timeout 600 --threads=50 --access-logfile=-
```
кулюч **--access-logfile=-** треба видалити, якщо такий ключ додавали.  
Якщо не видалити, то серед записів в json будуть попадатися текстові рядки від gunicorn access log.

## <a name="p5">5. Приклад реалізації</a>


Приклад реалізації логування наведено в репозиторії [FlaskLoggingJson](https://github.com/pavlo-shcherbukha/FlaskLoggingJson).
Сам форматер  знаходиться в [sh_app/shjsonformatter.py](https://github.com/pavlo-shcherbukha/FlaskLoggingJson/blob/main/sh_app/shjsonformatter.py)
А імплементація flask-додатку в [/sh_app/views.py](https://github.com/pavlo-shcherbukha/FlaskLoggingJson/blob/main/sh_app/views.py)

Якщо в .vscode/launch.json: 

```json
{

    "version": "0.2.0",
    "configurations": [
       
        {
            "name": "sh_app: Win Flask",
            "type": "python",
            "request": "launch",
            "module": "flask",
            "env": {
                "FLASK_APP": "sh_app.webapp",
                "FLASK_ENV": "development",
                "FLASK_DEBUG": "0",
                "LOGLEVEL":"DEBUG"
             },
            "args": [
                "run",
                "--no-debugger",
                "--no-reload",
                "--port", "8080",

            ],
            "jinja": true
        }        
    ]
}
```

- поставити змінну  **"LOGLEVEL":"INFO"**, то будуть логуватися тільки  http  записи.

```json
[

{"timestamp": "2024-11-04T15:26:15.1730726775Z", "level": "INFO", "label": "http_api", "ahostname": null, "message": "HTTP API REQ", "ausername": null, "http_api": {"method": "GET", "path": "/api/srvci", "request_content_type": null, "request_size": 0, "status": 422, "duration": 0.0, "time": 1730733975.9626393, "ip": "127.0.0.1", "host": "localhost", "params": {}, "response_content_type": "application/json", "response_size": 189}}
,{"timestamp": "2024-11-04T15:26:55.1730726815Z", "level": "INFO", "label": "http_api", "ahostname": null, "message": "HTTP API REQ", "ausername": null, "http_api": {"method": "GET", "path": "/api/health", "request_content_type": null, "request_size": 0, "status": 200, "duration": 0.0, "time": 1730734015.6419735, "ip": "127.0.0.1", "host": "localhost", "params": {}, "response_content_type": "application/json", "response_size": 44}}
,{"timestamp": "2024-11-04T15:27:26.1730726846Z", "level": "INFO", "label": "http_api", "ahostname": null, "message": "HTTP API REQ", "ausername": null, "http_api": {"method": "GET", "path": "/", "request_content_type": null, "request_size": 0, "status": 200, "duration": 0.03, "time": 1730734046.4681294, "ip": "127.0.0.1", "host": "localhost", "params": {}, "response_content_type": "text/html; charset=utf-8", "response_size": 591}}
,{"timestamp": "2024-11-04T15:27:26.1730726846Z", "level": "INFO", "label": "http_api", "ahostname": null, "message": "HTTP API REQ", "ausername": null, "http_api": {"method": "GET", "path": "/static/site.css", "request_content_type": null, "request_size": 0, "status": 304, "duration": 0.02, "time": 1730734046.724471, "ip": "127.0.0.1", "host": "localhost", "params": {}, "response_content_type": "text/css; charset=utf-8", "response_size": null}}
,{"timestamp": "2024-11-04T15:27:41.1730726861Z", "level": "INFO", "label": "http_api", "ahostname": null, "message": "HTTP API REQ", "ausername": null, "http_api": {"method": "GET", "path": "/about/", "request_content_type": null, "request_size": 0, "status": 200, "duration": 0.01, "time": 1730734061.2955894, "ip": "127.0.0.1", "host": "localhost", "params": {}, "response_content_type": "text/html; charset=utf-8", "response_size": 579}}
,{"timestamp": "2024-11-04T15:27:41.1730726861Z", "level": "INFO", "label": "http_api", "ahostname": null, "message": "HTTP API REQ", "ausername": null, "http_api": {"method": "GET", "path": "/static/site.css", "request_content_type": null, "request_size": 0, "status": 304, "duration": 0.0, "time": 1730734061.412634, "ip": "127.0.0.1", "host": "localhost", "params": {}, "response_content_type": "text/css; charset=utf-8", "response_size": null}}

]

```
- поставити змінну  **"LOGLEVEL":"DEBUG"**, то будуть логуватися   http  запити та специфічні логувальні записи.

```json

[
    {"timestamp": "2024-11-04T15:28:27.1730726907Z", "level": "DEBUG", "label": "sh_app.views.home", "ahostname": null, "message": "Home page logger", "ausername": null}
    ,{"timestamp": "2024-11-04T15:28:27.1730726907Z", "level": "INFO", "label": "http_api", "ahostname": null, "message": "HTTP API REQ", "ausername": null, "http_api": {"method": "GET", "path": "/", "request_content_type": null, "request_size": 0, "status": 200, "duration": 0.03, "time": 1730734107.6819003, "ip": "127.0.0.1", "host": "localhost", "params": {}, "response_content_type": "text/html; charset=utf-8", "response_size": 591}}
    ,{"timestamp": "2024-11-04T15:28:28.1730726908Z", "level": "INFO", "label": "http_api", "ahostname": null, "message": "HTTP API REQ", "ausername": null, "http_api": {"method": "GET", "path": "/static/site.css", "request_content_type": null, "request_size": 0, "status": 304, "duration": 0.02, "time": 1730734108.0400624, "ip": "127.0.0.1", "host": "localhost", "params": {}, "response_content_type": "text/css; charset=utf-8", "response_size": null}}
    ,{"timestamp": "2024-11-04T15:28:29.1730726909Z", "level": "DEBUG", "label": "sh_app.views.about", "ahostname": null, "message": "About page logger", "ausername": null}
    ,{"timestamp": "2024-11-04T15:28:29.1730726909Z", "level": "INFO", "label": "http_api", "ahostname": null, "message": "HTTP API REQ", "ausername": null, "http_api": {"method": "GET", "path": "/about/", "request_content_type": null, "request_size": 0, "status": 200, "duration": 0.01, "time": 1730734109.064762, "ip": "127.0.0.1", "host": "localhost", "params": {}, "response_content_type": "text/html; charset=utf-8", "response_size": 579}}
    ,{"timestamp": "2024-11-04T15:28:29.1730726909Z", "level": "INFO", "label": "http_api", "ahostname": null, "message": "HTTP API REQ", "ausername": null, "http_api": {"method": "GET", "path": "/static/site.css", "request_content_type": null, "request_size": 0, "status": 304, "duration": 0.0, "time": 1730734109.2231581, "ip": "127.0.0.1", "host": "localhost", "params": {}, "response_content_type": "text/css; charset=utf-8", "response_size": null}}
    ,{"timestamp": "2024-11-04T15:28:42.1730726922Z", "level": "DEBUG", "label": "sh_app.views.health", "ahostname": null, "message": "Health check", "ausername": null}
    ,{"timestamp": "2024-11-04T15:28:42.1730726922Z", "level": "DEBUG", "label": "sh_app.views.health", "ahostname": null, "message": "Sending response {'message': 'Remote debug demo', 'ok': True}", "ausername": null}
    ,{"timestamp": "2024-11-04T15:28:42.1730726922Z", "level": "INFO", "label": "http_api", "ahostname": null, "message": "HTTP API REQ", "ausername": null, "http_api": {"method": "GET", "path": "/api/health", "request_content_type": null, "request_size": 0, "status": 200, "duration": 0.01, "time": 1730734122.2962513, "ip": "127.0.0.1", "host": "localhost", "params": {}, "response_content_type": "application/json", "response_size": 44}}
    ,{"timestamp": "2024-11-04T15:29:20.1730726960Z", "level": "INFO", "label": "http_api", "ahostname": null, "message": "HTTP API REQ", "ausername": null, "http_api": {"method": "GET", "path": "/api/srvi", "request_content_type": null, "request_size": 0, "status": 404, "duration": 0.0, "time": 1730734160.7438104, "ip": "127.0.0.1", "host": "localhost", "params": {}, "response_content_type": "text/html; charset=utf-8", "response_size": 207}}
    ,{"timestamp": "2024-11-04T15:29:26.1730726966Z", "level": "INFO", "label": "http_api", "ahostname": null, "message": "HTTP API REQ", "ausername": null, "http_api": {"method": "GET", "path": "/api/srvci", "request_content_type": null, "request_size": 0, "status": 422, "duration": 0.0, "time": 1730734166.9499178, "ip": "127.0.0.1", "host": "localhost", "params": {}, "response_content_type": "application/json", "response_size": 189}}

]
```