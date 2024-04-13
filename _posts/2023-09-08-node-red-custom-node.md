---
layout: post
title: "Node-RED How to create custom node"
date: 2023-09-08 10:00:01
categories: [node-RED]
permalink: posts/2023-09-08/node-red-custom-node/
published: true
---

<!-- TOC BEGIN -->

<!-- TOC END -->

## <a name="p-1">Про що цей блог</a>

Працюючи з Node-Red мені знадобилося зробити логування в JSON спеціальної структури. Переглянув я існуючі  node - вони мене не влаштували. Вирішив зробити свою. До цього я раніше їх не робив, але, як виявилося, все не так складно. За 2-3 дні  уже більш-менш запрацювало  
Приклад можна знайти в моєму [github repo](https://github.com/pavlo-shcherbukha/node-red-custom-node.git)
В цьому блозі я ділюся доствідом, як створювати custom node. Фактично, мені прийшлося створити дві Node:
- [конфігураційну  node](https://nodered.org/docs/creating-nodes/config-nodes), як є глобальною для всього flow  файлу
- [функціональну (звичайну)  Node](https://nodered.org/docs/creating-nodes/first-node), але яка використовує параметри конфігураційної Node.
Зрозуміло, що Node [запакована в пакет](https://nodered.org/docs/creating-nodes/packaging), який можна помістити в npm  репозиторій, або у власний репозиторій.

Єдине, що я не зробив, так це не освоїв [unit тестів для нової node](https://nodered.org/docs/creating-nodes/first-node#testing-your-node-in-node-red). Але, сподіваюся я це зроблю в недалекому майбутньому.


По факту, я зробив собі **custom node**, що логує мені  роботу flows в JSON структуру, використовуючи winston логер. Найближчим  до моєї ідеє є [/node-red-contrib-flogger](https://flows.nodered.org/node/node-red-contrib-flogger).

Розібравшись з цим підходом, я  розумію, що в мене не буде великих проблем написати якийсь власний конектор до бази даних чи черги чи якогось іншого сервера. Підходи я відпрацював на цьому простому прикладі.

## <a name="p-2">2. Формалізація задачі та показ результатів, що вийшли</a>

Мені потрібно розробити custon Node, яка буде пропускати повідомлення "через себе" та логувати в файл або в консоль записи у вигляді json, а саме:
- саме повідомлення;
- глобальний контекст;
- flow контекст.
Локальний node-контекст не зможу логувати. Потрібно буде подумати з приводу логування вхідних реквізитів http запиту, но це не в цьому блозі.

Конфігурація Node розділяється на 2 розділи:
- глобальний, тобто налаштування спільні для всіх вузлів логування;
- локальний, тобто налаштування, специфічні для окремо взятого вузла.

До глобальних налаштувань відносяться такі налаштування:

- куди логувати: в файл чи в консоль;
- якщо в файл, то потрібно задати каталог та ім'я файлу логу;
- бажано, щоб таких конфігурацій можна було задати кілька: одну для помилок, другу для нормального потоку операцій

До локальних налаштувань відносяться:
- loglevel (info, debug, error, trace...)
- loglabek - мітка, яка б дала розуміння зякого вузла прийшов запис в лог


Структура лога повинна бути приблизно така:

```json
[
    {
        "hostname": "localhost",
        "label": "InReq",
        "level": "info",
        "message": {
            "flow_context": {},
            "global_context": {},
            "payload": {
                "id": "450"
            }
        },
        "timestamp": "2023-09-08T19:58:35.874874+03:00"
    },
    {
        "hostname": "localhost",
        "label": "GET-USER",
        "level": "info",
        "message": {
            "flow_context": {},
            "global_context": {},
            "payload": {
                "fullname": "Petro Petrenko",
                "id": "150",
                "phone": "222-33-44"
            }
        },
        "timestamp": "2023-09-08T19:59:02.278278+03:00"
    }
]

```

- hostname вказує на адресу віртуалки чи контейнера, де запущено екземпляр flow;
- label вказує на  джерело, відкіль прийшов запис;
- level  вказує на рівень логування;
- message  повідомлення або структура, що логується;
- timestamp  дата та час, коли зроблено запис.


На [pic-01](#pic-01)  показано простенький flow з Noda-ми  логування.

<kbd><img src="/assets/img/posts/2023-09-08-node-red-custom-node/doc/pic-01.png" /></kbd>
<p style="text-align: center;"><a name="pic-01">pic-01</a></p>

А на  [pic-02](#pic-02) показана параметризація вузлів логування. Як видно з малюнка, запараметризовано запис  логів у файли, але помилки пишуться в окремий файл, а інші повідомлення в інший файл. 

<kbd><img src="/assets/img/posts/2023-09-08-node-red-custom-node/doc/pic-02.png" /></kbd>
<p style="text-align: center;"><a name="pic-02">pic-02</a></p>

В самому правому віконці показана конфігурація глобального конфігуратора, коли логування виконується в консоль.

## <a name="p-3">3. Покрокова інструкція, як створювати nodes</a>

Перед тим, як програмувати бізнес логіку, потрібно, на мій погляд, визначитися з параметрами конфігурації та логікою UI, що дозволяє ввести ці конфігураційні параметри. Починати потрібно з цього. Ну а далі показую покроково як я це робив.
### <a name="p-3.1">3.1. Створення порожньої бібліотеки</a>
Створимо вузол **wnode**.
Перше, що робимо, це **npm init**  та вводимо  параметри, що запитуються [pic-03](#pic-03).  У вас створиться порожній **package.json** Тут головне, вказати найменування пакету, ключ **name**.

<kbd><img src="/assets/img/posts/2023-09-08-node-red-custom-node/doc/pic-03.png" /></kbd>
<p style="text-align: center;"><a name="pic-03">pic-03</a></p>

Далі, реєструємо node-red вузол **wnode** та .js файл, що зберігає  бізнес логіку  

```json
  "node-red": {
    "nodes": {
      "wnode": "wnode.js"
    }
  },

```
В результаті отримаємо такий от **package.json**

```json
{
  "name": "wnode",
  "version": "1.0.0",
  "description": "wnode-тестова custom node (demo)",
  "main": "wnode.js",
  "scripts": {
    "test": "test"
  },
  "node-red": {
    "nodes": {
      "wnode": "wnode.js"
    }
  },
  "author": "pasha",
  "license": "ISC"
}

```

Після цього в цьому ж каталозі встановлюємо потрібні залежності стандартною командою  **npm install**. Нижче показано, як я додаю бібліотеку **winston**

```bash
npm install winston

```
Далі створюємо 2 файли:
- wnode.js файл, де знаходиться javascript бізнеслогіка
- wnode.html файл, який описує поведінку UI

#### <a name="p-3.1.1">3.1.1. Порожній node.js файл</a>

Далі показано порожній .js файл в якому існує дві node`s: **WNode** - в якій  буде сконцентрована бізнеслогіка та  **WConfigNode** в якій буде сконцентрована логіка конфігурації. 

```js
    module.exports = function(RED) {
        function WNode(config) {
            var winston = require('./winston');
            
            RED.nodes.createNode(this,config);
            var node = this;
            this.filename = config.filename;
            node.on('input', function(msg) {
                msg.payload = msg.payload
                node.send(msg);
            });
        }

        RED.nodes.registerType("w-node",WNode);

        function WConfigNode(n) {
            RED.nodes.createNode(this,n)
        }

        RED.nodes.registerType("w-config", WConfigNode)

    }

```

Тут треба звернути увагу на такі особливості:

- *function WNode(config){...}*  де сконцентрована вся бізнес логіка;
- *RED.nodes.registerType("w-node",WNode)*  це рядок, що реєструє програмну функцію під ідентифікатором "w-node";
- *RED.nodes.createNode(this,config);* це рядок створеня **node** а в параметрі config передаються параметри конфігурації.

#### <a name="p-3.1.2">3.1.2. Підготовка html файлу конфігураційної Node</a>

Параметр confg наповнюється через html файл, **wnode.html**.  В цьому файлі є два обов'язкових розділи:
- *<script type="text/javascript"></script>* розділ, що описує програмну (реактивну частину UI)
- *<script type="text/x-red" data-template-name="w-config"></script>* розділ, що  описує html частину роботи UI


```html
<script type="text/javascript">
	RED.nodes.registerType('w-config',{

		category: 'config',

		defaults: {
            configname: {value:"", required:true},
            logto: {value:"",required:true},            
			logname: {value:"",required:false},
			logdir: {value:"",required:false},
		},

		label: function() {
			return this.configname
		},

		oneditprepare: function() {
            $("#node-config-input-logto").change(function() {
      			if ($(this).val() === "file") {
                    $("#node-config-fileprop").show()
      			} else {
                    $("#node-config-fileprop").hide()  
                }

		    })
		},

		oneditsave: function() {
		}
	})
</script>

<script type="text/x-red" data-template-name="w-config">
    <div class="form-row">
        <label for="node-config-input-configname"><i class="fa fa-tag"></i>Config name</label>
        <input type="text" id="node-config-input-configname" placeholder="Config name">
    </div>
    <div class="form-row">
		<label for="node-config-input-logto"><i class="fa fa-clock-o"></i> Log to</label>
		<select type="text" id="node-config-input-logto" placeholder="Log to">
			<option value="file">to File</option>
			<option value="console">to console</option>
		</select>
	</div>
    <div id="node-config-fileprop">
        <div id="node-config-input-logname-f" class="form-row">
            <label for="node-config-input-logname"><i class="fa fa-tag"></i> Log file name</label>
            <input type="text" id="node-config-input-logname" placeholder="Log file name">
        </div>
        <div id="node-config-input-logdir-f" class="form-row">
            <label for="node-config-input-logdir"><i class="fa fa-tag"></i> Log dir</label>
            <input type="text" id="node-config-input-logdir" placeholder="Log dir">
        </div>
    </div>    

</script>

```
На прикладі вище, показана конфігураційна NODE. Основна відмінність від звичайної, це наявність рядка:

```text
 	category: 'config',
```

та посиланню на конфігураційні поля передує префікс **node-config-input-**.
Далі, в  роздід **defaults** описуються конфігураційні поля.
В роздіді **oneditprepare** (по суті це реація на event *on_edit_prepare*) можна управляти видимістю заданих на реєстрацію полів. 
Зверніть увагу на div

```html

    <div id="node-config-fileprop">
        <div id="node-config-input-logname-f" class="form-row">
            <label for="node-config-input-logname"><i class="fa fa-tag"></i> Log file name</label>
            <input type="text" id="node-config-input-logname" placeholder="Log file name">
        </div>
        <div id="node-config-input-logdir-f" class="form-row">
            <label for="node-config-input-logdir"><i class="fa fa-tag"></i> Log dir</label>
            <input type="text" id="node-config-input-logdir" placeholder="Log dir">
        </div>
    </div>    

```
та умови при яких яких відображається чи не вібражається даний **div** в залежності від вибраного значення в полі *"node-config-input-logto"* === *defaults.logto*

```js
		oneditprepare: function() {
            $("#node-config-input-logto").change(function() {
      			if ($(this).val() === "file") {
                    $("#node-config-fileprop").show()
      			} else {
                    $("#node-config-fileprop").hide()  
                }

		    })
		},

```

Крім того, слід зауважити, що комбінація defaults.configname та динамічне присвоєння найменування config node *label: function() {return this.configname}* створюють такий ефект, що можна зберегти кілька конфігурацій і, потім,  розробляючи flow можна між ними переключатися.  

```js
		defaults: {
            configname: {value:"", required:true},
		},

		label: function() {
			return this.configname
		},


```

#### <a name="p-3.1.3">3.1.3. Підготовка html файлу основної Node</a>

Розділ для основної node  показано далі. Тут, як бачимо, все дуже схоже до попереднього розділу,

```html
<script type="text/javascript">
    RED.nodes.registerType('w-node',{
        category: 'output',
        color: '#FFAAAA',
        icon: "file.png",
        defaults: {
            name: {value:""},
            loglabel: {value:"",required:true},
            loglevel: {value:"INFO", required: true},
            configname: {type:"w-config"},
        },
        inputs:1,
        outputs:1,
        label: function() {
            return this.name||"w-node";
        }
    });
</script>

<script type="text/html" data-template-name="w-node">
    <div class="form-row">
        <label for="node-input-name"><i class="fa fa-tag"></i> Name</label>
        <input type="text" id="node-input-name" placeholder="Name">
    </div>
    <div class="form-row">
        <label for="node-input-loglabel"><i class="fa fa-tag"></i>Log Label</label>
        <input type="text" id="node-input-loglabel" placeholder="LogLabel">
    </div>
	<div class="form-row">
		<label for="node-input-loglevel"><i class="fa fa-exclamation"></i> Level</label>
		<select type="text" id="node-input-loglevel" placeholder="Log Level">
			<option value="ERROR">ERROR</option>
			<option value="WARN">WARN</option>
			<option value="INFO">INFO</option>
			<option value="DEBUG">DEBUG</option>
			<option value="TRACE">TRACE</option>
		</select>
	</div>
    <div class="form-row">
		<label for="node-input-configname"><i class="fa fa-cogs"></i> Log Config</label>
		<input type="text" id="node-input-configname" placeholder="Log Configuration">
	</div>

   
</script>

```

але є деякі відмінності:

- Категорія вже має інший ідентифікатор:  *category: 'output'*

- В розділі defaults  є посилання на нашу конфігураційну node:  *defaults: {configname: {type:"w-config"}}* і буде поле для її вводу в другому розділі

- Трошки інший алгоритм обчислення label  **label: function() {return this.name||"w-node";}**
- Префікс для конфігураційних полів повинен бути: **node-input-** (без config)
- Ну і тут продемонстровано, як  задавати поля з випадаючим списком (в принципі все по html)

Коли всі ці файли підготовані можна спробувати підключити пакет до Node-RED


### <a name="p-3.2">3.2 Підключеня пакету до node-red</a>

Для цого потрібно повернутися в каталог де інстальовано Node RED  та виконати команду npm install але з посиланням на каталог вашої node ,а не на npm репозиторій:

```bash
npm i /wnode

```

В результаті ваш пакет буде включено в основний **package.json** тільки з ознакою "file:": ** "wnode": "file:wnode",**

```json
{
  "name": "node-red-test01",
  "version": "1.0.0",
  "description": "testing nodre red development and deployment on openshift",
  "main": "index.js",
  "scripts": {
    "dev": "node  node_modules/node-red/red.js --settings ./dev/settings.js --userDir .  --port 1880 --verbose user-registration.json",
    "test": "node  node_modules/node-red/red.js --settings ./test/settings.js --userDir .  --port 8080 --verbose user-registration.json",
    "devloader": "node  node_modules/node-red/red.js --settings ./dev/settings.js --userDir .  --port 1880 --verbose loader.json",
    "testloader": "node  node_modules/node-red/red.js --settings ./test/settings.js --userDir .  --port 8080 --verbose loader.json",
    "start": "node  node_modules/node-red/red.js --settings ./prod/settings.js --userDir . --verbose --port 8080 $FLOW_NAME"
  },
  "author": "psh",
  "license": "ISC",
  "dependencies": {
    "node-red": "^3.0.2",
    "wnode": "file:wnode",
    "wshlog": "file:wshlog"
  }
}


```

Тепер запукаємо наш тестовий flow і дивимося що буде

```bash

npm run dev

```
Flow доступний за URL  http://localhost:1880

Ну а на [pic-04](#pic-04)  показно простенький приклад, що з того всього вийшло

<kbd><img src="/assets/img/posts/2023-09-08-node-red-custom-node/doc/pic-04.png" /></kbd>
<p style="text-align: center;"><a name="pic-04">pic-04</a></p>

Детально з прикладом можна ознайомитися за лінком: [node-red-custom-node.git](https://github.com/pavlo-shcherbukha/node-red-custom-node.git)