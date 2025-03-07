# Задание 5. Проектирование GraphQL API

## Ключевые ресурсы

На основе контракта Swagger выявлены следующие сущности:
1. Client
2. Document
3. Relative
И следующие запросы:
1. Получение информации о клиенте по ID (/clients/{id})
2. Получение списка документов клиента по ID (/clients/{id}/documents)
3. Получение информацию о родственниках клиента по ID (/clients/{id}/relatives)

## Эквивалентная схема GraphQL

Схема as is в GraphQL_as_is.txt

### Запросы на получение данных по этой схеме

1. Информация о клиенте по ID

query {
 clients(id: "id") {
    id
    name
    age
 }
}

2. Информация о документах клиента по ID

query {
 documents(id: "id") {
    id
    type
    number
    issueDate
    expiryDate
 }
}

3. Информация о родственниках клиента по ID

query {
 relatives(id: "id") {
    id
    relationType
    name
    age
 }
}

## Новые запросы GraphQL
Предполагаю, что в запросах от веб приложения и core-app может потребоваться полная информация о клиенте, которая включает информацию о клиенте, его родственниках и его документах. Данный запрос назовем client_details и добавим в type Query. Добавим type ClientDetails

Схема в to be в файле GraphQL_to_be.txt

### Запросы на получение данных по всем клиентам

Полная информация по клиенту. Убираем дублирующийся id у документов и родственников клиента

query {
    client_details(id: "id") {
        client {
            id
            name
            age
        }
        documents {
            type
            number
            issueDate
            expiryDate
        }
        relatives {
            relationType
            name
            age
        }
    }
}


 