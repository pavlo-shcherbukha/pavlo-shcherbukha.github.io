---
layout: post
title: "Send messages to ms teams channels using incoming webhooks"
date: 2023-09-08 10:00:01
categories: [node-RED]
permalink: posts/2024-04-21/sendmsgtoteamschannel/
published: true
---

<!-- TOC BEGIN -->

<!-- TOC END -->

## <a name="p-1">Про що цей блог</a>

Виникла потреба в відправленні повідомлень в канал MS Teams. По суті відправка  алертів  про нештатні ситуації. В найпростішому варіанті відправку можна реалізувати через [Incoming WebHooks до MS Teams](https://learn.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook?tabs=newteams%2Cdotnet). По цьому ж лінку написано, як створити Incoming Webhook в каналі [reate an Incoming Webhook](https://learn.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook?tabs=newteams%2Cdotnet#create-an-incoming-webhook).

Відправка повідомлень виконується через [Adaptive Cards ](https://learn.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/connectors-using?tabs=cURL#send-adaptive-cards-using-an-incoming-webhook). Типи **Cards** та інтерфейсів, що використовують ті чи інші cards описані за лінком [Features that support different card types](https://learn.microsoft.com/en-us/microsoftteams/platform/task-modules-and-cards/cards/cards-reference#features-that-support-different-card-types). 

Є ще одна особливість: через Adaptive Card неможна передати вкладення. Можна передати лише  зображення по  url.  Про це можна прочитати за лінком [CSV file not posting to teams channel using webhook url in python code](https://learn.microsoft.com/en-us/answers/questions/1183652/csv-file-not-posting-to-teams-channel-using-webhoo)

> You cannot send file attachments using webhook, they only support card format.
> 
> You can use Bots to send or receive files to user - https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/bots-filesv4


Для розробк на Python існує уже готовий пакет [pymsteams 0.2.2](https://pypi.org/project/pymsteams/).

Додатково можна використати такі лінки:

- [How to Send Microsoft Teams Messages with Python](https://www.datacamp.com/tutorial/how-to-send-microsoft-teams-messages-with-python);
- [github AdaptiveCards 2020.07](https://github.com/microsoft/AdaptiveCards/releases/tag/2020.07)
- [How to send message to teams using python.](https://dev.to/shadow_b/how-to-send-message-to-teams-using-python-496g)




## <a name="p-2">2. Постановка завдання для прототипу</a>

Для вивчення робити з Incoming webhook потрібно розробити webservice для выдправки повыдомлень в канал MSteams використовуючи  python flask webservice

https://github.com/pavlo-shcherbukha/pyteamsmsg

