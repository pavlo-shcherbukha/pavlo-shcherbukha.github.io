---
layout: default
title: Projects
permalink: /projects/
---
# Projects

##  [Google-sheet-to-db](https://github.com/pavlo-shcherbukha/google-sheet-to-db)

How to save data from google sheet to mysql database using google Apps script

The task is:

1. Create Mysql DB with any 10 columns
2. Write CRUD API server in node.js to manipulate data in created at step 1 DB via API
3. Create 10 columns table in Google sheet, fill 3 rows with dummy data
4. Write Google App Script application to connect spreadsheet with db via API created in step 2
5. You should be able to add/update all data from table to DB

Node.js application and mysql db server will be deployed on Red Hat Openshift cluster using openshift sendbox.

Google sheet and Google Apps Script will be on google. It is obvious

##  [Що таке ServerLess IBM Cloud Functions. Як з ним почати працювати](https://github.com/pavlo-shcherbukha/icf-r-01)

IBM Cloud Functions – це один из сучасних інструментів для хмарних платфлом, що реалізує концепцію Function-as-a Service (FAAS). 
Інструмент побудований на базі open-source  платформи Apache OpenWhisk.

Основнве особливості: 
- Не потребує app серверів  (Serverless)
- Пришвидшує розробку функцій.
- Можна використовувати  для запуска різних триггерів  чи - запускати за розкладом
- Не потребує  оплати, коли функція не працює (на відміну від app серверів )
- Автоматично масштабурється
- Може бути виставлена як API для інтеграції з іншими сервісами
- Можлива реалізація функцій  на популярних мовах програмувания  

Тут можна ознайомитися з лабораторними роботами:

- Ознайомитися з IBM Cloud Functions  на IBM Cloud Dashboard
- Створення простої функції з шаблону
- Створення власної вункції
- Як виставити функцію, яу REST API
- Знайомство с IBM CLI


## [IBM Cloud functions для середнього рівня](https://github.com/pavlo-shcherbukha/icf-r-02)

Продовження про IBM Cloud functions. 

* Інтеграція IBM Cloud Functions з хмарними сервісами  на прикладі  NoSqlDB IBM Cloudant
* Організація колективнох розробки
* Інтеграція IBM Cloud Functions з хмарним сервісом Cloud Object Storage (COS)
* IBM CLoud Functions  Розробка функцій з використанням сторонніх бібліотек.


## [WorkShop IBM-CLOUD-STARTER 2021. Building Cloud Native Application](https://github.com/pavlo-shcherbukha/IBM-CLoud-Starter)


Вступна  презентація про хмарні технології та їх використаня.


- Хмарні технології = Контейнери
- Хмарна архітектура на прикладі OpenShift
- Cloud-Native App and Microservices 
- Стек розробки (Development Stack)
- IBM-Cloud Catalog
- Lab-0 Встановлення програмного забезпечення
- Lab-1 Створення облікового запису на IBM-Cloud
- Lab-2 Приклад створення складного сервісу на базі Node-Red
- Lab-3 Створення простого Node.JS Back-end та Deployment в  public cloud використовуючи CI/CD toolchain


## [bnkapi - Побудова multicloud CloudNative додатків на платформі Open-Shift](https://github.com/pavlo-shcherbukha/bankapi-demo)

Метою цієї роботи є побудова моделі банківського BackEnd додатка з використанням Cloud-Native технологій у локальному центрі обробки даних на базі контейнерної платформи Open-Shift. Функціональність локального центру обробки даних буде розширена шляхом побудови гібридної хмарної інфраструктури та публікацією API у IBM-CLOUD. Опублікований API можна використовувати іншими програмами-клієнтами, наприклад, Watson Assistant у хмарі IBM, або клієнтами-сервісами інших виробників.

## [Інтеграційні рішения з використанням Watson Assistant ( чатбот в IBM Cloud)](https://github.com/pavlo-shcherbukha/WatsonAssistant-inegration-demo)

Показан побудова моделі віртуального асистента на базі Watson Assistant з використанням кількох варіантів інтеграції.

- Інтеграція з використанням IBM Cloud Functions
- Інтеграція з використанням Custom Application
