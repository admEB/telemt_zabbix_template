# Zabbix Template: Telemt MTProxy (Active Checks)

## Описание

Шаблон **Telemt MTProxy by Zabbix agent 2 active** предназначен для глубокого мониторинга прокси-сервера Telemt MTProxy. 

Шаблон осуществляет комплексный сбор данных: от общих метрик сервера (аптайм, общее количество подключений, таймауты, рейты ошибок) до статистики по каждому пользователю с помощью обнаружения (LLD). Сбор телеметрии реализован через активные проверки (Active checks) агента. Решение для мониторинга серверов, находящихся за NAT, строгими правилами межсетевого экрана и максимальной закрытости из вне.

## Ключевые возможности и преимущества

* **Сбор данных (High Performance)**: Шаблон минимизирует количество обращений к серверу за счет использования зависимых элементов данных и предварительной обработки. Вся статистика запрашивается крупными блоками, а Zabbix самостоятельно парсит необходимые значения.
* **LLD пользователей**: Автоматическое обнаружение пользователей позволяет вести точечный мониторинг. Шаблон отслеживает не только базовую активность, но и контролирует утилизацию лимитов — рассчитывает процент используемых IP-адресов и TCP-соединений от максимально разрешенных для каждого конкретного пользователя.
* **Логика триггеров**: В шаблон заложены алерты для оперативного реагирования на инциденты: отслеживание перезагрузки службы и контроль качества соединений через вычисление процента ошибок.
* **Визуализация "из коробки"**: Включает автоматически создаваемые графики для каждого пользователя, общие графики производительности и интегрированный дашборд.

## Требования

Для работы шаблона на целевом хосте должны быть установлены:
* **Zabbix Agent 2**
* **curl** — для выполнения запросов к API
* **jq** — для парсинга и обработки JSON-данных

## Установка

### 1. Настройка Zabbix Agent 2
Добавьте следующие параметры в конфигурационный файл агента (например, в `/etc/zabbix/zabbix_agent2.d/telemt.conf`):

```bash
UserParameter=mtproto.stats,curl -s [http://127.0.0.1:9091/v1/users](http://127.0.0.1:9091/v1/users) | jq '[.data[] | {username, max_unique_ips: (.max_unique_ips // 0), max_tcp_conns: (.max_tcp_conns // 0), active_unique_ips, current_connections, usage_connections_percent: (if (.max_tcp_conns // 0) > 0 then (.current_connections * 100 / .max_tcp_conns) else 0 end), usage_ips_percent: (if (.max_unique_ips // 0) > 0 then (.active_unique_ips * 100 / .max_unique_ips) else 0 end)}]'
UserParameter=mtproto.discovery,curl -s [http://127.0.0.1:9091/v1/users](http://127.0.0.1:9091/v1/users) | jq '{data: [.data[] | {"{#USERNAME}": .username}]}'
UserParameter=mtproto.total.connections,curl -s [http://127.0.0.1:9091/v1/users](http://127.0.0.1:9091/v1/users) | jq '[.data[].current_connections] | add'
UserParameter=mtproto.total.ips,curl -s [http://127.0.0.1:9091/v1/users](http://127.0.0.1:9091/v1/users) | jq '[.data[].active_unique_ips] | add'
UserParameter=mtproto.summary,curl -s [http://127.0.0.1:9091/v1/stats/summary](http://127.0.0.1:9091/v1/stats/summary) | jq '.data'
