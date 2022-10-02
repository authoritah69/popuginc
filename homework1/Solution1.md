# Домашка №0 

[Решение](https://github.com/authoritah69/popuginc/blob/master/homework0/popug.jpeg) нулевого урока в папке `homework0`, 
уже в мастере. На картинке опечатка, не `AnalyticsPusher`, a `AccountingPusher` (уже удалил борду, не могу поправить)


# Домашка №1

[Линк](https://miro.com/app/board/uXjVPR-JmmY=/?share_link_id=56841548359) на ивент-шторминг и модели
данных в Miro. 

### Ивент шторминг: 
Все в миро. Как надо рисовать бизнес цепочки в ивент шторминге я пока так и не понял, 
надеюсь получится улучшить позже.

### Данные в доменах и зависимости:
Все в миро. 3 домена - `Common`, `Task` и `Accounting`.

`Task` зависит от `Common`.

`Accounting` зависит от `Common`.

Своего домена у `Analytics`, похоже, нет, но сам сервис зависит от `Common` и `Task` доменов.  

### Сервисы:
Хоть нулевую домашку делал впопыхах, после первой домашки общая картина сервисов почти
не поменялась (линк сверху страницы) - некий `TaskWriter` получает по какому-то API команды 
от юзеров создать, перееасайнить и закрыть таски и пишет соответствующие ивенты в кафку. Все заинтересованные 
консьюмеры асинхронно их подбирают и агрегатят как хотят, пишут в базы на чтение. 
Друг на друга эти консьюмеры не зависят никак. Второй кафка продюсер - это некий `AccountingPusher`, 
который по внутреннему шедулеру раз в день запускает финансовые подсчеты (ну или экспозит очередной API
и его раз в день дергает по крону какая-то CI джоба). 

Единственный сервис, с которым строго синхронно общается каждый user-exposed сервис - `PopugAuth`,
потому что аутентификация и создание секьюрных сессий. 


### Бизнес События

#### Event: TaskCreated
Producer: `TaskWriter`

Consumers: `AccountingAggregator`, `TaskAggregator`, `AnalyticsAggregator`

Payload: 
```json
{
  "type": "Tasks.TaskCreated",
  "id": "TES-100500",
  "title": "Free rides for promotion purposes",
  "description": "First 3 parrot booty call rides are for free",
  "assignee": "Grzegorz Brzęczyszczykiewicz",
  "worth": 14,
  "timestamp": "2022-10-02T14:38:40.108Z"
}
```

#### Event: TaskClosed
Producer: `TaskWriter`

Consumers: `AccountingAggregator`, `TaskAggregator`, `AnalyticsAggregator`

Payload: 
```json
{
  "type": "Tasks.TaskClosed",
  "id": "TES-100500",
  "timestamp": "2022-10-02T14:38:40.108Z",
  "completedBy": "Grzegorz Brzęczyszczykiewicz"
}
```


#### Event: TaskReassigned
Producer: `TaskWriter`

Consumers: `AccountingAggregator`, `TaskAggregator`

Payload:
```json
{
  "type": "Tasks.TaskReassigned",
  "id": "TES-100500",
  "timestamp": "2022-10-02T14:38:40.108Z",
  "newAssignee": "Grzegorz Brzęczyszczykiewicz"
}
```

#### Event: ParrotCredited
Producer: `AccountingPusher`

Consumers: `AccountingAggregator`, `AnalyticsAggregator` 

Payload:

```json
{
  "type": "Accounting.ParrotCredited",
  "id": "Grzegorz Brzęczyszczykiewicz",
  "timestamp": "2022-10-02T14:38:40.108Z",
  "txId": "d2f1de52-8fa9-4f4e-95c3-980269d26a45",
  "reason": {
    "type": "ClosedTask",
    "details": "TES-100500"
  },
  "amount": "35"
}
```

#### Event: NegativeBalanceCarriedOver
Producer: `AccountingPusher`

Consumers: `AccountingAggregator`, `AnalyticsAggregator`

Payload:

```json
{
  "type": "Accounting.NegativeBalanceCarriedOver",
  "id": "Grzegorz Brzęczyszczykiewicz",
  "timestamp": "2022-10-02T23:59:59.999Z",
  "total": -15
}
```

### CUD события
Таких пока не вижу, все бизнес. Что я упускаю?