---
layout: post
title: "Keycloak vue.js node.js on openhsift"
date: 2023-03-02 10:00:01
categories: [keycloak, node.js, vue.js, openshift]
permalink: posts/2023-03-02/Keycloak vue.js node.js on openhsift/
published: true
---

<!-- TOC BEGIN -->

- [1. Інтеграція keycloak з vue.js, Node.js express в openhsift](#p-1)
- [2. Розгортаня keycloak  в OpenShift](#p-2)
- [3. Основні терміни адміністрування Keycloak](#p-3)
- [4. Ручна реєстрація програми клієнта](#p-4)
- [5. Налаштування прикладних ролей для програми клієнта](#p-5)
- [6. Створення користувачів для призначення їм ролей](#p-6)
- [7. Отримання авторизаційного токену по протоколу openid-connect](#p-7)
- [8. Захист Node.js express Rest API  за допомогою KeyCloak](#p-8)
<!-- TOC END -->

## <a name="p-1">Інтеграція keycloak з vue.js, Node.js express в openhsift </a>

Продук [Keycloack](https://www.keycloak.org/) є зараз типовим  інструментом для авторизації в Web-based системах.  Документацію можна знайти за лінками: 
- Основна документація знаходиться за лінком [documentation](https://www.keycloak.org/documentation);
- За цим лінком знаходяться більш сфокусовані описи [Фокусовані описи](https://www.keycloak.org/guides).

Ну на цьому зацікавився цим продуктом, щоб трохи розібратися як він працює та як його конфігурвати. Це більше про те, як підключитися клієнтом до keycloak  а не про те, як правильно його розгортати та конфігурувати.


## <a name="p-2">Розгортаня keycloak  в OpenShift</a>

За звичай keycloak  можна підняти в контейнері, але він страртує в development mode. Також, там треба прив'язати базу даних типу postgresql. Я вирішив піти більш простим шляхом і розгорнув його в хмарі RadHat в sendbox  OpsnShift.  Openshift sendbox можна створити за url: https://developers.redhat.com/developer-sandbox, а перед цим потрібно зарєструватися як developer на RedHat. Docker відкинув зразу, тому що  прочитав ліцензійні обмеження для корпорацій, що описані за лінокм [pricing](https://www.docker.com/pricing/) в самому низу стріник, і відмовився.

> Docker Desktop is free to use, as part of the Docker Personal subscription, for individuals, non-commercial open source developers, students and educators, and small businesses of less than 250 employees AND less than $10 million in revenue. Commercial use of Docker Desktop at a company of more than 250 employees OR more than $10 million in annual revenue requires a paid subscription (Pro, Team, or Business) to use Docker Desktop. While the effective date of these terms is August 31, 2021, there is a grace period until January 31, 2022 for those that require a paid subscription to use Docker Desktop.

Щоб не мучитися з  лінкуванням бази даних та самого  сервісу keycloak  я використав калог уже підготованих продуктів, що є вже в любому OpenShift і в пару кліків розгорнув додаток [pic-01](#pic-01). 

<kbd><img src="/assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-01.png" /></kbd>
<p style="text-align: center;"><a name="pic-01">pic-01</a></p> 

Через кілька хвилин я уже маю готовий keycloak:
<kbd><img src="/assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-02.png" /></kbd>
<p style="text-align: center;"><a name="pic-02">pic-02</a></p> 

І уже натискаємо на роут [pic-03](#pic-03), попадаємо в консоль адміністрування. Можливо потрібно трохи зачекати, поки обидва сервіса стартонуть. Ну, там час старту такий собі, відчутний, але не критично. 

<kbd><img src="/assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-03.png" /></kbd>
<p style="text-align: center;"><a name="pic-03">pic-03</a></p> 


Консоль адміністрування запросить логін та пароль. Вони знаходяться в env змінних Keycloak  так як показано на   [pic-04](#pic-04)

<kbd><img src="/assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-04.png" /></kbd>
<p style="text-align: center;"><a name="pic-04">pic-04</a></p>

Залогінился і вуаля - попадаємо в  консоль адмінітрування, в головний realm

<kbd><img src="/assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-05.png" /></kbd>
<p style="text-align: center;"><a name="pic-05">pic-05</a></p>


## <a name="p-3">Основні терміни адміністрування Keycloak</a>

Адміністрування в keycloak ділиться на **realm**-и. 
- **Realm**  це одниця адміністрування, що інкапсулює в собі набори: користувачів, ролей, груп та набори додаткових повноважень (чи  прав). Тому, переше, що робимо,  реєструємо свій realm. Назвемо його **shdemorealm**. На [pic-06](#pic-06) показані основні елементи realm.

<kbd><img src="/assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-06.png" /></kbd>
<p style="text-align: center;"><a name="pic-06">pic-06</a></p>

Keycloak підтримує як OpenID Connect (розширення OAuth 2.0), так і SAML 2.0. Коли говоримо про security, перше, що потрібно вирішити, це те, що з двох ви збираєтеся використовувати. Я вирішую використовувати OpenID Connect (розширення OAuth 2.0), тому в подальшому мова йде тільки про нього.

- **Client** - це прикладні додатки, що зареєстровані в окремо взятій realm. За звичай додатки аутентифікуються за допомогою clinet ID  або client ID та client secret. Ще є аутентифікація через JWT  токен, але зараз її не розглядаємо. Client ID and Client Secret - це традиційний метод, описаний у специфікації OAuth2. Клієнт має секрет, який повинен бути відомий як прикладному додатку, так і серверу Keycloak.

##  <a name="p-4">Ручна реєстрація програми клієнта</a>

Створюємо клієнта в client ID **DemoApp1**, як показано на [pic-06](#pic-06):

<kbd><img src="/assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-06.png" /></kbd>
<p style="text-align: center;"><a name="pic-06">pic-06</a></p>


Після реєстрації зразу попадаємо на вікно [pic-07](#pic-07)

<kbd><img src="/assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-07.png" /></kbd>
<p style="text-align: center;"><a name="pic-07">pic-07</a></p>

Тут треба звернути увагу на **Access Type**=public при логіні не поребує client Secret.  А при **Access Type**=confidential потребує client Secret. Зараз залишаємо **public**.


Треба звернути увагу на **Standard Flow Enabled**  треба включити в **"ON"**. Таким чином авторизація піде по стандартному OAuth2.0 за допомогою обміну autorization_code.

Далі потрібно налаштувати URL  додатка так, як показано на [pic-08](#pic-08).

<kbd><img src="/assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-08.png" /></kbd>
<p style="text-align: center;"><a name="pic-08">pic-08</a></p>

Тобто, поки що нас цікавлять Root URL  та "правильний" redirect URL.  По ньому відбувається  переадресація у випадку успішного  логіна. Інші, пока що не задіяні.

Все, можна сказати в найпростішому варіанті клієнт-додаток зареєстровано.

## <a name="p-5">Налаштування прикладних ролей для програми клієнта</a>


Після  реєстрації додатку-клієнта  потрібно запараметризувати рольовий доступ до ресурсів програми. Рольовий досутп може ділитися на  по ролям. А ролі діляться на Realm Roles  та Client Roles. 

- Realm Roles  доступні для всього realm. Цей варіант не зовсім вигідний. Але прийнятний в деяких випадках
- Client Roles -  доступні для окремо зареєстрованого клієнта. Цей варіант, як на мене, більш прийнятний. Тому на ньому і зупинимося.

Для нашого майбутньго додатку створемо 2 ролі:

- app_viewer дозволяє тільки читати дані:
- app_editor дозволяє читати та змінювати дані.


Для цього створюємо ролі, як показано на [pic-09](#pic-09)

<kbd><img src="/assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-09.png" /></kbd>
<p style="text-align: center;"><a name="pic-09">pic-09</a></p>

Щоб ролі передавалися на клієнта, потрібно пересідчитися, що вони включені в client scope, як на [pic-10](#pic-10).

<kbd><img src="/assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-10.png" /></kbd>
<p style="text-align: center;"><a name="pic-10">pic-10</a></p>


## <a name="p-6">Створення користувачів для призначення їм ролей</a>

Набір коритсувачів є  єдиним на весь realm. Але користувачів можна розділити на групи. Набір груп теж єедний на весь realm. А от групи можна вже поэднати з ролями. Так і зробимо.

Процес створення користувачыв показано на [pic-11](#pic-11).

<kbd><img src="/assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-11.png" /></kbd>
<p style="text-align: center;"><a name="pic-11">pic-11</a></p>

Тепер створимо групи користовучів і поділимо користувачів на  дві групи:

- **assistent_users** -  група кристувачів,  що  повинна тільки читати дані. Тобто цій групі повинна відповідати прикладна роль **app_viewer**.

- **manager_users** - група користувачів, що можуть змінювати дані. Тобто цій групі повинна відповідати роль **app_editor**. 

Процес створення груп показано на [pic-12](#pic-12) та  [pic-13](#pic-13).


<kbd><img src="/assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-12.png" /></kbd>
<p style="text-align: center;"><a name="pic-12">pic-12</a></p>


<kbd><img src="/assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-13.png" /></kbd>
<p style="text-align: center;"><a name="pic-13">pic-13</a></p>

Далі, потрібно пройтися по користувачах і додати кожного у відповідін групи так, як показано на [pic-14](#pic-14).

<kbd><img src="/assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-14.png" /></kbd>
<p style="text-align: center;"><a name="pic-14">pic-14</a></p>

Тут треба зазначити, що якщо keycloak інтегрувати з Active Directory -  то групи користувачів будть зразу відображатися і розносити користувачів по групах в KeyCloak  не потрібно. Це робиться в Active Directory. У підсумку, користувачі по групах рознесені так, як показано на [pic-15](#pic-15).

<kbd><img src="/assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-15.png" /></kbd>
<p style="text-align: center;"><a name="pic-15">pic-15</a></p>


На цьому етапі  співставлення: користувачі-групи-client roles виконано. можна переходити до етапа тестування логіну та отримання авторизаційногг токену.

# <a name="p-7">Отримання авторизаційного токену по протоколу openid-connect</a>

Для перевірки авторизації потрібно визначити, для початку, знайти "правильні" URL. Для цього. ідемо в налаштування realm та отримуємо json, як показано на [pic-16](#pic-16).

<kbd><img src="/assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-16.png" /></kbd>
<p style="text-align: center;"><a name="pic-16">pic-16</a></p>

Далі беремо url в ключі **token_endpoint** та формуємо  http - запит на KeyCloack

- Method: Http=POST
- URL ${token_endpoint}
- Http Headers

```text

    Content-Type: application/x-www-form-urlencoded

```

- Request Body:

```text
    grant_type=password&client_id=DemoApp1&username=usr1&password=11111111

```

- Response Body:

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJaUUhtZ0oya2R1b1FPOG12NTVoYU1vOXJKd1hMYVBGZ3VJRU9JejA5Y0xvIn0.eyJleHAiOjE2NzgyODAxODgsImlhdCI6MTY3ODI3OTg4OCwianRpIjoiMjE3NGNhNmUtZjcxMC00MDFkLWJiOGMtYzBjOTBlZWI0N2VhIiwiaXNzIjoiaHR0cHM6Ly9zc28tcGFzaGFreC1kZXYuYXBwcy5zYW5kYm94LW0zLjE1MzAucDEub3BlbnNoaWZ0YXBwcy5jb20vYXV0aC9yZWFsbXMvc2hkZW1vcmVhbG0iLCJhdWQiOiJhY2NvdW50Iiwic3ViIjoiNGJiNTQzZDktOGQ4NS00ZTcxLTlkOTYtMTA1YWNiNTE2MjNlIiwidHlwIjoiQmVhcmVyIiwiYXpwIjoiRGVtb0FwcDEiLCJzZXNzaW9uX3N0YXRlIjoiMzBhN2YwODMtNGFjZi00ZWY2LTg0MGEtZDA4YTYzZmMyNzAyIiwiYWNyIjoiMSIsInJlYWxtX2FjY2VzcyI6eyJyb2xlcyI6WyJkZWZhdWx0LXJvbGVzLXNoZGVtb3JlYWxtIiwib2ZmbGluZV9hY2Nlc3MiLCJ1bWFfYXV0aG9yaXphdGlvbiJdfSwicmVzb3VyY2VfYWNjZXNzIjp7IkRlbW9BcHAxIjp7InJvbGVzIjpbImFwcF92aWV3ZXIiXX0sImFjY291bnQiOnsicm9sZXMiOlsibWFuYWdlLWFjY291bnQiLCJtYW5hZ2UtYWNjb3VudC1saW5rcyIsInZpZXctcHJvZmlsZSJdfX0sInNjb3BlIjoicHJvZmlsZSBlbWFpbCIsInNpZCI6IjMwYTdmMDgzLTRhY2YtNGVmNi04NDBhLWQwOGE2M2ZjMjcwMiIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwibmFtZSI6ItCf0LXRgtGA0L4g0J_QtdGC0YDQtdC90LrQviIsInByZWZlcnJlZF91c2VybmFtZSI6InVzcjEiLCJnaXZlbl9uYW1lIjoi0J_QtdGC0YDQviIsImZhbWlseV9uYW1lIjoi0J_QtdGC0YDQtdC90LrQviJ9.S-xsR0RTac3YwF7hVX5f3N1W4qkoXcIXnnfIQVMKRAi99LCt4UqEKGIGBf_QtbGikMLnm7sSHJAesJIOoVURYklVNM0Hxb6xgy4gy6KGcgxabbeuDdLJsN2OVENTtFL9m3GQAAlN71w7ka8MmMxcxqYhEAxQtVwvCU1pa6tpkOpoQKpRB9CeNRuhx9KdziymvNR4AEhD8Q1QqB0yLvyjxQ_pkKGT808IpWPezvqMGXuLQ2yEfDV8sc3PFyQMhDCwLj5W-egvyJJq2bIgssNNYJ69WsX_WcnfPWOisErGIMijcNq7rFCEwrmIUdrTaTxyEfeJfJgU0MjJd1a-440WIA",
  "expires_in": 300,
  "refresh_expires_in": 1800,
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJkYTVmZWIwOS1hNWYxLTQwZjMtYjc5MC1iYTI0NGJkYTIyMDIifQ.eyJleHAiOjE2NzgyODE2ODgsImlhdCI6MTY3ODI3OTg4OCwianRpIjoiNjk0MTRkMGYtMTAyZS00Yzc1LThlNmQtNjRmZmY1ZjhmODhjIiwiaXNzIjoiaHR0cHM6Ly9zc28tcGFzaGFreC1kZXYuYXBwcy5zYW5kYm94LW0zLjE1MzAucDEub3BlbnNoaWZ0YXBwcy5jb20vYXV0aC9yZWFsbXMvc2hkZW1vcmVhbG0iLCJhdWQiOiJodHRwczovL3Nzby1wYXNoYWt4LWRldi5hcHBzLnNhbmRib3gtbTMuMTUzMC5wMS5vcGVuc2hpZnRhcHBzLmNvbS9hdXRoL3JlYWxtcy9zaGRlbW9yZWFsbSIsInN1YiI6IjRiYjU0M2Q5LThkODUtNGU3MS05ZDk2LTEwNWFjYjUxNjIzZSIsInR5cCI6IlJlZnJlc2giLCJhenAiOiJEZW1vQXBwMSIsInNlc3Npb25fc3RhdGUiOiIzMGE3ZjA4My00YWNmLTRlZjYtODQwYS1kMDhhNjNmYzI3MDIiLCJzY29wZSI6InByb2ZpbGUgZW1haWwiLCJzaWQiOiIzMGE3ZjA4My00YWNmLTRlZjYtODQwYS1kMDhhNjNmYzI3MDIifQ.m6MbJ1bVlbixDzURut5vOPmWVS5aD2o2GCw3v20YOY0",
  "token_type": "Bearer",
  "not-before-policy": 0,
  "session_state": "30a7f083-4acf-4ef6-840a-d08a63fc2702",
  "scope": "profile email"
}

```
Ну, або у вигляді curl

```bash
curl -X POST -k -H 'Content-Type: application/x-www-form-urlencoded' -i 'https://[domain.name]/auth/realms/shdemorealm/protocol/openid-connect/token' --data 'grant_type=password&client_id=DemoApp1&username=usr1&password=11111111'

```


Ну а якщо зайти в меню **sessions** або **users**,  то можна побачити всі підключені сесії жо жаного клієнта, та є можливісь всіх, або окремо взятого користувача - відключити [pic-17](#pic-17).

<kbd><img src="/assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-17.png" /></kbd>
<p style="text-align: center;"><a name="pic-17">pic-17</a></p>

Під одним логіном можна отримати кілька токенів і вони всі будуть дійсні, якщо їх період дії пересікається. Ну можна вибрати ще одного користувача, usr4  [pic-18](#pic-18).

<kbd><img src="/assets/img/posts/2023-03-02-keycloak-openshift/doc/pic-18.png" /></kbd>
<p style="text-align: center;"><a name="pic-18">pic-18</a></p>


# <a name="p-8">Захист Node.js express Rest API  за допомогою KeyCloak</a>

В документації до Keycloak [Securing Applications and Services Guide](s://www.keycloak.org/docs/latest/securing_apps/index.html) є розділ по інтеграції додатків Node.js з keycloak [2.3. Node.js adapter](https://www.keycloak.org/docs/latest/securing_apps/index.html#_nodejs_adapter). Рекомендується використовувати пакет [keycloak-connect](https://www.npmjs.com/package/keycloak-connect), при цьому він парцює  разом з [express-session](https://www.npmjs.com/package/express-session).

Тому для демонстрації підготовано приклад  простого додатку [todo_srvc - Node.js exptress  REST API with keycloak security](https://github.com/pavlo-shcherbukha/todo_srvc), що використовує  [keycloak-connect](https://www.npmjs.com/package/keycloak-connect) для захисту RestAPI. Опист розгортання додтаку та розробені API описано в [readme.md](https://github.com/pavlo-shcherbukha/todo_srvc/blob/main/README.md). Додаток використовує realm та client_id,  що були створені в попередньому розділі.

Завантаження бібліотеки  [keycloak-connect](https://www.npmjs.com/package/keycloak-connect) відубвається у файлі [/server/config/keycloak-config.js](https://github.com/pavlo-shcherbukha/todo_srvc/blob/main/server/config/keycloak-config.js).  А вже саме її підклчення для викоритсання, разом з [express-session](https://www.npmjs.com/package/express-session), виконується в файлі [/server/server.js](https://github.com/pavlo-shcherbukha/todo_srvc/blob/main/server/server.js):

```js
// Включаємо  session memory store
const memoryStore = new session.MemoryStore()

app.use(session({
  secret: 'mySecret',
  resave: false,
  saveUninitialized: true,
  store: memoryStore
}))


// включаємо keycloack
const keycloak = require('./config/keycloak-config.js').initKeycloak(memoryStore);
app.use(keycloak.middleware());

```

Також, потрібно звернути увагу на те, що всі АПІ, які не повинні використовувати захист keycloak  потрібно розмістити вище цього участку коду. Інакше, keycloak буде перевіряти наявність http-заголовка Authorization з токеном:

```text
  Authorization: Bearer <token>

```

А, для прикладу, коли у вас використовується fronend,  то браузер шле на backend http запити **options** і явно без цього заголовка. Тому, в  цій демці методи **options** та метод перевірки доступності сервера "api/health"  розміщені до момента підключення keycloak.

Як було  показано раніше,  на рівні клієнта створено 2 ролі: **app_viewer** та **app_editor**.  Для захисту API можна використати конструкцію **keycloak.protect**,  де в масиві  передати список доступних ролей.

```js
keycloak.protect(  [ 'app_editor' ,'app_viewe' ]  )
```

Ось для прикладу:

```js
app.get('/api/todos',  keycloak.protect(  [ 'app_editor' ,'app_viewe' ]  ), function(req, res) {
  let label='todos';
  applog.info( 'call api/todos method', label);
  try{
      let result=i_todos;
      return res.status(200).json( result );
  } 
  catch (err){
      applog.error( `Error ${err.message} `, label);
      errresp=applib.HttpErrorResponse(err)
      applog.error( `Error result ${errresp.Error.statusCode} ` + JSON.stringify( errresp )   ,label);
      return res.status(errresp.Error.statusCode ).json(errresp);    

  }

});  

app.post('/api/todo',  keycloak.protect( [ 'app_editor' ]),  function(req, res) {
  let label='todo';
  applog.info( 'call api/todo method', label);
  let body=req.body;
  try { 
      applog.info( 'Check propery [name]', label);
      if (!body.hasOwnProperty("name")){
        throw new apperror.ValidationError( 'key [name] is absend' );
      }
      applog.info( 'Check propery [description]', label);
      if (!body.hasOwnProperty("description")){
        throw new apperror.ValidationError( 'key [description] is absend' );

      }
      applog.info( 'Check propery [owner]', label);
      if (!body.hasOwnProperty("owner")){
        throw new apperror.ValidationError( 'key [owner] is absend' );
      }
      applog.info( 'Return result', label);
      
      body["id"] = uuid.v4()
      i_todos.push(  body )
      let result={"id": body.id};
      return res.status(200).json( result );

  } 
  catch (err){
    applog.error( `Error ${err.message} `, label);
    errresp=applib.HttpErrorResponse(err)
    applog.error( `Error result ${errresp.Error.statusCode} ` + JSON.stringify( errresp )   ,label);
    return res.status(errresp.Error.statusCode ).json(errresp);

  }

});

```

Але, як на мене, це не дуже гнучний метод. Мені б хотілося, з токена отримати інформацію про користувача, та  номер сесії. Більш того, хотілося б якось запараметризувати відповідність URL (path)  та доступних ролей. Цього можна досягти, якщо  прочитиати увжано документацію до бібліотеки, де сказано, що  в **keycloak.protect()**  можна використовувати не тільки масив ролей а і свою функцію, яка в якості парамтерів приймає token та requset, так, як показано далі фрагмент функції та приклад її використання. Перевірка наявності тієї чи іншої ролі виконується функцією **token.hasRole( "rolename")**. А сама функція контролю доступа повинна повертати boolean: true-дозволено, false-не дозволено.

```js
/**
 * Перевірка доступа
 * @param {*} token 
 * @param {*} request 
 * @returns  true or false
 */
function checkAccess(token, request) {
  let label='checkAccess';
  let is_role=false ;
  if ( token.hasRole( "rolename") ) {
      is_role=true ;

  }

  applog.info(`checkAccess Method: ${request.method} Path: ${request.path}`, label);
  return is_role;
}  


app.post('/api/todo',   keycloak.protect(  checkAccess  ) ,  function(req, res) {
  let label='todo';
  applog.info( 'call api/todo method', label);
});  


```

От цей підхід і можна використати. Для цього потібно підготувати json стурктуру, що пов'язує:
- http  метод;
- path (частину url);
- масив ролей, яким достуний виклик даного методі.

На приклад, зробимо такий **/server/config/accessRoles.json** :

```json
[
  {"method": "GET",    "path": "/api/todos", "accessroles": ["app_editor", "app_viewer" ]},
  {"method": "POST",   "path": "/api/todo", "accessroles": ["app_editor"]},
  {"method": "GET",    "path": "/api/todo/:todoid", "accessroles": ["app_viewer", "app_editor" ]},
  {"method": "DELETE", "path": "/api/todo/:todoid", "accessroles": ["app_editor"]}
]

```
Для співставлення path реального URL  та заданого в файлі використаємо пакет: **url-pattern**.  Доречі, виявив, що пакет на працює з масками імен, що майть "_":  ":todoid - співставляє", а ":todo_id - не співставляє".   Фінальний варіант функції та її використання показані уже в git репозиторії. На додачу до цього додамо логування в метода реквізити username та state (що аналогічно ідентифікатору сесії). 

Також, слід звернути увагу на частину куду в функції **checkAccess**, а саме:
- Параметр token надходить у вигляді розшифрованої JSON об'єкту, де в ключі **resource_access**  мжна побачити  ролі користувача для заданого client-id, а в ключі **realm_access** доступні користувачу realm ролі;

```json
{
  token: "eyJhbGc.......jEmYLHmeg",
  clientId: "DemoApp1",
  header: {
    alg: "RS256",
    typ: "JWT",
    kid: "jott31qT3_Vutuz6yt9k2......lsJizig",
  },
  content: {
    exp: 1678552102,
    iat: 1678551802,
    jti: "21954b73-6579-4e42-b260-8f08840b0084",
    iss: "https://hostname/auth/realms/DemoApp1",
    aud: "account",
    sub: "228501ff-8d5c-4224-9fe7-8a72fa5db426",
    typ: "Bearer",
    azp: "DemoApp1",
    session_state: "9310df83-1f52-43b1-b64f-6cc74d090c8f",
    acr: "1",
    "allowed-origins": [
      "",
    ],
    realm_access: {
      roles: [
        "offline_access",
        "uma_authorization",
        "default-roles-iitregsrvcr",
      ],
    },
    resource_access: {
      DemoApp1: {
        roles: [
          "app_editor",
        ],
      },
      account: {
        roles: [
          "manage-account",
          "manage-account-links",
          "view-profile",
        ],
      },
    },
    scope: "email profile",
    sid: "9310df83-1f52-43b1-b64f-6cc74d090c8f",
    email_verified: false,
    name: "Микола Роблюусьо",
    preferred_username: "usr1",
    given_name: "Микола",
    family_name: "Роблюусьо",
  },
  signature: new Uint8Array([53, .........2]),
  signed: "eyJ.....QviJ9",
}


```

- цей токен через перемери сесії передається далі в запит на всі інші API;

```js
function checkAccess(token, request) {
  let label='checkAccess';
  let is_role=false ;
  applog.info(`checkAccess Method: ${request.method} Path: ${request.path}`, label);
  // в параметри http сесії записуємо token-об'єкт
  applog.info("Set session param ", label);
  request.session["keycloak-token"]=token;
  ///..........
}

app.get('/api/todos',  keycloak.protect( checkAccess ), function(req, res) {
  let label='todos';
  // отримуэмо з запиту об'єкт-token  
  let ssnk=req.session['keycloak-token'];

  let alogctx= new LogContext();
  let alog= new AppLogger();
  alog.LogContext=alogctx;      
  
  // з token  читаю id сесії та логін користувача і заисую в лог для подальшого використання
  alog.State=ssnk.content.session_state;  
  alog.Username=ssnk.content.preferred_username;
  
  
  alog.info( 'call api/todos method', label);
  // ...........
});


```

- Backend  сервіс може перевіряти доступ з інших client-id,  що внесені до одного й того ж realm. Для цього в ролі потрібно вказувати client-id. На приклад:

```text
[
  /* роль перевіряються в client-id  backend*/
  {"method": "GET",    "path": "/api/todos", "accessroles": ["app_editor", "app_viewer" ]},
  /*перевіряється роль для client-id "demoapp2"*/
  {"method": "POST",   "path": "/api/todo", "accessroles": ["demoapp2:app_editor"]},
  {"method": "GET",    "path": "/api/todo/:todoid", "accessroles": ["demoapp2:app_viewer", "demoapp2:app_editor" ]},
  /*В цьому прикладіперевіряється роль, що задана на весь realm*/
  {"method": "DELETE", "path": "/api/todo/:todoid", "accessroles": ["realm:app_support"]}
]

```

Таким чином, один back-end   може перевіряти доступи кількох front-end  в рамках одного realm. А, для приклду, групі support  можна зробити розширені доступа до всіх методів за допомогою введення одної спільної ralm- ролі.


Ось лог роботи сервісу, коли методи виклкаємо для користуачів usr1  - app_viewer  та usr4 - app_editor. По логу можна побачити, що метод POST Path: /api/todo (label=createTodo) достпно для користувача usr4 та не доступно для usr1, чого і хотіли досягти. И бачимо, що викли кожного метода супрводжується логуванням, в якому присутні username - логін користуача та state-ідентифікатор сесії. Тобто по ньому можна відслідкувати послідовність викликів сесії. 

```json
        [
            {
                "hostname": "localhost",
                "label": "server",
                "level": "info",
                "message": "SERVER HAS STARTED",
                "state": null,
                "timestamp": "2023-03-13T21:23:13.765765+02:00",
                "username": null
            },
            {
                "hostname": "localhost",
                "label": "server",
                "level": "info",
                "message": "LISTENING  PORT= 8080 on HOST localhost",
                "state": null,
                "timestamp": "2023-03-13T21:23:13.767767+02:00",
                "username": null
            },
            {
                "hostname": "localhost",
                "label": "checkAccess",
                "level": "info",
                "message": "checkAccess Method: GET Path: /api/todos",
                "state": null,
                "timestamp": "2023-03-13T21:24:01.980980+02:00",
                "username": null
            },
            {
                "hostname": "localhost",
                "label": "checkAccess",
                "level": "info",
                "message": "Set session param ",
                "state": null,
                "timestamp": "2023-03-13T21:24:01.983983+02:00",
                "username": null
            },
            {
                "hostname": "localhost",
                "label": "checkAccess",
                "level": "info",
                "message": "Check parmission of: usr1 - Петро Петренко state= a9552b74-24ac-4926-87ec-1ccc4d6b026a",
                "state": null,
                "timestamp": "2023-03-13T21:24:01.985985+02:00",
                "username": null
            },
            {
                "hostname": "localhost",
                "label": "checkAccess",
                "level": "info",
                "message": "Check parmission : usr1 - Петро Петренко state= a9552b74-24ac-4926-87ec-1ccc4d6b026a  against Method: GET Path: /api/todos  RESULT: true",
                "state": null,
                "timestamp": "2023-03-13T21:24:01.993993+02:00",
                "username": null
            },
            {
                "hostname": "localhost",
                "label": "todos",
                "level": "info",
                "message": "call api/todos method",
                "state": "a9552b74-24ac-4926-87ec-1ccc4d6b026a",
                "timestamp": "2023-03-13T21:24:01.997997+02:00",
                "username": "usr1"
            },
            {
                "hostname": "localhost",
                "label": "checkAccess",
                "level": "info",
                "message": "checkAccess Method: POST Path: /api/todo",
                "state": null,
                "timestamp": "2023-03-13T21:24:21.643643+02:00",
                "username": null
            },
            {
                "hostname": "localhost",
                "label": "checkAccess",
                "level": "info",
                "message": "Set session param ",
                "state": null,
                "timestamp": "2023-03-13T21:24:21.645645+02:00",
                "username": null
            },
            {
                "hostname": "localhost",
                "label": "checkAccess",
                "level": "info",
                "message": "Check parmission of: usr1 - Петро Петренко state= a9552b74-24ac-4926-87ec-1ccc4d6b026a",
                "state": null,
                "timestamp": "2023-03-13T21:24:21.647647+02:00",
                "username": null
            },
            {
                "hostname": "localhost",
                "label": "checkAccess",
                "level": "info",
                "message": "Check parmission : usr1 - Петро Петренко state= a9552b74-24ac-4926-87ec-1ccc4d6b026a  against Method: POST Path: /api/todo  RESULT: false",
                "state": null,
                "timestamp": "2023-03-13T21:24:21.651651+02:00",
                "username": null
            },
            {
                "hostname": "localhost",
                "label": "checkAccess",
                "level": "info",
                "message": "checkAccess Method: GET Path: /api/todos",
                "state": null,
                "timestamp": "2023-03-13T21:24:52.391391+02:00",
                "username": null
            },
            {
                "hostname": "localhost",
                "label": "checkAccess",
                "level": "info",
                "message": "Set session param ",
                "state": null,
                "timestamp": "2023-03-13T21:24:52.394394+02:00",
                "username": null
            },
            {
                "hostname": "localhost",
                "label": "checkAccess",
                "level": "info",
                "message": "Check parmission of: usr4 - Денис Денисов state= c41a18f6-7617-4415-a477-86c5950f691c",
                "state": null,
                "timestamp": "2023-03-13T21:24:52.395395+02:00",
                "username": null
            },
            {
                "hostname": "localhost",
                "label": "checkAccess",
                "level": "info",
                "message": "Check parmission : usr4 - Денис Денисов state= c41a18f6-7617-4415-a477-86c5950f691c  against Method: GET Path: /api/todos  RESULT: true",
                "state": null,
                "timestamp": "2023-03-13T21:24:52.399399+02:00",
                "username": null
            },
            {
                "hostname": "localhost",
                "label": "todos",
                "level": "info",
                "message": "call api/todos method",
                "state": "c41a18f6-7617-4415-a477-86c5950f691c",
                "timestamp": "2023-03-13T21:24:52.403403+02:00",
                "username": "usr4"
            },
            {
                "hostname": "localhost",
                "label": "checkAccess",
                "level": "info",
                "message": "checkAccess Method: POST Path: /api/todo",
                "state": null,
                "timestamp": "2023-03-13T21:25:06.717717+02:00",
                "username": null
            },
            {
                "hostname": "localhost",
                "label": "checkAccess",
                "level": "info",
                "message": "Set session param ",
                "state": null,
                "timestamp": "2023-03-13T21:25:06.720720+02:00",
                "username": null
            },
            {
                "hostname": "localhost",
                "label": "checkAccess",
                "level": "info",
                "message": "Check parmission of: usr4 - Денис Денисов state= c41a18f6-7617-4415-a477-86c5950f691c",
                "state": null,
                "timestamp": "2023-03-13T21:25:06.721721+02:00",
                "username": null
            },
            {
                "hostname": "localhost",
                "label": "checkAccess",
                "level": "info",
                "message": "Check parmission : usr4 - Денис Денисов state= c41a18f6-7617-4415-a477-86c5950f691c  against Method: POST Path: /api/todo  RESULT: true",
                "state": null,
                "timestamp": "2023-03-13T21:25:06.725725+02:00",
                "username": null
            },
            {
                "hostname": "localhost",
                "label": "createtodo",
                "level": "info",
                "message": "call post api/todo method",
                "state": "c41a18f6-7617-4415-a477-86c5950f691c",
                "timestamp": "2023-03-13T21:25:06.729729+02:00",
                "username": "usr4"
            },
            {
                "hostname": "localhost",
                "label": "createtodo",
                "level": "info",
                "message": "Check propery [name]",
                "state": "c41a18f6-7617-4415-a477-86c5950f691c",
                "timestamp": "2023-03-13T21:25:06.732732+02:00",
                "username": "usr4"
            },
            {
                "hostname": "localhost",
                "label": "createtodo",
                "level": "info",
                "message": "Check propery [description]",
                "state": "c41a18f6-7617-4415-a477-86c5950f691c",
                "timestamp": "2023-03-13T21:25:06.734734+02:00",
                "username": "usr4"
            },
            {
                "hostname": "localhost",
                "label": "createtodo",
                "level": "info",
                "message": "Check propery [owner]",
                "state": "c41a18f6-7617-4415-a477-86c5950f691c",
                "timestamp": "2023-03-13T21:25:06.736736+02:00",
                "username": "usr4"
            },
            {
                "hostname": "localhost",
                "label": "createtodo",
                "level": "info",
                "message": "Return result",
                "state": "c41a18f6-7617-4415-a477-86c5950f691c",
                "timestamp": "2023-03-13T21:25:06.738738+02:00",
                "username": "usr4"
            }
        ]
```













