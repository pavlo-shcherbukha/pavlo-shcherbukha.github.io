---
layout: post
title: "Python Flask app with TLS on openshift"
date: 2023-03-30 10:00:01
categories: [Python, Flask, openshift]
permalink: posts/2023-03-30/py_flask_app_on_openshift_tls/
published: true
---

<!-- TOC BEGIN -->

<!-- TOC END -->

## <a name="p-1">Постановка проблеми</a>

При розробці сучасних багатосервісних систем частно виникає необхідність захисту даних на трансопортному рівні. Тому це було першопричиною для того щоб розібратися в цьому, а потім написати цей блог. А в процесі вивчення тми виникли додаткові аргументи в цій сфері.

На приклад маємо таку архітектуру компонентів:

```mermaid
    C4Container
    title Service oriented app
    
    Person(customer, Customer, "A customer in front of the laptop")

    Container_Boundary(c2, "Application services") {
        Container(app1_service, "Back End service", "", "")
        Container(app2_service, "Back End service", "", "")
    }

    Container_Boundary(c1, "Database Rest API") {
       
        Container(appdbservice, "Rest API", "", "Ineract with  database only")
        ContainerDb(database, "Database", "SQL Database")
       

    }

    Rel( customer, app1_service, "user interacttion","http", "1")
    Rel( customer, app2_service, "user interacttion","http", "2")
    
    Rel(app1_service,  "appdbservice", "http", "3")
    Rel(app2_service,  "appdbservice", "http", "4")

    BiRel(appdbservice,  "database", "sql-net", "5")

    Rel(customer,  "appdbservice", http, "Forbidden interaction", "0")

    UpdateRelStyle(customer,  "appdbservice" $textColor="red",$lineColor="red", $offsetY="-80", $offsetX="-120")


    UpdateRelStyle(customer, app1_service, $textColor="blue",$lineColor="blue", $offsetY="-50", $offsetX="10")
    UpdateRelStyle(customer, app2_service, $textColor="blue",$lineColor="blue", $offsetY="-140", $offsetX="-110")
    
    UpdateLayoutConfig($c4ShapeInRow="1", $c4BoundaryInRow="2")

```

Синіми та чорними лініями показана дозволена взаємодія клієнта (сині) та сервісів (чорні). Червона лінія показує  заборонену взаємодію. Може виникнути питання чому ця дія заборонена. На компоненті RestApi скоріше за все немає аутентифікації користувача. Більш того, backend сервіси можуть організовувати с RestApi сервісом взаємодію, по специфічному шаблону взаємодії типу: webpooling чи webhook -  для того щоб не перевантажити сервіс, чи отримати дані послідовно, порціями, а потім користувачу віддати вже все. При чому, customer  це не завжди вреднючий користувач. Просто він користується такми фронтом, де програміст зробив "покращення", тому що так швидше.


Тут стає питання, як забезпечити тількт дозволені комунікації між сервісами. Більш того, За звичай існує 2 середовища: продуктивне та тестове. І, потрібно максимально ізолювати їх між собою. І не завжди мереживні технології дозволяють це зробити. Тому, одним із швидих  методів дозволити тільки очевидні комунікації між сервісами може забезпечити використання TLS  протоколів з різними типами атворизації.



<iframe
  src="https://github.com/pavlo-shcherbukha/tls-self-sign-certs#tls-self-sign-certs-%D0%BD%D0%B0%D0%B1%D1%80%D1%96-%D0%BA%D1%80%D0%BE%D0%BA%D1%96%D0%B2-%D0%B4%D0%BB%D1%8F-%D0%B3%D0%B5%D0%BD%D0%B5%D1%80%D0%B0%D1%86%D1%96%D1%97-%D1%81%D0%B0%D0%BC%D0%BE%D0%BF%D1%96%D0%B4%D0%BF%D0%B8%D1%81%D0%BD%D0%B8%D1%85-tls--%D1%81%D0%B5%D1%80%D1%82%D0%B8%D1%84%D1%96%D0%BA%D0%B0%D1%82%D1%96%D0%B2"
  style="width:100%; height:300px;"
></iframe>



