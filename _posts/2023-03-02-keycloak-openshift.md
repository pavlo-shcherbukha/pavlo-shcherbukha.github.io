---
layout: post
title: "Keycloak vue.js node.js couchdb on openhsift"
date: 2023-03-02 10:00:01
categories: [keycloak, node.js, vue.js, couchdb, openshift]
permalink: posts/2023-03-02/Keycloak vue.js node.js couchdb on openhsift/
published: true
---

<!-- TOC BEGIN -->

- [1. Інтеграція keycloak з vue.js, Node.js express в openhsift](#p-1)
- [2. Розгортаня keycloak  в OpenShift](#p-2)
- [3. Основні терміни адмінітрування Keycloak](#p-3)
<!-- TOC END -->

## <a name="p-1">Інтеграція keycloak з vue.js, Node.js express в openhsift </a>

Продук [Keycloack](https://www.keycloak.org/) є зараз типовим  інструментом для авторизації в Web-based системах.  Документацію можна знайти за лінком: 
- Основна документація знаходиться за лінком [documentation](https://www.keycloak.org/documentation);
- За цим лінком знаходяться більш фокусовані описи [Фокусовані описи](https://www.keycloak.org/guides).

Ну на цьому зацікавився цим продуктом, щоб трохи розібратися як він працює та як його конфігурвати.


## <a name="p-2">Розгортаня keycloak  в OpenShift</a>

За звичай keycloak  можна підняти в контейнері, але він страртує в development mode. Так також там треба прив'язати базу даних типу postgresql. Я вирішив піти більш простим шляхом і роззоргнув його в хмарі RadHat в sendbox  OpsnShift.  Openshift sendbox можна створити за url: https://developers.redhat.com/developer-sandbox, а перед цим потрібно зарєструватися як developer на RedHat. Docker відкинув зразу, тому що  прочитав ліцензійні обмеження для корпорацій.

Щоб не мучитися з  лінкуванням бази даних та самого  сервісу keycloak  я використав калог уже підготованих продуктів, що є вже в любому OpenShift і в пару кліків розгорнув додаток [pic-01](#pic-01). 

<kbd><img src="../assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-01.png" /></kbd>
<p style="text-align: center;"><a name="pic-01">pic-01</a></p> 

Через кілька хвилин я уже маю готовий keycloak:
<kbd><img src="../assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-02.png" /></kbd>
<p style="text-align: center;"><a name="pic-02">pic-02</a></p> 

І уже натискаємо на роут [pic-03](#pic-03), попадаємо в консоль адміністрування. Можливо потрібно трохи зачекати, поки обидва сервіса стартонуть. Ну, там час старту такий собі, відчутний, але не критично. 

<kbd><img src="../assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-03.png" /></kbd>
<p style="text-align: center;"><a name="pic-03">pic-03</a></p> 


Консоль адміністрування запросить логін та пароль. Вони знаходяться в env змінних Keycloak  так як показано на   [pic-04](#pic-04)

<kbd><img src="../assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-04.png" /></kbd>
<p style="text-align: center;"><a name="pic-04">pic-04</a></p>

Залогінился і вуаля - попадаємо в  консоль адмінітрування, в головний realm

<kbd><img src="../assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-05.png" /></kbd>
<p style="text-align: center;"><a name="pic-05">pic-05</a></p>


## <a name="p-3">Основні терміни адмінітрування Keycloak</a>

Адміністрування в keycloak ділиться на **realm**-и. 
- **Realm**  це одниця адміністрування, що інкапсулює в собі набори: користувачів, ролей, груп та набори додаткових повноважень (чи  прав). Тому, переше, що робимо,  реєструємо свій realm. Назвемо його **shdemorealm**. На [pic-06](#pic-06) показані основні елементи realm.

<kbd><img src="../assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-06.png" /></kbd>
<p style="text-align: center;"><a name="pic-06">pic-06</a></p>

Keycloak підтримує як OpenID Connect (розширення OAuth 2.0), так і SAML 2.0. Коли говоримо про security, перше, що потрібно вирішити, це те, що з двох ви збираєтеся використовувати. Я вирішую використовувати OpenID Connect (розширення OAuth 2.0), тому в подальшому мова йде тільки про нього.

- **Client** - це прикладні додатки, що зареєстровані в окремо взятій realm. За звичай додатки аутентифікуються за допомогою clinet ID  або client ID та client secret. Ще є аутентифікація через JWT  токен, але зараз її не розглядаємо. Client ID and Client Secret - це традиційний метод, описаний у специфікації OAuth2. Клієнт має секрет, який повинен бути відомий як прикладному додатку, так і серверу Keycloak.

### Ручна реєстрація програми клієнта

Створюємо клієнта в client ID **DemoApp1**, як показано на [pic-06](#pic-06):

<kbd><img src="../assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-06.png" /></kbd>
<p style="text-align: center;"><a name="pic-06">pic-06</a></p>


Після реєстрації зразу попадаємо на вікно [pic-07](#pic-07)

<kbd><img src="../assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-07.png" /></kbd>
<p style="text-align: center;"><a name="pic-07">pic-07</a></p>

Тут треба звернути увагу на **Access Type**=public при логіні не поребує client Secret.  А при **Access Type**=confidential потребує client Secret. Зараз залишаємо **public**.


Треба звернути увагу на **Standard Flow Enabled**  треба включити в **"ON"**. Таким чином авторизація піде по стандартному OAuth2.0 за допомогою обміну autorization_code.

Далі потрібно налаштувати URL  додатка так, як показано на [pic-08](#pic-08).

<kbd><img src="../assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-08.png" /></kbd>
<p style="text-align: center;"><a name="pic-08">pic-08</a></p>

Тобто, поки що нас цікавлять Root URL  та "правильний" redirect URL.  По ньому відбувається  переадресація у випадку успішного  логіна. Інші, пока що не задіяні.

Все, можна сказати в найпростішому варіанті клієнт-додаток зареєстровано.

### Налаштування прикладних ролей для програми клієнта


Після  реєстрації додатку-клієнта  потрібно запараметризувати рольовий доступ до ресурсів програми. Рольовий досутп може ділитися на  по ролям. А ролі діляться на Realm Roles  та Client Roles. 

- Realm Roles  доступні для всього realm. Цей варіант не зовсім вигідний. Але прийнятний в деяких випадках
- Client Roles -  доступні для окремо зареєстрованого клієнта. Цей варіант, як на мене, більш прийнятний. Тому на ньому і зупинимося.

Для нашого майбутньго додатку створемо 2 ролі:

- app_viewer дозволяє тільки читати дані:
- app_editor дозволяє читати та змінювати дані.


Для цього створюємо ролі, як показано на [pic-09](#pic-09)

<kbd><img src="../assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-09.png" /></kbd>
<p style="text-align: center;"><a name="pic-09">pic-09</a></p>

Щоб ролі передавалися на клієнта, потрібно пересідчитися, що вони включені в client scope, як на [pic-10](#pic-10).

<kbd><img src="../assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-10.png" /></kbd>
<p style="text-align: center;"><a name="pic-10">pic-10</a></p>


### Створення користувачів для призначення їм ролей.

Набір коритсувачів є  єдиним на весь realm. Але користувачів можна розділити на групи. Набір груп теж єедний на весь realm. А от групи можна вже поэднати з ролями. Так ы зробимо.
#### Створемо користувачів

Процес створення користувачыв показано на [pic-11](#pic-11).

<kbd><img src="../assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-11.png" /></kbd>
<p style="text-align: center;"><a name="pic-11">pic-11</a></p>

Тепер створимо групи користовучів і поділимо користувачів на  дві групи:

- **assistent_users** -  група кристувачів,  що  повинна тільки читати дані. Тобто цій групі повинна відповідати прикладна роль **app_viewer**.

- **manager_users** - група користувачів, що можуть змінювати дані. Тобто цій групі повинна відповідати роль **app_editor**. 

Процес створення груп показано на [pic-12](#pic-12) та  [pic-13](#pic-13).


<kbd><img src="../assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-12.png" /></kbd>
<p style="text-align: center;"><a name="pic-12">pic-12</a></p>


<kbd><img src="../assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-13.png" /></kbd>
<p style="text-align: center;"><a name="pic-13">pic-13</a></p>

Далі, потрібно пройтися по користувачах і додати кожного у відповідін групи так, як показано на [pic-14](#pic-14).

<kbd><img src="../assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-14.png" /></kbd>
<p style="text-align: center;"><a name="pic-14">pic-14</a></p>

Тут треба зазначити, що якщо keycloak інтегрувати з Active Directory -  то гурпи користувачів будть ззразу відображатися і розносити коритсувачів по групах в KeyCloak  не потрібно. Це робиться в Active Directory. У підсумку, користувачі по групах рознесені так, як показано на [pic-15](#pic-15).

<kbd><img src="../assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-15.png" /></kbd>
<p style="text-align: center;"><a name="pic-15">pic-15</a></p>


На цьому етапі  співставлення: користувачі-групи-client roles виконано. можна переходити до етапа тестування логіну.