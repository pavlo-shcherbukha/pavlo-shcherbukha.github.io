---
layout: post
title: "FABRIC DEVELOPER WIKI"
date: 2026-09-01 10:00:01
categories: [spark, Microsoft Fabric]
permalink: posts/2026-09-01/Fabricdeveloperwiki/
published: true
---

<!-- TOC BEGIN -->

I. СТАНДАРТИ ТА ГОВЕРНАНС (Обов’язково до виконання)

- [1. Налаштування середовища (Environments) — Управління пакетами та Runtime](#p-1")

- [2. Моделювання та проектування БД](#p-2)
- [2.1. Тип числових даних](#p-2.1)
- [2.2. Коментування об'єктів](#p-2.2)
- [2.3 Правила іменування таблиць та ствопців в Lake house](#p-2.3)
- [2.4. Системні поля та ключі (PK, Identity)](#p-2.4)
- [2.5. Секціонування таблиць (Partitioning)](#p-2.5)

- [3. Культура розробки в Notebooks](#p-3)
- [3.1 Типологія ноутбуків (DDL, Manual, Pipeline)](#p-3.1)
- [Ідемпотентність та чистота коду](#p-3.2)
- [3.3. Обробка помилок](#p-3.3)

II. БІБЛІОТЕКА ТЕХНІЧНИХ ПАТТЕРНІВ (Cookbook)

- [4. Робота з файловою системою (OneLake/Files)](#p-4)
- [4.1. Шляхи та маніпуляції з файлами](#p-4.1)
- [4.2. Швидке очищення папок через notebookutils](#p-4.2)

- [5. Оптимізація та Capacity Management](#p-5)
- [5.1. Боротьба з помилками 429/430](#p-5.1)
- [5.2. Вирішення конфліктів запису (ConcurrentAppend)](#p-5.2)

- [6. Просунуті сценарії Spark та SQL](#p-6)
- [6.1. Еволюція схеми (Schema Evolution)](#p-6.1)
- [6.2. Ізоляція даних через проксі-Views](#p-6.2)
- [6.3. Виклик Stored Procedures з Notebook](#p-6.3)
- [6.4. Візуальні кубики VS кодінг в otebooks](#p-6.4)

- [6.5. Шаблони кодінгу в notebook](#p-6.5)

- [7. Тестування та QA](#p-7)
- [7.1. Генерація тестових даних](#p-7.1)
- [7.2. Про моделі промислових пристроїв](#p-7.2)
- [7.3. Про моделі http сервісів](#p-7.3)

III. РЕСУРСИ ТА ПОСИЛАННЯ

- [8. Корисна документація та Medium-блоги](#p-8)

- [9. Оновлення та Roadmap Fabric](#p-9)

<!-- TOC END -->


## <a name="p-1">1. Налаштування середовища (Environments) — Управління пакетами та Runtime</a>

**Fabric Environment** ,  що є еквівалентом **python virtual environment**

Відображається (обведено рамкою)

<kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-03-1.png" /></kbd>
<p style="text-align: center;"><a name="pic-03-1">pic-03-1</a></p>

Краще створити спільне середовище для всього Workspace.

- У лівому меню Fabric натисніть + New (або створіть через Workspace) і виберіть Environment.

<kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-03-2.png" /></kbd>
<p style="text-align: center;"><a name="pic-03-2">pic-03-2</a></p>


- У налаштуваннях середовища перейдіть у вкладку Public Libraries.

- Натисніть Add from PyPI і введіть назву пакету (наприклад, opencv-python або faker для ваших тестових даних).

- Важливо: Натисніть кнопку Save та **обов'язково Publish**. Публікація займає 2-3 хвилини.

- Після цього в налаштуваннях вашого Notebook (вгорі панель "Home") у випадаючому списку Environment виберіть створене вами середовище.

Скріншот чітко показує, що ваш ноутбук зараз підключений до "Workspace default".

Логіка "додати все в дефолт" здається найпростішою, але у Fabric є один нюанс: "Workspace default" — це не зовсім Environment, який можна редагувати так само вільно, як кастомний. Це скоріше "заглушка" або базові налаштування тенанта.

Ось як це працює насправді і чому  не можете просто натиснути "редагувати" на дефолті:

1. Дефолт vs Кастомний Environment

Workspace Default: Це базовий образ Spark з уже встановленими бібліотеками (Pandas, PySpark, тощо). Щоб змінити його "назавжди" для всього воркспейсу, потрібні права адміна Fabric у налаштуваннях самого Workspace (Workspace settings -> Engineering/Science -> Spark settings).

Ваш кастомний Environment (який ви створили): Це повноцінний ізольований об'єкт. Ви можете мати один для IoT (з Faker, paho-mqtt), а інший для ML (з scikit-learn). Він уже включає повний набор пакетів з **Workspace Default**

2. Як зробити ваш Environment "головним" (Важливо для Pipeline)

Щоб ваші Pipeline та Notebook працювали автоматично з вашим пакетом Faker (як на скрині), у вас є два шляхи:

Шлях А: Переключити конкретний Notebook (те, що ви бачите на скрині)

Натисніть на випадаючий список, куди вказує рамка.

Виберіть зі списку ваш створений Environment.

Тепер увага: Коли ви додасте цей Notebook у Pipeline, він "запам'ятає", що йому потрібен саме цей Environment.

Шлях Б: Зробити ваш Environment дефолтним для всього Workspace (Рекомендовано)

Якщо ви хочете, щоб будь-який новий об'єкт одразу бачив ваші пакети:

Перейдіть у Workspace settings (шестерня зліва знизу або в меню воркспейсу).

Не використовуємо Environment Workspace default. ЇЇ можна побачити при створенні notebook

 <kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-04-4.png" /></kbd>
<p style="text-align: center;"><a name="pic-04-4">pic-04-4</a></p>

або в налаштуваннях Workspace створити свою власну або як на малюнку використати env Environment

 <kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-04-5.png" /></kbd>
<p style="text-align: center;"><a name="pic-04-5">pic-04-5</a></p>

Це дозволить встановити свої сторонні бібліотеки та використовувати env-змінні, щоб не хардкодити URL, шляхи до файлів чи інші параметри.

Встановлення  додаткових Python бібіліотек з Pypi-репозиторію  можливе тільк в custom  environment, як показано на малюнку

<kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-04-6.png" /></kbd>
<p style="text-align: center;"><a name="pic-04-6">pic-04-6</a></p>


Лінкт на документацію 

- [External repositories](https://learn.microsoft.com/en-us/fabric/data-engineering/environment-manage-library#external-repositories)

- [Summary of library management best practices](https://learn.microsoft.com/en-us/fabric/data-engineering/library-management#summary-of-library-management-best-practices)


Встановлення звичних env-зміних для всієї custom Fabric environment виконується, як показано на малюнку [pic-04-7](#pic-04-7)

<kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-04-7.png" /></kbd>
<p style="text-align: center;"><a name="pic-04-7">pic-04-7</a></p>

**Увага:** Після внесення змінних не забуваємо натиснути кнопку **"Publish"**

Ключ для внесення:

```text
spark.nodeproperty.env.TELEGRAM_TOKEN
spark.nodeproperty.env.URL

```

Щоб прочитати env-змінні в Notebook використовуємо код:

```py

# Через spark.conf.get
print("Отримуємо через spark.conf.get")
print(spark.conf.get("spark.nodeproperty.env.TELEGRAM_TOKEN"))
print(spark.conf.get("spark.nodeproperty.env.URL"))

```

Підтвердження, що такий підхід працює показано на [pic-04-8](#pic-04-8) 

<kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-04-8.png" /></kbd>
<p style="text-align: center;"><a name="pic-04-8">pic-04-8</a></p>


У розподілених системах, таких як Apache Spark, різниця між «локальною зміною оточення» та «конфігурацією кластера» є критичною. Тому os.getenv() не працює і боротися за os.getenv() у контексті Fabric/Spark справді не варто.

Коли ви виконуєте import os; os.environ[...] = ... у клітинці ноутбука, ви змінюєте оточення лише на Driver node (вузлі-керівнику).

Якщо ваш код виконує трансформації локально (наприклад, підключення до API в циклі на драйвері), os.getenv спрацює.

Але якщо ви захочете використати цей секрет всередині Spark udf (User Defined Function) або при паралельному зчитуванні даних, ваші Executors (робочі вузли) про цю змінну нічого не знатимуть.

Варіант зі spark.conf гарантує, що конфігурація прокидається через SparkContext на всі вузли кластера автоматично.


## <a name="p-2">2. Моделювання та проектування БД</a>

У Fabric ми не створюємо бази даних через CREATE DATABASE. Кожен Lakehouse — це і є наша база даних. Якщо треба розділити логіку (наприклад, для платежів), створюємо новий Lakehouse trm_payment_lh і додаємо його в Explorer ноутбука."

Варіант Б: Кілька Lakehouse (Архітектурний)

Ви створюєте окремий об'єкт Lakehouse під назвою trm_payment_lh.

Кожен Lakehouse має свій SQL Endpoint. Ви можете давати права доступу колегам на весь "платіжний" Lakehouse окремо від інших даних.

Як звернутися з одного ноутбука до іншого: Ви просто додаєте обидва Lakehouse до вашого ноутбука (кнопка Add data items зліва на вашому скрині). Після цього ви можете звертатися до них через повне ім'я:

```py
SELECT * FROM psh_exch_lh.table1
UNION 
SELECT * FROM trm_payment_lh.table2
```
В даному прикладі **psh_exch_lh** та **trm_payment_lh**  - є Lakehouse.

Якщо ви хочете побачити, де ви зараз "знаходитеся", виконайте:

```py

print(spark.catalog.currentDatabase())
```

### <a name="p-2.1">2.1 Тип числових даних</a>

Оскільки працюємо з фінансами (amount, charge), при завантаженні, Spark може визначити їх як double. В облікових задачах **краще одразу приводити їх до DecimalType(18,2)**, щоб уникнути проблем з плаваючою комою, можемо вионувати приведення типів як в прикладі:

```py

df1 = spark.read.json(f"{file_pth}/{file_name}")
df_fixed = df1.withColumn("amount", col("amount").cast("decimal(18,2)")) \
              .withColumn("charge", col("charge").cast("decimal(18,2)"))

```

Розбір можливих причини (Overflow)

У JSON значення "charge": 510.22.
Нехай визначення в DDL: CHARGE DECIMAL(3, 2).

- Перша цифра (Precision = 3): Це загальна кількість цифр у числі (і до, і після коми).

- Друга цифра (Scale = 2): Це кількість знаків після коми.

- Результат: DECIMAL(3, 2) дозволяє зберігати числа лише від -9.99 до 9.99.

Число 510.22 потребує як мінімум DECIMAL(5, 2) (3 цифри до коми + 2 після). 

Для фінансових задач на виробництві краще брати запас, щоб не перестворювати таблиці щотижня. Оптимально використовувати DECIMAL(18, 2) (класика для грошей) або хоча б (10, 2).

Виправлений DDL:

```py
spark.sql("""
    CREATE TABLE TERM_PAYMENTS (
        PAYMENT_ID    BIGINT,
        TX_ID         STRING,
        TERMINAL_ID   STRING,
        OPDATE        DATE,  -- Виправив опичатку з OPATE
        NAME          STRING,
        PASSPORT      STRING,
        AMOUNT        DECIMAL(10, 2), -- до 99,999,999.99
        CHARGE        DECIMAL(10, 2), -- до 99,999,999.99
        STATUS        STRING,
        FILE_NAME     STRING
    )
    USING DELTA
""")

```

### <a name="p-2.2">2.2. Коментування об'єктів</a>


1. Коментування стовпців та таблиць при створенні таблиць.

При ствренні таблиці Lake house зразу є можливість додавати назви стовпців та таблиць. Це не стосується Fabric DWH.

```py
spark.sql(
    """
        CREATE TABLE sensors.sensor_telemetry_m2 (
            sensor_id     STRING      COMMENT "ID датчика (напр., 'TMP-REQ-001')", 
            facility_id   STRING      COMMENT "Склад / Цех",
            equipment_id  STRING      COMMENT "Холодильна камера / Термостат",
            timestamp     TIMESTAMP   COMMENT "Час зняття показника",
            temperature   DOUBLE      COMMENT "Значення температури",
            humidity      DOUBLE      COMMENT "Вологість (часто йде в парі у фармі)",
            status        STRING      COMMENT "'OK', 'ALARM', 'MAINTENANCE'",
            date          DATE        COMMENT "Колонка для партиціювання"
        )
        USING DELTA
        PARTITIONED BY (date) COMMENT "Датчики модель 2"
    """
)

```

1. Коментування стовпців та таблиць для вже існуючих об'єктів.

**Додавання зміна коментаря для існуючих таблиць Lake house**

```py

ALTER TABLE clients SET TBLPROPERTIES ('comment' = 'Новий опис: Таблиця містить верифіковані дані клієнтів банку');
```

Або простіший синтаксис (залежно від версії Spark, але цей надійніший):

```py
 COMMENT ON TABLE clients IS 'Ця таблиця зберігає історію транзакцій клієнтів';
```

**Додавання зміна коментаря для стовпців існуючих таблиць Lake house**

```py
ALTER TABLE clients 
ALTER COLUMN email COMMENT 'Електронна пошта (обов’язково у форматі name@domain.com)';
```

Важливо: При зміні коментаря колонки тип даних (STRING, BIGINT тощо) залишається незмінним, змінюється лише метадата.

**Перевірка результату коментування**

Щоб переконатися, що коментарі успішно додані, використовуйте команду DESCRIBE:

- Коротко: DESCRIBE clients (покаже колонки, типи та коментарі до колонок).

- Детально: DESCRIBE TABLE EXTENDED clients (покаже коментар до всієї таблиці в секції Detailed Table Information).

<kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-03-3.png" /></kbd>
<p style="text-align: center;"><a name="pic-03-3">pic-03-3</a></p>



### <a name="p-2.3">2.3. Правила іменування (Таблиці, Стовпці, Views)</a>



#### <a name="p-2.3.1">2.3.1 Правила іменування таблиць</a>

Назви таблиць виконуються в нижньому регістрі (snake_case): brz_customer_orders, brz_product_categories і виконуються тільки латиницею.

- brz_ (Bronze) — сирі, незмінені дані (наприклад, brz_sap_orders).

- slv_ (Silver) — очищені, дедупліковані дані (slv_orders).

- gld_ (Gold) — агреговані дані, готові до звітів (gld_sales_monthly).

- Таблиці фактів закінчуються на _fact, наприклад, gld_sales_fact_, gld_inventory_fact).

- Таблиці вимірів закінчубться на _dim (наприклад, gld_customer_dim, gld_product_dim).

#### <a name="p-2.3.2">2.3.2. Правила іменування стовпців</a>

Назви таблиць виконуються в нижньому регістрі (snake_case) і виконуються тільки латиницею.

- ID / Ключі: Завжди використовуйте _id або _key (наприклад, customer_id, product_key).

- Дати: Використовуйте _dt для дат (order_dt) та _ts  для timestamp  (created_dt, loading_ts).

- Прапорці (Boolean): Починайте з is_ або has_ (is_active, has_discount).

- Метрики / Кількість: Використовуйте amt (amount), qty (quantity), pct (percentage) або total_ (total_revenue, item_qty).

- не можна створювати дві колонки, які відрізняються лише регістром (наприклад, ID та id).

- Заборонені символи в назвах таблиць та колонок: ,;{}()=!? та пробіли. Викороистовуємо тільки ASCII  символи.

#### <a name="p-2.3.3">2.3.3. Правила іменування views</a>

- Назви View повинні відповідати бізнес-логіці, описаній у п. [2.3.1](#p-2.3.1), але можуть містити префікс v_ для відрізнення від фізичних таблиць.

- Звичайні view, що побудовані на основі одної таблиці  утворюються з назви таблиці шляхом додавання v_ ( v_gld_customer_dim)

### <a name="p-2.4">2.4. Системні поля та ключі (PK, Identity)</a>


Для аудиту та дебагінгу в Lakehouse (особливо в шарах Bronze та Silver) рекомендується додавати системні колонки. Варто визначити їх іменування:

- _load_ts — час завантаження рядка.

- _source_file — назва файлу-джерела (ви це вже використовували в прикладах коду).

- _is_deleted — прапорець м'якого видалення.

Використання підкреслення на початку (_) візуально відокремлює технічні поля від бізнес-даних

#### <a name="p-2.4.1">2.4.1. Іменування ключів (Primary Key / Surrogate Key)</a>

Оскільки в Lakehouse часто використовуються синтетичні ключі (Surrogate Keys), використовуємо такий підхід:

- Первинний ключ (PK): Рекомендується використовувати найменування таблиці та суфікс   _pk. Наприклад: brz_customer_pk.

- Бізнес-ключ (BK): Якщо  зберігаєvj оригінальний ID з джерела (наприклад, з SAP чи Salesforce), додаємо префікс джерела. Наприклад: sap_order_id.

- Зовнішній ключ (FK): Повинен називатися точно так само, як PK у батьківській таблиці. Це значно спрощує авто-зв'язування (Auto-detect relationships) у Power BI.

PK/FK — це підказка для Power BI: Вони не блокують INSERT дублікатів, але допомагають Power BI автоматично будувати зв'язки в режимі Direct Lake.

<a name="IDENTITY">Суррогатні ключі</a>: Для сурогатних ключів (SK) рекомендується використовувати механізм GENERATED ALWAYS AS IDENTITY. Це забезпечує автоматичну генерацію унікальних ID на рівні Delta-протоколу.

Важливо: Не розраховуйте на сувору послідовність чисел (1, 2, 3...) — через паралелізм Spark можливі "дірки" в нумерації. Це не є помилкою і допустимо для архітектури Lakehouse.

**Синтаксис створення (SQL)**:
Найпростіший спосіб — визначити стовпець при створенні таблиці через SQL-скрипт у Fabric:

```py
CREATE TABLE brz_sap_orders (
    order_sk BIGINT GENERATED ALWAYS AS IDENTITY (START WITH 1 INCREMENT BY 1),
    order_id INT,
    order_date DATE,
    customer_id INT
)
```

- GENERATED ALWAYS: Spark сам генерує значення. Ви не можете вставити своє значення в цей стовпець (це гарантує цілісність).

- GENERATED BY DEFAULT: Дозволяє системі генерувати ключ, але якщо ви передасте своє значення, воно буде збережене (корисно при міграції історичних даних).

**Використання в PySpark**:

Якщо ви створюєте таблиці через DataFrame API, ви можете використовувати метод withColumn з функціями, але IDENTITY краще визначати саме на рівні схеми таблиці (через spark.sql), щоб Delta Lake самостійно керувала лічильником.

**Гарантія унікальності, але не послідовності**: В розподілених системах (Spark) IDENTITY гарантує, що кожен ID буде унікальним, але вони можуть мати пропуски (наприклад, 1, 2, 5, 6, 10). Це відбувається через паралельну обробку даних різними вузлами. Для сурогатного ключа це нормально.

**Тип даних**: Завжди використовуйте BIGINT для IDENTITY стовпців. У великих Lakehouse-таблицях звичайний INT може швидко закінчитися.

**Відсутність UPDATE для IDENTITY**: Ви не можете змінити значення в стовпці GENERATED ALWAYS після того, як рядок було вставлено.

**Як поводиться стовпець IDENTITY під час виконання операції MERGE**:

1. Автоматичне ігнорування при вставці (INSERT)

Коли ви виконуєте WHEN NOT MATCHED THEN INSERT *, вам не потрібно (і не можна) вказувати стовпець IDENTITY. Delta Lake сама побачить, що в цільовій таблиці є автоінкремент, і згенерує нове значення для нових рядків.

```py
MERGE INTO gld_customers AS target
USING stg_customers AS source
ON target.customer_bk = source.customer_bk
WHEN MATCHED THEN
  UPDATE SET target.customer_name = source.customer_name -- Стовпець SK тут не чіпаємо!
WHEN NOT MATCHED THEN
  INSERT (customer_bk, customer_name) -- IDENTITY стовпець (customer_sk) пропускаємо
  VALUES (source.customer_bk, source.customer_name);

```

2. Заборона на оновлення (UPDATE)

Це головний "запобіжник". Якщо ви спробуєте зробити UPDATE SET target.identity_col = ..., Spark видасть помилку. IDENTITY стовпець за визначенням є незмінним (immutable) для існуючого рядка. Це гарантує, що ваші зв'язки (FK) в інших таблицях не "зламаються".

3. Identity та Schema Evolution

Якщо ви використовуєте опцію option("mergeSchema", "true") у PySpark під час мерджу, Spark може автоматично додавати нові стовпці, але він ніколи не змінить тип існуючого IDENTITY стовпця і не спробує перетворити звичайний стовпець на IDENTITY "на льоту". Це захищає структуру вашого Gold-шару.


Для міграції (Migration Mode): Якщо ви переносите дані зі старої системи і хочете зберегти старі ID, використовуйте GENERATED BY DEFAULT AS IDENTITY. Тоді MERGE дозволить вставити ваші значення. Якщо ж стовпець GENERATED ALWAYS, вставити своє значення не вдасться навіть через MERGE.

Продуктивність: Використання IDENTITY у MERGE практично не впливає на швидкість, оскільки генерація значень відбувається на рівні запису файлів Parquet.

Помилка "Casting": Іноді при MERGE виникає помилка, якщо типи даних у source та target відрізняються (наприклад, INT vs BIGINT). Оскільки IDENTITY завжди рекомендується як BIGINT, переконайтеся, що вхідні дані для бізнес-ключів також мають сумісні типи.

#### <a name="p-2.4.2">2.4.2. Обмеження (Constraints)</a>

Це найважливіша частина для розробника у Fabric, бо Delta Lake в Fabric не перевіряє цілісність даних на запис так, як SQL.

>Informational Constraints: У Fabric Lakehouse обмеження PRIMARY KEY та FOREIGN KEY є інформаційними. Це означає, що система вірить вам на слово, що дані унікальні. Якщо ви запишете дублікати, Fabric не видасть помилку, але звіти в Power BI (Direct Lake) можуть показувати неправильні цифри.

- NOT NULL: Це єдине обмеження, яке реально перевіряється Spark-двигуном при записі.

- Завжди виконуйте дедуплікацію даних у Notebook (через dropDuplicates або merge), перш ніж записувати їх у таблицю з визначеним PK.


### <a name="p-2.5">2.5. Секціонування (Partitioning)</a>

Секціонування розподіляє дані по окремих папках на основі значень одного або кількох стовпців. Це дозволяє двигуну (Spark або SQL) пропускати непотрібні файли при запитах (Partition Pruning).

1. Коли варто секціонувати? 

У Fabric діє правило: "Менше — це більше".

- Розмір таблиці: Починайте думати про секціонування, лише якщо обсяг даних у таблиці перевищує 10–20 ГБ. Не секціонуйте таблиці обсягом менше 10 ГБ. Це створює занадто багато метаданих для OneLake.

- Малі таблиці: Якщо таблиця менша за 1 ГБ, секціонування лише сповільнить роботу через надмірну кількість дрібних файлів (проблема "Small Files Problem").

- Ідеальна кількість секцій — від 10 до 1000. Якщо секцій стає більше 10 000, продуктивність SQL Endpoint різко падає.


- V-Order: Пам'ятайте, що Fabric автоматично застосовує V-Order (оптимізацію стиснення Parquet). Вона часто працює настільки ефективно, що таблиця на кілька мільйонів рядків чудово працює і без секціонування.

- Z-Order vs Partitioning: Для колонок, які часто використовуються у фільтрах (наприклад, product_id), замість секціонування краще використовувати Z-Order (оптимізацію всередині файлів), оскільки вона не створює зайвих папок.

2. Вибір стовпця для секціонування

Правильний вибір стовпця — запорука продуктивності:

- Низька кардинальність (Low Cardinality): Стовпець повинен мати обмежену кількість унікальних значень (зазвичай до 1000).

- Типові кандидати: Year, Month, Region, Category.

- Погані кандидати: Timestamp, ID, Email. Секціонування за TransactionID створить мільйони папок, що призведе до деградації OneLake.

3. Рекомендований підхід до дат

Ніколи не секціонуйте за повним timestamp. Якщо потрібно секціонувати за часом, створіть обчислювальні стовпці:

- load_date (рік-місяць-день)

- load_year_month (наприклад, 202405) — найкращий варіант для більшості великих таблиць.

4. Синтаксис (PySpark)

При створенні таблиці в Notebook використовуйте метод .partitionBy():

```py

df.write.format("delta").partitionBy("order_date").saveAsTable("fact_sales")
```

```py
spark.sql("""
                CREATE TABLE sensors.sensor_telemetry (
                    sensor_id     STRING      COMMENT "ID датчика (напр., 'TMP-REQ-001')", 
                    facility_id   STRING      COMMENT "Склад / Цех",
                    equipment_id  STRING      COMMENT "Холодильна камера / Термостат",
                    timestamp     TIMESTAMP   COMMENT "Час зняття показника",
                    temperature   DOUBLE      COMMENT "Значення температури",
                    humidity      DOUBLE      COMMENT "Вологість (часто йде в парі у фармі)",
                    status        STRING      COMMENT "'OK', 'ALARM', 'MAINTENANCE'",
                    date          DATE        COMMENT "Колонка для партиціювання"
                )
                USING DELTA
                PARTITIONED BY (date) COMMENT "Датчики"
          """
)

```

5. Перевірка структури в OneLake


Після секціонування ваша таблиця в провіднику OneLake буде виглядати як ієрархія папок:
fact_sales/Year=2024/Month=05/part-0001-xyz.parquet.

Перевірка цілісності Delta-архіву,  як Delta Lake почувається з файлами на диску. Викорстаємо команду, щоб подивитися на фізичні файли:

```py
import os

# через звичайний Python, якщо Fabric то msutils.fs.ls :
path = "lab_11/lakehouse/sensors.db/sensor_telemetry"
print(f"Кількість партицій (днів): {len([d for d in os.listdir(path) if 'date=' in d])}")

> Кількість партицій (днів): 16

```

6. Пам'ятайте:

Після секціонування ваша таблиця в провіднику OneLake буде виглядати як ієрархія папок:
fact_sales/Year=2024/Month=05/part-0001-xyz.parquet

Unique Constraints + Partitioning: Пам'ятайте, що якщо ви вказуєте стовпці як PRIMARY KEY (інформаційні), вони обов'язково повинні включати стовпці секціонування, щоб Power BI та SQL Endpoint могли коректно працювати з метаданими.

Over-partitioning: Застережіть розробників: "Секціонування таблиці на 100 МБ — це технічний борг, а не оптимізація".

Maintenance: Використовуйте команду OPTIMIZE (у Fabric це робиться автоматично в межах обслуговування Delta), щоб групувати дрібні файли всередині секцій.


## <a name="p-3">3. Культура розробки в Notebooks</a>

Щоб  було легко працювати в команді і швидко розбиратися в чудому коді оформлення NoteBook повинно теж регулватися  деякими правилами. Будемо розрізнятик кілька типів Notebook і їх бажано не змішувати. Зважаючи на те, що Notebooks прекрасно рендерять комірки типу "Markdown" - то notebooks потрібно забезпечувати коментарями та описами, бажано з номерованими кроками. Такми чином, автоматично будується і відображається зміст Notebook в Fabric, що забезпечує швидку навігацію.

### <a name="p-3.1">3.1. Типологія ноутбуків (DDL, Manual, Pipeline)</a>

1. Notebook для оформлернрня DDL-скриптів, що створюють об'єкти в Lake house чи в Fabric DWH

Використовуються для створення об'єктів в базах даних.

2. Notebook для ручного використання.

Використовуються для тестування, побудови якихось тимчасових звітів, прототипування чи виконання якихось разових чи тимчастових задач.

3. Notebook для використання в pipelines.

З назви зрозуміло шо ці Notebooks використовуються в інтеграційних сценаріях. 

### <a name="p-3.2">3.2. Ідемпотентність та чистота коду</a>

Хоча Notebook дозволяє змішувати мови, використовуючи "магічні" команди (%%sql, %%pyspark), для командної роботи краще визначити пріоритет: **"Один ноутбук — одна мова"**. 

Якщо це ETL-процес на PySpark, потрібно уникати вкраплень SQL-комірок всередині, оскільки це ускладнює дебагінг через змінні.

#### <a name="p-3.2.1"> 3.2.1. Загальні вимоги до чистоти коду (Clean Code)</a>

**Коментарі**: Використовуйте Markdown-комірки для опису логіки блоків, а не лише коментарі # всередині коду. Ноутбук має читатися як технічна стаття.

**Очищення ресурсів**: В кінці ноутбука обов'язково зупиняйте тимчасові View (spark.catalog.dropTempView).

**Вивід даних**: Видаляйте df.show() або display(df) або print("....") з фінальних версій ноутбуків для пайплайнів, щоб не засмічувати логи та не витрачати ресурси на рендеринг таблиць.

**Імпорт пакетів**:

Імпорт необхідних пакетів потрібно виносити в одно з перших кормірок, так notebook працює стабільніше.

**Використання бібліотеки фукнцій**

Якщо потрібно створити якусь бібліотеку фукнцій, яка потім буде викликатися в інших комірках, то функції вписуються в окремій комірці.


#### <a name="p-3.2.2"> 3.2.2. Вимоги до "DDL" Notebook (Тип 1)

**Ідемпотентність**: Кожен скрипт повинен бути написаний так, щоб його можна було запустити повторно без помилок (використання CREATE TABLE IF NOT EXISTS).

**Версійність**: Додавайте в першій комірці коментар з датою створення та автором, або посилання на таску в Jira/Azure DevOps.

Якщо створюємо структуру таблиць для Lake house - то використовуємо SPARK SQL. На приклад:

**Варыант 1:**
```py
spark.sql("""
    CREATE TABLE IF NOT EXISTS psh_dwh_lh.dbo.ev2_silver_customers_dim (
            cust_id      STRING COMMENT 'Унікальний ID клієнта',
            cust_name    STRING COMMENT 'Наіменування клієнта',
            cust_doc     STRING COMMENT 'Документ клієнта',
            cust_status  STRING COMMENT 'Статус клієнта',
            upload_id    STRING COMMENT 'Унікальний id завантаження в систему',
            idt          TIMESTAMP COMMENT 'Дата та час завантаження в систему
    ) USING DELTA;
""")
``` 

**Варіант 2:**

```py
print(f"Create Table psh_dwh_lh.dbo.ev2_silver_customers_dim")

spark.sql("DROP TABLE IF EXISTS psh_dwh_lh.dbo.ev2_silver_customers_dim")

spark.sql("""
    CREATE TABLE psh_dwh_lh.dbo.ev2_silver_customers_dim (
            cust_id       STRING,
            cust_name     STRING,
            cust_doc      STRING,
            cust_status   STRING,
            upload_id     STRING,
            idt           TIMESTAMP
    )  USING DELTA;  
"""
)

print(f"Table created")

```

При цьому слід звернути увагу на нотацію, де точно вказується назва Lake house, що робить Notebook не залежним від поточного підключення.

Якщо створюємо структуру таблиць для Fabric DWH - то використовуємо комірки notebook T-SQL. І пишемо вже як звичайний  SQL - скрипт в форматі T-SQL.  

Треба уникати автоматичної зміни схеми (Schema Evolution). Краще використовувати ручне управлыння схемам. На приклад:

```sql
ALTER TABLE psh_dwh_lh.dbo.ev2_silver_customers_dim 
ADD COLUMNS (cust_email STRING COMMENT 'Пошта клієнта для розсилок');

```

Уникати автоматичної еволюції  потрібно хоча б тому, щоб Spark не "фантазував" з приводу типів стовпців. Якщо ж є підстави для автоматичної еволюції схеми, то це виконується таким чином: 

```py
# Додавання опції mergeSchema дозволяє Spark додати нові колонки з DataFrame у таблицю
df.write.format("delta") \
    .mode("append") \
    .option("mergeSchema", "true") \
    .saveAsTable("psh_dwh_lh.dbo.ev2_silver_customers_dim")
```

Для Gold-шару краще використовувати ALTER TABLE замість автоматичної еволюції схеми - завжди.


**Оптимізація та обслуговування (Maintenance)**:

Оскільки ноутбуки створюють Delta-таблиці, варто мати періодичне обслуговування. Наприклад, раз на тиждень запускати ноутбук типу 1 (DDL/Admin), який виконує:

- OPTIMIZE (для групування дрібних файлів);

- VACUUM (для видалення старих версій файлів та економії місця в OneLake).

```py

# Приклад для обслуговування таблиць
spark.sql("OPTIMIZE psh_dwh_lh.dbo.ev2_silver_customers_dim")
spark.sql("VACUUM psh_dwh_lh.dbo.ev2_silver_customers_dim RETAIN 168 HOURS") # зберігаємо історію за 7 днів

```

#### <a name="p-3.2.3"> 3.2.3. Вимоги до "ручних" Notebook (Тип 2)

Назви ручних ноутбуків повинні починатися з префікса  dev_ та містити ім'я розробника. Це допоможе не плутати їх з продуктовими скриптами в загальному переліку Lakehouse. І кразе їх розміщати в спеціальній папці Workspace **DEV_TOOLS**. І в таких notebooks теж повинен бути опис.


#### <a name="p-3.2.4"> 3.2.4. Вимоги до Notebooks для використання в pipelines (Тип 3)

Оскільки ці ноутбуки є частиною автоматизації, вони повинні мати сувору структуру комірок:

- Комірка параметрів (Parameters Cell): Обов'язкове використання тегу parameters для передачі значень з Data Factory чи з Pipeline чи з попередьої activity (наприклад, load_date, source_system).

- Комірка ініціалізації: Підключення до Lakehouse, імпорт стандартних бібліотек (pyspark.sql.functions).

- Комірка логування: Початок запису логів (ви згадували ELK/Azure Monitor — тут саме місце для ініціалізації логера).

- В Pipeline-ноутбуках завжди тримайте mergeSchema вимкненим (Schema Evolution вимкнена), окрім випадків, коли ви свідомо очікуєте динамічну структуру від джерела. Це запобігає появі "сміттєвих" колонок через помилки в коді.


### <a name='3.3'>3.3. Обробка помилок</a>

- В обчислювальних комірках повинні використовуватися блоки try.... exept. Повинна зразу розроблятися стратегія  error handling (обробку помилок) у ноутбуках для пайплайнів. Кожний notebook повинен повертати в pipline  значеня результату роботи Notebook використовуючи **mssparkutils.notebook.exit**:

```py
# return result frimom notebook  
mssparkutils.notebook.exit(json.dumps(result_data))
```
А в pipeline результат треба правильно обробляти, викороистовуючи  вираз: @activity('NotebookName').output.result.exitValue.

- Timeout: Стандартний таймаут для ноутбуків у пайплайнах  виьираємо 2 години, щоб у разі "зациклення" вони не спалювали ресурси капасіті (CU) безкінечно.


## <a name="p-4">4.Робота з файловою системою (OneLake/Files)</a>

Пам'ятай: 

- Якщо пишеш дані через Pandas (to_csv, to_json), завжди додавай префікс /lakehouse/default/ . 

- Якщо через Spark (df.write), можна використовувати прямий шлях Files/... або Tables/...."

1. Якщо ви використовуєте pandas.

Pandas — це стандартна бібліотека Python, вона не знає про "магію" Lakehouse автоматично. Для неї потрібно вказувати шлях через локальну точку монтування /lakehouse/default/.... .

```py

# Для Pandas/OS шлях має починатися з кореневого каталогу Fabric
output_path = f"/lakehouse/default/Files/bronze/{file_name}"

# Тоді запис спрацює:
df.to_json(output_path, orient='records', lines=True)

```

2. Якщо ви використовуєте spark

Spark у Fabric вже "знає", де знаходиться його дефолтний Lakehouse, тому він сприймає відносні шляхи.

```py
output_path = f"Files/bronze/{file_name}"
# Або через ABFSS (найбільш надійний варіант для Cloud)
# abfss://workspace_id@onelake.dfs.fabric.microsoft.com/lakehouse_id/Files/bronze/...

```

Чому це важливо: Це треба знати, щоб не витрачати час на пошук помилки  "No such file or directory".

Якщо ви залишите просто Files/bronze/, Pandas спробує створити папку в тимчасовій робочій директорії самого вузла (node), де виконується ноутбук. Після завершення сесії ці дані просто зникнуть, і ви не побачите їх у провіднику (Explorer) зліва.

Порада для перевірки:
Додайте цей блок на початку вашого генератора, щоб переконатися, що папка існує (Pandas сам її не створить):

```py
import os

bronze_path = "/lakehouse/default/Files/bronze"
if not os.path.exists(bronze_path):
    os.makedirs(bronze_path)
    print(f"Папку {bronze_path} створено.")
```

### <a name="p-4.1">4.1 Шляхи та маніпуляції з файлами</a>

**mssparkutils.fs** являє собою пакет служюових програм для роботи з різними файловими системами, включаючи Azure Data Lake Storage (ADLS) 2,  BLOB-Storage Azure. 

Детально можна прочитати тут: [Служебные программы файловой системы](https://learn.microsoft.com/ru-ru/fabric/data-engineering/microsoft-spark-utilities).


Чому mssparkutils.fs кращий за стандартні методи:

- Масштабованість: Він розуміє, що файл лежить не на жорсткому диску, а в розподіленому сховищі.

- Швидкість: Операції з копіювання або видалення відбуваються на рівні інфраструктури Azure/Fabric, а не через прошарок Python-драйвера.

- Зручність: Вам не потрібно прописувати складні рядки підключення або токени доступу — він автоматично використовує права користувача, під яким запущено Notebook.

**Отримати інформацію про доступні функції**

```py

from notebookutils import mssparkutils
mssparkutils.fs.help()

```

Відовідь буде приблизно така:

```text
mssparkutils.fs provides utilities for working with various FileSystems.

Below is overview about the available methods:

cp(from: String, to: String, recurse: Boolean = false): Boolean -> Copies a file or directory, possibly across FileSystems

mv(from: String, to: String, recurse: Boolean = false): Boolean -> Moves a file or directory, possibly across FileSystems

ls(dir: String): Array -> Lists the contents of a directory

mkdirs(dir: String): Boolean -> Creates the given directory if it does not 
exist, also creating any necessary parent directories

put(file: String, contents: String, overwrite: Boolean = false): Boolean -> Writes the given String out to a file, encoded in UTF-8

head(file: String, maxBytes: int = 1024 * 100): String -> Returns up to the first 'maxBytes' bytes of the given file as a String encoded in UTF-8

append(file: String, content: String, createFileIfNotExists: Boolean): Boolean -> Append the content to a file

rm(dir: String, recurse: Boolean = false): Boolean -> Removes a file or directory

exists(file: String): Boolean -> Check if a file or directory exists

mount(source: String, mountPoint: String, extraConfigs: Map[String, Any]): Boolean -> Mounts the given remote storage directory at the given mount point

unmount(mountPoint: String): Boolean -> Deletes a mount point

mounts(): Array[MountPointInfo] -> Show information about what is mounted

getMountPath(mountPoint: String, scope: String = ""): String -> Gets the local path of the mount point

Use mssparkutils.fs.help("methodName") for more info about a method.

```

### <a name="p-4.2">Швидке очищення папок через notebookutils</a>

```py
# Шлях до вашої папки
path = "Files/bronze"

# Видаляємо папку рекурсивно (true означає видалити і всі вкладені файли/папки)
mssparkutils.fs.rm(path, True)

# Створюємо порожню папку назад
mssparkutils.fs.mkdirs(path)

print(f"Каталог {path} очищено.")

```

#### <a name="p-4.2.1">Видалення лише вмісту (якщо треба зберегти папку)</a>

```py

path = "Files/bronze"

# Отримуємо список усіх файлів у папці
files = mssparkutils.fs.ls(path)

for f in files:
    mssparkutils.fs.rm(f.path, True)

print(f"Всі файли в {path} видалено.")

```

Що ще корисного вміє цей пакет?

Окрім видалення файлів, ось топ-функцій, які вам точно знадобляться в роботі з Lakehouse:

- Перегляд вмісту папки:
    mssparkutils.fs.ls("Files/bronze") — повертає список об'єктів з метаданими (розмір, назва, шлях).

- Копіювання та переміщення:
    mssparkutils.fs.cp(source, destination, True) — копіює файли або цілі папки.

- Робота з секретами:
    mssparkutils.credentials.getSecret("KeyVaultName", "SecretName") — якщо вам потрібно під'єднатися до зовнішньої бази даних, не засвічуючи пароль у коді.

- Допомога:
    Ви завжди можете викликати mssparkutils.fs.help(), щоб побачити список усіх доступних команд прямо в блокноті.


- Видалення файлів, тільки *.csv

```py
path = "Files/bronze"
target_extension = ".csv"

# Отримуємо список об'єктів
files = mssparkutils.fs.ls(path)

for f in files:
    # f.name — це ім'я файлу, f.path — повний шлях
    if f.name.lower().endswith(target_extension):
        print(f"Видаляю: {f.name}")
        mssparkutils.fs.rm(f.path)

print("Очищення завершено.")
```

**Трюк із динамічним створенням папок (Mounting)**

Ще одна крута фішка mssparkutils — це можливість монтування (mount). Ви можете під’єднати зовнішній Azure Blob Storage або ADLS Gen2 прямо як локальну папку в Lakehouse. Це дозволяє працювати з гігабайтами даних у хмарі так само легко, як із папкою Files/bronze.
Чому це важливо для "Bronze" шару?

Зазвичай архітектура Medallion (Bronze -> Silver -> Gold) працює так:

- Bronze: Сюди ми "скидаємо" все як є (Raw data).

- Silver: Тут ми читаємо ці файли через Spark, чистимо їх і записуємо вже в каталог Tables як Delta-таблиці.

- Gold: Агреговані дані для звітів.

Використання mssparkutils.fs.rm() у папці Bronze — це стандартна практика для очищення "сирих" даних після того, як вони успішно пройшли обробку і перетворилися на таблиці в Silver-шарі.

Як перевірити, скільки місця займає ваша папка Files/bronze перед тим, як її видаляти? Можу підказати короткий скрипт для цього.

**Скрипт для розрахунку розміру папки**

Цей код рекурсивно проходить по папці та виводить загальний розмір у зручному форматі (МБ або ГБ):

```py
def get_dir_size(path):
    total_size = 0
    try:
        files = mssparkutils.fs.ls(path)
        for f in files:
            if f.isDir:
                # Якщо це підпапка, викликаємо функцію рекурсивно
                total_size += get_dir_size(f.path)
            else:
                total_size += f.size
        return total_size
    except:
        return 0

path = "Files/bronze"
size_bytes = get_dir_size(path)

# Конвертація для зручності клієнта
size_mb = size_bytes / (1024 * 1024)
size_gb = size_mb / 1024

print(f"--- Звіт по папці {path} ---")
print(f"Розмір у байтах: {size_bytes}")
print(f"Розмір у МБ: {size_mb:.2f} MB")
print(f"Розмір у ГБ: {size_gb:.4f} GB")

```


## <a name="p-5">Оптимізація та Capacity Management</a>

### <a name="p-5.1">Боротьба з помилками 429/430</a>


Це класична ситуація для Trial ліцензії. Ви просто "вижали" максимум з того безкоштовного вузла, який вам виділив Microsoft.

Чому це сталося?
Fabric Trial має обмежену кількість CU (Capacity Units). Коли ви запускаєте масове читання (spark.read.json(json_files)), Spark намагається підняти паралельні процеси для кожного файлу. Якщо файлів багато, а потужність тріалу маленька (зазвичай це еквівалент F2 або F64, але з низьким пріоритетом), система спрацьовує як запобіжник: "Stop, ти споживаєш ресурси швидше, ніж дозволяє твій безкоштовний тариф".

Як це "лікувати":

- Зачекайте 2-5 хвилин: Fabric використовує систему "smoothing" (згладжування). Якщо ви дали пікове навантаження, вам треба трохи почекати, поки "штрафні бали" згорять.

- Обмежте паралелізм: Якщо ви використовували ThreadPoolExecutor, зменште max_workers до 2 або 3.

- Перевірте Monitoring Hub: Зліва на панелі Fabric є іконка спідометра (Monitoring hub). Зайдіть туди і завершіть (Cancel) усі завислі або старі сесії Spark. Вони можуть "тримати" ваші ліміти.

- Збільште час життя сесії: У налаштуваннях Workspace (Spark settings) можна виставити автоматичне завершення сесії через 10-20 хвилин, щоб вони не висіли марно.

#### <a name="p-5.1.1">5.1.1. "TooManyRequests (430) у Fabric Trial: Це не баг коду, а обмеження безкоштовної потужності</a>

"Помилка 430 (TooManyRequests): Це ознака 'throttling' (обмеження швидкості). Якщо виникає при масовій обробці:

- Не запускайте ноутбук занадто часто поспіль.

- Закрийте непотрібні вкладки з іншими ноутбуками (кожна вкладка — це активна сесія).

- Якщо файлів дуже багато, обробляйте їх пачками (batches), а не всі 500 одразу."


#### <a name="p-5.1.2">5.1.2 "CapacityLimitExceeded (429) у Fabric Trial: Це не баг коду, а обмеження безкоштовної потужності</a>

"CapacityLimitExceeded (429) у Fabric Trial: Це не баг коду, а обмеження безкоштовної потужності.

- Причина: Занадто багато паралельних Spark-завдань за короткий проміжок часу.

- Рішення для розробки: Зменшити кількість одночасних потоків у ThreadPoolExecutor (наприклад, до 2) або використовувати послідовну обробку пакетів.

- Для продуктиву: У реальному проекті обирається відповідний рівень Capacity (наприклад, F64), де такі ліміти значно вищі."

Якщо у  колеги теж піднятий тріал, спробуйте працювати в різних Workspace, але на одному тенанті — це іноді допомагає розділити ліміти (якщо вони не обмежені на рівні всього тенанта).

**Аналіз ситуації (по скріншотах)**

 <kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-01.png" /></kbd>
<p style="text-align: center;"><a name="pic-01">pic-01</a></p>


 <kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-02.png" /></kbd>
<p style="text-align: center;"><a name="pic-02">pic-02</a></p>


CapacityUnits 64: Ваш пробний період (Trial) має загальну потужність 64 CU. Це досить багато (еквівалент серйозного вузла).

0/16 Capacity units used/maximum: А ось тут криється нюанс. Хоча загальна ємність 64, на ваш конкретний Spark-пул (або воркспейс) наразі виділено ліміт лише 16 CU.

Other workspaces: На третьому скріншоті видно велику золотисту смугу. Це означає, що інші ваші воркспейси або процеси вже "з’їли" майже весь доступний бюджет потужності вашого тріалу.


###  <a name="p-5.2">Вирішення конфліктів запису (ConcurrentAppend)</a>  

Помилка ConcurrentAppendException у Microsoft Fabric (та загалом у середовищі Delta Lake) — це класична проблема паралелізму. Вона виникає, коли декілька процесів намагаються одночасно додати (append) дані в одну й ту саму таблицю, і система не може гарантувати цілісність транзакції.
Хоча Delta Lake дозволяє читати дані під час запису, одночасний запис вимагає чіткої черговості.
Коли ви записуєте дані в Delta-таблицю, Fabric створює новий файл логу в папці _delta_log. Якщо два джобі (наприклад, два Notebooks або Pipeline) намагаються створити наступну версію таблиці (наприклад, 00001.json) одночасно:

- Перший процес успішно фіксує зміни.

- Другий процес бачить, що версія, яку він хотів створити, вже зайнята, і "падає" з помилкою ConcurrentAppendException.

Методи вирішення та обходу

Ось стратегії, які допоможуть уникнути цієї помилки, від найпростіших до архітектурних:

1. Оптимізація розкладу (Retry Logic)

Найпростіший спосіб — не дати процесам "штовхатися" в дверях.

    Рознесення в часі: Налаштуйте тригери Pipeline так, щоб вони не запускалися одночасно.

    Автоматичні повтори (Retries): У налаштуваннях Pipeline для активності Notebook або Dataflow встановіть Retry count (наприклад, 3) та Retry interval (наприклад, 30-60 секунд). Це дозволить другому процесу дочекатися завершення першого.

2. Використання V-Order та Write Optimization

В налаштуваннях Spark у Fabric можна увімкнути функції, які пришвидшують запис, зменшуючи "вікно", під час якого може виникнути конфлікт:

    spark.databricks.delta.optimizeWrite.enabled = true

    spark.databricks.delta.autoCompact.enabled = true

3. Partitioning (Партиціонування)

Якщо ваші процеси записують дані в різні логічні частини таблиці (наприклад, один процес пише дані за "Регіон А", а інший — за "Регіон Б"), використовуйте партиціонування за цим полем.

    Delta Lake краще обробляє паралельні записи, якщо вони фізично рознесені по папках-партиціях.

4. Перехід на ID-based Upsert (Merge)

Замість простого append, розгляньте використання операції MERGE. Хоча вона складніша, вона має власні механізми обробки конфліктів. Однак, зверніть увагу, що MERGE також може викликати ConcurrentUpdateException, якщо два процеси оновлюють одні й ті самі рядки.

5. Архітектурний підхід: "Staging" таблиці

Якщо у вас десятки процесів, що пишуть в одну таблицю одночасно:

    Кожен процес пише у свою окрему тимчасову (staging) таблицю.

    Окремий "майстер-процес" (Master Job) збирає дані з усіх staging-таблиць і одним махом переносить їх в основну фінальну таблицю


Якщо помилка виникає рідко — допоможе звичайний Retry у Pipeline. Якщо постійно — потрібно переглянути архітектуру запису або впровадити чергу (Queue), де повідомлення про запис обробляються послідовно.

Як реалізувати Retry у  Варіанті 2  обробника файлів [Exch_ParallelFileMergeComply.ipynb](/notebooks/Exch_ParallelFileMergeComply.ipynb)

Оскільки у Exch_ParallelFileMergeComply.ipynb  використовуєтmcz ThreadPoolExecutor, помилки конфліктів там найбільш імовірні. Ось як можна модифікувати функцію обробки:

```py
import time
from py4j.protocol import Py4JJavaError

def merge_file_with_retry(file_path, max_retries=3):
    for attempt in range(max_retries):
        try:
            # Ваша логіка зчитування та MERGE
            df = spark.read.json(file_path)
            df.createOrReplaceTempView("current_file_view")
            
            spark.sql("""
                MERGE INTO TERM_PAYMENTS AS target
                USING current_file_view AS source
                ON target.TX_ID = source.tx_id
                ... -- інша частина вашого MERGE
            """)
            return True # Успішно
        except Exception as e:
            error_msg = str(e)
            # Перевіряємо, чи це помилка конфлікту або лімітів
            if "ConcurrentAppendException" in error_msg or "429" in error_msg:
                if attempt < max_retries - 1:
                    wait = (attempt + 1) * 3 # Проста прогресія паузи
                    print(f"Помилка конфлікту у {file_path}. Спроба {attempt+1}, чекаємо {wait}с...")
                    time.sleep(wait)
                    continue
            print(f"Критична помилка у {file_path}: {error_msg}")
            raise e

```

## <a name="p-6">Просунуті сценарії Spark та SQL</a>

Як ще можна замінити  відсітні Sequence (як в ORACLE) чи IDENTITY (як в MS SQL) окрім [описаного метода](#IDENTITY) . По факту можна використовувати такі підходи:

1. Генерація ID на рівні ETL (Найкращий варіант)

Якщо ви завантажуєте дані через Fabric Notebook (PySpark) або Data Factory, ви можете генерувати сурогатні ключі під час завантаження:

```text
    У Spark (Notebook) це функція monotonically_increasing_id().

    У SQL можна використовувати ROW_NUMBER() OVER(ORDER BY (SELECT NULL)) + (максимальний ID в таблиці).
```

2. Використання хеш-ключів (Hash Keys)

Замість цифр (1, 2, 3...) використовуйте унікальний рядок, згенерований з бізнес-ключа. Це дуже популярно в Data Vault.

```SQL
    -- Приклад для терміналів
    UPDATE payment.TERMINALS_D
    SET TERMINAL_ID = CAST(HASHBYTES('SHA2_256', CAST(SourceTerminalCode AS VARCHAR)) AS BIGINT);
```

3. "Modern Data Stack" підхід (Hash Keys)

Варіант, який зараз є стандартом для Data Vault 2.0 та великих Lakehouse: Surrogate Hash Keys.
Замість послідовного числа (1, 2, 3...) ви генеруєте TERMINAL_ID як HASH від природного ключа (наприклад, від коду терміналу в системі-джерелі).

Функція: 

```py
    HASHBYTES('SHA2_256', UPPER(TRIM(SourceTerminalCode)))
```

Перевага: Ви можете завантажувати дані паралельно з 10 різних джерел, і вам не потрібно звертатися до жодної центральної таблиці чи Sequence. Одна і та сама сутність завжди отримає один і той самий ID.

- **Тимчасові таблиці**: T-SQL обожнює тимчасові таблиці. Замість складних вкладених підзапитів чи CTE (хоча вони є), часто простіше написати SELECT ... INTO #MyTempTable.

- **Cross-database queries**: Ви можете легко робити JOIN між таблицями з Warehouse та таблицями з Lakehouse (через SQL Endpoint), просто вказуючи повне ім'я: [Workspace].[Database].[Schema].[Table].

- "Top Sessions" запитів в DWH, це  "Top Sessions" з Oracle

В системному view sys.dm_pdw_exec_requests ( це  "Top Sessions" з Oracle). Там можна побачити, які запити (від тих самих Notebooks чи Power BI) зараз "кладуть" вашу систему. Це допоможе  знайти винуватця Notebook_1, який вантажить процесор.

- **Коментарі до таблиць та полів таблиці**: На даний момент Fabric Warehouse не підтримує COMMENT ON TABLE або sp_addextendedproperty безпосередньо через T-SQL скрипти так, як це робить повноцінний SQL Server або Oracle. Як це працює зараз: Ви пишете чистий DDL (CREATE TABLE...), а описи полів та таблиць зазвичай додаються через візуальний редактор семантичної моделі або через зовнішні інструменти (Tabular Editor), якщо модель не "Default". Оскільки ми звикли до DDL-скриптів, у Fabric Warehouse ваш основний інструмент — це власне SQL-скрипти всередині Workspace. Ви можете зберігати свої CREATE та ALTER скрипти прямо в проекті. Найкраща практика зараз: 

- використовувати звичайні SQL-коментарі -- або /* ... */ всередині збережених процедур та скриптів.

- писати скрипти в Notebook з типом комірок T-SQL. Notebook повноцінно заміняє файли  *.sql. Вона є "природнім" елементом-об'єктом workspace. Завдякі багатій підтримці markdown дозволяє там написати досить широкі коментарі

- Семантична модель: навіщо вона.

Хоча це інструмент для Power BI, у Fabric Warehouse вона створюється автоматично (Default Semantic Model).

1. Логічні зв'язки: Коли ви малюєте зв'язки в інтерфейсі моделі, ви не створюєте фізичні FOREIGN KEY з перевіркою цілісності (як в класичних БД). Ви просто кажете Power BI: «Коли користувач візьме поле з цієї таблиці, з'єднай його з тією».

2. Коментарі: Так, описи (Descriptions), які ви вносите в семантичну модель, бачать користувачі в Power BI Desktop. Це допомагає їм не питати вас щодня: «А що означає колонка TX_TYPE_01?».


Поточна версія Fabric дозволяє створювати в рамках одного DWH різні схеми даних  (як в OTRACLE).

```py
/* 1. Створення схеми, якщо вона не існує */
IF NOT EXISTS (SELECT * FROM sys.schemas WHERE name = 'payment')
BEGIN
    EXEC('CREATE SCHEMA [payment]')
END
```

Тобто, замість сувати все в схему dbo, таблиці різних додатків (різних сутностей) можна рознести по  схемам.

Не зважаючи на те, що в Source Control  попадають DDL всіх об'єктів бази даних, зміни DDL в БД краще вести окремо. На приклад в спеціалізовній  Notebook. Крім того, треба мати на увазі, що  в SQL explorer Fabric DWH в папці **Queries** (**Queries/my queries**, **Queries/shared queries** ) - не зберігаються в Source Control

Notebook вигідніше одного або кількох sql скриптів (файлів з типом *.sql) тому що: 

- notebook можна зробити  з комірками типу  T-SQL

- Notebook моє комірки типу Markdown. Тому там можна зробити нормлаьний опис для  кожної T-SQL комірки чи групи TSQL кмірок [pic-05-3](#pic-05-3). 

- В notebook можна запустити всі комірки послідовно, як і sql скрипт, так і окремо вибрані комірки, що не можливо для sql-скрипта.

- Notebook гарно зберігається в Source Control ситемі


Для ведення проектів DDL можна зробити в workspace таку ієрахію папок, як на [pic-05-2](#pic-05-2).

<kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-05-2.png" /></kbd>
<p style="text-align: center;"><a name="pic-05-2">pic-05-2</a></p>

Така ієрархія дасть можливість розробникам, що працюють над одним проектом бачити зміни один оного, відпрацьовувати аудит, тому що зміни розділені по RFC, ну і колективно працювати над проектом.

<kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-05-3.png" /></kbd>
<p style="text-align: center;"><a name="pic-05-3">pic-05-3</a></p>


### <a name="p-6.1"> 6.1. Еволюція схеми (Schema Evolution)</a>

В Microsoft Fabric, як і в класичному Delta Lake, схема таблиці за замовчуванням є жорсткою. Це захищає від "забруднення" даних випадковими новими колонками. Однак, коли зміни є легітимними, ми використовуємо механізми еволюції.

1. Автоматична еволюція схеми (.option("mergeSchema", "true"))

Найпоширеніший спосіб. Якщо ви додаєте нові колонки до DataFrame і хочете, щоб вони автоматично з'явилися в Delta-таблиці, використовуйте цей параметр під час запису.

```py
# Приклад запису з еволюцією схеми
df.write.format("delta") \
    .mode("append") \
    .option("mergeSchema", "true") \
    .save("Tables/your_table_name")
```

Що дозволяє Schema Evolution:

- Додавання нових колонок (найчастіший кейс).

- Зміна Nullability (з NOT NULL на NULL).

Що НЕ дозволяє (потребує overwrite або перестворення):

- Зміна типу даних існуючої колонки (наприклад, з Integer на String).

- Перейменування існуючих колонок (для цього потрібен Mapping).

- Видалення колонок.

2. Ручна еволюція через SQL (ALTER TABLE)

Якщо ви працюєте в Spark SQL ноутбуці, ви можете додати колонку вручну перед записом даних:

```sql
ALTER TABLE your_lakehouse.your_table_name 
ADD COLUMNS (new_column_name STRING AFTER existing_column);

```

3. Schema Enforcement (Захист схеми)

Якщо ви не вкажете .option("mergeSchema", "true"), Spark видасть помилку AnalysisException, якщо структура DataFrame не збігається зі структурою таблиці.

**Правило:** В Pipeline-ноутбуках завжди тримайте mergeSchema вимкненим, окрім випадків, коли ви свідомо очікуєте динамічну структуру від джерела. Це запобігає появі "сміттєвих" колонок через помилки в коді.

4. Обробка несумісних змін (Overwrite Schema)

Якщо тип даних колонки змінився на джерелі (наприклад, ID став рядком замість числа), автоматична еволюція не спрацює. В такому разі потрібно використовувати overwriteSchema:

```py

# УВАГА: Це видалить старі дані в колонці або потребує повної перезапису таблиці!
df.write.format("delta") \
    .mode("overwrite") \
    .option("overwriteSchema", "true") \
    .saveAsTable("your_table_name")

```


При використанні V-Order (оптимізація запису в Fabric), Schema Evolution працює стандартно, але пам'ятайте, що після великих змін схеми варто запустити OPTIMIZE для перерахунку статистики та підтримки продуктивності SQL Endpoint.




### <a name="p-6.2"> 6.2. Ізоляція даних через проксі-Views</a>

Відмінність від реляційних баз даних полягає в тому, що Fabric DWH це T-SQL Engine  натягнутий проверх DataLake, щоб забезпечити роботу PowerBi. Відповідно, випала та функціональність, що не підтримується DataLake.

- Первинні ключі  Deffered (відкладені), тобто вони не використовуються при вставці даних. Це як індекс, для підказки оптимізатору при операціяї SELECT. Як наслідок, легко можуть появитися дублі.

- Відсутні Foreign Key

Вони просто відсутні. Вони моделюються емантичними моделями для підтримки PowerBI.

- Відсутність Індексів:  У Fabric DWH немає CREATE INDEX. Система покладається на Columnstore (стиснення по колонках) та розподілені обчислення. Ваша задача — правильно підібрати типи даних, щоб не "роздувати" таблиці.

- Transaction Isolation: Тут підтримується рівень Snapshot Isolation. Тобто читачі не блокують письменників (схоже на те, як працює Undo Tablespace в Oracle, але реалізовано через версіонування файлів)

- View з GROUP BY: чи це «онлайн»?

У Fabric Warehouse звичайні VIEW завжди обчислюються динамічно (онлайн).

```text
    Як це працює: Коли ви робите SELECT * FROM MyView, рушій Polaris підставляє код View у ваш запит і виконує агрегацію (Group By) в момент звернення.

    Чи є Materialized Views? У класичному розумінні (як в Oracle, де дані фізично зберігаються) — ні. У Fabric Warehouse зараз немає індексованих або матеріалізованих View.

    Проблема продуктивності: Якщо у вас мільйони рядків і складні Join-и всередині View з Group By, Power BI може «підторможувати».

    Рішення: Якщо продуктивність динамічного View стає проблемою, розробники Gold-шару зазвичай перетворюють це на фізичну таблицю (через INSERT INTO ... SELECT або CREATE TABLE AS SELECT у Pipeline), яку оновлюють за розкладом.
```

4. При створенні об'єктів бази даних треба користуватися таким підходм (структурою):

- CREATE SCHEMA [private]; — для таблиць. Тут лежать фізичні таблиці

- CREATE SCHEMA [public]; — для View. Тут створюється перший рівень абстракації, коли користувач звітів не має прямого доступу до таблиць і не має інформації про їх фізичну структуру. Усі VIEW у схемі public робимо просто як проксі:


```SQL

    CREATE VIEW [public].[Sales] AS SELECT * FROM [private].[FactSalesOrder];
```

Таким чином ми  ізолюємо фізичне зберігання від логічного представлення. Це дозволяє нам змінювати структуру таблиць, не ламаючи звіти користувачів.


### <a name="p-6.3"> 6.3. Виклик Stored Procedures з Notebook</a>

**Stored Procedures** (Процедури): Це основний інструмент ELT. Замість того, щоб налаштовувати трансформацію у візуальному Pipeline, ви кладете весь свій SQL-код у процедуру. Pipeline просто викликає її однією командою Execute Stored Procedure. Це набагато легше версіонувати та дебажити.

- Виклик Stored Procedur з Notebook

```py
    %%sql
    -- 1. Додаємо новий термінал
    EXEC payment.sp_UpsertTerminal @TerminalID = 101, @TerminalName = 'Термінал А1', @IsActive = 1;

    -- 2. Оновлюємо той самий термінал (змінюємо назву)
    EXEC payment.sp_UpsertTerminal @TerminalID = 101, @TerminalName = 'Термінал А1 - Оновлено', @IsActive = 1;

    -- 3. Дивимось результат
    SELECT * FROM payment.TERMINALS_D;

```

**User-Defined Functions** (Функції): В основному використовуються для розрахунків (наприклад, конвертація валют або складні бізнес-правила), які потрібно перевикористовувати в багатьох View.

- Security (Row-Level Security)

Ось де функції стають критично важливими. Якщо ви хочете, щоб розробник з одного воркспейсу не бачив дані іншого в межах однієї таблиці Gold, ви пишете Security Function і прив'язуєте її до таблиці через Security Policy. Це працює ідентично до Oracle VPD (Virtual Private Database).

- у Microsoft Fabric немає концепції GRANT TO PUBLIC у тому розумінні, як це було в класичному SQL Server чи Oracle. Безпека тут працює за принципом "спочатку заборонено все", але з певними особливостями ієрархії Fabric Workspace. Ось як архітектурно правильно реалізувати схему «Приховані таблиці — Публічні View» у вашому DWH.

1. Розподіл по Схемах (Schema-based Security)

Це найбільш "чистий" спосіб. Ви створюєте дві схеми: одну для сирих даних, іншу — для презентаційного шару.

Схема data (або stg): Тут лежать ваші реальні таблиці.

Схема reports (або dbo): Тут лежать лише VIEW.

2. Механізм приховування (DENY vs GRANT)

У Fabric Warehouse ви керуєте доступом на рівні схем за допомогою стандартних T-SQL команд. Щоб реалізувати вашу ідею:

**Даємо доступ до View:**

```SQL
GRANT SELECT ON SCHEMA::reports TO [user_or_group];

```
**Забороняємо доступ до таблиць**:
Вам навіть не потрібно явно писати DENY, якщо ви не дали SELECT на схему data. Користувач просто не побачить таблиць у списку об'єктів або отримає помилку доступу при спробі SELECT * FROM data.Table.

**Рівні доступу у Workspace (Важливий нюанс!)**
У Fabric є ролі Workspace: Admin, Member, Contributor, Viewer.

Якщо ваші колеги мають роль Contributor або вище, вони автоматично бачать УСІ таблиці та коди. Ролі Workspace мають пріоритет над SQL-дозволами.

Щоб ваша схема з приховуванням запрацювала, користувачі повинні мати роль Viewer у Workspace + специфічні GRANT у самому SQL Warehouse.

**Використання "App" для кінцевих користувачів**

Якщо ваша мета — щоб люди бачили лише звіт (як той, що ми щойно зібрали), а не структуру бази взагалі:

Ви публікуєте Power BI App.

Користувачі мають доступ лише до App.

Вони не мають доступу до Workspace, а отже, навіть не знають про існування таблиць у DWH. Семантична модель ходить в базу під вашим ім'ям або Service Principal.

### <a name="p-6.4">Візуальні кубики VS кодінг в otebooks</a>

Суперечка — "класика" в Data Engineering. Інфраструктурники часто люблять Low-code/No-code (як Data Factory Copy Activity), бо це простіше моніторити, воно "стандартне" і нібито надійніше.
Але як тільки з'являється складна логіка, MERGE з бізнес-правилами, обробка фінансових розрахунків або робота з нестандартними JSON — візуальні "кубики" стають монстрами, які неможливо підтримувати.

У Microsoft Fabric наша стратегія — підтримувати баланс між кубаким і кодингом на прикладі паралельної обробки файлів. У Fabric є два основні способи "розпаралелити"  паралельні трансформації:

1. "Зовнішня" паралелізація (через Pipeline)

Це найпростіший шлях, який сподобається архітекторам, бо вони бачать структуру процесу:

- Використовуйте активність ForEach у Pipeline.

- Налаштуйте її на папку Files/bronze.

- Всередині ForEach поставте виклик вашого Notebook.

У налаштуваннях ForEach поставте галочку Sequential = False та встановіть Batch count (наприклад, 5 або 10).

**Результат:** Pipeline запустить кілька сесій Spark одночасно для різних файлів.

2. "Внутрішня" паралелізація (через Spark)

Якщо файлів дуже багато (десятки, сотні чи тисячі), краще не плодити багато сесій ноутбуків, а обробити все за один раз самим Spark, Для прикладу:

- Замість того, щоб читати один файл 12144_2026-01-31.json, ви читаєте всю папку:
    df = spark.read.json("Files/bronze/*.json").

- Spark сам розподілить ці дані по вузлах кластера (Worker Nodes).

- Потім ви робите один великий MERGE для всього датафрейму.

Це набагато ефективніше, бо Spark оптимізує план виконання для всієї маси даних одразу.

Аргументи:

Коли вони кажуть, що "ноутбуки — це некеровано":

- **Source Control (Git Integration):** У Fabric ноутбуки підключаються до Azure DevOps або GitHub. Весь ваш код версіонується, як і будь-який інший бекенд-проект. Та і взагалі Fabric notebook в своєму бінарному вигляді являє собою py-файл. Компоненти Pipeline  та і самі Pipeline являють собою json-файли. Тобто, немає різниці, що мержити. Треба тільки мати на увазі, що json-файли всі мержаться погано, особливо коли багато змін чи  конфліктів, на відміну від звичайного .py - файлу. 

- **Compute Isolation:** Ви можете призначити ноутбуку конкретний "Pool" ресурсів, щоб він не "з'їв" пам'ять всього воркспейсу.

- **Unit Testing:** У ноутбуці ви можете написати тести для своєї логіки трансформації, чого майже неможливо зробити у візуальних Mapping Data Flows без болю.

- **Якщо комусь хочеться візуальності:** Використовуйте Data Wrangler у Fabric. Це інструмент всередині ноутбука, який генерує Python-код для очищення даних через графічний інтерфейс. Це такий собі "компроміс": ви працюєте візуально, але на виході — чистий, професійний код у клітинці ноутбука.

- **Декомпозиція пайплайнів:**

- Parent Pipeline: Відповідає за оркестрацію (перелік файлів, цикл).

- Child Pipeline: Відповідає за unit-роботу (обробка одного об'єкта).

**Перевага:** Це дозволяє уникнути 'прісного' коду та спрощує відстеження помилок через Output конкретної ітерації For Each."

- Child Pipeline повинен повертати змінну-значенн результат своєї роботи, а Parent Pipeline повинна його аналізувати і приймати рішення, що робити з помилкою, якщо така виявлена.


- Повернення результату виконання з Child pipline в Parent pipline використовуйте елемент **Set Veriable** з параметром PipeLine Return Value

 <kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-04-1.png" /></kbd>
<p style="text-align: center;"><a name="pic-04-1">pic-04-1</a></p>

Якщо хочемо звернутися до параметрів pipline  треба використовувати **Expressions**. Лінк на документацію: [Expressions and functions for Data Factory in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/data-factory/expression-language) .

**Приклади:**

**@activity('Mergilka').output.result.exitValue** отримати результат роботи попереднього кубика з назвою 'Mergilka' як String

**@json(activity('Mergilka').output.result.exitValue)** отримати результат роботи попереднього кубика з назвою 'Mergilka' як json

**@pipeline().RunId** отримати  runid  поточного pipeline 

**@pipeline().PipelineName** отримати найменування поточного pipline

- Для того, щоб в рамках одного pipeline  передати змінну, до якої будуть звертатися інші кубики, що стоять далеко після поточного використовуйте елемент **Set Veriable** з параметром PipeLine Variable

<kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-04-2.png" /></kbd>
<p style="text-align: center;"><a name="pic-04-2">pic-04-2</a></p>



### <a name="p-6.5">Шаблони кодінгу в notebook</a>

- **Параметризація:** Завжди в NoteBooks використовуйте Parameter cell  для шляхів до папок чи для отримання параметрів від попередніх вузлів роботи pipekine. Це дозволяє використовувати один ноутбук для Test та Prod середовищ [pic-04-3](#pic-04-3).

 <kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-04-3.png" /></kbd>
<p style="text-align: center;"><a name="pic-04-4">pic-04-3</a></p>

- **Не викорситовуємо хардкодінг секретів та параметриів конфігурації таких як URL чи щось подібне**

Для зберігання використовуємо env-зміні вашої Custom Environemt   [pic-04-7](#pic-04-7), [pic-04-8](#pic-04-8). Для доступу до env-змінних з коду notebook використовуємо такий підхід


```py

# Через spark.conf.get
print("Отримуємо через spark.conf.get")
print(spark.conf.get("spark.nodeproperty.env.TELEGRAM_TOKEN"))
print(spark.conf.get("spark.nodeproperty.env.URL"))

```

Для більш сарйозних секретів використовуємо створюємо Azure Key Vault окремо від Fabric, записуємо туди секрети. Хоча розробники цього і не роблять, але це частина загальної архітектурно освіти хмари, тому знати повинні. Але визначення що пишемо в Azure Key Vault, а що в "spark.nodeproperty.env" лежить на архітекторі/аналітикаї. Розробникам повинно бути доведено, відкіль  читати ті чи інші налаштування. Зате розробники повинні знати як це прочитати. 
В Notebook звертаєтеся до параметрів в  Azure Key Vault через бібліотеку **mssparkutils**.

```py

# Отримання секрету з Azure Key Vault
from notebookutils import mssparkutils

# "my-key-vault" — назва вашого Key Vault
# "api-password" — назва секрету всередині сховища
secret_value = mssparkutils.credentials.getSecret("https://my-key-vault.vault.azure.net/", "api-password")

# Тепер використовуємо secret_value у підключенні

```

Детальніше про  mssparkutils можна почитати за лінком: [Утилиты для учётных записей](https://learn.microsoft.com/ru-ru/fabric/data-engineering/microsoft-spark-utilities#credentials-utilities) та [Accessing Azure Key Vault (AKV) from Notebook](https://learn.microsoft.com/en-us/fabric/data-engineering/spark-best-practices-security#accessing-azure-key-vault-akv-from-notebook).


**Використання Fast Authentication (для SQL/Data sources)**

Якщо ви працюєте з Azure SQL, ADLS або іншими сервісами Azure, найкращий спосіб — взагалі не використовувати логіни та паролі.

    Service Principal або Managed Identity: Надайте права самій ємності Fabric на доступ до ресурсу в Azure.

    У коді Notebook можна використовувати mssparkutils.credentials.getToken("audience") для отримання токена доступу без введення пароля.


- Для роботи з файлами використовуємо mssparkutils.fs

Детальніше читати за лінком [Служебные программы файловой системы](https://learn.microsoft.com/ru-ru/fabric/data-engineering/microsoft-spark-utilities#file-system-utilities).


- Щоб викликати з одного notebook інший notebook використовуємо пакет mssparkutils.notebook

Детальніше можна прочитати за лінком [Служебные программы для ноутбуков](https://learn.microsoft.com/ru-ru/fabric/data-engineering/microsoft-spark-utilities#notebook-utilities) .

Тут можна запустити з одного notebook інший. Можна запустити в паралель кілька  notebooks.

Звернути увагу на використання цього пакету, коли notebook в складі pipeline завершив свою роботу і повинен повернути результат.

Приклад такого виходу показано нижче:

```py
# пока робимо просто вивод імені файлу
print(f"Обробка файла: {file_name}")

import json
from notebookutils import mssparkutils
from pyspark.sql.functions import input_file_name, regexp_extract
import os

# 1. Налаштування шляхів
source_folder = "Files/bronze"
archive_folder = "Files/bronze/archive" # Або інша папка поза зоною читання
target_table = "TERM_PAYMENTS"

# Створюємо папку архіву, якщо її немає
mssparkutils.fs.mkdirs(archive_folder)

# 2. Перевіряю наявність  JSON файлу
json_file = f"{source_folder}/{file_name}"

result_data = {"file_processed": json_file, "status": "In progress", "rows_affected": 0}

try:

    if not mssparkutils.fs.exists(json_file):
        print("Не знайдено для обробки файлу: {json_file}")
        # Піднімаємо помилку, щоб зупинити виконання
        raise FileNotFoundError(f"Критична помилка: Файл {json_file} не знайдено в сховищі!")        
    else:
        print(f"Починаємо обробку {json_file} файлу...")
    
    # 3. Читання та MERGE (як ми обговорили раніше)

    raw_df = spark.read.json(f"{json_file}")
    df_with_meta = raw_df.withColumn("file_name", regexp_extract(input_file_name(), r"([^/]+)$", 1))
    df_with_meta.createOrReplaceTempView("v_batch_incoming_data")

    # Виконуємо MERGE
    spark.sql(f"""
        MERGE INTO {target_table} AS target
        USING v_batch_incoming_data AS source
        ON target.TX_ID = source.tx_id
        WHEN MATCHED THEN
            UPDATE SET target.STATUS = 'UPDATED', 
                       target.FILE_NAME = source.file_name,
                       target.AMOUNT = source.amount,
                       target.CHARGE = source.charge
        WHEN NOT MATCHED THEN
            INSERT (TX_ID, TERMINAL_ID, OPDATE, NAME, PASSPORT, AMOUNT, CHARGE, STATUS, FILE_NAME)
            VALUES (source.tx_id, source.terminal, source.opdate, source.name, source.passport, 
                    source.amount, source.charge, 'NEW', source.file_name)
    """)

    # 4. АРХІВАЦІЯ: Переносимо файли тільки ПІСЛЯ успішного MERGE
    print("MERGE завершено успішно. Переносимо файли в архів...")
    destination = os.path.join(archive_folder, file_name)
        
    # Переміщуємо файл
    mssparkutils.fs.mv(json_file, destination, overwrite=True)
    
    print(f"Файл: {json_file}  переміщено в {archive_folder}.")
    # 4. Формуємо успішний результат
    result_data["status"] = "Success"
except Exception as e:
    print( f"❌ Помилка у файлі {json_file}: {str(e)}" )
    result_data["status"] = "Error"
    result_data["error"] = str(e)

# 3. ЄДИНА точка виходу з ноутбука. 
#ристовуємо json.dumps для гарантії валідності формату для Pipeline    
mssparkutils.notebook.exit(json.dumps(result_data)) 
```

**Увага:** Вихід повинен бути тільки один, тому в даном приклді він винесений за try-Exept. Повинен повертати завжди String.  exitValue має обмеження по розміру (близько 2-4 МБ), тому не варто повертати через нього величезні JSON-масиви.

Доступ до результата в рамках pipelien виконується через Set Variable з expression


```text
# як текст
@activity('Mergilka').output.result.exitValue

# як json
@json(activity('Mergilka').output.result.exitValue)
```
В документації ключс **result**  пропущений. Треба звернути на це увагу.


- В notebooks, що використовуються в pipline  не використовувати функції виводу

В notebooks, що використовуються в pipline  не використовувати функції виводу  SparkDataframe.dispalay() або PandasDataFrame.Head(). Вони дуже уповільнюють роботу nogtebook.

- В notebooks мінімізувати використання python print()

В notebooks мінімізувати використання python print(), тому, що вони не логуються. Замість цього використовувати пакет стандартного python логгера. Детально можна прочитати за лінком [Python logging in a notebook](https://learn.microsoft.com/en-us/fabric/data-engineering/author-execute-notebook#python-logging-in-a-notebook).

```py
import logging

# Customize the logging format for all loggers
FORMAT = "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
formatter = logging.Formatter(fmt=FORMAT)
for handler in logging.getLogger().handlers:
    handler.setFormatter(formatter)

# Customize log level for all loggers
logging.getLogger().setLevel(logging.INFO)

# Customize the log level for a specific logger
customizedLogger = logging.getLogger('customized')
customizedLogger.setLevel(logging.WARNING)

# logger that use the default global log level
defaultLogger = logging.getLogger('default')
defaultLogger.debug("default debug message")
defaultLogger.info("default info message")
defaultLogger.warning("default warning message")
defaultLogger.error("default error message")
defaultLogger.critical("default critical message")

# logger that use the customized log level
customizedLogger.debug("customized debug message")
customizedLogger.info("customized info message")
customizedLogger.warning("customized warning message")
customizedLogger.error("customized error message")
customizedLogger.critical("customized critical message")

```

Як показано на малюнку, код прекрасно виводить все ы в консоль.
<kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-04-9.png" /></kbd>
<p style="text-align: center;"><a name="pic-04-9">pic-04-9</a></p>


- Для паралельної обробки стартися використовувати паралелізацію SPARK, а не багатопоточність pyhton.

Для цього використовуємо Spark UDF- функції. Щось на кшталт такого:


```py

from faker import Faker
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType

fake = Faker()
# Створюємо UDF, щоб Spark міг генерувати імена паралельно на всіх нодах
fake_name_udf = udf(lambda: fake.name(), StringType())

df = spark.range(0, 1000).withColumn("fake_name", fake_name_udf())
display(df)

```

Замість **ThreadPoolExecutor**

```py
with ThreadPoolExecutor(max_workers=16) as executor:
    futures = [executor.submit(fetch_data, version_key, end_url, headers, time_filter, cost_type_filter, account_filter, company_filter, costelement_filter) for version_key, end_url, time_filter, cost_type_filter, account_filter, company_filter, costelement_filter in request_list]
    for future in tqdm(as_completed(futures), total=len(futures)):
        version_key, values, month, cost_type, account_condition = future.result()
        data_accumulator[version_key].extend(values)

```


## <a name="p-7">Тестування та QA</a>

### <a name="p-7.1">7.1. Генерація тестових даних</a>

Якщо хтось ще думає, що вам ададуть тестові дані, особливо, сторонні постачальники, то ви дуже помиляєтется. Зазвичай, запукається паралельна робта обох команд і тстові дані кодна команда готує самстійно, зідно  завдання на розробку. Якзо по завданню на розробку не можливо визначити, який вигляд повинні мати тестові дані - то таке завання на розробку не якісне і повинно бути перероблене.

**Порада щодо генерації даних у Spark.**

Коли будете писати свій Notebook для заповнення Bronze, спробуйте одразу використати Spark-оптимізований підхід. Замість того, щоб генерувати дані в циклі (що повільно), використовуйте spark.range разом з функціями withColumn:

```py
from pyspark.sql import functions as F

# Створюємо мільйон рядків миттєво
df = spark.range(0, 1000000) \
    .withColumn("sensor_id", (F.rand() * 100).cast("int")) \
    .withColumn("timestamp", F.current_timestamp()) \
    .withColumn("value", F.rand() * 100)

# Записуємо в Lakehouse як Delta-таблицю
df.write.format("delta").mode("overwrite").save("Tables/iot_bronze")
```

Якщо це  пакет типу Faker (пакет для генерації реалістичних даних), треба мати  на увазі, що його краще використовувати через pandas_udf, щоб Spark міг розподілити генерацію тексту по всіх вузлах кластера.

Щоб не робити генерацію в циклі for (що в Spark повільно), можна іти таким шляхом:

```py
from faker import Faker
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType

fake = Faker()
# Створюємо UDF, щоб Spark міг генерувати імена паралельно на всіх нодах
fake_name_udf = udf(lambda: fake.name(), StringType())

df = spark.range(0, 1000).withColumn("fake_name", fake_name_udf())
display(df)
```

### <a name="p-7.2">7.2. Про моделі промислових пристроїв</a>

При розробці програмного забезпечення, що приймає та обробляє дані від сенсорів промислового обладнання важлиов використовувати підхід моделювання промислових сенсорів. Це дає можливість згенерувати різні сценарії поведінки обладнання, та відтестувати реакцію на різні сценарії.  Notebook в  даному випадку  дуже допомагає. Для прикладу, в наведеному фрагменті коду показан модель датчика температури холодильника, що моделює не тільки нормальну поведінку а і анормлаьну поведінку датчика.




```python
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
from pyspark.sql.functions import col, to_date

def generate_failing_compressor(days=14):
    """
    Генератор температури холодильника з поганим компресором
    """
    # Базова синусоїда (цикли охолодження)
    time = np.linspace(0, days * 24, days * 24 * 60) # раз на хвилину
    
    # Поступове зростання амплітуди (компресор довше качає)
    degradation = np.linspace(0, 2.5, len(time)) 
    
    # Зростання базової лінії (фреон витікає)
    drift = np.linspace(0, 1.5, len(time))

    # тут коливання компресора з фіксованою частотою
    #temp = 4.0 + 1.5 * np.sin(time * 2 * np.pi / 0.5) + degradation * np.random.rand(len(time)) + drift

    # Робимо частоту прогресуючою (компресор вмикається частіше)
    for i in range( 1, len(time)):
      frequency_boost = 1 + (i / len(time)) * 0.5  # частота зростає на 50% до кінця
      cycle_effect = 1.5 * np.sin(time * 2 * np.pi / 0.5 * frequency_boost)
      temp = 4.0 + cycle_effect + degradation * np.random.rand(len(time)) + drift  

    
    return temp

def generate_sensor_data(num_sensors=10, days=1):
    """
      Генерація даних сенсорів температури
    """
    data = []
    start_time = datetime.now() - timedelta(days=days)
    
    for s_id in range(1, num_sensors + 1):
        sensor_name = f"TMP-REQ-{s_id:03d}"
        # Визначаємо базову стабільну температуру для кожного датчика
        base_temp = np.random.uniform(4.5, 5.5)
        # Визначаємо температуру для датчика від холодольника з вмираючим компресором
        base_temp_arr=generate_failing_compressor(days=days)
        base_temp_index=0

        # генеруємо температуру в вибраному часовому діапазоні
        for minute in range(days * 24 * 60):
            ts = start_time + timedelta(minutes=minute)
            
            # Додаємо невеликий шум (фізичні коливання)
            noise = np.random.normal(0, 0.1)

            # тут визначаємо, який датчик буде з вмираючим компресором
            if sensor_name=='TMP-REQ-004':
                # вмираючий компресор
                temp = base_temp_arr[base_temp_index] + noise
            else:
                # нормлаьний датчик
                temp = base_temp + noise
    

            # Імітуємо аномалію (наприклад, відкриті двері холодильника раз на добу)
            if 600 <= (minute % 1440) <= 615:
                #temp += (minute % 600) * 0.5 # Швидке зростання температури
                temp += np.random.normal(5, 10) * 0.5
            
            status = "OK" if 2.0 <= temp <= 8.0 else "ALARM"
            
            data.append({
                "sensor_id": sensor_name,
                "facility_id": "KYIV-WH-01",
                "equipment_id": f"FRIDGE-{s_id:02d}",
                "timestamp": ts,
                "temperature": float(round(temp, 2)),
                "humidity": float(round(np.random.uniform(40, 60), 2)),
                "status": status,
                "date": ts.date()
            })
            base_temp_index += 1
            
    return spark.createDataFrame(pd.DataFrame(data))

# Генеруємо та записуємо в наш Lakehouse
df_telemetry = generate_sensor_data(num_sensors=10, days=15)
df_telemetry.write.format("delta").mode("append").saveAsTable("sensors.sensor_telemetry_m2")

print("Генерація даних закінчена і записана в  Delta table Модель 2.")
```

Як реузльтат, на  [pic-06-01](#pic-06-01) відображає дані і поведінку  датчика температури з нормального холодильника, 

<kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-06-01.png" /></kbd>
<p style="text-align: center;"><a name="pic-06-01">pic-06-01</a></p>

а [pic-06-02](#pic-06-02) та [pic-06-03](#pic-06-03) відображають  поведінку датчика температури, у якого компресор працює погано 

<kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-06-02.png" /></kbd>
<p style="text-align: center;"><a name="pic-06-02">pic-06-02</a></p>


<kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-06-03.png" /></kbd>
<p style="text-align: center;"><a name="pic-06-03">pic-06-03</a></p>


Відповідно, це дасть можливість відлагодити роботу програмного забезпечення по прийому  даних, обрахуванню, та відображення на дашбордах чи в звітах.

### <a name="p-7.3">7.3. Про моделі http сервісів</a>

Якщо виникає необхідність, щоб Fabric зверталася до зовнішніх серверів за даними, то у більшості випадків розробка програмного забезпечення починається задовго до того, як буде отримано доступ до тих http - серверів, і знову ж таки, у більшості випадків ті сервери не будуть мати сестових версій, а матимуть тільки продуктивні. Щоб відпрацювати всі суенарії поведінки можна створювати моделі цих серверів використовуюючи Server Less функції вбудовані в Fabric. Це значно спростить  відпрацювання всіляких сценаріїв, зважаючи, що http протокол по своїй природі не надійний. Практика показує, що використання моделей більше ніж на 90% дозволяє відпрацювати всі сценарії обробки. 


## <a name="p-8">Корисна документація та Medium-блоги</a>

1. [Microsoft Fabric documentation](https://learn.microsoft.com/en-us/fabric/)

2. [End-to-end data lifecycle in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/fundamentals/data-lifecycle)

3. [Fabric Data Engineering documentation](https://learn.microsoft.com/en-us/fabric/data-engineering/)

    3.1. [Fabric Data Engineering documentation. Summary of library management best practices](https://www.google.com/url?sa=E&source=gmail&q=https://learn.microsoft.com/en-us/fabric/data-engineering/library-management%23summary-of-library-management-best-practices)

4. [Microsoft OneLake documentation](https://learn.microsoft.com/en-us/fabric/onelake/)

5. [Fabric Real-Time Intelligence documentation](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/)

6. [Microsoft Fabric decision guide: copy activity, Copy job, dataflow, Eventstream, or Spark](https://learn.microsoft.com/en-us/fabric/fundamentals/decision-guide-pipeline-dataflow-spark)

7. [Limitations of Fabric Data Warehouse](https://learn.microsoft.com/en-us/fabric/data-warehouse/limitations)

    7.1. [Get Started with Fabric DWH](https://learn.microsoft.com/en-us/fabric/fundamentals/decision-guide-data-store?toc=/fabric/data-warehouse/toc.json&bc=/fabric/data-warehouse/toc.json)

    7.2. [Transact-SQL reference (Database Engine)](https://learn.microsoft.com/en-us/sql/t-sql/language-reference?view=fabric&preserve-view=true)

    7.3. [sql-server-samples](https://github.com/Microsoft/sql-server-samples/tree/master/samples)

    7.4. [CREATE SCHEMA](https://learn.microsoft.com/en-us/sql/t-sql/statements/create-schema-transact-sql?view=fabric&preserve-view=true)

    7.5. [DROP SCHEMA](https://learn.microsoft.com/en-us/sql/t-sql/statements/drop-schema-transact-sql?view=fabric)

    7.6. [CREATE TABLE](https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-azure-sql-data-warehouse?view=fabric)

    7.7. [DROP TABLE](https://learn.microsoft.com/en-us/sql/t-sql/statements/drop-table-transact-sql?view=fabric)

    7.8. [CREATE VIEW](https://learn.microsoft.com/en-us/sql/t-sql/statements/create-view-transact-sql?view=fabric)

    7.9. [DROP VIEW](https://learn.microsoft.com/en-us/sql/t-sql/statements/drop-view-transact-sql?view=fabric)


8. [Limitations in SQL database in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/database/sql/limitations)

9. [What is the SQL analytics endpoint for a lakehouse?](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-sql-analytics-endpoint)

10. [Databricks tables](https://docs.databricks.com/aws/en/tables)


11. [Why Microsoft Fabric’s Copy Data Templates Might Fail Your Production ETL (And How to Fix It)  Part 1](https://medium.com/@pashashcherbukha/why-microsoft-fabrics-copy-data-templates-might-fail-your-production-etl-and-how-to-fix-it-part-1-1118e804e8a1)

12. [Why Microsoft Fabric’s Copy Data Templates Might Fail Your Production ETL (And How to Fix It)  Part 2](https://medium.com/@pashashcherbukha/why-microsoft-fabrics-copy-data-templates-might-fail-your-production-etl-and-how-to-fix-it-4f644628e9d7)

13. [Why Microsoft Fabric’s Copy Data Templates Might Fail Your Production ETL (And How to Fix It)  Part 3](https://medium.com/@pashashcherbukha/why-microsoft-fabrics-copy-data-templates-might-fail-your-production-etl-and-how-to-fix-it-24ee20204c29)

14. [What is the Microsoft Fabric Capacity Metrics app?](https://learn.microsoft.com/en-us/fabric/enterprise/metrics-app)


15. [Microsoft Fabric: External repositories](https://learn.microsoft.com/en-us/fabric/data-engineering/environment-manage-library#external-repositories)

16. [Microsoft Fabric: Summary of library management best practices](https://learn.microsoft.com/en-us/fabric/data-engineering/library-management#summary-of-library-management-best-practices)


17. [Rihab Feki: Microsoft Fabric](https://rihab-feki.medium.com/)

18. [Microsoft Fabric, Code, and CI/CD Pipelines — Designing Your Deployment Workflow: Managing DEV, TEST, and PROD Environments](https://blog.devops.dev/microsoft-fabric-code-and-ci-cd-pipelines-designing-your-deployment-workflow-managing-dev-bb10b1f67d0b)

19. [How to maintain sanity between DEV-STG-PROD in Fabric? — Tracking Changes via Deployment Pipeline](https://uselessai.in/how-to-maintain-sanity-between-dev-stg-prod-in-fabric-tracking-changes-via-deployment-pipeline-984cb201f5d2)

20. [How to Design a 3‑Tier Workspace Architecture in Microsoft Fabric](https://medium.com/data-science-collective/designing-a-secure-workspace-architecture-in-microsoft-fabric-db7fd112f200)


21. [Expressions and functions for Data Factory in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/data-factory/expression-language) .



22. [Лінк на документацію  про ролі в Workspaces наведено тут:Microsoft Fabric workspace roles](https://learn.microsoft.com/en-us/fabric/fundamentals/roles-workspaces#-workspace-roles).

<kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-05-1.png" /></kbd>
<p style="text-align: center;"><a name="pic-05-1">pic-05-1</a></p>


23. [What is Fabric User data functions?](https://learn.microsoft.com/en-us/fabric/data-engineering/user-data-functions/user-data-functions-overview)

24. [Create a Fabric User data functions item](https://learn.microsoft.com/en-us/fabric/data-engineering/user-data-functions/create-user-data-functions-portal)


25. [OneLake security and item permissions](https://learn.microsoft.com/en-us/fabric/onelake/security/data-access-control-model#onelake-security-and-item-permissions).

26. [Row-level security in OneLake preview](https://learn.microsoft.com/en-us/fabric/onelake/security/row-level-security)

27. [Recommended architecture Best practices for OneLake security](https://learn.microsoft.com/en-us/fabric/onelake/security/best-practices-secure-data-in-onelake#primary-pattern)


## <a name="p-9">Оновлення та Roadmap Fabric</a>

1. [Connect to the Microsoft Fabric Release Plan report](https://learn.microsoft.com/en-us/fabric/fundamentals/service-connect-fabric-release-plan)

