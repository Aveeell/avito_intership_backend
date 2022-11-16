# Тестовое задание на позицию стажёра-бэкендера
---
##Стек
* **Язык разработки**: *Golang*
* **БД**: *MySQL*
* **Драйвер БД**: *sql*
---
## Запуск приложения

`docker-compose up`

---

##Архитктура БД

Таблица **users**
Содержит в себе информацию о пользователях:
```
+----+---------+----------+
| id | balance | reserved |
+----+---------+----------+
```

Таблица **products**
Содержит в себе информацию о услугах:
```
+----+----------+-------+
| id | title    | price |
+----+----------+-------+
```

Таблица **history**
Содержит в себе историю всех операций:
```
+----+---------+-------------+------+---------+
| id | user_id | transaction | date | comment |
+----+---------+-------------+------+---------+
```

Таблица **bookkeeping**
Содержит в себе данные о подтвержденных покупках:
```
+----+---------+---------+-------+------+
| id | user_id | product | price | date |
+----+---------+---------+-------+------+
```

Все таблицы, за исключением **products**, создаются пустыми
Таблица products содержит в себе три записи:
```
+----+----------+-------+
| id | title    | price |
+----+----------+-------+
|  1 | guitar   |   500 |
|  2 | drum set |  1500 |
|  3 | pick     |     5 |
+----+----------+-------+
```

---
##Задания

1) Метод начисления средств на баланс. Принимает id пользователя и сколько средств зачислить. ✅

Реализован метод **POST** `/user/balance/add`
Если пользователя с таким id не существует, то создается новая запись в БД, также добавляется запись в историю с комментарием `user creation` или `edit balance`
```
{
    "id"    //id пользователя
    "sum":  //сумма зачисления
}
```

2) Метод резервирования средств с основного баланса на отдельном счете. Принимает id пользователя, ИД услуги, ИД заказа, стоимость. ✅

Реализован метод **POST** `/order/create`
Резервирует сумму с баланса пользователя и добавляет запись в историю транзакций с комментарием `trying to buy X`
```
{
    "order_id":     //id заказа
    "user_id":      //id пользователя
    "product_id":   //id услуги
    "sum":          //стоимость услуги
}
```

3) Метод признания выручки – списывает из резерва деньги, добавляет данные в отчет для бухгалтерии. Принимает id пользователя, ИД услуги, ИД заказа, сумму. ✅

Реализован метод **POST** `/order/approve`
Списывает сумму с резерва пользователя и добавляет запись в историю транзакций с комментарием `approved purchase X`
```
{
    "order_id":     //id заказа
    "user_id":      //id пользователя
    "product_id":   //id услуги
    "sum":          //стоимость услуги
}
```

4) Метод получения баланса пользователя. Принимает id пользователя. ✅

Реализован метод **GET** `/user/balance`
Показывает состояние баланса пользователя
```
{
    "id" //id пользователя
}
```
```
{
    "ID": 1,
    "Balance": 1500,
    "Reserved": 0
}
```
---
##Дополнительные задания

1) Реализовать метод для получения месячного отчета. На вход: год-месяц. На выходе ссылка на CSV файл. ✅

Реализован метод **GET** `/bookkeeping`
Формируется файл **data.csv**, который можно скачать по ссылке в ответе сервера
```
{
    "date"  //дата в формате "yyyy-mm"
}
```
```
{
    "Link": "http://localhost:8080/bookkeeping/download"
}
```

2) Необходимо предоставить метод получения списка транзакций с комментариями откуда и зачем были начислены/списаны средства с баланса. Необходимо предусмотреть пагинацию и сортировку по сумме и дате. ✅

Реализован метод **GET** `/history`
Принимает следующие параметры (*id обязателен, остальные - нет*):
```
{
    "id"        //id пользователя
    "offset"    //пропуск первых N записей
    "limit"     //максимальное количество записей
    "sort"      //тип сортировки <DESC | ASC>
    "sort_col"  //по какому столбцу производить сортировку
}
```

**Запрос**
```
{
    "id": 1
}
```
**Ответ**
```
[
    {
        "ID": 1,
        "Transaction": 1500,
        "Date": "2022-11-16T21:43:45Z",
        "Comment": "user creation"
    },
    {
        "ID": 2,
        "Transaction": 500,
        "Date": "2022-11-16T21:44:31Z",
        "Comment": "trying to buy guitar"
    },
    {
        "ID": 3,
        "Transaction": 500,
        "Date": "2022-11-16T21:45:09Z",
        "Comment": "approved purchase guitar"
    }
]
```

####Примеры пагинации
**Запрос**
```
{
    "id": 1,
    "offset": 1,
    "limit": 2
}
```
**Ответ**
```
[
    {
        "ID": 2,
        "Transaction": 500,
        "Date": "2022-11-16T21:44:31Z",
        "Comment": "trying to buy guitar"
    },
    {
        "ID": 3,
        "Transaction": 500,
        "Date": "2022-11-16T21:45:09Z",
        "Comment": "approved purchase guitar"
    }
]
```
**Запрос**
```
{
    "id": 1,
    "sort_col": "id",
    "sort_type": "DESC"
}
```
**Ответ**

```
[
    {
        "ID": 3,
        "Transaction": 500,
        "Date": "2022-11-16T21:45:09Z",
        "Comment": "approved purchase guitar"
    },
    {
        "ID": 2,
        "Transaction": 500,
        "Date": "2022-11-16T21:44:31Z",
        "Comment": "trying to buy guitar"
    },
    {
        "ID": 1,
        "Transaction": 1500,
        "Date": "2022-11-16T21:43:45Z",
        "Comment": "user creation"
    }
]
```