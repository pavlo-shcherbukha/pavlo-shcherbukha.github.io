---
layout: post
title: "Python - flask інтеграція з Redis, gevent"
date: 2022-09-02 10:00:01
categories: [python, flask]
permalink: posts/2022-09-02/python-flask-3/
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

Це продовження записок початківця про python flask. В попереніх серіях було:

- [Python - flask start](https://pavlo-shcherbukha.github.io/posts/2022-09-02/python-flask-1/)
- [Python - flask запуск в контейнері від RadHat UBI8](https://pavlo-shcherbukha.github.io/posts/2022-09-02/python-flask-2/)
- [Python, як підключити .dll, .so бібліотеки](https://pavlo-shcherbukha.github.io/posts/2022-06-11/python-dll-lib-find/)

В цьому блозі:

В [Python - flask запуск в контейнері від RadHat UBI8](https://pavlo-shcherbukha.github.io/posts/2022-09-02/python-flask-2/) було показано, що в Linux контейнері під Gunicorn додаток запускається в кілька потоків. Тепер виникла інша проблема, як управляти всіма цими потоками. Ну, уявимо собі, що на сервіс приходить якась команда і всі потоки повинні виконати її. Що це може бути за команда:

- ну завантажити файл з бази даних;
- змінити налаштування (параметризацію сервісу);
- виконати якийсь фоновий процес.

Для організації всього цього було прийняте рішення використати Publish/Subscribe схему на базі in-memory DB Redis. Якщо можна так сказати, то для Cloud Native applicatoin Redis виконує функцію пам'яті для public та shared змінних. Але, опублікувати в Redis -  то не не проблема. Пролема буде прочитати. Якщо просто запустити на Flask такий от цикл:

```py

    while True:
        time.sleep(4.0)
        lpid = os.getpid()
        log( str(lpid) + " : timeout ")
        for message in sub.listen():
            if message['type'] != 'message':
                continue
            log( "Pid: "+ str(lpid) + " - Get message: " +  json.dumps( message['data'] ) ) 


```

То, цикл працювати буде. А от web service  працювати не будуть, на відміну від Node.js, де event loop уже вбудована і цієї проблеми не має. А мені потрібно, щоб працювали web service і щоб  всі підписники слухали чергу і виконували,  команди, отримані з черги.

Тому  цей блог присвячений тому, як Python Flask applicatoin інтегрувати з Redis по принципу public/subscribe.









## <a name="p-2">Розгортання Flask  applicatin  для  продуктивної експлуатації</a>





## <a name="p-3">Побудова образу локально</a>



## <a name="p-5">Висновки</a>

Це елементарні приклади для початківця щоб  запустити python appliction в docker контейнері та навчитися з ним працювати. Ну а  в репозиторії програмного коду показані типові приклади побудови api/
Сам репозиторій з прикладом знаходиться за лінком: [py-flask-app-smpl-restapi-for-ubi8](https://github.com/pavlo-shcherbukha/py-flask-app-smpl-restapi-for-ubi8/tree/main)

