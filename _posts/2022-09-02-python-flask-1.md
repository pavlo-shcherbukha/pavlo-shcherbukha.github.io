---
layout: post
title: "Python - flask start"
date: 2022-09-02 10:00:01
categories: [python, flask]
permalink: posts/2022-09-02/python-flask-1/
published: true
---

## Зміст

<!-- TOC BEGIN -->
1. [Про що цей блог](#p-1)
2. [Вибір інстументів розробки та налаштування середовища розробки](#p-2)
2.1. [Вибір фреймворку](#p-2.1)
2.2. [Вибір середовища розробки](#p-2.2)
2.3. [Вибір середовища виконання](#p-2.3)
3. [Створення початкового мінімального проекту на python Flask на Wi64](#p-3)
3.1. [Створити проект в Visual Studio Code](#p-3.1)
3.2. [Створення віртуального середовища](#p-3.2)
3.3. [Устанвока необхідних біблиотек](#p-3.3)
4. [Створення найпростішого скрипта для flask app та запуск applicaiton  з назвою за замлвчуванням в DEVELOPENT mode](#p-4)
5. [Створення найпростішого скрипта для flask app та запуск applicaiton  з індивідуальною назвою в DEVELOPENT mode](#p-5)
6. [Створення найпростішого скрипта для flask app та запуск applicaiton  з індивідуальн ою назвою в режимі DEBUG
](#p-6)
7. [Взаємодія з Git repo](#p-7)
8. [Висновки](#p-8)

<a name="p-1"></a>



3.4. [](#)
<a name="p6.1">6.1. Шлюз для взаємодії з ПФУ</a>
<!-- TOC END -->



## <a name="p-1">Про що цей блог</a>

Постало передімною завдання, що вимагає розробки webservice з бібліотеками, що є тільки для python, але для win платформи та для Linux.  Тому прийшлося швидко вивчати Python ну і  щось  зразу писати. Ну а працювати все йе повинно під kubernetes  платформою Openshift. Тому цей блог орієнтований на початківців, хто, можливо опинився в схожій ситуації. Більш того, мені здається, що це буде серія блогів, тому що в один все це втулити складно. Тож поїхали


## <a name="p-2">Вибір інстументів розробки та налаштування середовища розробки</a>


### <a name="p-2.1">Вибір фреймворку</a>
Для розробки Web Services на Python  google каже, що  найширше використовуються фреймоворки Django  та Flask. Кому цікаво, їх порівняльні характеристики можна почитати за посиланням: [Flask Vs Django: Which Python Framework to Choose?](https://www.interviewbit.com/blog/flask-vs-django/). Я вибрав Flask  тому що мені потрібні маленькі сервіси, де кожен сервіс виконує  свою роботу, а не масивні application. Тому я зупинився на Flask. Тимбільше, що для Node.js developer шаблон додатку чомусь дуже мені нагадав Node.js express. Тому для розробки вибрав [Flask](https://flask.palletsprojects.com/en/2.2.x/#user-s-guide). І у мене існують обмеження зовнішнбої бібіліотеки: не вище Python 3.9.


### <a name="p-2.2">Вибір середовища розробки</a>
В якості середвоища розробки вибрав Visual Code Studio з плаіном Python. 

<kbd><img src="/assets/img/posts/2022-09-02-python-flask-1/doc/pic-01.png" /></kbd>
<p style="text-align: center;"><a name="pic-01">pic-01</a></p>   


За дінком нписано, як налаштувати VSC для роботи з Flask [Using Flask in Visual Studio Code](https://code.visualstudio.com/docs/python/tutorial-flask) і тут є корисний приклад: [python-sample-vscode-flask-tutorial](https://github.com/microsoft/python-sample-vscode-flask-tutorial).




### <a name="p-2.3">Вибір середовища виконання</p>

Тут розклад простий. Розробку та выдладку вести на win-64. Виконуватися повинно в контейнері Linux на RedHat OpenShift. В якості базового контейнера вибрано сертифікований RedHat  [UBI8 з адаптацією під Python 3.9,  ubi8/python-39 ](https://catalog.redhat.com/software/containers/ubi8/python-39/6065b24eb92fbda3a4c65d8f). 


## <a name="p-3">Створення початкового мінімального проекту на python Flask на Wi64</a>


Перед цим, звичайно, на машині повинно бути встановлено Python 3.9.  

### <a name="p-3.1">Створити проект в Visual Studio Code</a>

Створив каталог та перенести з [python-sample-vscode-flask-tutorial](https://github.com/microsoft/python-sample-vscode-flask-tutorial) папку **.vscode** в ній в файлі **launch.json**  записана конфsгурація  для debug різних фраймворків на Python та **launch.json settings.json** де записано, що старт додатку відбувається не з загального каталогу, де установдено Python  а з віртуального середовища, що індивідуальне  для  окремо взятого проекту. Далі потрібно створити віртуальне середовище запуску

### <a name="p-3.2">Створення віртуального середовища</a>

* Як створити віртуальне середовище, описано за лінком:  [Installing packages using pip and virtual environments¶](https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/)

- Створення віртуального середовища виконується командою

```text
py -m venv env

```
, що запускається в терміналі VSC.

<kbd><img src="/assets/img/posts/2022-09-02-python-flask-1/doc/pic-02.png" /></kbd>
<p style="text-align: center;"><a name="pic-02">pic-02</a></p>   


В результаті виконання команди в проекті створяться каталог env, з підкаталогами [pic-3](#pic-3)


<kbd><img src="/assets/img/posts/2022-09-02-python-flask-1/doc/pic-03.png" /></kbd>
<p style="text-align: center;"><a name="pic-03">pic-03</a></p>   


Для перевірки, що virtual environment запуститься можна виконати в терміналі VSC: 

```
.\env\Scripts\activate.ps1   
```

<kbd><img src="/assets/img/posts/2022-09-02-python-flask-1/doc/pic-04.png" /></kbd>
<p style="text-align: center;"><a name="pic-04">pic-04</a></p>   


Якщо виникає помилка при запуску скрипта Activate.ps1, то 
потрібно запустити WinShell с правами адміністратора і виконати команду

```text
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser 

```

Інші рецепти не допомгли.


На цьому віртуальне середовище створено. Переходимо до інсталювання необхіних бібліотек.


### <a name="p-3.3">Устанвока необхідних біблиотек</a>

Процес установки необхідних бібліотек дуже схожить з Node.js

Можна ставити бібліотеки поіменно, як в цьому прикладі, інсталюємо бібліотку requests:

```bash
py -m pip install requests

```

А можна створити файл залежностей **requirements.txt** в корені проекту, та встановити всі залужності за списком. Для мене цей варіант більш прийнятний, тому що при збірці в контейнері, якраз використовується пакетна установка.

```bash
py -m pip install -r requirements.txt
```

Тому ствроюю проситьй набір бібілиотек в **requirements.txt**

```text
Flask
requests

```

та запускаю 

```bash
py -m pip install -r requirements.txt
```


<kbd><img src="/assets/img/posts/2022-09-02-python-flask-1/doc/pic-05.png" /></kbd>
<p style="text-align: center;"><a name="pic-05">pic-05</a></p> 


А в каталозі **env\Scripts** з'явився файл flask.exe 



<kbd><img src="/assets/img/posts/2022-09-02-python-flask-1/doc/pic-06.png" /></kbd>
<p style="text-align: center;"><a name="pic-06">pic-06</a></p> 

Таким чином мінімаьне віртуальне середивище для запуска Flask application сконфігуроване.

##  <a name="p-4">Створення найпростішого скрипта для flask app та запуск applicaiton  з назвою за замлвчуванням в DEVELOPENT mode</a>

Створимо app.py в кореневому каталозі проекту

```py
from flask import Flask
app = Flask(__name__)


@app.route("/")
def home():
    return "Hello, Flask This IS APP.PY!"

```

Та запустимо його командою: 

```bash
python -m flask run
```

Запуститься додакток за замовчуванням: app.py

<kbd><img src="/assets/img/posts/2022-09-02-python-flask-1/doc/pic-07.png" /></kbd>
<p style="text-align: center;"><a name="pic-07">pic-07</a></p> 

Ну і зразу  сервіс доступний по http://localhost:5000


<kbd><img src="/assets/img/posts/2022-09-02-python-flask-1/doc/pic-08.png" /></kbd>
<p style="text-align: center;"><a name="pic-08">pic-08</a></p> 


## <a name="p-5">Створення найпростішого скрипта для flask app та запуск applicaiton  з індивідуальною назвою в DEVELOPENT mode</a>

Створимо mainprog.py  в кореневому каталозі проекту

```py
from flask import Flask
app = Flask(__name__)


@app.route("/")
def home():
    return "Hello, Flask! This is mainprog.py !"

```

Запустити програму з назвою, що відрізняється від app.py  можна кільком методами.

1. Задати назву головного модуля в командному рядку з ключем --app

```bash
   python -m flask --app mainprog.py  run 
```

<kbd><img src="/assets/img/posts/2022-09-02-python-flask-1/doc/pic-09.png" /></kbd>
<p style="text-align: center;"><a name="pic-09">pic-09</a></p> 


2. Шляхом задання env змінної **FLASK_APP**


потрібно в  терміналі WSC встановити 

- Для PowerShell

```bash
 $env:FLASK_APP = "mainprog.py"

```

Переглянути зачення можна командою:

```bash

   $env:FLASK_APP
``` 

- Для CMD

```bash

SET FLASK_APP=mainprog.py

echo %FLASK_APP%

```

ну і запустити flask

```bash
python -m flask run
```

- для PowerShell
<kbd><img src="/assets/img/posts/2022-09-02-python-flask-1/doc/pic-10.png" /></kbd>
<p style="text-align: center;"><a name="pic-10">pic-10</a></p> 


- для CMD
<kbd><img src="/assets/img/posts/2022-09-02-python-flask-1/doc/pic-11.png" /></kbd>
<p style="text-align: center;"><a name="pic-11">pic-11</a></p> 

## <a name="p-6">Створення найпростішого скрипта для flask app та запуск applicaiton  з індивідуальн ою назвою в режимі DEBUG</a>

Для запусу в режимі DEBUG потрібно внести конфігурацію в файл **.vscode/launch.json** в розділ Python: Flask

- В розділ **env** в змінну **FLASK_APP** потрібно внести назву стартового модуля **mainprog.py**
- Додати необхідні env-змінні, як для прикладу додана **DB_NAME**, що додаток повинен читати при старті програми

```text
        {
            "name": "Python: Flask",
            "type": "python",
            "request": "launch",
            "module": "flask",
            "env": {
                "FLASK_APP": "mainprog.py",
                "FLASK_ENV": "development",
                "FLASK_DEBUG": "0",
                "DB_NAME": "ORACLE-1234"
            },
            "args": [
                "run",
                "--no-debugger",
                "--no-reload"
            ],
            "jinja": true
        },


```

Далі переключаємося в режим DEBUG і запускаємо додаток. 

1. Для вибору режима DEBUG переходимо на піктограму

<kbd><img src="/assets/img/posts/2022-09-02-python-flask-1/doc/pic-12.png" /></kbd>
<p style="text-align: center;"><a name="pic-12">pic-12</a></p>


2. Для вибору типу запуску натискаємо на випадаючий список та вибираємо Flask


<kbd><img src="/assets/img/posts/2022-09-02-python-flask-1/doc/pic-13.png" /></kbd>
<p style="text-align: center;"><a name="pic-13">pic-13</a></p>


3. <a name="p-7">Запустити програму</a>


<kbd><img src="/assets/img/posts/2022-09-02-python-flask-1/doc/pic-14.png" /></kbd>
<p style="text-align: center;"><a name="pic-14">pic-14</a></p>


3.1. При **ПЕРШОМУ** запуску вас просить вибрати інтерпритатор. То потрібно вибирати інтерпритетор віртуального середовища:


<kbd><img src="/assets/img/posts/2022-09-02-python-flask-1/doc/pic-15.png" /></kbd>
<p style="text-align: center;"><a name="pic-15">pic-15</a></p>


Ну і в результаті видно, що програма стартонула і навіть зупинилася на breakpint.



<kbd><img src="/assets/img/posts/2022-09-02-python-flask-1/doc/pic-16.png" /></kbd>
<p style="text-align: center;"><a name="pic-16">pic-16</a></p>

Аде, зазвичай, в додаток потрібно передавати env-змінні, що параметризують його роботу. Під час конфігурації режиму DEBUG  була додана змінна ** "DB_NAME": "ORACLE-1234"**. Спробуємо її вичатити та повернути її значення у відповіді. Для цього трохи змінемо файл **mainprog.py**

```py
import os
from flask import Flask

app = Flask(__name__)


@app.route("/")
def home():

    dbname=os.environ.get('DB_NAME');
    
    if dbname==None:
        return "Hello, Flask! This is mainprog.py ! ENV DB_NAME IS NOT DEFINED"
    else:    
        return "Hello, Flask! This is mainprog.py ! ENV DB_NAME =" + dbname

```

І вуаля, значення змінної отримано!

<kbd><img src="/assets/img/posts/2022-09-02-python-flask-1/doc/pic-17.png" /></kbd>
<p style="text-align: center;"><a name="pic-17">pic-17</a></p>


## <a name="p-7">Взаємодія з Git repo</a>

Якщо працюємо з Git  реазторієм, то потрібно налаштувати файл .gitignore щоб не  завантажувати в Git біділіотеки, тимчасові файли і  таке інше. Взяв з прикладу  [python-sample-vscode-flask-tutorial](https://github.com/microsoft/python-sample-vscode-flask-tutorial). Мені він здався найбільш універсальним.



## <a name="p-8">Висновки</a>

Це елементарні приклади для початківця щоб розгорнути середовище розробки  та перевірити його працездатність.
Сам репозиторій з прикладом знаходиться за лінком: [py-flask-app-smpl](https://github.com/pavlo-shcherbukha/py-flask-app-smpl)

