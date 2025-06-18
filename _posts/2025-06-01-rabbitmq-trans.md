---
layout: post
title: "Queue transactions"
date: 2025-06-01 10:00:01
categories: [Node-Red,  RabbitMQ]
permalink: posts/2025-06-01/queue-transaction/
published: true
---

<!-- TOC BEGIN -->
- [1. Про що цей блог](#p1)
- [2. Сценарій 1. "Чиста" транзакційність (File Transfer/Data Replication)](#p2)
- [3. Сценарій 2: Асинхронна обробка з можливістю тимчасових помилок (Банківські платежі)](#p3)
- [4. Сценарій 3. Синхронно-асинхронні веб-сервіси (одноразова обробка)](#p4)
- [5. Висновки та додаткові міркування](#p5)
- [6. Прототип реалізацію метод **Retry queue** та **Dead Letter Queues** (Сценарій 2) на базі RabbitMQ](#p6)
- [6.1 Створення конфігурації **Retry**  на прикладі RabbitMQ](#p6.1)
- [6.2 Створеня прототипу на Node.js](#p6.2)
- [6.3 Node-Red: Невдала спроба створеня прототипу](#p6.3)
<!-- TOC END -->

## <a name="p1">1. Про що цей блог</a>

В багатьох мануалах продажників, що продають програмні продукти типу "черги" деінде дуже випирає, що їх продукт відрізняється підтримкою транзакцій. В цьому блозі наведено міркування з приводу корисності чи потрібності транзакційності відносно того, які сценарії обробки нам потрібно реалізувати. 

## <a name="p2">2. Сценарій 1. "Чиста" транзакційність (File Transfer/Data Replication)</a>

Через черги налаштована передача файлів з одного каталога в інший. Або ж  пакет записів бази даних публікується в чергу одним повідмоленням, а при читанні цей пакет зберігається як файл і  записується в каталог. В деяких випадках може бути важливою послідовність файлів. І, в якийсь момент, не вистачило місця, щоб записати файл в каталог. Виникла помилка і транзакція відкотилася, а повідомлення - повернулося в чергу. Весь процес обміну зупинився, до тих пір, поки не додадуть вільного місця для запису файла. Не зважаючи на те, що процес передачі зупинився, не порушилася послідовність надходження даних і, такми чином, дані залишилися цілісними. Такий же сценарій може виникнути, коли через чергу реалізована пакетна реплікація даних. І там теж важлива послідовність надходження та обробки пакетів.

**Тут працює принцип:** Якщо операція (наприклад, запис файлу, або пакетна реплікація) не може бути завершена успішно, то нічого не повинно бути записано або змінено, і стан системи має залишатися незмінним. Повідомлення повертається в чергу, щоб бути обробленим пізніше, коли умови для успішного виконання відновляться.

У таких системах передбачається наявністиь механізми моніторингу, які сигналізують про проблеми (наприклад, відсутність місця на диску). Після усунення проблеми, повідомлення, що повернулося, може бути успішно оброблене.

**Обмеження:** Слід пам'ятати, що така "чиста" транзакційність на рівні черги (тобто, повернення повідомлення) не забезпечує двофазний коміт або розподілені транзакції між кількома системами (наприклад, черга + база даних + файлова система + http). Для цього потрібні складніші патерни, такі як ["Outbox Pattern"](https://microservices.io/patterns/data/transactional-outbox.html) або ["Saga Pattern"](https://microservices.io/patterns/data/saga.html), які повинні гарантувати атомарність між різними сервісами. 
Відверто кажучи, я ці патерни не використовував, бо обходився більш простими рішеннями. 


## <a name="p3">3. Сценарій 2: Асинхронна обробка з можливістю тимчасових помилок (Банківські платежі)</a>

В банківській системі створені платежі: різні, різного типу, різних клієнтів. Між собою платежі не зв'язані. Вони відправляються в чергу і обробляються окремо. Кожний платіж обробляється окремо від інших. Але в цьому процесі велика вірогідність надходження не коректних (не валідних) даних в одному з платежів і тоді платіж не можливо обробити. Тобто завжди виникатиме помилка, доки не зміняться умови: змінять ліміт транзакцій для рахунку, на рахунок списання надійдуть кошти, виконаються якісь додаткові перевірки. Якщо використати **класичну транзакційність** *(Сценарій 1)*, то такий платіж повернеться в чергу і зупинить обробку інших платежів. В такому випадку класична транзакційність шкодить. І тоді використовують інший підхід. А саме метод **Retry queue** та **Dead Letter Queues**. Тобто, повідомлення-платіж відправляється в іншу чергу ( "черги повторних спроб"  **Retry queue**) і через якийсь проміжок часу повідомлення-платіж знову ререкладається в основну чергу для повторної спроби обробки. За час очікування може успішно обробитися багато "нормальних" платежів. Потім знову спробує обробитися платіж з **Retry queue**. Такий підхід зручний, коли ви очікуєте тимчасові помилки відносно одного повідомлення в черзі помилка не повинна зупиняти потік обробки інших. Або ж тимчасова не доступність інфраструктури не повинна зупиняти весь процес обробки повідомлень. В такого роду сценаріях навіть термін такий з'явився "отруйне повідомлення". Якщо таке "отруйне повідомлення" буде безкінечну кількість разів перекладатися в основну чергу (також є велика вірогідність накопичення такого роду повідомлень) то продуктивність системи може різко падати. Тому, зазвичай, кількість спроб **Retry**  обмежують. Іколи всі спроби вичерпані, то таке повідмолення відправляється до "черги мертвих повідомлень" (DLQ - Dead Letter Queues).

**Переваги цього методу:**
- Ізоляція проблем: "Проблемні" повідомлення не блокують обробку інших, коректних повідомлень.
- Автоматизація відновлення: Тимчасові збої (мережеві проблеми, тимчасова недоступність сервісу) можуть бути автоматично виправлені за допомогою механізму повторних спроб.
- Моніторинг: DLQs слугують чудовим місцем для моніторингу повідомлень, які не були оброблені, що дозволяє швидко виявляти та усувати системні проблеми.

Такий сценарій обробки притаманний більше для фонових процесів - обробників. 

## <a name="p4">4. Сценарій 3. Синхронно-асинхронні веб-сервіси (одноразова обробка)</a>

Як побудувати синхронно-асинхронний веб сервіс, описано за лінком [Async Web Service with RabbitMQ](https://pavlo-shcherbukha.github.io/posts/2025-05-08/asyncws-rabbitmq/). 

У сценаріях, де клієнт очікує швидку синхронну відповідь від веб-сервісу, а обробка в черзі є фоновою частиною запиту, транзакційність на рівні черги втрачає сенс. Мета – швидко відповісти клієнту про прийняття запиту (або про помилку у запиті) і не тримати його в очікуванні фонової обробки.

Якщо виникає помилка при обробці повідомлення з черги, її слід зафіксувати (логи, метрики), можливо, відправити повідомлення до DLQ для подальшого аналізу, але ні в якому разі не повертати до основної черги для повторної спроби, оскільки це не має жодного сенсу для кінцевого користувача або веб-сервісу, який вже отримав відповідь.

## <a name="p5">5. Висновки та додаткові міркування</a>

**Таким чином:** необхідність транзакційності (чи то "класичної", чи то з використанням DLQ/Retry) повністю залежить від бізнес-вимог та характеру операції. Не існує єдиного "правильного" підходу.

**Принцип виконання операцій:**  Незалежно від обраного підходу, дуже важливо, щоб операції, які обробляються з черги,відповідали принципу, що повторне виконання тієї ж операції з тим самим повідомленням (наприклад, якщо воно повернулося в чергу або було переоброблено з DLQ) не повинно призводити до небажаних побічних ефектів або подвоєння результатів (наприклад, подвійного списання грошей). Це критично для надійності розподілених систем.

**Моніторинг:** Завжди налаштовуйте моніторинг для ваших черг, особливо для DLQ. Накопичення повідомлень у DLQ є явним індикатором проблем у системі, які потребують уваги.

Особливості в "транзакційності":
- Атомарність обробки повідомлення: повідомлення буде успішно оброблене повністю або не оброблене взагалі.
- Розподілені транзакції: Це складніший патерн, який забезпечує атомарність операцій, що охоплюють кілька систем (наприклад, базу даних, зовнішній API та чергу).

## <a name="p6">6. Прототип реалізацію метод **Retry queue** та **Dead Letter Queues** (Сценарій 2) на базі Rabb</a>

Зробити прототип на базі класичної транзакційності - задача досить очевидна, тому мабуть немає сенсу на неї витрачати час. А сценарій синхронно-асинхронного web service описано в попередньому блозі [Async Web Service with RabbitMQ](https://pavlo-shcherbukha.github.io/posts/2025-05-08/asyncws-rabbitmq/) і там уже є прототип. Тому залишається **Retry queue**. Оособливістю цього прототипа є необхідність створення відповідної конфігурації черг RabbitMq, що пов'язані з цим прототипом (з вашою бізнеслогіогікою).
Сам прототип знаходиться за лінком [asyncretry-p](https://github.com/pavlo-shcherbukha/asyncretry-p).
---
Зразу потрібно зауважити, що на Node-Red мені не вдалося зробити такий прототип, тому, що бульшість вузлів: або дуже старі, або є обмеження фукнціональності вузла що не дозволяють це зробити. Проблема в тому, що вузли самі, автоматично створюють чергу, якщо її немає. А якщо ви вже створили конфігурацію черг та обмінників з додатковими аргуентами, то отримаєте помилку підключення, тому що простенька декларація черги у вузлі не співпадає з тим, що на справді створено в RabbitMQ


Так, в каталозі **test**  знаходиться  bash-скрипт для створення конфігурації **create-cfg.sh** . Нижче показано його фрагменти для створення конфігурації з поясненнями. 

В цьому фрагменті заповнено ключ **"arguments"**. А ноди для Node-Red  не передбачають його заповення.
```bash
curl -u $XUSR:$XPSW -X PUT \
-H "Content-Type: application/json" \
-d '{"auto_delete":false,"durable":true,"arguments":{"x-dead-letter-exchange": "retry-exchange"}}' \
http://$XHOST:$XPORT/api/queues/%2F/main-queue
```
 В цей момент і виникає помилка. На додаток в багатьох комплекатх вузлів Node-Red відсутнє ручне управління транзакцією. 
В реальному житті мені приходилося дуже часто використовувати саме цей шаблон, правда на чергах IBM MQ. Тому я змушений був написати прототип на Node.JS.
---

### <a name="p6.1">6.1 Створення конфігурації **Retry**  на прикладі RabbitMQ</a>

Конфігурація показана на [pic-101](#pic-101)

<kbd><img src="/assets/img/posts/2025-04-08-asyncws-rabbit/doc/pic-101.svg" /></kbd>
<p style="text-align: center;"><a name="pic-101">pic-101</a></p>

По суті є основна черга **main_queue** та exchange **main_exchange** через який в **main_queue**  потрапляють повідомлення. 
Для повторної обробки є додаткові черга **retry-queue** та exchange **retry-exchange**.
Якщо за задану кількість циклів повідомлення не вдалося обробити, до воно відкидається в чергу **parking-queue**.  
Створити конфігурацію в один клік можна за допомогою bash скрипта [/tests/create-cfg.sh](https://github.com/pavlo-shcherbukha/asyncretry-p/blob/main/tests/create-cfg.sh), використовуючи адміністративний Rest API. 
Видалити конфігурацію в один клік можна за допомогою bash скрипта [/tests/delete-cfg.sh](https://github.com/pavlo-shcherbukha/asyncretry-p/blob/main/tests/delete-cfg.sh).

Всі інші  bash-скрипти в катагозі **test** наведені для демонстрації базових можливостей адміністративного rest API.

**Міркування з приводу особливостей обробки повідомлення в прив'язці до RabbitMQ:**

- **Якщо повідомлення оброблено успішно**. 

Надсилаємо в канал повідомлення **ACK**, закриваючи транзакцію (щось на кшталт commit  в базах даних).

```js
    channel.ack(msg);
    console.log(" [x] Message ACKed");
```

Відповідь публікується в чергу **reply-queue**. Публікація повідмолення відбувається таким чином:

```js
    // Якщо треба відправити відповідь у replyTo:
    if (msg.properties.replyTo) {
        channel.sendToQueue(
            msg.properties.replyTo,
            Buffer.from('{"status":"ok"}'),
            { correlationId: msg.properties.correlationId, contentType: 'application/json' }
        );
    }
```
Як бачимо, тут exhange  не вказується (або вказується пустий рядок), а в якості routing key вказується назва черги. При такій публікації, повідомлення пройде через exchange: (AMQP default). По суті, до цього службового exchange "прив'язуються" всі черги з routing key який співпадає з назвою черги.

- **Якщо виникла помилка при обробці повідомлення**.
У разі помилки повідомлення повинно автоматично відправитися в чергу **retry_queue**. В цій черзі повідомлення деякий час полежить в надії на те, що зміняться зовнішні умови і повідомлення  повернеться в чергу main_queue і  попаде на обробку повторно. Щоб повідомленя могло автоматично перенестися в іншу чергу, потрібно зробити наступне ( далі ідуть фрагменти з [/tests/create-cfg.sh](https://github.com/pavlo-shcherbukha/asyncretry-p/blob/main/tests/create-cfg.sh) та пояснення до них):

1. При створенні черги  **main-queue** потрібно вказати додаткові аргументи *"arguments":{"x-dead-letter-exchange": "retry-exchange"}*, де ключ **"x-dead-letter-exchange"** вказує на назву exchange, що перенаправить повідомлення в **retry_queue**. 

Тут показано скрипт для створення main_queue. 

```bash
echo "Creating RabbitMQ main-queue"
curl -u $XUSR:$XPSW -X PUT \
-H "Content-Type: application/json" \
-d '{"auto_delete":false,"durable":true,"arguments":{"x-dead-letter-exchange": "retry-exchange"}}' \
http://$XHOST:$XPORT/api/queues/%2F/main-queue

```
Тут показано скрипт для створення main_exchange. 

```bash
echo "Creating RabbitMQ main-exchange"
curl -u $XUSR:$XPSW -X PUT \
-H "Content-Type: application/json" \
-d '{"type":"direct","durable":true,"auto_delete":false,"internal":false,"arguments":{}}' \
http://$XHOST:$XPORT/api/exchanges/%2F/main-exchange
```

А тут зв'язувіання queue та exchange. 

```bash
echo "Bind queue to exchange main"
curl -u $XUSR:$XPSW -X POST \
-H "Content-Type: application/json" \
-d '{"routing_key":"srvc.transact.cash","arguments":{}}' \
http://$XHOST:$XPORT/api/bindings/%2F/e/main-exchange/q/main-queue
```
Як бачимо, що повідомлдення попаде в чергу, якщо буде вказано routing key: **"srvc.transact.cash"**

2. Створити **retry-exchange**.
Тут є одна особливість: не повинен змінюватися оригінальний **routing key**. Для цього ідеально підходить **fanout exchange**. Exchange такого типу ігнорує **routing key**, але його не видаляє і не змінює, а просто копіює і передає далі. **Fanout exchange** прив'язує до себе черги з порожнім **routing key**.

**Створення  retry exchange типу fanout**

```bash
echo "Creating RabbitMQ retry-exchange"
curl -u $XUSR:$XPSW -X PUT \
-H "Content-Type: application/json" \
-d '{"type":"fanout","durable":true,"auto_delete":false,"internal":false,"arguments":{}}' \
http://$XHOST:$XPORT/api/exchanges/%2F/retry-exchange

```
3. Створити чергу **retry-queue**.
При створенні черги знадобляться додаткові аргументи:
- треба встановити термін "життя" повідомлення **TTL**;
- треба встановити **"x-dead-letter-exchange"** з назвою exchange, що пов'язаний з **main_queue**.

Тут показано, створення черги. TTL "x-message-ttl": 30000 вказує, що повідомлення буде "жити" в **retry_queue** 30 секунд. 

```bash
echo "Creating RabbitMQ retry-queue"
curl -u $XUSR:$XPSW -X PUT \
-H "Content-Type: application/json" \
-d '{"auto_delete":false,"durable":true,"arguments":{ "x-message-ttl": 30000, "x-dead-letter-exchange": "main-exchange" }}' \
http://$XHOST:$XPORT/api/queues/%2F/retry-queue

```
Тобто повідомлення з retry-queue  з тим же routing key  через main-exchange  знову попаде в main_queue  через 30 секунд. А в retry_queue  воно "пропаде".

4. Зв'яжемо exchange  та retry-queue.

```bash
echo "baind retry q of exhcange retry"
curl -u $XUSR:$XPSW -X POST \
-H "Content-Type: application/json" \
-d '{"routing_key":"","arguments":{}}' \
http://$XHOST:$XPORT/api/bindings/%2F/e/retry-exchange/q/retry-queue

```
тут треба звернути увагу, що **"routing_key":""** (пустий) . 

5. В програмному коді, при виникненні помилки треба в чергу надіслати сигнал **NACK**.

Щось на кшталт оцього фраменту коду.
```js
    if (parsedContent.amount > 100.00) {
        console.error(" [!] Amount exceedds limit:", parsedContent.amount);
        channel.nack(msg, false, false);
        console.log(" [x] Message NACKed due to invalid amount");
        return;       
    }
```

В даному випадку реалізовано перекладання повідомлення в retry-queue, але таке повідомлення може ходити безкінечну кількість раз. Щоб такого не сталося вводиться ще одна черга **parking_queue**. В цю чергу перекладаються повідомлення, що використали задану кількість спроб обробки і не обробилися.

Декларація **parking_queue**

```bash
echo "Creating RabbitMQ parking queue"

curl -u $XUSR:$XPSW -X PUT \
-H "Content-Type: application/json" \
-d '{"auto_delete":false,"durable":true,"arguments":{}}' \
http://$XHOST:$XPORT/api/queues/%2F/parking-queue

```

Публікувати повідомлення будемо так як і в reply-queue через default exchange.

```js
    const xDeath = msg.properties.headers['x-death'];
    let retryCount = 0;
    if (xDeath && Array.isArray(xDeath) && xDeath.length > 0) {
        retryCount = xDeath[0].count;
    }

    if (retryCount >= MAX_RETRY) {
        // Перенаправити у parking queue
        channel.sendToQueue( parking_queue, msg.content, msg.properties);
        channel.ack(msg);
        console.log('Message sent to parking queue');
        return;
    } 

```
Тут треба звернути увагу на властивості (заголовки) повідмолення. Коли ви в канал передаєте NACK, то автоматично додається у вдастивість (заголовок) повідомелння headers.x-death[]. Це масив структур. В кожній структурі є ключ **counter**, що показує скульки раз повідомлення попадало на обробку.

```json
{ 
  "contentType": "application/json",
  ...
  ...,  
  "headers": { 
    "x-death": [
       {"counter": 3, .....},
       {"counter": 1, .....}        
    ]
  }
}

```
З приводу створення конфігурації - все.

### <a name="p6.2">6.2 Створеня прототипу на Node.js</a>

Чому взявся спершу створити прототип на Node.js - та тому, що в своїй базі всі ноди Node-Red побудовані навколо бібіліотеки Node.JS [amqplib](https://www.npmjs.com/package/amqplib). Знаходиться прототип в каталозі [msg-srvc](https://github.com/pavlo-shcherbukha/asyncretry-p/tree/main/msg-srvc). Складається прототип з двох елементів:
- публікатор повідомлень [/msg-srvc/publisher.js](https://github.com/pavlo-shcherbukha/asyncretry-p/blob/main/msg-srvc/publisher.js).

В чергу *main_queue** (через exchange **main-exchange**) публікує повідомленя - json

```json
{"num": 1000, "dbt": "1001001", "krd": "1007222", "amount": 10.23, "remark": "cash payment"}
```

А до повідомлення додаються заголовки:

```json
{ 
    "persistent": true,
    "correlationId": "${correlationId}",
    "replyTo": "${replyToQueue}",
    "contentType": "application/json",
    "headers": {
        "x-cerrelation-id": "${correlationId}",
        "x-request-type": "cash-transaction"

    }
}
```

сама публікація покказана тут:

```js
    const correlationId = uuidv4();
    const replyToQueue  = "reply-queue";
    const exchange= "main-exchange";
    const routingKey = "srvc.transact.cash";
    .....
    .....

    // Publish message to the exchange with routing key
    channel.publish(exchange, routingKey, Buffer.from( JSON.stringify(msg) ),
                { 
                    persistent: true,
                    correlationId: correlationId,
                    replyTo: replyToQueue,
                    contentType: 'application/json',
                    headers: {
                        'x-cerrelation-id': correlationId,
                        'x-request-type': 'cash-transaction'

                    }
                }); 

```

- споживач повідомлень [/msg-srvc/consumer.js](https://github.com/pavlo-shcherbukha/asyncretry-p/blob/main/msg-srvc/consumer.js)

Споживач підключається до **main-queue**. При обробці повідомлень побудована така логіка. Якщо в отриманому повідомленні ключ **amount** більше 100.00, то генерується прикладна помилка. Відповідно, повідомлення відправиться в Retry. Пізніше, коли повідомлення спробує обробитися 3 рази і не обробиться - воно відправиться в **parking-queue**.
Мені не вдалося протестувати повідомлення тільки за допомогою адмівністративного Rest API -  тому і написав на Node.js.

Ствремо конфігурацію RabbitMQ  та опублікуємо кілька "нормальних" повідомлень, та одне "ядовите", що поверне його в retray щоб подивитися, як працює конфігурація:

```bash
..../asyncretry/tests $ ./create-cfg.sh
Create RabbitMQ Retry queue configuration
Creating RabbitMQ retry-queue
Creating RabbitMQ main-queue
Creating RabbitMQ reply-queue
Creating RabbitMQ parking queue
Creating RabbitMQ retry-exchange
Creating RabbitMQ main-exchange
Bind queue to exchange main
baind retry q of exhcange retry
press any key to continue

..../asyncretry/tests $ cd ..
..../asyncretry $ cd ./msg-srvc
..../asyncretry/msg-srvc $ ls
consumer.js  node_modules  package.json  package-lock.json  publisher.js
..../asyncretry/msg-srvc $ npm run pub

> publisher@1.0.0 pub
> node publisher.js

 [x] Sent '{"num":1000,"dbt":"1001001","krd":"1007222","amount":10.23,"remark":"cash payment"}' with routing key 'srvc.transact.cash'
.../asyncretry/msg-srvc $ npm run pub

> publisher@1.0.0 pub
> node publisher.js

 [x] Sent '{"num":1000,"dbt":"1001001","krd":"1007222","amount":10.23,"remark":"cash payment"}' with routing key 'srvc.transact.cash'
.../asyncretry/msg-srvc $ npm run pub

> publisher@1.0.0 pub
> node publisher.js

 [x] Sent '{"num":1000,"dbt":"1001001","krd":"1007222","amount":210.23,"remark":"cash payment"}' with routing key 'srvc.transact.cash'
..../asyncretry/msg-srvc $ 

```
Останнє повідомлення має реквізит **amount** > 100.00 і тому викличе помилку обробки (за бізнес логікою).

Запустимо читання повідомлень з черги та ролдивимось по логах, що відбужеться:

```bash
..../asyncretry/msg-srvc $ npm run con

> publisher@1.0.0 con
> node consumer.js

 [*] Waiting for messages. To exit press CTRL+C
Retry count: 0
Retry count: undefined
 [x] Received message with correlationId: a295d1fc-7470-4795-91d5-da2a8bb95ab0
 [x] Received message with contentType: application/json
 [x] Received: {"num":1000,"dbt":"1001001","krd":"1007222","amount":10.23,"remark":"cash payment"}
 [X] Parsing content: ok
 [x] Message ACKed
Retry count: 0
Retry count: undefined
 [x] Received message with correlationId: 3eba9aee-6864-4a4c-a642-e0fe069ca283
 [x] Received message with contentType: application/json
 [x] Received: {"num":1000,"dbt":"1001001","krd":"1007222","amount":10.23,"remark":"cash payment"}
 [X] Parsing content: ok
 [x] Message ACKed
Retry count: 0
Retry count: undefined
 [x] Received message with correlationId: 6a19fff1-560b-4e24-8bc4-b94d34487b72
 [x] Received message with contentType: application/json
 [x] Received: {"num":1000,"dbt":"1001001","krd":"1007222","amount":210.23,"remark":"cash payment"}
 [X] Parsing content: ok
 [!] Amount exceedds limit: 210.23
 [x] Message NACKed due to invalid amount
Retry count: 1
Retry count: [
  {
    "count": 1,
    "reason": "expired",
    "queue": "retry-queue",
    "time": {
      "!": "timestamp",
      "value": 1750251800
    },
    "exchange": "retry-exchange",
    "routing-keys": [
      "srvc.transact.cash"
    ]
  },
  {
    "count": 1,
    "reason": "rejected",
    "queue": "main-queue",
    "time": {
      "!": "timestamp",
      "value": 1750251770
    },
    "exchange": "main-exchange",
    "routing-keys": [
      "srvc.transact.cash"
    ]
  }
]
 [x] Received message with correlationId: 6a19fff1-560b-4e24-8bc4-b94d34487b72
 [x] Received message with contentType: application/json
 [x] Received: {"num":1000,"dbt":"1001001","krd":"1007222","amount":210.23,"remark":"cash payment"}
 [X] Parsing content: ok
 [!] Amount exceedds limit: 210.23
 [x] Message NACKed due to invalid amount
Retry count: 2
Retry count: [
  {
    "count": 2,
    "reason": "expired",
    "queue": "retry-queue",
    "time": {
      "!": "timestamp",
      "value": 1750251800
    },
    "exchange": "retry-exchange",
    "routing-keys": [
      "srvc.transact.cash"
    ]
  },
  {
    "count": 2,
    "reason": "rejected",
    "queue": "main-queue",
    "time": {
      "!": "timestamp",
      "value": 1750251770
    },
    "exchange": "main-exchange",
    "routing-keys": [
      "srvc.transact.cash"
    ]
  }
]
 [x] Received message with correlationId: 6a19fff1-560b-4e24-8bc4-b94d34487b72
 [x] Received message with contentType: application/json
 [x] Received: {"num":1000,"dbt":"1001001","krd":"1007222","amount":210.23,"remark":"cash payment"}
 [X] Parsing content: ok
 [!] Amount exceedds limit: 210.23
 [x] Message NACKed due to invalid amount
Message sent to parking queue
^C
.../asyncretry/msg-srvc $ 

```
Як бачимо по логах, після 3 спроб обробити повідомлення, його життєвий цикл закінчивчя в **parking_queue**.

На [pic-102](#pic-102) можна побачити початкове розподілення повідомлень  по чергам (на верхньому малюнку).

<kbd><img src="/assets/img/posts/2025-04-08-asyncws-rabbit/doc/pic-102.svg" /></kbd>
<p style="text-align: center;"><a name="pic-102">pic-102</a></p>

Та фінальне розподілення повідомлень по чергам: 
- два нормальних знаходяться в retry-queue
- одне (ядовите) перекочувало в parking_queue.


Додаткову, хочу звернути увагу на інформацію, яка присутня у властивості **"x-death"** 

```json
"x-death": [ 
  {
    "count": 2,
    "reason": "expired",
    "queue": "retry-queue",
    "time": {
      "!": "timestamp",
      "value": 1750251800
    },
    "exchange": "retry-exchange",
    "routing-keys": [
      "srvc.transact.cash"
    ]
  },
  {
    "count": 2,
    "reason": "rejected",
    "queue": "main-queue",
    "time": {
      "!": "timestamp",
      "value": 1750251770
    },
    "exchange": "main-exchange",
    "routing-keys": [
      "srvc.transact.cash"
    ]
  }
]


```

### <a name="p6.3">6.3 Node-Red: Невдала спроба створеня прототипу</a>

На жаль на Node-Red такий прототип створити не вийшло.
- Перша причнина, це те, що на вузлах не можливо вказати додаткові аргументи при створенні черги, які я вказував через адміністративне API 
  
- Друга причина, це те, що при відсутності черги - consumer  автоматично її пробує створити, але без додаткових аргумсентів. А коли RabbitMQ отримує команду на створення чекрги, яка вжек існує, то вона порівнює їх парампетри. І, у випадку якщо параметри черг відрізнюяться - вузол не підключається до rabbit MQ.

Яому так зроблено - питання риторичне. Я кілька вузлів перепробував - результат вийшов не успішний.