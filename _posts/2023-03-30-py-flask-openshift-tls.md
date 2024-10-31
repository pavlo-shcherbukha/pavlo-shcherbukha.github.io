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

При розробці сучасних багатосервісних систем частно виникає необхідність захисту даних на трансопортному рівні. Тому це було першопричиною для того щоб розібратися в цьому, а потім написати цей блог. А в процесі вивчення теми виникли додаткові аргументи в цій сфері.

На приклад маємо таку архітектуру компонентів:

<kbd><img src="/assets/img/posts/2023-03-30-py-flask-openshift-tls/doc/pic-02.png" /></kbd>
<p style="text-align: center;"><a name="pic-02">pic-02</a></p>

Синіми та чорними лініями показана дозволена взаємодія клієнта (сині) та приладних сервісів (чорні). Червона лінія показує  заборонену взаємодію. Може виникнути питання, чому ця дія заборонена. На компоненті RestApi скоріше за все немає аутентифікації користувача. Більш того, backend сервіси можуть організовувати с RestApi сервісом взаємодію, по специфічному шаблону взаємодії типу: webpooling чи webhook -  для того щоб не перевантажити сервіс, чи отримати дані послідовно, порціями, а потім користувачу віддати вже все. При чому, customer,  це не завжди вреднючий користувач. Просто він користується такми фронтом, де програміст зробив "покращення", тому що так швидше.


Тут стає питання, як забезпечити тільки дозволені комунікації між сервісами. Більш того, За звичай існує 2 середовища: продуктивне та тестове. І, потрібно максимально ізолювати їх між собою. І не завжди мереживні технології дозволяють це зробити. Тому, одним із швидих  методів дозволити тільки очевидні комунікації між сервісами може забезпечити використання TLS  протоколів з різними типами атворизації. Звичайно, це буде вимагати впровадження процесу управління ключами. Але Для тестовго і продуктового середовища можна випустити різні набори сертифікатів і тоді сервіси буде важко переплутати. 



Можна побудувати для захисту таку, архітектуру для приклау.

<kbd><img src="/assets/img/posts/2023-03-30-py-flask-openshift-tls/doc/pic-03.png" /></kbd>
<p style="text-align: center;"><a name="pic-03">pic-03</a></p>

