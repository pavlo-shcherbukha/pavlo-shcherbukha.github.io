---
layout: post
title: "FABRIC DEVELOPER WIKI"
date: 2026-09-01 10:00:01
categories: [spark, Microsoft Fabric]
permalink: posts/2026-09-01/Fabricdeveloperwiki/
published: true
---



# FABRIC DEVELOPER WIKI

<!-- TOC BEGIN -->

- [1. Проблема установки додаткових пакетів pyhton або управління pyhton environment](#p-1")
- [2. Генерація тестових даних](#p-2)
- [3.  Створення бази данних](#p-3)
- [4. Тип числових даних](#p-4)
- [5. Шляхи до файлів, що на LakeHouse в каталозі Files](#p-5)
- [6. Службові функції по роботі з файлами від Microsoft notebookutils](#p-6)
- [6.1. Повне видалення папки та її створення заново (найшвидший)](#p-6.1)
- [6.2. Видалення лише вмісту, якщо треба зберегти папку](#p-6.2)
- [7. На що звернути увагу при аудиті](#p-7)
- [7.1. Чек-лист для аудиту, щоб розібратися в коді](#p-7.1)
- [8. Помилки Capacity та як з ними боротися](#p-8)
- [8.1. Отримали помилку: TooManyRequestsForCapacity: Помилка 430](#p-8.1)
- [8.2. CapacityLimitExceeded (429) у Fabric Trial: обмеження безкоштовної потужності](#p-8.2)
- [8.3. ConcurrentAppendException: декілька процесів намагаються одночасно додати (append) дані в одну й ту саму таблицю<](#p-8.3)
- [9. Правила кодінгу чи розробки - Coding Good Practice](#p-9)
- [9.1. Візуальні кубики VS кодінг](#p-9.1)
- [9.2. Налаштування WorkSpace](#p-9.2)
- [9.3. Кодінг в notebook](#p-9.3)
- [9.4. Автоматизація через Pipelines](#p-9.4)

- [10. Microsoft Fabric workspace roles](#p-10)
- [11. Fabric DWH](#p-11)
- [11.1. Лінки на різного роду документацію по DWH](#p-11.1)
- [11.2. Огляд особливостей та відмінностей Fabric DWH від звичних реляційних баз даних](#p-11.2)
- [11.3. Good Practice по проектуванні структури Fabric DWH](#p-11.3)

- [12. Використання функцій Fabric](#p-12)

- [13. Прототипування](#p-13)


<!-- TOC END -->

## <a name="p-1"> Проблема установки додаткових пакетів pyhton або управління pyhton environment</a>


**Fabric Environment** ,  що є еквівалентом **python virtual environment**
Відображається обведено рамкою

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

Ви все правильно помітили. Скріншот чітко показує, що ваш ноутбук зараз підключений до "Workspace default".

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
    Виберіть Data Engineering/Science -> Spark settings.
    Там є вкладка Environment.
    Виберіть ваш створений Environment як Set as default.

## <a name="p-2">Генерація тестових даних</a>

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

Ось лайфхак, щоб не робити генерацію в циклі for (що в Spark повільно):

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

## <a name="p-3">Створення бази данних</a>

У Fabric ми не створюємо бази даних через CREATE DATABASE. Кожен Lakehouse — це і є наша база даних. Якщо треба розділити логіку (наприклад, для платежів), створюємо новий Lakehouse trm_payment_lh і додаємо його в Explorer ноутбука."

Варіант Б: Кілька Lakehouse (Архітектурний)

Ви створюєте окремий об'єкт Lakehouse під назвою trm_payment_lh.

    Чому це круто: Кожен Lakehouse має свій SQL Endpoint. Ви можете давати права доступу колегам на весь "платіжний" Lakehouse окремо від інших даних.

    Як звернутися з одного ноутбука до іншого: Ви просто додаєте обидва Lakehouse до вашого ноутбука (кнопка Add data items зліва на вашому скрині). Після цього ви можете звертатися до них через повне ім'я:

```py
SELECT * FROM psh_exch_lh.table1
UNION 
SELECT * FROM trm_payment_lh.table2
```

Якщо ви хочете побачити, де ви зараз "знаходитеся", виконайте:

```py

print(spark.catalog.currentDatabase())
```

## <a name="p-4">Тип числових даних</a>

Оскільки працюємо з фінансами (amount, charge), при завантаженні з JSON Spark може визначити їх як double. В облікових задачах **краще одразу кастувати їх до DecimalType(18,2)**, щоб уникнути проблем з плаваючою комою, можемо вионувати приведення типів як в прикладі:

```py

df1 = spark.read.json(f"{file_pth}/{file_name}")
df_fixed = df1.withColumn("amount", col("amount").cast("decimal(18,2)")) \
              .withColumn("charge", col("charge").cast("decimal(18,2)"))

```

Розбір причини (Overflow)

У JSON значення "charge": 510.22.
Ваше визначення в DDL: CHARGE DECIMAL(3, 2).

    Перша цифра (Precision = 3): Це загальна кількість цифр у числі (і до, і після коми).

    Друга цифра (Scale = 2): Це кількість знаків після коми.

    Результат: DECIMAL(3, 2) дозволяє зберігати числа лише від -9.99 до 9.99.

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


## 1. Додавання коментаря до всієї таблиці

```py

ALTER TABLE clients SET TBLPROPERTIES ('comment' = 'Новий опис: Таблиця містить верифіковані дані клієнтів банку');
```

Або простіший синтаксис (залежно від версії Spark, але цей надійніший):

```py
 COMMENT ON TABLE clients IS 'Ця таблиця зберігає історію транзакцій клієнтів';
```

## 2. Додавання/Зміна коментаря для конкретної колонки

```py
ALTER TABLE clients 
ALTER COLUMN email COMMENT 'Електронна пошта (обов’язково у форматі name@domain.com)';
```

Важливо: При зміні коментаря колонки тип даних (STRING, BIGINT тощо) залишається незмінним, змінюється лише метадата.


## 3. Перевірка результату

Щоб переконатися, що коментарі успішно додані, використовуйте команду DESCRIBE:

- Коротко: DESCRIBE clients (покаже колонки, типи та коментарі до колонок).

- Детально: DESCRIBE TABLE EXTENDED clients (покаже коментар до всієї таблиці в секції Detailed Table Information).


## <a name="p-5">Шляхи до файлів, що на LakeHouse в каталозі Files</a>

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

## <a name="p-6">Службові функції по роботі з файлами від Microsoft notebookutils</a>

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
mkdirs(dir: String): Boolean -> Creates the given directory if it does not exist, also creating any necessary parent directories
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

### <a name="p-6.1">Повне видалення папки та її створення заново (найшвидший)</a>

```py
# Шлях до вашої папки
path = "Files/bronze"

# Видаляємо папку рекурсивно (true означає видалити і всі вкладені файли/папки)
mssparkutils.fs.rm(path, True)

# Створюємо порожню папку назад
mssparkutils.fs.mkdirs(path)

print(f"Каталог {path} очищено.")

```

### <a name="p-6.2">Видалення лише вмісту (якщо треба зберегти папку)</a>

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

## <a name="p-7">На що звернути увагу при аудиті</a>


Це дуже зрілий підхід. Малювати архітектуру "As Is" (як є) — це найкращий спосіб виявити хаос, який часто ховається за гучними менеджерськими термінами. Коли ви побачите реальний шлях даних від джерела до звіту, "больові точки" випливуть самі собою: десь це дублювання даних, десь — надмірні витрати, а десь — "крихкі" Pipeline, які падають щопонеділка.

Ось на що варто звернути увагу в екосистемі Fabric саме з технічної точки зору:

1. Перевірка "Pipeline Health"

Коли дивимося їхні пайплайни, зверніть увагу на:

- Hardcoded paths: Чи зашиті шляхи до папок (як той самий Files/bronze) прямо в код, чи вони винесені в параметри?

- Retry Policy: Чи налаштовані автоматичні повтори при збоях? У хмарі мережеві "мигтіння" — це норма, і без ретраїв пайплайни стають нестабільними.

- Incremental Load: Чи вони щоразу перекачують всі дані з джерела (Full Load), чи тільки нові (Incremental)? Останнє — це величезна економія грошей і ресурсів клієнта.

2. Структура OneLake (File vs Table)

Часта помилка початківців у Fabric — зберігати все як файли в розділі Files.

Якщо дані вже структуровані (мають чіткі колонки), вони мають бути в Tables у форматі Delta. Це дає можливість використовувати SQL, пришвидшує роботу в рази завдяки механізму V-Order і дозволяє робити "Travel back in time" (відкат до попередніх версій даних).

3. Notebooks vs Data Factory

Подивитися, де вони роблять трансформації.

- Якщо все на Notebooks — це гнучко, але важко підтримувати (код може стати "макаронним").

- Якщо все на Data Factory (Dataflows Gen2) — це зручно для бізнес-користувачів, але іноді дорожче за ресурсами Spark.

- Best Practice: Поєднання. Важка обробка — в Spark Notebooks, оркестрація та прості переміщення в Pipeline.

Що можна запропонувати як "Quick Win" (швидка перемога):

-  Використати mssparkutils для аудиту обсягів — це ідеальний кандидат.

Можна запропонувати їм "Data Lake Health Check Report":

- Автоматичний розрахунок розмірів папок (те, що ми обговорили).

- Список "забутих" тимчасових файлів, які ніхто не видаляє.

- Вимырювання (розрахунок) та Візуалізація швидкості росту даних у Bronze за останній тиждень/місяць.



Коли  аналызуэмо архітектуру "As Is", звернути увагу на те, як дані потрапляють у Fabric. Це Shortcuts (посилання на інші хмари) чи реальний Copy Activity? Це критично для розуміння затримок (latency).


### <a name="p-7.1">Чек-лист для аудиту, щоб розібратися в коді</a>

Ось структурований чек-лист для аудиту, який допоможе  розібратися в коді і знайти конкретні аргументи для майбутньої презентації "To Be" архітектури.

Ці питання розділені на технічні блоки, щоб  «промацати» систему з різних сторін.

📋 Чек-лист аудиту Microsoft Fabric (As-Is)

1. Рівень даних (OneLake Storage)

- Співвідношення Files vs Tables: Який відсоток даних лежить у розділі Files (CSV/Parquet/JSON) замість Tables (Delta)?

Чому це важливо: Тільки Tables підтримують V-Order (фірмове стиснення Microsoft), яке робить звіти Power BI блискавичними.
Схема іменування: Чи є чіткий стандарт назв папок? (Наприклад: SourceSystem/Entity/Year/Month/).
Shortcuts: Чи використовуються "Shortcuts" (посилання на S3/ADLS/Dataverse)?
Ризик: Якщо дані не копіюються, а лише посилаються, це може створювати мережеві затримки при складних розрахунках.

2. Рівень обробки (Spark & Notebooks)

**Параметризація:** Чи зашиті (hardcoded) назви WorkspaceID, LakehouseID та шляхи до папок у ноутбуках?
Все має передаватися через параметри (mssparkutils.notebook.run(..., {"param": "value"})).

**Spark Session Management:** Чи налаштовані кастомні "Environments" (бібліотеки, версія Spark, потужність вузлів)?

Економія: Якщо для простого видалення файлів запускається потужний кластер на 10 вузлів — це "гроші на вітер".
Обробка помилок: Що станеться, якщо файл у bronze прийде з битою структурою? Чи є блоки try-except і логування помилок у окрему таблицю? Та і взагалі, як, щодо логування?

3. Рівень пайплайнів (Data Factory)

**Метод завантаження:** Це Full Overwrite (щоразу все заново) чи Incremental Load (тільки дельти)?

**Біль:** Повне перезавантаження великих обсягів — це найпоширеніша "больова точка" бюджету.

**Секрети та безпека:** Де зберігаються паролі до джерел? (Мають бути в Azure Key Vault, до якого Fabric звертається через mssparkutils.credentials).

**Оркестрація:** Чи є залежності між пайплайнами? Якщо Bronze не оновився, чи зупиниться автоматично Silver?

4. Рівень продуктивності та вартості

**Partitioning:**Чи партиційовані великі таблиці (наприклад, по даті)?
**Ефект:** Без партицій Spark змушений вичитувати всю таблицю навіть для звіту за один день.
**Vacuum & Optimize:**  Чи запускаються ці команди для Delta-таблиць?
Важливо: Без VACUUM старі версії файлів (після видалень/оновлень) будуть вічно займати місце і клієнт буде за них платити.

💡 Для "менеджерського" звіту:

Коли знайдемо проблему, фіксємо її за формулою:

Проблема (немає Incremental load) ➡️ Наслідок (довгий час очікування звітів + зайві витрати на Spark) ➡️ Рішення (впровадження Watermark-завантаження).

## <a name="p-8">Помилки Capacity та як з ними боротися</a>

### <a name="p-8.1">Отримали помилку: TooManyRequestsForCapacity (Помилка 430)</a>

Це класична ситуація для Trial ліцензії. Ви просто "вижали" максимум з того безкоштовного вузла, який вам виділив Microsoft.

Чому це сталося?
Fabric Trial має обмежену кількість CU (Capacity Units). Коли ви запускаєте масове читання (spark.read.json(json_files)), Spark намагається підняти паралельні процеси для кожного файлу. Якщо файлів багато, а потужність тріалу маленька (зазвичай це еквівалент F2 або F64, але з низьким пріоритетом), система спрацьовує як запобіжник: "Stop, ти споживаєш ресурси швидше, ніж дозволяє твій безкоштовний тариф".

Як це "лікувати":

- Зачекайте 2-5 хвилин: Fabric використовує систему "smoothing" (згладжування). Якщо ви дали пікове навантаження, вам треба трохи почекати, поки "штрафні бали" згорять.

- Обмежте паралелізм: Якщо ви використовували ThreadPoolExecutor, зменште max_workers до 2 або 3.

- Перевірте Monitoring Hub: Зліва на панелі Fabric є іконка спідометра (Monitoring hub). Зайдіть туди і завершіть (Cancel) усі завислі або старі сесії Spark. Вони можуть "тримати" ваші ліміти.

- Збільште час життя сесії: У налаштуваннях Workspace (Spark settings) можна виставити автоматичне завершення сесії через 10-20 хвилин, щоб вони не висіли марно.

"Помилка 430 (TooManyRequests): Це ознака 'throttling' (обмеження швидкості). Якщо виникає при масовій обробці:

- Не запускайте ноутбук занадто часто поспіль.

- Закрийте непотрібні вкладки з іншими ноутбуками (кожна вкладка — це активна сесія).

- Якщо файлів дуже багато, обробляйте їх пачками (batches), а не всі 500 одразу."


### <a name="p-8.2">"CapacityLimitExceeded (429) у Fabric Trial: Це не баг коду, а обмеження безкоштовної потужності</a>

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


###  <a name="p-8.3">ConcurrentAppendException: декілька процесів намагаються одночасно додати (append) дані в одну й ту саму таблицю</a>  

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

## <a name="p-9">Правила кодінгу чи розробки - Coding Good Practice</a>

### <a name="p-9.1">Візуальні кубики VS кодінг</a>

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

Якщо файлів дуже багато (сотні чи тисячі), краще не плодити багато сесій ноутбуків, а обробити все за один раз самим Spark:

- Замість того, щоб читати один файл 12144_2026-01-31.json, ви читаєте всю папку:
    df = spark.read.json("Files/bronze/*.json").

- Spark сам розподілить ці дані по вузлах кластера (Worker Nodes).

- Потім ви робите один великий MERGE для всього датафрейму.

Це набагато ефективніше, бо Spark оптимізує план виконання для всієї маси даних одразу.

Аргументи:

Коли вони кажуть, що "ноутбуки — це некеровано":

- **Source Control (Git Integration):** У Fabric ноутбуки підключаються до Azure DevOps або GitHub. Весь ваш код версіонується, як і будь-який інший бекенд-проект. Та і взагалі notebook в своєму бінарному вигляді являє собою json-файлію. Компоненти Pipeline  та і самі Pipуline теж являють собою json-файли. Тобто, немає різниці, що мержити. Треба тільки мати на увазі, що json-файли всі мержаться погано, особливо коли багато змін чи  конфліктів. 

- **Compute Isolation:** Ви можете призначити ноутбуку конкретний "Pool" ресурсів, щоб він не "з'їв" пам'ять всього воркспейсу.

- **Unit Testing:** У ноутбуці ви можете написати тести для своєї логіки трансформації, чого майже неможливо зробити у візуальних Mapping Data Flows без болю.

- **Якщо комусь хочеться візуальності:** Використовуйте Data Wrangler у Fabric. Це інструмент всередині ноутбука, який генерує Python-код для очищення даних через графічний інтерфейс. Це такий собі "компроміс": ви працюєте візуально, але на виході — чистий, професійний код у клітинці ноутбука.

- **Декомпозиція пайплайнів:**

- Parent Pipeline: Відповідає за оркестрацію (перелік файлів, цикл).

- Child Pipeline: Відповідає за unit-роботу (обробка одного об'єкта).

**Перевага:** Це дозволяє уникнути 'прісного' коду та спрощує відстеження помилок через Output конкретної ітерації For Each."


### <a name="p-9.2">Налаштування WorkSpace</a>


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

Підтвердження, що такий підхід прауює поаказано на [pic-04-8](#pic-04-8) 
<kbd><img src="doc/pic-04-8.png" /></kbd>
<p style="text-align: center;"><a name="pic-04-8">pic-04-8</a></p>


У розподілених системах, таких як Apache Spark, різниця між «локальною зміною оточення» та «конфігурацією кластера» є критичною. Тому os.getenv() не працює і боротися за os.getenv() у контексті Fabric/Spark справді не варто.

Коли ви виконуєте import os; os.environ[...] = ... у клітинці ноутбука, ви змінюєте оточення лише на Driver node (вузлі-керівнику).

Якщо ваш код виконує трансформації локально (наприклад, підключення до API в циклі на драйвері), os.getenv спрацює.

Але якщо ви захочете використати цей секрет всередині Spark udf (User Defined Function) або при паралельному зчитуванні даних, ваші Executors (робочі вузли) про цю змінну нічого не знатимуть.

Варіант зі spark.conf гарантує, що конфігурація прокидається через SparkContext на всі вузли кластера автоматично.

### <a name="p-9.3">Кодінг в notebook</a>

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



### <a name 9.4>Автоматизація через Pipelines</a>

- Обробка помилок: Використовуйте "On Fail" гілку (червона стрілка) для відправки сповіщень про помилки чи запуску процесу retry.

- Schedule: Налаштовуйте запуск (Trigger) раз на годину або по події появи файлу"

- Повернення результату виконання з Child pipline в Parent pipline використовуйте елемент **Set Veriable** з параметром PipeLine Return Value

 <kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-04-1.png" /></kbd>
<p style="text-align: center;"><a name="pic-04-1">pic-04-1</a></p>

Якщо хочемо звернутися до параметрів pipline  треба використовувати **Expressions**. Лінк на документацію: [Expressions and functions for Data Factory in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/data-factory/expression-language) .

**Приклади:**

**@activity('Mergilka').output.result.exitValue** отримати результат роботи попереднього кубика з назвою 'Mergilka' як String

**@json(activity('Mergilka').output.result.exitValue)** отримати результат роботи попереднього кубика з назвою 'Mergilka' як json

**@pipeline().RunId** отримати  runid  поточного pipeline 

**@pipeline().PipelineName** отримати найменування поточного pipline

- Для того, щоб в рамках одного pipekine треба передати якусь змінну, до якої будуть звертатися інші кубики, що стоять далеко попереду поточного використовуйте елемент **Set Veriable** з параметром PipeLine Variable

<kbd><img src="doc/pic-04-2.png" /></kbd>
<p style="text-align: center;"><a name="pic-04-2">pic-04-2</a></p>


## <a name="p-10">Microsoft Fabric workspace roles</a>

Лінк на документацію наведено тут:
[Microsoft Fabric workspace roles](https://learn.microsoft.com/en-us/fabric/fundamentals/roles-workspaces#-workspace-roles).

<kbd><img src="/assets/img/posts/2026-09-01-fabricdevwiki/doc/pic-05-1.png" /></kbd>
<p style="text-align: center;"><a name="pic-05-1">pic-05-1</a></p>


Про контроль доступа до OneLake можна почитати тут: 

- [OneLake security and item permissions](https://learn.microsoft.com/en-us/fabric/onelake/security/data-access-control-model#onelake-security-and-item-permissions).

- [Row-level security in OneLake preview](https://learn.microsoft.com/en-us/fabric/onelake/security/row-level-security)

- [Recommended architecture Best practices for OneLake security](https://learn.microsoft.com/en-us/fabric/onelake/security/best-practices-secure-data-in-onelake#primary-pattern)


## <a name="p-11">Fabric DWH</a>

### <a name="p-11.1">Лінки на різного роду документацію по DWH</a>
- [Get Started with Fabric DWH](https://learn.microsoft.com/en-us/fabric/fundamentals/decision-guide-data-store?toc=/fabric/data-warehouse/toc.json&bc=/fabric/data-warehouse/toc.json)

- [Transact-SQL reference (Database Engine)](https://learn.microsoft.com/en-us/sql/t-sql/language-reference?view=fabric&preserve-view=true)

- [sql-server-samples](https://github.com/Microsoft/sql-server-samples/tree/master/samples)

- [CREATE SCHEMA](https://learn.microsoft.com/en-us/sql/t-sql/statements/create-schema-transact-sql?view=fabric&preserve-view=true)

- [DROP SCHEMA](https://learn.microsoft.com/en-us/sql/t-sql/statements/drop-schema-transact-sql?view=fabric)

- [CREATE TABLE](https://learn.microsoft.com/en-us/sql/t-sql/statements/create-table-azure-sql-data-warehouse?view=fabric)

- [DROP TABLE](https://learn.microsoft.com/en-us/sql/t-sql/statements/drop-table-transact-sql?view=fabric)

- [CREATE VIEW](https://learn.microsoft.com/en-us/sql/t-sql/statements/create-view-transact-sql?view=fabric)

- [DROP VIEW](https://learn.microsoft.com/en-us/sql/t-sql/statements/drop-view-transact-sql?view=fabric)

 
### <a name="p-11.2">Огляд особливостей та відмінностей Fabric DWH від звичних реляційних баз даних</a> 

Відмінність від реляційних баз даних полягає в тому, що Fabric DWH це T-SQL Engine  натягнутий проверх DataLake, щоб забезпечити роботу PowerBi. Відповідно, випала та функціональність, що не підтримується DataLake.

- Первинні ключі  Deffered (відкладені), тобто вони не використовуються при вставці даних. Це як індекс, для підказки оптимізатору при операціяї SELECT. Як наслідок, легко можуть появитися дублі.

- Відсутні Foreign Key

Вони просто відсутні. Вони моделюються емантичними моделями для підтримки PowerBI.

- Відсутність Індексів:  У Fabric DWH немає CREATE INDEX. Система покладається на Columnstore (стиснення по колонках) та розподілені обчислення. Ваша задача — правильно підібрати типи даних, щоб не "роздувати" таблиці.

- Transaction Isolation: Тут підтримується рівень Snapshot Isolation. Тобто читачі не блокують письменників (схоже на те, як працює Undo Tablespace в Oracle, але реалізовано через версіонування файлів)

- View з GROUP BY: чи це «онлайн»?

У Fabric Warehouse звичайні VIEW завжди обчислюються динамічно (онлайн).

```text
    Як це працює: Коли ви робите SELECT * FROM MyView, рушій Polaris підставляє код View 
    у ваш запит і виконує агрегацію (Group By) в момент звернення.

    Чи є Materialized Views? У класичному розумінні (як в Oracle, де дані фізично зберігаються) — ні. У Fabric Warehouse зараз немає індексованих або матеріалізованих View.

    Проблема продуктивності: Якщо у вас мільйони рядків і складні Join-и всередині View з Group By, Power BI може «підторможувати».

    Рішення: Якщо продуктивність динамічного View стає проблемою, розробники Gold-шару зазвичай перетворюють це на фізичну таблицю (через INSERT INTO ... SELECT або CREATE TABLE AS SELECT у Pipeline), яку оновлюють за розкладом.
```

- Функції та Процедури в Fabric DWH

**Stored Procedures** (Процедури): Це основний інструмент ELT. Замість того, щоб налаштовувати трансформацію у візуальному Pipeline, ви кладете весь свій SQL-код у процедуру. Pipeline просто викликає її однією командою Execute Stored Procedure. Це набагато легше версіонувати та дебажити.

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

- Відсітні Sequence (як в ORACLE) чи IDENTITY (як в MS SQL). По факту треба використовувати такі підходи:

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
Перевага: Ви можете завантажувати дані паралельно з 10 різних джерел, і вам не потрібно звертатися до жодної центральної таблиці чи Sequence. Один і той самий термінал завжди отримає один і той самий ID.

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


- Правила видимості об'єктів в БД



### <a name="p-11.3">Good Practice по проектуванні структури Fabric DWH</a>

2. Поточна версія Fabric дозволяє створювати в рамках одного DWH різні схеми даних  (як в OTRACLE).

```py
/* 1. Створення схеми, якщо вона не існує */
IF NOT EXISTS (SELECT * FROM sys.schemas WHERE name = 'payment')
BEGIN
    EXEC('CREATE SCHEMA [payment]')
END
```

Тобто, замість сувати все в схему dbo, таблиці різних додатків (різних сутностей) можна рознести по  схемам.

3. Не зважаючи на те, що в Source Control  попадають DDL всіх об'єктів бази даних, зміни DDL в БД краще вести окремо. На приклад в спеціалізовній  Notebook. Крім того, треба мати на увазі, що  в SQL explorer Fabric DWH в папці **Queries** (**Queries/my queries**, **Queries/shared queries** ) - не зберігаються в Source Control

Як на мій погляд то Notebook вигідніше одного або кількох sql скриптів (файлів з типом *.sql) тому що: 

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

4. При створенні об'єктів бази даних треба користуватися таким підходм (структурою):

- CREATE SCHEMA [private]; — для таблиць. Тут лежать фізичні таблиці

- CREATE SCHEMA [public]; — для View. Тут створюється перший рівень абстракації, коли користувач звітів не має прямого доступу до таблиць і не має інформації про їх фізичну структуру. Усі VIEW у схемі public робимо просто як проксі:


```SQL

    CREATE VIEW [public].[Sales] AS SELECT * FROM [private].[FactSalesOrder];
```

Таким чином ми  ізолюємо фізичне зберігання від логічного представлення. Це дозволяє нам змінювати структуру таблиць, не ламаючи звіти користувачів.


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


## <a name="p-12">Використання функцій Fabric</a>

- [What is Fabric User data functions?](https://learn.microsoft.com/en-us/fabric/data-engineering/user-data-functions/user-data-functions-overview)

- [Create a Fabric User data functions item](https://learn.microsoft.com/en-us/fabric/data-engineering/user-data-functions/create-user-data-functions-portal)


## <a name="p-13">Прототипування</a>

### <a name="p-13.1">Оброка файлів за подіями eventStream</a>

Опис прототипу знаходиться за лінком: [Event Driven file uploading](shcherbukha.github.io/posts/2026-05-01/Fabric.EventDrivenFileProcessing-en/)

### <a name="p-13.2">Оброка даних з датчиків в RealTime та перетворення сирих даних в бізнес - сутності</a>

Опис прототипу знаходиться за лінком:
[Microsoft Fabric. Прагматичний AI та Цифрові двійники: Чому 5 рядків математики іноді цінніші за гігабайтні нейромережі](https://pavlo-shcherbukha.github.io/posts/2026-06-10/Fabric.%20AL-vs-Engeneering/)