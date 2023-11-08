---
layout: post
title: "Node-RED How to make unit tests"
date: 2023-09-08 10:00:01
categories: [node-RED]
permalink: posts/2023-11-04/node-red-utests/
published: true
---

<!-- TOC BEGIN -->

<!-- TOC END -->

## <a name="p-1">Про що цей блог</a>

Приводом для напиcання цього блогу стало те, що з  Node-RED працюють в більшості не  програмісти, а АСУПТ-шники, які легко розбираються в залізяках технологічного процесу, але їм трохи складніше розібратися з програмуванням , а швидше за все просто не вистачає часу, бо залізячча вимагає багато уваги бо воно не таке гнучке як програма і має кучу особливостей, які просто так не виправиш, як в програмі.  Тому, це спроба  більш простою мовою  донести як використовувати типові підходи, які вже звичні для програмістів.  Одним з таких підходів є розробка  unit  тестів та  їх використання в парадигмі "test driven development". 
На сайті nodered.org є пряма вказівка на ті ж самі інструментни [Написання unit тестів для нової node](https://nodered.org/docs/creating-nodes/first-node#testing-your-node-in-node-red). Але цей приклад наведено для тестування custom node.  Але, для тестування web api  не потрібно так багато писати. По суті потрібно:

- сформувати запит
- отримати відповідь
- перевірити відповідь на допустимі наявні поля, якісь ключові значення.


## <a name="p-2">2. Інструменти для виконання unit тестів web API</a>

Для  тестування використовуються такі інстументи:
- [mocha](https://www.npmjs.com/package/mocha), з додатковими бібліотеками:
    - [should](https://github.com/tj/should.js),
    - [expect](https://github.com/Automattic/expect.js),
    - [chai](https://www.chaijs.com/)
Цей інструмент використовується як головний для організації тестування.

- [supertest](https://www.npmjs.com/package/supertest)
Це високорівнева абстракція для  створення http запитів

Інсталяція виконується традиційною інструкцією

```bash
npm i mocha chai supertest
```
## <a name="p-3">3. Структура тест кейсів</a>

За звичай всі тести оформлюються в окремому каталозі **/test** і кожний набір тестів оформляється в окремому файлі.
В загальному структура кожного набора unit тестів обрамлена комбінацією **describe**. А кожний тест описується конструкцію **it**. В одному файлі може бути кілька конструкцій **describe**. Але треба мати на увазі, що вони будуть виконуватися асинхронно в паралельному режимі. Тому,  в більшості випадків  достатньо мати один **describe** в одному файлі. приклад наведено нижче. 

```js

describe('Коротка назва unit test', function() {
  it('Тест 1. Очікується відповідь status code=200', function(done) {

  });

  it('Тест 2. Очікується відповідь status code=422', function(done) {

  });

  it('Тест 3. Очікується відповідь status code=201', function(done) {

  });


});
```
Далі, загальна структура файлу unit test складається з таких частин:

```js

// підклчаємо основні модулі js
const mocha = require('mocha');
const chai = require('chai');
const request = require('supertest');

// Підключаємо основні бібліотеки що допомагають аналізувати структуру відповіді
const expect = require('chai').expect;
const assert = require('chai').assert;
const should = require('chai').should();

// Це URL  нашого сервісу
let baseurl="http://localhost:1880"

// Обрамляємо набір тестів, в средені якого вони виконуються послідовно
describe('Тестування методів Rest API ведення користувачів user-registration.json', function () {

    // Омисуємо один unit test
    it('Heath check Перевырка працездатності сервісу. Очікуємо http-200 ', function (done) {
        //Якщо цей  "it"  потрібно виключити із проходження тестів, то розкоментувати this.skip();
        //this.skip();

        //З допомгою supertest описуємо http запит  типу http-get  на path /health
        // .expect(200) - очікуємо відповідь http status code 200
        // .end  - callback де ми обробляємо отриманий результат відповіді, ну або помилку.
        request( baseurl )
            .get('/health')
            .set('Content-Type', 'application/json')
            .set('Accept', 'application/json' )
            .expect(200)
            .end(function (err, res) {
                if (err) {
                    // В протоколі тестування відмичається що оримана помилка і тест не пройшли
                    // done(err); закінсує тест з помилкою 
                    done(err);
                } else {
                    // отримуємо відповідь
                    var lrsp = res.body;
                    // аналізуємо, що у відповіді повинні бути поля "ok" та "msg"
                    res.body.should.have.property('ok');
                    res.body.should.have.property('msg');
                    // тут, перевіряємо що отримане значення поля "ok" повинно дорівнювати trus
                    res.body.ok.should.equal(true);
                    //закінчує тест успішно.
                    done();
                }
    });
});

```

По факту, можна  на першому етапі взагалі залишити "done(err); done();" і потім доповнювати по мірі розробки flow.
По факту, тести можна запускати з командного рядку:

```bash
// зпуск для windows
node node_modules\mocha\bin\_mocha --timeout 99999 --colors unit-tests\api-utest-1.js

```
Але, як на мене, простіше додати в package.json, який платформенно не залежний і запускати через нього

```json
  "scripts": {
    "dev": "node  node_modules/node-red/red.js --settings ./dev/settings.js --userDir .  --port 1880 --verbose user-registration.json",
    "start": "node  node_modules/node-red/red.js --settings ./prod/settings.js --userDir . --verbose --port 8080 $FLOW_NAME",
    "utest1": "mocha --timeout 99999 --colors unit-tests/api-utest-1.js"
  }

```
і запускати командою npm.

```bash
npm run utest1
```


<kbd><img src="/assets/img/posts/2023-09-08-node-red-custom-node/doc/pic-03.png" /></kbd>
<p style="text-align: center;"><a name="pic-03">pic-03</a></p>
#### <a name="p-3.1.2">3.1.2. Підготовка html файлу конфігураційної Node</a>
#### <a name="p-3.1.3">3.1.3. Підготовка html файлу основної Node</a>
### <a name="p-3.2">3.2 Підключеня пакету до node-red</a>

