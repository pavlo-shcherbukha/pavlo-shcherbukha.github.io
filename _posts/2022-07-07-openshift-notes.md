---
layout: post
title: "Нотатки з Openshift"
date: 2022-07-07 10:00:01
categories: [open-shift]
permalink: posts/2022-07-07/openshift-notes/
published: true
---
 

# Нотатки з управління Openshift

Тут використовуються версія  Openshit OKD 4.6 і вище


## Видалити Pod, що "завис" в стані terminating

Pods можуть зависати в стані завершення через роботу finalizers або запущений ресурс не закривається (не виключається). Як правило, pods слід дозволити завершити самостійно, оскільки вони існують для очищення будь-чого після контейнера.

```bash
oc delete pod podname -n namesapce  --grace-period=0 --force

```

Отримати дані для  metadata -> finalizersта перевірити стан помилок можна через  POD yaml

```bash
 oc get pod example-pod-name-1-xyz -n namespace -o yaml > example-pod.yaml
```
Інформація про статус та помилку будуть в lastState terminated section.


https://www.techbeatly.com/how-to-delete-a-pod-with-terminating-state-in-openshift-or-kubernetes-cluster/
https://access.redhat.com/solutions/2317401



## Як масштабувати  кількість POD-ів


Масшабується такою командою.   'dc/bankid-p' - це назва DeploymentConfig

```bash
oc scale --replicas=1 dc/bankid-p
```

Якщо потрібно повністю зупинити  параметр --replicas  потрібно скинути в 0 

```bash

oc scale --replicas=0 dc/bankid-p

```

## Прочитати логи з PODа


Прочитати логи з PODа можна командою

```
  oc logs  pod-name

```  

Але, якщо потрібно весь час читати логи, то додаемо ключ -f

```
  oc logs  pod-name  -f

```  


## Копіювати файли з PODа на робочу станцію

```
oc rsync pod-name:/opt/app-root/src/logs log
```

В цьому прикладі з pod по path /opt/app-root/src/logs файли копіюються в каталог log на робочій станції

А в цьому прикладі файли з поточного каталога робочої станції [.] копіюються на POD pod_name в каталог  [/opt/app-root/src]

```bash 
echo "copy files"
oc rsync . pod_name:/opt/app-root/src --delete --no-perms=true --watch=true
cd ..
```

## Відкотитися на вибрану версію успішного  deployment

Інколи виникає ситуація, що вимагає відкотитися на попредню, не обов'язково останню версію deployment

Більш детально можна подивитися за лінком [ Openshift 4.7. Rolling back a deployment](https://docs.openshift.com/container-platform/4.7/applications/deployments/managing-deployment-processes.html)


А якщо коротко то: 

- Переглянути історію deployment   

```bash
   oc rollout history dc/mysrvc-p

```

В результаті будемо  мати щось таке:

```text
REVISION        STATUS          CAUSE
38              Complete        image change
39              Complete        image change
40              Failed          newer deployment was found running
41              Complete        newer deployment was found running
42              Complete        config change
43              Complete        image change
44              Complete        image change
45              Complete        image change
46              Failed          newer deployment was found running
47              Complete        config change
48              Complete        image change
````

- Вернутися на вибрану revision:  

В даному випадку стартуємо з revision з номером 40

```bash

oc rollout undo dc/mysrvc-p  --to-revision=50

```

При цьому слід прийняти до уваги, що при rollout на попередню версю відключаютья всі тригери в DecpoymntConfig. І це зупинить автоматичний deployment.  

- Повернути все  в автомат: 

```bash

oc set triggers dc/mysrvc-p --auto

```