Користувач  володіє тільк кореневим сертифікатом, що видано всьому додатку і може зати на  тільки ті сервіси, що  приймають сервісну авторизацію. І не запитують сертифікат клієнта. А сервіси уже комунікують сіж собою за домогою клієнтської аутентифікації. Тобто Сервер до якого звертається клієнт , запитує клієнтський сертифікат. Щоб розубратися в тому як працює TLS За лінком:  [tls-self-sign-certs Набрі кроків для генерації самопідписних TLS сертифікатів](https://github.com/pavlo-shcherbukha/tls-self-sign-certs#tls-self-sign-certs-%D0%BD%D0%B0%D0%B1%D1%80%D1%96-%D0%BA%D1%80%D0%BE%D0%BA%D1%96%D0%B2-%D0%B4%D0%BB%D1%8F-%D0%B3%D0%B5%D0%BD%D0%B5%D1%80%D0%B0%D1%86%D1%96%D1%97-%D1%81%D0%B0%D0%BC%D0%BE%D0%BF%D1%96%D0%B4%D0%BF%D0%B8%D1%81%D0%BD%D0%B8%D1%85-tls--%D1%81%D0%B5%D1%80%D1%82%D0%B8%D1%84%D1%96%D0%BA%D0%B0%D1%82%D1%96%D0%B2) описаний простий набір кроків як розробнику швидко строрити набір tls сертифікатів та демка на Node.js express  як перевірити їх працездатність. Тут можна легко згенерувати набір сертифікатів для серверно та клієнтської аутентифікації. Також наведені приклади на Node.JS для перевірки робти різних варіантів аутентифікацій, використовуючи тільк що згенеровані ключі та сертифікати.  Ці кроки уже кілька разів використовував, коли виникала необхідність змоделювати роботу інших сервісів. Виявилося дуже зручно.  Звичайно, наведена архітектура не едний можливий шлях. Це вже  прилінкувалося у зв'язку з вивченнм, як використати TLS з Python-Flask додатками. Основним завданням у мене було  розібратися, як запустити Python flask сервіси за tls  протоколом на Openshift. Найбльш коротко і лаконічно  описано  про можливості  використання транспортного захисту в роутерах openshift за лінком [Secure your-microservices with RedHat OpenShft Routes](https://masamh.gitbook.io/secure-your-app-on-rhos/). Ось [pic-01](pic-01)

<kbd><img src="/assets/img/posts/2023-03-30-py-flask-openshift-tls/doc/pic-01.png" /></kbd>
<p style="text-align: center;"><a name="pic-01">pic-01</a></p>


показані  комбінації  можливих варіантів захисту.

1. Без захисту, по чистому http. Трафіук взагалі ніяк не шифрується
2. В цьому варіанті, трафік шифрується до  самого роутера OpenShift,  а далі використовується чистий http.
3. До роутера трафік захищений одним набороб сертифікатів, а після роутера і до самого сервісу трафік шифрується уже іншим набором.
4.  Роутер просто пропускає через себе TLS  трафік, і розшифровується він уже на самому сервісі.


Щоб якось перевірити це та зрозуміти як це робити розроблений приклад за лінком: [tls-pyflask-srvc Запуск Python-Flask application за https](https://github.com/pavlo-shcherbukha/tls-pyflask-srvc#tls-pyflask-srvc-%D0%B7%D0%B0%D0%BF%D1%83%D1%81%D0%BA-python-flask-application-%D0%B7%D0%B0-https). В цьому репозиторіх, як раз і використані підходи 2 та 4 з [pic-01](pic-01).

В проекті існує 2 приклади deployment:

- **\openshift\ubi8_docker_deployment**  TLS  доходить до Flask application

- **\openshift-tls-edge\ubi8_docker_deployment** TSL  доходить тількт до route openshift,  а далі трафік уже не захищений.

В варіанті **\openshift\ubi8_docker_deployment** python запускається за gunicorn, тому ніяких змін в python код вносити не потрібно. Потрібно зробити 2 кроки, а саме:

- всі кореневі  (CA) сертифікати, що є в ланцюжку потрібно покласти в трастове сховище Linux. Це робиться при зборці docke image і описується Dockerfile.

Ось цитата з Dockerfile, для Redhat Ubi8-python базового образу

```bash
# Put self signed CA into trust store
RUN cp /tmp/src/sh_app/tlscert/ca-crt.pem /usr/share/pki/ca-trust-source/anchors/
RUN ls /usr/share/pki/ca-trust-source/anchors/
RUN update-ca-trust

```

- Серврений ключ та серверний сертифікат вносяться в спеціальні ключі командного рядку при запуску gunicorn: --keyfile --certfile 

```yaml
        spec:
          containers:
          - env:
            - name: GUNICORN_CMD_ARGS
              value: --workers=1 --worker-connections=2000  --bind=0.0.0.0:8080 --access-logfile=---keyfile /opt/app-root/src/sh_app/tlscert/server-key.pem --certfile /opt/app-root/src/sh_app/tlscert/server-crt.pem


```
Ну і далі в програмному коді нічго міняти не портібно в Deploymentconfig  також ні, окрім роутера. В роутері додається додатковий ключ **tls** із значеннмя termination: passthrough 

```yaml
    spec:
      host: "${HOSTNAME}"
      port:
        targetPort: ${{PORTNUMBER}}
      # новий розділ start  
      tls:
        termination: passthrough
      # новий розділ stop  
      to:
        kind: "Service"
        name: ${APP_SERVICE_NAME}
        weight: null


```


- В варіанті **\openshift-tls-edge\ubi8_docker_deployment**  запуск самого додатку взагалі ніяк не відрізняється від запуску в режимі без захисту. Але основні зміни виконуються в роутері. Створюється так званий edg- роутер, куди додаються зненеровані CA та серврені ключ та сертифікат.

В роутері з`являється ноий блок **tls**, куди  вносяться сертифікати та ключі

```yaml
    spec:
      host: "${HOSTNAME}"
      port:
        targetPort: ${{PORTNUMBER}}
      # новий розділ start  
      tls:
        termination: edge
        certificate: |
          -----BEGIN CERTIFICATE-----
          # 
          -----END CERTIFICATE-----
        key: |
          -----BEGIN PRIVATE KEY-----

          -----END PRIVATE KEY-----
        caCertificate: |
          -----BEGIN CERTIFICATE-----
   
          -----END CERTIFICATE-----
        insecureEdgeTerminationPolicy: Allow
      wildcardPolicy: None
      # новий розділ stop
      to:
        kind: "Service"

```


### Генерація сертифікатів TLS

Принципи та послідовність короків для генераціх сертифікатів  описана за [tls-self-sign-certs Набрі кроків для генерації самопідписних TLS сертифікатів](https://github.com/pavlo-shcherbukha/tls-self-sign-certs#readme). 

- Для варіанту **\openshift\ubi8_docker_deployment** достатньо поміняти тільки свої реквізити в конфігураційних файлах openssl в розділі req_distinguished_name, залишивши localhost в CN  для серверного сертифікату.

Для варіанту **\openshift-tls-edge\ubi8_docker_deployment** в сертифікатах в реквізиті CN  вказував маску URL в ca.cnf  та в server.cnf 

```text
CN                     = *.apps.sandbox-myowndomain.openshiftapps.com
```

Використовується серверна аутентифікація. Тому у відповідності з посиланням- сертифікати генеруються в 4 кроки.
1









