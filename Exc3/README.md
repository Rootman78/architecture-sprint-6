# Задание 3. Переход на Event-Driven архитектуру

## Проблемы, связанные с увеличением нагрузки:

1. При большом количестве обращений к системам страховых компаний, в связи с увеличением их количества, мы будем вынуждены регулярно посылать большее количество запросов к разным источникам. Это сильно увеличит нагрузку.
2. Синхронные запросы к большому количеству систем страховых компаний в случае неудачных попыток связи будут тратить ресурсы системы
3. В случае отсутствия связи хотя бы с одной из компаний, мы можем не получить полной информации о тарифах, и не сможем продавать страховки по актуальным тарифам
4. В случае запроса ins-comp-settlement к ins-product-aggregator через REST раз в сутки, в случае сбоя, мы не получаем вовремя информацию. Обновить ее мы сможем только через день, при следующем запросе.
5. При запросе из  core-app к ins-product-aggregator каждые 15 минут избыточны, так как тарифы меняются не так часто. Правильно было бы получать данные только по факту изменений.
6. При запросе ins-comp-settlement к core-app могут возникнуть те же проблемы, что и в пункте 4 

## Варианты решения:
Схема в файле InsureTech_C4_сontainer-diagram_new.drawio.xml

1. Для взаимодействия между ins-comp-settlement, ins-product-aggregator, core-app использовать очередь брокера сообщений Apache Kafka. Для каждого взаимодействия используем отдельный топик. Apache Kafka развернем на нескольких серверах для обеспечения отказоустойчивости. Это решение позволит решить проблемы 4, 5, 6
2. Со взаимодействием между  ins-product-aggregator, ins-comp-settlement и страховыми компаниями я не нашел 100% удачного решения. Есть 2 варианта решения в зависимости от способа взаимодействия с внешними сервисами, на которые мы не можем влиять. Первый вариант я изобразил на схеме:
	1. Планирую использовать брокер сообщений Apache Kafka для взаимодействия между внутренними и внешними сервисами. Для этого настроем Kafka в режиме "точка-точка", то есть будем гарантировать, что доставка 100% проходит в один конец и нет потери данных, которые могли бы произойти при падении серверов страховых компаний. Так же используем разные топики для каждого взаимодействия. Брокера будет дополнять модуль Kafka Connect, который позволяет взаимодействовать с конечными точками выстраивая запросы в очередь. Это решило бы проблему 1, 2, 3.
	2. Этот вариант будем использовать в случае, если страховая не может работать с очередями сообщений. В этом случае нам придется, как и раньше, синхронно использовать REST / SOAP / GraphQL. В дополнение мы используем паттерн Circuit Breaker, благодаря которому в случае падения API страховых компаний, мы не будем тратить ресурсы на пустые запросы. Это решит проблему описанную в пункте 2
	
## Вопросы по паттернам Event-Streaming и Transactional Outbox:

Я не вижу на этой схеме необходимости в применениях этих паттернов так как:
1. Event-Streaming будет избыточен, так как нет необходимости в непрерывных потоках данных, которые сильно нагружает систему
2. Transactional Outbox нам то же не потребуется, так как у большинства документов( как договоры, данные лиц и т.д) есть какие-то уникальные поля либо группа уникальности, по которой при чтении сообщений из Kafka мы сможем понять, что оно не дублируется. А дополнительные обращения к outbox таблице требуют дополнительные ресурсы. 


