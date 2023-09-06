---
layout: post
title: "Node.js how to create and process multipart/form-data requests"
date: 2023-09-06 10:00:01
categories: [Node.js]
permalink: posts/2023-09-06/2023-09-06-nodejs-multipart-formdata/
published: true
---


## <a name="p-1">Як в Node.js передати та обробити multipart/form-data дані</a>



Не дуже часто приходиться працювати з multipart/form-data даними. Але коли потрібно, то практично кожного разу починаю шукати матеріал поновій. Тому я зібрав невеличку колекцію прикладів, щоб потім не шукати. Вони не закривають всі питання і всі задачі. Але для початку підійдуть. 

Приклад для вивчення наведено в репозиторії за лінком: [Як створити та оборобити http multipart/form-data запити в node.js ](https://github.com/pavlo-shcherbukha/nodejs-multipart-form-data-). 

В репозиторії 3 додатки:

- servermp - додаток, що обробляє запити multipart/form-data, використовуючи бібліотеку [multiparty](https://www.npmjs.com/package/multiparty). Тут є html сторінка, через яку можна завантажити файл та перевірити роботу бібіліотеки.

- servereu - додаток, що обробляє запити multipart/form-data, використовуючи бібліотеку [express-fileupload](https://www.npmjs.com/package/express-fileupload).  Тут також є html сторінка, через яку можна завантажити файл та перевірити роботу бібіліотеки.



- worker -  додаток, що формує програмно запити типу multipart/form-data, з допомогою бібліотеки axios, та відправляє їх на servermp або  servereu, в залежності від конфігурації в .env-файлі. Запити формуються в  двх варіантах:  з одним файлом та з двома. На приймаючій стороні обробка даних виконується однаково, що з html форми, що з запиту, створеного програмно (для завантаження одного файлу). Для  обробки кількох файлів UI  не розробляв, а програмно запит обробляється по іншому роутеру

Не розглядав Streaming.

Додатково до поису бібліотек викорситовував такі лінки:

- https://axios-http.com/docs/multipart 
- https://maximorlov.com/send-a-file-with-axios-in-nodejs/ 
- https://www.npmjs.com/package//axios?activeTab=readme

Деталі налаштування та запуску можна прочитати за посиланням:  https://github.com/pavlo-shcherbukha/nodejs-multipart-form-data-/blob/master/README.md





