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



## Як масштабувати  кількісьб POD-ів


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



