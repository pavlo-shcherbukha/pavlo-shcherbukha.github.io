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
- [3. ](#p3)
<!-- TOC END -->

## <a name="p1">1. Постановка проблеми</a>

При розробці Python Flask додатків виникла необхідність логувати їх роботу специфічним чином, щоб  унаслідувати вже існуючий формат. 
На Pypi можна знайти багато логерів, але кожний з них вирішує свою, специфічну проблему, але однозначну не мою.
Прийшлося трошки почитати документацію 


## <a name="p2">2. Корисна документація</a>

- [Python JSON Logging](https://aminalaee.dev/posts/2022/python-json-logging/)
По суті це базовий ресурс, з якого взято більшість підходів.

- [logging — Logging facility for Python](https://docs.python.org/3/library/logging.html#logrecord-attributes)

Лінк на базову документацію по пакету [logging](https://pypi.org/project/logging/).
А ось і документація україгською [logging — Можливість журналювання для Python](https://docs.python.org/uk/3.9/library/logging.html).

- [Structured log files in Python using python-json-logger](http://web.archive.org/web/20201130054012/https://wtanaka.com/node/8201)
Тут трошки про структуру логування в  Python.
- [Injecting Request Information](https://flask.palletsprojects.com/en/2.3.x/logging/#injecting-request-information)
про те, як достукатися до інформації з запиту flask

корисне, що можна взяти з пакету json-logging

## <a name="p3">3. Опис логування</a>

Основними вимогами до логування є такі:

- Запис логу повинна мати json-структуру з такими реквізитами:

  - timestamp - мітка часу
  - level - рівень логування (DEBUG, INFO, WARNING, ERROR, CRITICAL)
  - label - помітка участка кода, де виконується програмний код
  - ahostname - localhost або найменування host
  - message - log-повідомлення
  - ausername - - login користовича

Приклад json:

```json
[
 {"timestamp": "2024-10-31T16:24:38.1730384678Z", "level": "DEBUG", "label": "sh_app.views", "ahostname": null, "message": "debug message", "ausername": null}
,{"timestamp": "2024-10-31T16:24:38.1730384678Z", "level": "INFO", "label": "sh_app.views", "ahostname": null, "message": "INFO ...... ......MESSAGE", "ausername": null}
,{"timestamp": "2024-10-31T16:24:38.1730384678Z", "level": "DEBUG", "label": "sh_app.views", "ahostname": null, "message": "START TEST ERROR", "ausername": null}
,{"timestamp": "2024-10-31T16:24:38.1730384678Z", "level": "ERROR", "label": "sh_app.views", "ahostname": null, "message": "division by zero", "ausername": null, "stack_info": "/home/psh/psh_dev/github-io/github-io/tz-000026-flask-json-logging/FlaskLoggingJson/tz-000001-init/FlaskLoggingJson/sh_app/views.py", "filename": "views.py", "lineno": 103, "module": "views"}
,{"timestamp": "2024-10-31T16:24:38.1730384678Z", "level": "INFO", "label": "http_api", "ahostname": null, "message": "INFO ...... ......MESSAGE", "ausername": null, "http_api": {"label": "lblblblblbl"}}
]
```

Так як додаток являє собою webservice  то логер повинен логувати ще і параметри http запитів









