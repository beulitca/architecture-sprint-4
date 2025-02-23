
# Выбор и настройка мониторинга в системе

## Мотивация:
Мониторинг - часть требований от бизнеса, без данных о сторонних пользователях API сложно принимать бизнес решения.

В условиях потери клиентов необходимо оперативно выявить слабые места системы на основе наблюдений за суточной активностью отдельных узлов и спланировать оптимальный ход работ и затрат по их устраненнению. Учитывая низкую скорость поставки, в первую очередь внимание следует уделить тем узлам, которые принесут максимальный позитивный эффект при минимальном объеме разработки.  

Мониторинг представляет собой корректирующее действие по предотвращению потери клиентов в будущем и обеспечению своевременной реакции на сбои в системе до того, как проблема коснулась пользователя. 

Позволит сократить время и стоимость разработки, обеспечить стабильный рост числа клиентов и получение прибыли.


## Конфигурация мониторинга
Все метрики для Prometheus должны иметь метки instance и job

### Мониторинг БД (4 золотых сигнала)
Мониторинг должен подходить в случае добавления кеширования.

Метрики:

Duration вместо Latancy
<span style="color:red">Response Time of shop db instance</span>
<span style="color:red">Response Time of MES db instance</span>
<span style="color:red">Response Time of S3 storage</span>

Трафик:
Number of connections for shop db instance
Number of connections for MES db instance

Ошибки:
<span style="color:red">Number of errors for shop db instance</span>
<span style="color:red">Number of errors for MES db instance</span>

Насыщенность:
Memory Utilisation for shop db instance
Memory Utilisation for MES db instance

Дополнительно, контроль диска:
Size of S3 storage
Size of shop db instance
Size of MES db instance

### Мониторинг API (RED)
Requests Rate:
Для отслеживания роста нагрузки и соотношения с числом ошибок
Number of requests (RPS) for internet shop API
Number of requests (RPS) for CRM API
Number of requests (RPS) for MES API

Number of simultanious sessions for shop API
Number of simultanious sessions for CRM API
Number of simultanious sessions for MES API

Для отслеживания эффективности работы API, отсутствия зависаний при наличии большого количества идентичных запросов
Number of requests (RPS) per user for internet shop API
Number of requests (RPS) per user for CRM API
Number of requests (RPS) per user for MES API

Errors:
Number of HTTP 200 for shop API
Number of HTTP 200 for CRM API
Number of HTTP 200 for MES API
Number of HTTP 200 for MES API с меткой method="GET"
Number of HTTP 500 for shop API
Number of HTTP 500 for CRM API
Number of HTTP 500 for MES API

Duration:
Response time for shop API
Response time for CRM API
Response time for MES API

Дополнительно (т к сервисы работают с тяжеловесными 3d-моделями):
Memory Utilisation for shop API
Memory Utilisation for MES API

CPU % for shop API
CPU % for MES API


### Мониторинг очереди (USE, т к есть возможность считать число необработанных сообщений в буфере и корректировать настройки back pressure, ручной разбор китичных сообщений)

Utilization:
Number of message in flight in RabbitMQ

Saturation:
Number of dead-letter-exchange letters in RabbitMQ

Errors:
Number of dead-letter-exchange letters in RabbitMQ 

## План действий

1. Определить набор отслеживаемых метрик
2. Собрать данные - Развернуть агентов-экспортеров Prometheus в узлах:
shop db instance, MES db instance, S3 storage
shop API, CRM API, MES API
RabbitMQ
3. Отправить данные на хранение и анализ - Поднять сервер наблюдаемости:
развернуть компоненты Prometheus(Retrieval,TSDM, HTTP-сервер) и Grafana - настроить соответствующие yml файлы для систем, указать в разделе scrape_configs для Prometheus список сервисов для извлечения метрик, составить общий docker-compose, запустить контейнеры
4. Сформировать отчеты - настроить борды в Grafana по собираемым метрикам. В т ч: для API MES полезно будет визуализироать метрику  RPS / CPU Usage, построить гистограммы по числу ошибок API.  
5. Настроить уведомления в Grafna или Prometheus Alertmanager на e-mail Тимлида, DevOps, PO, Тестировщика;  в Telegram Тимлида, DevOps, Тестировщика




