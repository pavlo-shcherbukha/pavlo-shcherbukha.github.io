---
layout: post
title: "ibm cloud Code Engine"
date: 2023-03-30 10:00:01
categories: [ibmcloud]
permalink: posts/2023-05-11/ibmcloud-codeengine/
published: true
---

<!-- TOC BEGIN -->
- [1. Постановка проблеми](#p-1)
    - [1.1. Посилання на потрібну документацію](#p-1.2)
- [2. Установка IBM CLI та необхідних плагінів.](#p-2)
- [3. Швидкий логін в IBM CLoud](#p-3)
    - [3.1. Зайдемо в IBM CLoud з допомогою CLI по логін/пароль](#p-3.1)
    - [3.2. Згенеруємо API-KEY  для подальших логінів та Вивантажемо його у вигляді json-файлу](#p-3.2)
    - [3.3. Зайдемо в IBM CLoud з допомогою CLI по API-KEY](#p-3.3)
- [4. Deployment додатка в IBM Cloud Engine](#p-4)
    - [4.1. Створити  project](#p-4.1)
    - [4.2. Прив'язування container registry. Створити API-KEY для доступу до container registry](#p-4.2)
    - [4.3. Прив'язування container registry. Створення сontainer registry](#p-4.3)
    - [4.4. Application Depoyment. Створення профіля для побудови контейнера **build**](#p-4.4)
    - [4.5. Application Depoyment. Безпосередньо побудова контейнера **buildrun**](#p-4.5)
    - [4.6. Application Depoyment. Запуск application](#p-4.6) 
    - [4.7. Application Depoyment. Зв1язування application з іншими хмарними сервісами **application bind**](#p-4.7)
    - [4.8. Application Depoyment. Масштабування](#p-4.8)

<!-- TOC END -->

## <a name="p-1">Постановка проблеми</a>

Років 5  тому було розроблено додаток, частина якого працювала в IBM Cloud на  платформі CloudFoundary. Зараз ця платформа виводиться із експлуатації і виникла проблема, куди мінгрувати. Одним з очевидних рішень є міграція та розгортання додатку на serverles  платформі  [CodeEngine](https://cloud.ibm.com/codeengine/overview), що є одним з варіантів використання  kubernetis knative. Початковим крокам з вивченнмя цієї платформи і присвячений цей блог. 

Основними відправними моментами є такі твердження:

- для вивчення розроблено прототип  Node.js сервера, що має своє rest api  та взаємодіє з БД Cloudant, що теж розгорнута в хмарі. Прототип моделює, щось схоже на електронний магазин. Деталі налаштування та опис Rest API наведено в git repo додатка, що названо: [IBM Code Engine Node.js BackEnd with Cloudant](https://github.com/pavlo-shcherbukha/cenodebesrvc.git);
- deployment на CodeEngine буде виконуватися з допомогою IBM Cloud Command Line Interface.  


### <a name="p-1.2">Посилання на потрібну документацію</a>

- [Getting started with the IBM Cloud CLI](https://cloud.ibm.com/docs/cli?topic=cli-getting-started)
- [General IBM Cloud CLI (ibmcloud) commands](https://cloud.ibm.com/docs/cli?topic=cli-ibmcloud_cli)
- [IBM Cloud Code Engine CLI](https://cloud.ibm.com/docs/codeengine?topic=codeengine-cli)

- [ Cloudsant Client libraries](https://cloud.ibm.com/docs/Cloudant?topic=Cloudant-client-libraries#client-libraries)


Але, основна неприємніть з документцією та, що вона написана в алфавідному порядку. А щоб вибрати правильнй набір та послідовність виконання команд - потрібно витратити багато часу, щоб все перепробувати та вибрати оптимальний шлях.  


В результаті багатьох спроб, мені здається, що  вибрав оптимальний шлях deployment за допомогою CLI, що дає можливість явно контролювати всі ресурси, які використовуються на кожному кроці deployment. Ну, а далі, ці команди можна вбудувати в яку небуть локальну pipeline,  та виконати deployment.


## <a name="p-2">Установка IBM CLI та необхідних плагінів.</a>

Перш за все для работи знадоиться встановити IBM Cloud CLI, за лінком [IBM Cloud CLI](https://cloud.ibm.com/docs/cli?topic=cli-getting-started#step1-install-idt). Також знадобиться встановити два розширення (plugins):

- Плагін для роботи з Codeengine [Install the IBM Cloud Code Engine CLI](https://cloud.ibm.com/codeengine/cli)

```bash
ibmcloud plugin install code-engine
```

- Плагін для роботи з Container Registry  [ IBM Cloud® Container Registry ](https://cloud.ibm.com/docs/Registry?topic=Registry-registry_overview#registry_overview). Як встановити плагін описано за лінком: [container-registry CLI plug-in](https://cloud.ibm.com/docs/Registry?topic=Registry-registry_setup_cli_namespace#cli_namespace_registry_cli_install)

```bash
ibmcloud plugin install container-registry
```

Перевірка того, що плагіни встановлені:

```bash
ibmcloud plugin list
```

 
<kbd><img src="/assets/img/posts/2023-05-11-ibmcloud-codeengine/doc/pic-01.png" /></kbd>
<p style="text-align: center;"><a name="pic-01">pic-01</a></p>


І для довідки, щоб отримати перелік всіх плагінів, потрібно виконати команду: 

```bash
ibmcloud plugin repo-plugins
```

<kbd><img src="/assets/img/posts/2023-05-11-ibmcloud-codeengine/doc/pic-02.png" /></kbd>
<p style="text-align: center;"><a name="pic-02">pic-02</a></p>


## <a name="p-3">Швидкий логін в IBM CLoud</a>

За звичай в IBM Cloud можна зайти по вашому логін пароль. Але, якщо вам потрібно запустити якийсь devops  процес то потрібно виконати описану далі послідовність команд. З її допомгою зненерувати API-KEY  і уже його використовувати для входу в IBM Cloud. Ну, щось схоже з логіном то token  в RedHat Openshift. Таким чином, в цьому розділі: 
- Зайдемо в IBM CLoud з допомогою CLI по логін/пароль;
- Згенеруємо API-KEY  для подальших логінів
- Вивантажемо його у вигляді json-файлу
- Зайдемо в IBM CLoud з допомогою CLI по API-KEY




###  <a name="p-3.1">Зайдемо в IBM CLoud з допомогою CLI по логін/пароль</a>

Приклад команди [ibmcloud login](https://cloud.ibm.com/docs/cli?topic=cli-ibmcloud_cli#ibmcloud_login):

```bash

 ibmcloud login -a https://cloud.ibm.com -u myname@gmail.com -r eu-gb -g default

```
Де: 
- -a API_ENDPOINT
The API endpoint. For example, cloud.ibm.com

- -u username

- -r REGION
ідентифікатори регіонів можна подивитися за лінком: [codeengine-regions](https://cloud.ibm.com/docs/codeengine?topic=codeengine-regions). Я традиційно виьираю eu-gb

- -g RESOURCE_GROUP

Найменування групи ресурсів. За замовочуваням створбється група ресурсів "default"

Мені не подобається вводити логін і пароль, тому я входжу через onetime password з Cloud Dashboard:

<kbd><img src="/assets/img/posts/2023-05-11-ibmcloud-codeengine/doc/pic-03.png" /></kbd>
<p style="text-align: center;"><a name="pic-03">pic-03</a></p>

І найбільш для мене прийнятний  і найшвидший, через явний onetime password:

```bash

 ibmcloud login -a https://cloud.ibm.com -u myname@gmail.com  --sso -r eu-gb -g default

```
де:

- --sso
    Specify this option to log in with a [federated ID](https://cloud.ibm.com/docs/account?topic=account-federated_id&interface=cli). Using this option prompts you to authenticate with your single sign-on provider and enter a one-time passcode to log in.


Реpультат входу в CLI  показано на [pic-04](#pic-04).
<kbd><img src="/assets/img/posts/2023-05-11-ibmcloud-codeengine/doc/pic-04.png" /></kbd>
<p style="text-align: center;"><a name="pic-04">pic-04</a></p>


###  <a name="p-3.2">Згенеруємо API-KEY  для подальших логінів та Вивантажемо його у вигляді json-файлу</a>

Можливість зайти в CLI і з допомогою згенерированого **API-KEY** описана за лінком:  [API-KEY](https://cloud.ibm.com/docs/cli?topic=cli-ibmcloud_commands_iam).

Створити API-KEY можна комнадою: 

```bash
ibmcloud iam api-key-create NAME [-d DESCRIPTION] [--file FILE] [--lock] 
```

* NAME (required)
    Найменування API key, який буде створений.
* -d DESCRIPTION (опціонально)
    Опис API-KEY.
* --file FILE
    Зберегти інформацію про API key у заданый файл на laptop.
* --lock
    Заблокувати API key.


Таким чином, створений API-KEY запишеться в локальний файл и будемо його викорстовувати в подальших опреаціях. 
Створимо його командою:

```bash
ibmcloud iam api-key-create examplekey -d "APIKEY FOR EXAMPLE" --file psh-example-key.json 

```
Реpультат  показано на [pic-05](#pic-05).
<kbd><img src="/assets/img/posts/2023-05-11-ibmcloud-codeengine/doc/pic-05.png" /></kbd>
<p style="text-align: center;"><a name="pic-05">pic-05</a></p>

А сам **psh-example-key.json** вигляає так:[pic-06](#pic-06)



<kbd><img src="/assets/img/posts/2023-05-11-ibmcloud-codeengine/doc/pic-06.png" /></kbd>
<p style="text-align: center;"><a name="pic-06">pic-06</a></p>


### <a name="p-3.3">Зайдемо в IBM CLoud з допомогою CLI по API-KEY</a>

- логін в ibm cloud за прамим значенням API-KEY,  на приклад, з env var

```bash
ibmcloud login -a cloud.ibm.com --apikey 96**cV-ez*********************** -r eu-gb  -g Default


```

Або з використанням створеного файла

```bash
ibmcloud login -a cloud.ibm.com --apikey @psh-example-key.json -r eu-gb  -g Default

```

<kbd><img src="/assets/img/posts/2023-05-11-ibmcloud-codeengine/doc/pic-07.png" /></kbd>
<p style="text-align: center;"><a name="pic-07">pic-07</a></p>

На цьому, вивчення можливостей команди login  закінчено. Нам цього достатньо.



## <a name="p-4">Deployment додатка в IBM Cloud Engine</a>

Deployment буде виконуватися з [Dockerfile](https://github.com/pavlo-shcherbukha/cenodebesrvc/blob/main/Dockerfile), що лежить рядом з source code.


### <a name="p-4.1">Створити  project</a>

Project це сутність, що групує ваші application  та jobs  в єдине ціле. До нього ж прив'язується Container Registry на який складаються образи  створиених app. Project  створюється командою [ibmcloud ce project create](https://cloud.ibm.com/docs/codeengine?topic=codeengine-cli#cli-project-create). Створиме проект **sh-storage-prh** 

```bash
   ibmcloud ce project create --name sh-storage-prh  

```
Результат виконання показано на [pic-08](#pic-08)

<kbd><img src="/assets/img/posts/2023-05-11-ibmcloud-codeengine/doc/pic-08.png" /></kbd>
<p style="text-align: center;"><a name="pic-08">pic-08</a></p>

**Довідково**: 

- для переключення між проектами використовуються команди:
[**ibmcloud ce project select --name PROJECT_NAME**, Select a project as the current context ](https://cloud.ibm.com/docs/codeengine?topic=codeengine-cli#cli-project-select)

- відобразити, контекст поточного проекту
[**ibmcloud ce project current**  Display the details of the project that is currently targeted](https://cloud.ibm.com/docs/codeengine?topic=codeengine-cli#cli-project-current)

- отримати список всіх проектів:
[ibmcloud ce project list](https://cloud.ibm.com/docs/codeengine?topic=codeengine-cli#cli-project-list)


### <a name="p-4.2">Прив'язування container registry. Створити API-KEY для доступу до container registry</a>


Щоб створити відповідний API-KEY  потрібно зайти так, як показано на [pic-13](#pic-13) в IBM Cloud Dashboard та бажано скачати JSON-файл.
<kbd><img src="/assets/img/posts/2023-05-11-ibmcloud-codeengine/doc/pic-13.png" /></kbd>
<p style="text-align: center;"><a name="pic-13">pic-13</a></p>

### <a name="p-4.3">Прив'язування container registry. Створення сontainer registry</a>

Container registry так же як і docker registry зберігають образи побудованих контейнерів. До проекта прив'язується  сontainer registry. Звичайно, до цього потрібно переключитися на відповідний контекст проекту
[**ibmcloud ce project select](https://cloud.ibm.com/docs/codeengine?topic=codeengine-cli#cli-project-select).

Створення сontainer registry, що прив'язаний до відповідного Codeengine Project виконується командою [ibmcloud ce registry create](https://cloud.ibm.com/docs/codeengine?topic=codeengine-cli#cli-registry-create). При цьомуствориться namesapce  та одноіменний secret до нього. 


```bash

ibmcloud ce registry create --name sh-storage-prh-rgstr --server uk.icr.io --username iamapikey --password ********

```
В якості ключа **--password** вносимо отримане значення API-KEY.

- --server  [Local registry regions](https://cloud.ibm.com/docs/Registry?topic=Registry-registry_overview#registry_regions_local). Використав той же регіон, де і CodeEngine  розгорнуто.

Результат виконання показано на [pic-10](#pic-10).

<kbd><img src="/assets/img/posts/2023-05-11-ibmcloud-codeengine/doc/pic-10.png" /></kbd>
<p style="text-align: center;"><a name="pic-10">pic-10</a></p>


Перевірити виконання команди можна виконавши  [ibmcloud ce registry list](https://cloud.ibm.com/docs/codeengine?topic=codeengine-cli#cli-registry-list) (отримати список container registry namespaces, створених для code engine):

```bash
ibmcloud ce registry list
```
 та 

 [ibmcloud ce registry get](https://cloud.ibm.com/docs/codeengine?topic=codeengine-cli#cli-registry-get)

 ```bash
 ibmcloud ce registry get --name sh-storage-prh-rgstr
 ```

 А secret  можна побачити в cloud Dashboard так, як показано на [pic-09](#pic-09).

<kbd><img src="/assets/img/posts/2023-05-11-ibmcloud-codeengine/doc/pic-09.png" /></kbd>
<p style="text-align: center;"><a name="pic-09">pic-09</a></p>

На цьому етапі закінчені необхідні передумови для створення додатка і можна переходити безпосередньо до створення додатка **Application Depoyment**. 

### <a name="p-4.4">Application Depoyment. Створення профіля для побудови контейнера **build**</a>

Розділ команд build використовується для  опису методу побудови контейнера с програмного коду. В моєму випадку програмний код знаходиться в публічному github repo [IBM Code Engine Node.js BackEnd with Cloudant](https://github.com/pavlo-shcherbukha/cenodebesrvc.git), а сам Dockerfile,  в якому описано, як будувати знаходиться в корені цього ж repo [Dockerfile](https://github.com/pavlo-shcherbukha/cenodebesrvc/blob/main/Dockerfile). В якості базовго образу взято RedHat UBI8  контейнер, адаптований під Node.js.

Виконується командою [ibmcloud ce build create](https://cloud.ibm.com/docs/codeengine?topic=codeengine-cli#cli-build-create)
Для простоти я зібрав усе в CMD  файл /deployent/cli-crt-build.cmd. І, як можете бачити, тут і далі для доступу до container registry використовується secret, що був створений два кроки назад.  


```bash

set BLDSRC=https://github.com/pavlo-shcherbukha/cenodebesrvc.git
set BLDBRANCH=tz-000001-init
set BLDIMAGE= uk.icr.io/sh-storage-prh-rgstr/sh-storage-be

pause

ibmcloud ce build create --name sh-storage-be-bld --source %BLDSRC% --commit %BLDBRANCH% --strategy dockerfile --size medium --image %BLDIMAGE% --registry-secret sh-storage-prh-rgstr

pause

```
Успішне виконання команди показано на [pic-15](#pic-15), а що побачимо в UI - на [pic-14](#pic-14)

<kbd><img src="/assets/img/posts/2023-05-11-ibmcloud-codeengine/doc/pic-15.png" /></kbd>
<p style="text-align: center;"><a name="pic-15">pic-15</a></p>

<kbd><img src="/assets/img/posts/2023-05-11-ibmcloud-codeengine/doc/pic-14.png" /></kbd>
<p style="text-align: center;"><a name="pic-14">pic-14</a></p>


### <a name="p-4.5">Application Depoyment. Безпосередньо побудова контейнера **buildrun**</a>

Безпосердньо побудова образу виконується командою [ibmcloud ce buildrun submit](https://cloud.ibm.com/docs/codeengine?topic=codeengine-cli#cli-buildrun-submit)

```bash
ibmcloud ce buildrun submit --name sh-storage-be-bld-run --build sh-storage-be-bld
```
де **--build** вказує на попередньо створений build.

На [pic-16](#pic-16) показано результат виконання команди. 


<kbd><img src="/assets/img/posts/2023-05-11-ibmcloud-codeengine/doc/pic-16.png" /></kbd>
<p style="text-align: center;"><a name="pic-16">pic-16</a></p>

подивитися лог виконання можно за допомогою [ibmcloud ce buildrun logs](https://cloud.ibm.com/docs/codeengine?topic=codeengine-cli#cli-buildrun-logs)

```bash
ibmcloud ce buildrun logs --buildrun sh-storage-be-bld-run --follow

```

Отримати статус виконання можна командою [ibmcloud ce buildrun get](https://cloud.ibm.com/docs/codeengine?topic=codeengine-cli#cli-buildrun-get):

```bash
ibmcloud ce buildrun get --name sh-storage-be-bld-run
```
і результат виконнная
```text
C:\PSHDEV\PSH-WorkShops\github-io\tz-000013-codeengine\code-engine-app\cenodebesrvc\tz-000001-init\cenodebesrvc\deployment>ibmcloud ce buildrun get --name sh-storage-be-bld-run
Getting build run 'sh-storage-be-bld-run'...
For troubleshooting information visit: https://cloud.ibm.com/docs/codeengine?topic=codeengine-troubleshoot-build.
Run 'ibmcloud ce buildrun events -n sh-storage-be-bld-run' to get the system events of the build run.
Run 'ibmcloud ce buildrun logs -f -n sh-storage-be-bld-run' to follow the logs of the build run.
OK

Name:          sh-storage-be-bld-run
ID:            2f5ed2d1-dd55-4df0-abf1-d1df5a1333f7
Project Name:  sh-storage-prh
Project ID:    15716649-8035-45d9-ad3f-d87e886838cd
Age:           4m45s
Created:       2023-05-26T10:43:52+03:00

Summary:       Succeeded
Status:        Succeeded
Reason:        All Steps have completed executing
Source:
  Commit SHA:     f7d05480be091a31b1db9ecc9425dd66e8c22b50
  Commit Author:  Pavlo Shcherbukha
Image Digest:  sha256:6eda768bfe355151080c4f35337d39d14f377e85ef5ca06d9970e013e2a60aa1

Build Name:  sh-storage-be-bld
Image:       uk.icr.io/sh-storage-prh-rgstr/sh-storage-be

C:\PSHDEV\PSH-WorkShops\github-io\tz-000013-codeengine\code-engine-app\cenodebesrvc\tz-000001-init\cenodebesrvc\deployment>
```

Таким чином, на цьому етапі ми вже маємо побудований контейнер, покладений в IBM container registry. В UI  результат можна побачити, пройшовши за кроками, що показані на [pic-17](#pic-17)

<kbd><img src="/assets/img/posts/2023-05-11-ibmcloud-codeengine/doc/pic-17.png" /></kbd>
<p style="text-align: center;"><a name="pic-17">pic-17</a></p>


### <a name="p-4.6">Application Depoyment. Запуск application</a> 

Запуск application відбувається комнадою [ibmcloud ce application create](https://cloud.ibm.com/docs/codeengine?topic=codeengine-cli#cli-application-create).

Для зручності змінні винеснеі в env та створений cmd файл що знаходиться в репозиторії в ./deployment/cli-crt-app-img.cmd.
На цьому етапі задаються env змінні, що необхідні для роботи контейнера. Зв'язування з сервісами IBM хмари виконуються іншою командою, після сторення application. Також хочу звернути увашу на реквізити  **--cpu 1 --memory 2G**, де вказується кількість ресурсів, що виділяються для даного application та **--max-scale** , **--min-scale** - що вказують на кількість єкземплярів.



```bash

set DB_URL=CLOUDANT_URL=https://****************-bluemix.cloudantnosqldb.appdomain.cloud
set DB_APIKEY=CLOUDANT_APIKEY=*********************************
set DB_NAME=STORAGE_DBNAME=storagedb
set BLDIMAGE= uk.icr.io/sh-storage-prh-rgstr/sh-storage-be
pause

ibmcloud ce application create --name sh-storage-be1 --image %BLDIMAGE% --registry-secret sh-storage-prh-rgstr --env %DB_URL% --env %DB_APIKEY% --env %DB_NAME% --cpu 1 --memory 2G --port 8080

pause


```

Команда виконується не зразу, а прохожить деякий час, поки application стартоне. Результат виконання показано на [pic-18](#pic-18)

<kbd><img src="/assets/img/posts/2023-05-11-ibmcloud-codeengine/doc/pic-18.png" /></kbd>
<p style="text-align: center;"><a name="pic-18">pic-18</a></p>


На [pic-19](#pic-19)  показано, як теж саме побачити через UI, та  поставлено наголос, що коли не задана
<kbd><img src="/assets/img/posts/2023-05-11-ibmcloud-codeengine/doc/pic-19.png" /></kbd>
<p style="text-align: center;"><a name="pic-19">pic-19</a></p>

кількість instances,  то за замовчуванням їх 0, і додаток не стартонутий. Це означає, що вхідний http  запит буде перший раз виконуватися дуже довго. щоб пофіксити, треба  задати мінімальну кількість instances 1. 
Але, з іншої сторони, з application  немає доступу до БД cloudant, який виконується командою: [ibmcloud ce application bind](https://cloud.ibm.com/docs/codeengine?topic=codeengine-cli#cli-application-bind) - тобто зв'язує code engine з іншими хмарними сервісами і наявність 0 instances мене повністю влаштовує, тому що додаток не крешиться із за недоступності сервісую. Тому переходимо до наступного кроку **application bind**.

### <a name="p-4.7">Application Depoyment. Зв1язування application з іншими хмарними сервісами **application bind**</a>

Виконується командою: [ibmcloud ce application bind](https://cloud.ibm.com/docs/codeengine?topic=codeengine-cli#cli-application-bind).
В даному випадку мені  порібно зв'язати appliction з БД Cloudant.

```bash

ibmcloud ce application bind --name sh-storage-be1 --service-instance Cloudant-0m   --service-credential "Service credentials-1"

```

За результатм на [pic-20](#pic-20) видно що зв'язування відбулося і application виконує redeployment. 

<kbd><img src="/assets/img/posts/2023-05-11-ibmcloud-codeengine/doc/pic-20.png" /></kbd>
<p style="text-align: center;"><a name="pic-20">pic-20</a></p>


Тепер залишився останній крок - це налаштувати горизонтальне та вертикальне масштабування, встановивши кількість інстансів,  ЦПУ, пам'ять

### <a name="p-4.8">Application Depoyment. Масштабування</a>

Оновимо наш application  командою [ibmcloud ce application update](https://cloud.ibm.com/docs/codeengine?topic=codeengine-cli#cli-application-update)

```bash
ibmcloud ce application update --name sh-storage-be1 --cpu 0.5 --memory 2G  --min-scale 1  --max-scale 3
```


За результатм на [pic-21](#pic-21) видно що ми досягли бажаної конфігурації. 

<kbd><img src="/assets/img/posts/2023-05-11-ibmcloud-codeengine/doc/pic-21.png" /></kbd>
<p style="text-align: center;"><a name="pic-21">pic-21</a></p>


Тепер можна переходити до тестування додатку.













