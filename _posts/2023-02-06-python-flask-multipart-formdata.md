---
layout: post
title: "Python-flask how to create and process multipart/form-data requests"
date: 2023-02-06 10:00:01
categories: [python, flask]
permalink: posts/2023-02-06/2023-02-06-python-flask-multipart-formdata/
published: true
---


## <a name="p-1">Як в python Flask передати та обробити multipart/form-data дані</a>

В одному з проектів прийшлося перевикористати метод, який розбирає web форму, та передає дані у вигляді прикріпленого файлу. Мені потрібно було повторити це на python тільки програмно. Пошуки по інтернету не дали якогось системного розуміння, тому спробував розробити собі приклад, та розібратися з цим більш детально.

Приклад для вивчення наведено в репозиторії за лінком: [Як створити та оборобити http multipart/form-data запити в python flask ](https://github.com/pavlo-shcherbukha/py-flask-multipart-form-data). В наведених прикладах моделюються запити з html-форми та запити, що створені програмно з використанням двох бібіліотек:

- [urllib3](https://urllib3.readthedocs.io/en/stable/user-guide.html);

- рідного python модуля [request](https://pypi.org/project/requests/) та сервісної бібліотеки [requests-toolbelt](https://github.com/requests/toolbelt).



Сказати, що я там знайшов великі відмінності - не можу, тому що прицип формуфання запитів схожий. Але використання "рідного"  python  [request](https://pypi.org/project/requests/) модуля мені якось ближче.

В прикладах наведено:

- як фрмувати фейковий файл як attachent,  ну на приклад, Blob, вичитаний з бази даних та дані форми;
- як передати один файл та дані форми;
- як передати кілька файлів та дані форми.

На приймаючій стороні обробка даних виконується однаково, що з html форми, що з запиту, створеного програмно. Слід зробити наголос: щоб обробка була однаковою, слід розуміти що на стороні Flask  приймаючий  Flask.Request  використовує під капотом бібліотеку  werkzeug [werkzeug.ImmutableMultiDict how to parse ](https://tedboy.github.io/flask/generated/generated/werkzeug.ImmutableMultiDict.html), і вимагає розуміння, як розбирати дані, що збережені в **werkzeug.ImmutableMultiDict**, ну і як правильно підставити в запит дані форми на стороні формування запиту. Ну і основна  складність, коли потрібно передати та розібрати кілька файлів. 

Правда не розглянутим залишився streaming.  Ну, сподіваюсь, то трохи пізніше. 




