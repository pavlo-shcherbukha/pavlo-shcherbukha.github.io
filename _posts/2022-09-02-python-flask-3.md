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
2. [Розгортання Redis локально в Docker контейнері](#p-2)
3. [Короткий довідник по командах Redis які можна виконати з допомогою redis-клієнта](#p-3)
  * 3.1. [[SET, GET, INCR]  Встановлення - читання ключа та його значення](#p-3.1)
  * 3.2. [[KEYS]  Отримати список всіх ключів в БД](#p-3.2)
  * 3.3. [[HSET, HGET, HDEL, HEXISTS]  Встановлення - читання hash-ключа](#p-3.3)
  * 3.4. [[DEL, EXISTS] Видалити ключ, перевірити навність ключа](#p-3.4)
  * 3.5. [Як зберегти JSON в Redis](#p-3.5)
  * 3.6. [Корисні посилання по redis](#p-3.6)

4. [Використання Redis сумісно з Python-flask application](#p-4)
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

## <a name="p-2">Розгортання Redis локально в Docker контейнері</a>

Перш, ніж  інтегрувати Python application з Redis  потрібно навчитися впевнено розгортати сам redis. А, якщо зважити на те, що потрібно запустити як мінімум 2 сервіси: python service і БД redis - то краще зразу  запускати все це з допомогою docker-compolser, якщо запускати на вланому laptop. 

Для прикладів вибрана redis-5 з офіційного образу redis на [Docker hub з тегом версії redis:5.0.14-alpine ](https://hub.docker.com/_/redis).  Для старту достатньо підготувати **docker-composer.yaml**

```yaml

version: '3.8'
services:
  redisserver:
    image: redis:5.0.14-alpine
    restart: always
    ports:
      - '6379:6379'
    command: redis-server --save 20 1 --loglevel warning --requirepass 22

```

В цьому фрагмені **Redis** стартує з авторизацією по паролю [--requirepass 22].
Для запуску, потрібно перейти в каталог з файлом **docker-composer.yaml** та і запустити старт командою

```bash
docker compose up

```
На екарні отримаєте щось схоже на таке.
<kbd><img src="/assets/img/posts/2022-09-02-python-flask-3/doc/pic-01.png" /></kbd>
<p style="text-align: center;"><a name="pic-01">pic-01</a></p> 

Якщо подивитися на контейнери, що стартонули, то можна побачити redis:

В іншій cmd сесії запускаємо:

```bash
  docker ps
```

На екарні отримаєте щось схоже на таке.
<kbd><img src="/assets/img/posts/2022-09-02-python-flask-3/doc/pic-02.png" /></kbd>
<p style="text-align: center;"><a name="pic-02">pic-02</a></p> 



Тепер спробуємо підключитися до контейнера з допомогою redis-cli. Тут ми нічого не інсталюємо наробочу станцію, а заходиво в середину контейнера, шляхом запука redis-cli:

```bash
    docker exec  -u root -it 1c78b4161df0 redis-cli -a 22

```

Тобто, тут ми заходимо по ssh як root в контейнер з id 1c78b4161df, запускаючи redis-cli  з авторизацією, вказуючи пароль до БД з ключем **-a**


На екарні отримаєте щось схоже на таке.
<kbd><img src="/assets/img/posts/2022-09-02-python-flask-3/doc/pic-03.png" /></kbd>
<p style="text-align: center;"><a name="pic-03">pic-03</a></p> 

Ну і для перевірки з'єднання задаємо комнаду "PING", а у відповідь отримуємо PONG.
Таким чином в мінімальному вигляді контейнре запущено.



## <a name="p3">Короткий довідник по командах Redis які можна виконати з допомогою redis-клієнта</a>

Особисто я Redis-cli не користуюся. Але в цтому розділі хочу паказати деякі можливості redis, які в майбутньому буду викистовувати через спеціальні мовні бібіліотеки 

### <a name="p-3.1">[SET, GET, INCR]  Встановлення - читання ключа та його значення</a>

Найпершиою і найпростішою операцією є запам'ятати пари: ключ-знаення. Цікавим варіаном є можливість зберігання ключа заданий період часу (секінд, мілісекунд).

 - Встановити ключ та значення

```text
set myKey myvaluse
```

- Прочитати значення ключа

```text
get myKey

```
<kbd><img src="/assets/img/posts/2022-09-02-python-flask-3/doc/pic-04.png" /></kbd>
<p style="text-align: center;"><a name="pic-04">pic-04</a></p> 

Встановити ключ, що "живе" протягом визначеного часу. На прикладі, що показано видно, як встановлено значення ключа на 4 секунди. А потім, через якийсь час - ключ "пропадає"

```text

#
# set myKey value [expiration EX seconds|PX milliseconds] [NX|XX]

 set myKey XCODE EX 4

```

<kbd><img src="/assets/img/posts/2022-09-02-python-flask-3/doc/pic-05.png" /></kbd>
<p style="text-align: center;"><a name="pic-05">pic-05</a></p> 

Тут показано простий варіант установки лічільника і його постійного збільшення

```text
127.0.0.1:6379> set xcntr 0
OK
127.0.0.1:6379> get xcntr
"0"
127.0.0.1:6379> incr xcntr
(integer) 1
127.0.0.1:6379> incr xcntr
(integer) 2
127.0.0.1:6379> incr xcntr
(integer) 3
127.0.0.1:6379> incr xcntr
(integer) 4
127.0.0.1:6379> incr xcntr
(integer) 5
127.0.0.1:6379> get xcntr

```

<kbd><img src="/assets/img/posts/2022-09-02-python-flask-3/doc/pic-06.png" /></kbd>
<p style="text-align: center;"><a name="pic-06">pic-06</a></p> 


###  <a name="p-3.2">[KEYS]  Отримати список всіх ключів в БД</a>

Отримати всі ключі в БД можа командою

```text
  # keys pattern

```

На приклад:

```text
127.0.0.1:6379> keys *
1) "jsondata"
2) "myhash"
3) "xcntr"
4) "counter"
5) "shkey1"

```


### <a name="p-3.3">[HSET, HGET, HDEL, HEXISTS]  Встановлення - читання hash-ключа</a>

Ця команда під одним ключем може зберігати кілька значень FIELD: VALUE. Обмеження - прочитатит можна тілько по одному полю.
Збережемо під ключем sh-book  опис книжки: назву, автора, рік публікації. В принципі, це аналогічно збереженню JSON

{

  "sh-book": {"title": "", "author": "", published: 2009}
}


```text

   hset sh-book  title  "The Museum of Abandoned Secrets"
   hset sh-book  author "Oksana Zabuzhko"
   hset sh-book  published "2009" 

```

або ж можна ввести групову команду

```text
hset sh-book  title  "Second Attempt" author "Oksana Zabuzhko" published "2005" 

```

- Перевірити наявність поля в ключі

```text
  hexists sh-book title
```

- Видалити поле в ключі

```text

  hdel sh-book title

```

По суті ми зберегли та к би мовити плоский json

### <a name="p-3.4">[DEL, EXISTS] Видалити ключ, перевірити навність ключа</a>

```text
  del shkey
```

```text
127.0.0.1:6379> keys *
 1) "myhash"
 2) "xcntr"
 3) "counter"
 4) "shhkey2"
 5) "gey"
 6) "shkey1"
 7) "jsondata"
 8) "get"
 9) "sh-book"
10) "shhkey"
127.0.0.1:6379> del shhkey
(integer) 1
127.0.0.1:6379> keys *
1) "myhash"
2) "xcntr"
3) "counter"
4) "shhkey2"
5) "gey"
6) "shkey1"
7) "jsondata"
8) "get"
9) "sh-book"
127.0.0.1:6379>

```

Так можна перевірити наявнісмть ключа

```text
127.0.0.1:6379> keys *
1) "myhash"
2) "xcntr"
3) "counter"
4) "shhkey2"
5) "gey"
6) "shkey1"
7) "jsondata"
8) "get"
9) "sh-book"
127.0.0.1:6379> exists sh-book
(integer) 1
127.0.0.1:6379>
```

### <a name="p-3.5">Як зберегти JSON в Redis</a>

Допустимо у нас є JSON:

{ "title": "Second Attempt" ,  "author": "Oksana Zabuzhko", "published": 2005 }

то зберегти його можна звичайною команою **set**
Зберегти його можна звичайною командою set

```text
set book1 '{ "title": "Second Attempt" ,  "author": "Oksana Zabuzhko", "published": 2005 }'

```

Ось результат роботи комнади:

```text
127.0.0.1:6379> set book1 '{ "title": "Second Attempt" ,  "author": "Oksana Zabuzhko", "published": 2005 }'
OK
127.0.0.1:6379> get book1
"{ \"title\": \"Second Attempt\" ,  \"author\": \"Oksana Zabuzhko\", \"published\": 2005 }"
127.0.0.1:6379>
```

### <a name="p-3.6">Корисні посилання по redis</a>

- https://stackoverflow.com/questions/40678865/how-to-connect-to-remote-redis-server
- https://redis.io/docs/manual/security/
- https://hub.docker.com/_/redis
- https://stackoverflow.com/questions/7537905/how-to-set-password-for-redis
- https://github.com/docker-library/redis/blob/d36a031654f52ab95601de1f8a841765177cf702/5/alpine/Dockerfile
- https://hub.docker.com/_/redis
- https://stackoverflow.com/questions/33304388/calling-redis-cli-in-docker-compose-setup
- https://geshan.com.np/blog/2022/01/redis-docker/
- https://redis.io/docs/manual/cli/
- https://redis.io/docs/getting-started/
- https://packagist.org/packages/predis/predis
- https://stackoverflow.com/questions/49375856/using-composer-wallet-redis-where-to-see-the-card-in-redis-service 
- https://github.com/hyperledger-archives/composer-tools/tree/master/packages/composer-wallet-redis
- https://geshan.com.np/blog/2022/01/redis-docker/
- https://stackoverflow.com/questions/69140125/docker-redis-seeding-data-cannot-execute-redis-cli-via-dockerfile-or-via-she
- https://stackoverflow.com/questions/60828349/redis-cli-from-docker-compose-redis-container-doesnt-catch-any-keys-set-up-thro
- https://hub.docker.com/r/bitnami/redis





## <a name="p-4">Використання Redis сумісно з python flask application</a>

В цьому розділі показана проста демка, для демонстрації того, як використати  redis сумісно  з python flask applicatoin. Програмний код демонстрашки можна знайти за лінком [Flask app Rest API та взаємодія з Redis](https://github.com/pavlo-shcherbukha/py-flask-app-smpl-redis). Опис бібліотеки Python для роботи з redis знаходиться за лінком: [redis 4.3.4](https://pypi.org/project/redis/) або ж  прямо на github [redis-py](https://github.com/redis/redis-py). Цікаві і потрібні, як на мій погляд, приклади наедені в [ndexing / querying JSON documents](https://redis.readthedocs.io/en/stable/examples/search_json_examples.html) 

В демці розглядається, як запустити redis  та Python web service, що зверта
ться до redis  на совєму ноутбуці в контейнерах, викорситовуючи docker-composer. Це, так би мовити, створення та запуск середовища розробки на своєму laptop.

### Опис програмного коду демки.

Redis використовується в модулі **views.py**. Підключення до redis описано у наведеному фрагменті:

```py
log("Підключення до Redis")

irds_host = os.getenv('RDS_HOST');
irds_port = os.getenv('RDS_PORT');
irds_psw = os.getenv('RDS_PSW');

log('Підключеня до redis: ' + 'host=' + irds_host  )
log('Підключеня до redis: ' + 'Порт=' + str(irds_port)  )
log('Підключеня до redis: ' +  'Пароль: ' + irds_psw )

red = redis.StrictRedis(irds_host, irds_port, charset="utf-8", password=irds_psw, decode_responses=True)
log(" Trying PING")
log("1=======================")
rping=red.ping()
log( str(rping) )
if rping:
    log("redis Connected")
    #sub = red.pubsub()    
    #sub.subscribe( ichannel )    
else:
    log("redis NOT CONNECTED!!!")    

log("2=======================")

```

#### Підключення до redis
Як видно, парамери підключення параметризуються в ENV-variables  сервісу: RDS_HOST, RDS_PORT, RDS_PSW, які вичитуються при старті сервісу і повинні бути задані при старті контейнера (якщо сервіс стартує в контейнері).
Підклюення (чи то створення об'єкту підклченя) описано в команді: **red = redis.StrictRedis**  .....  А ось перевырити можливість взаємодії сервіса та redis  можна з допопогою команди **PING**: 

```text
rping=red.ping()

```
Тобто, якщо rping=True, значить до redis сервіс підключився. 

<kbd><img src="/assets/img/posts/2022-09-02-python-flask-3/doc/pic-07.png" /></kbd>
<p style="text-align: center;"><a name="pic-07">pic-07</a></p> 

#### Проста функція лічилька кількості викликів АПІ

Використаемо просут команду бібліотеки: [ incrby(name, amount=1)](https://redis.readthedocs.io/en/stable/commands.html?highlight=incrby#redis.commands.cluster.RedisClusterCommands.incrby). Також, при старті сервісу ключ потрібно створити, а потім бажано ще і прочитати. Для цього використаємо комнади:

- [set(name, value, ex=None, px=None, nx=False, xx=False, keepttl=False, get=False, exat=None, pxat=None)](https://redis.readthedocs.io/en/stable/commands.html?highlight=incrby#redis.commands.core.CoreCommands.set)

- [get(name)](https://redis.readthedocs.io/en/stable/commands.html?highlight=incrby#redis.commands.core.CoreCommands.get)

Цю задачу вирішують два простих фагменти кода.

Тут при старті створюється ключ в БД
```py
i_apicntr_key="APICALLS"
if rping:
    log("redis Connected")

    log("set predefined key by 0 value: " +  i_apicntr_key ) 
    red.set(i_apicntr_key, 0)
    log("Check the valuse of key: " + i_apicntr_key )
    log( "Read value: " + str( red.get(i_apicntr_key) ) )
```

А тут є функція, що виконє інкремент значення ключа

```py
#==================================================
# Функція підраховування викликів API
#
#=================================================
def apicallscntr():
    l_label="apicallscntr"
    log("Старт", l_label)
    return red.incrby( i_apicntr_key, 1)
```

Ну і далі виклик цієї функції вмонтовуємо в оброники викликів API

#### Розробка API методів, що дозволяюь створити ключ, прочитати занчення ключа, отримати список всіх ключів в Redis


Для читання ключів використовується функціія: [ keys(pattern='*', **kwargs)](https://redis.readthedocs.io/en/stable/commands.html?highlight=incrby#redis.commands.cluster.RedisClusterCommands.keys).

У відповіді повинні отримати масив всіх існуючих ключів, типу такого: 

```json
{
  "list": ["shhkey2", "shkey1", "APICALLS", "book1", "myhash", "jsondata", "get", "counter", "xcntr", "sh-book", "gey"]
}
```


Ну а для створення ключа використовуєму функцію: - [set(name, value, ex=None, px=None, nx=False, xx=False, keepttl=False, get=False, exat=None, pxat=None)](https://redis.readthedocs.io/en/stable/commands.html?highlight=incrby#redis.commands.core.CoreCommands.set)

Тіло POST запиту:

```json
{
"keyname": "test1",
"keyvalue": "test1 value"
}

```


Для демонстрації взаємодії з redis цього  досить. Тепер проблема, як це запустити у себе на машині.


### Запуск та налашьування контейнерів 


Коли програмний код уже існує, потрібно налаштувати узгожений запуск двох контейнерів:

- контейнера redis

- контейнера з PYthon  додатком на laptop

Для цього використано **docker-composer**. Хоч, выдверто кажучи, я його ы не дуже люблю. Мені більш звичним є такі платформи як Openshift та  kubernetes.  Але для цього випадку прийшлося використати **docker-composer**. Основним конфігураційним файлом, що зв'язує різні контейнери між собою є yaml-файл: **docker-compose.yaml**:

```yaml
version: '3.8'
services:
  redisserver:
    image: redis:5.0.14-alpine
    restart: always
    ports:
      - '6379:6379'
    command: redis-server --save 20 1 --loglevel warning --requirepass 22
  smplapp-srvc-redis:
    build:
      context: ./
      dockerfile: Dockerfile_local 
    ports:
      - "8081:8080" 
    links:
      - "redisserver:redis"     
    environment:
      GUNICORN_CMD_ARGS: "--workers=1 --bind=0.0.0.0:8080 --access-logfile=-"
      APP_MODULE: "hello_app.webapp"
      RDS_HOST: "redisserver"
      RDS_PORT: 6379
      RDS_PSW: "22"

```

На pic-08  показано з яких частин цей файл склажається:
<kbd><img src="/assets/img/posts/2022-09-02-python-flask-3/doc/pic-08.png" /></kbd>
<p style="text-align: center;"><a name="pic-08">pic-08</a></p> 

1. Описуэ запуск контейнера redis
2. Описує запуск контейнера python application



На pic-09  показані основні конфігураційні елементи, пов'язані з конфігуруванням  запуску redis  

<kbd><img src="/assets/img/posts/2022-09-02-python-flask-3/doc/pic-09.png" /></kbd>
<p style="text-align: center;"><a name="pic-09">pic-09</a></p> 

1. Кореневий елемен окрем взятого сервісу, що параметризуємо. В подальшому на нього можна буде посилатися.

2. Вказує на образ, з якого буде створюватися та стартувати контейнер.

3. Вказує на визначення портів в форматі [локальний порт вашого laptop]:[порт контейнера].

4. Вказує на команду, що перекриває комнаду Dockerfile **CMD**  при старті контейнера. В даному випадку ми додаємо додаткові параметри при старті сервісу БД.


На pic-10  показані основні конфігураційні елементи, пов'язані з конфігуруванням  запуску контейнера додатку на Python 

<kbd><img src="/assets/img/posts/2022-09-02-python-flask-3/doc/pic-10.png" /></kbd>
<p style="text-align: center;"><a name="pic-10">pic-10</a></p> 


1. Кореневий елемен окрем взятого сервісу, що параметризуємо. В подальшому на нього можна буде посилатися.

2. Вказує на Dockerfile, з якого буде створюватися  образ та стартувати контейнер. При цьому, **context** вказує на path  до Dockerfile **.** - значить поточний каталог, а **dockerfile** вказує на найменування Dockerfile  в якому описані правила побудови  образу.  

3. Вказує на визначення портів в форматі [локальний порт вашого laptop]:[порт контейнера].

4. Вказує на блок **links**, що так би мовити зв'язує два контейнера між собою так, що вони можуть між собою взаємодіяти по tcp/ip. В даному випадку вказано, що сервіс **smplapp-srvc-redis** може звертатися до сервісу з назвою хоста "redisserver". **:redis** це альтернативна назва.

5. Вказує на блок environments змінних які потрібно задати для старту сервісу.  Цифрою 6 показано, як сконфігуровано підключення до redis.


### Запуск контейнерів docker composer.

Запукаються командою

```bash
docker compose up
```
Але мені потрібно, щоб при запуску сервіс **smplapp-srvc-redis** перебудовував образ, бо я в процесі розробки можу міняти програмний код. Для цього використовується команда: --build smplapp-srvc-redis, що запускає перебудову образа 


``````bash
docker compose up --build smplapp-srvc-redis
```

Зупинити сервыс можна по CTRL + C  або просто: 


```bash

docker compose stop
```
