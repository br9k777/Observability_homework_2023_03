# Домашка 1 для Otus

## Задача
1. На виртуальной машине установите любую open source CMS которая включает в себя следующие компоненты: nginx, php-fpm, database (MySQL or Postgresql)
2. На этой же виртуальной машине установите Prometheus exporters для сбора метрик со всех компонентов системы (начиная с VM и заканчивая DB, не забудьте про blackbox exporter который будет проверять доступность вашей CMS)
3. На этой же или дополнительной виртуальной машине установите Prometheus задачей которого будет раз в 5 секунд собирать метрики с экспортеров
4. На этой же или дополнительной виртуальной машине установите Alertmanager и сконфигурируйте его таким образом чтобы в случае недоступности какого либо компонента был отправлен alert с важность Critical в один из канал оповещений (канал оповещений на выбор: slack or telegram)
- Задание со звездочкой 1 (повышенная сложность) - на VM с установленной CMS слишком много “портов экспортеров торчит наружу” и они доступны всем, попробуй настроить доступ только по одному и добавить авторизацию
- Если вы выполнили задание со звездочкой номер 1 то - добавить SSL

Как развернуть CMS взято [отсюда](https://admin812.ru/razvertyvanie-wordpress-s-nginx-php-fpm-i-mariadb-s-pomoshhyu-docker-compose.html)

## Требования перед запуском
  - docker
  - docker compose
  - plugin loki, ставится `make setup`

  Для удобства пользуюсь [direnv](https://github.com/direnv/direnv/blob/master/docs/installation.md).
Для создания окружения к нему выполнить `make prepare-dir-env`

## Запуск 
  `make run`
## Endpoints
 В качестве reverse proxy использован traefik
 саму cms спрятал за доменом http://cms.example.com/, его прописал в `/etc/hosts`
- prometheus - http://127.0.0.1/prometheus
- grafana - http://127.0.0.1/grafana
- traefik dashboard - http://127.0.0.1/dashboard/#/
- php admin для обращения к базе cms - http://127.0.0.1/database/
- prometheus - http://127.0.0.1/blackbox
- prometheus - http://127.0.0.1/alertmanager

## Exporters
- cadvisor для сбора метрик о контейнерах
- node-exporter для сбора метрик о сервер
- traefik для сбора метрик об обращениях к серверу
- mysql-exporter для сбора метрик базы mysql
- blackbox-exporter для проверки доступности компонентов системы

## пароли 
 Все пароли в Makefile, переменными в самом начале.
 (Для grafana `admin:hm1_grafana_pass`)

## результат
- сбор части метрик настроен через `docker_sd_configs` через labels на контейнерах.  
  Это удобно так как достаточно редактировать только compose файл.   
  Но работает только при работе объектами сервисов в k8s или swarm - если контейнер падает, prometheus 
  сам убирает job на него, и по функции `up` не удастся понять, что сбор метрик будет провален.  
- в alertmanager настроены два receivers `./homework/wordpress-compose/alertmanager/config.yml` :
  - `telegram` - для предупреждений которые могут сменить статус на resolved
  - `telegram_without_resolve` - для оповещений которы не могут сменить статус  
    (например `compose_service_down` - может зафиксировать только факт падения контейнера) 
- настроен alert rules в prometheus - `./homework/wordpress-compose/prometheus/alert.rules.yml` 
  - site_down - оповещение когда один из портов проверяемых через blackbox не прошел проверку
  - BlackboxSlowProbe - медленный http ответ от компонента
  - BlackboxProbeHttpFailure - не корректный http ответ от компонента
  - service_down - когда не удалось получить метрики

### Проверка оповещения
Останавливаем один из компонентов:
- `docker compose stop loki`
- `docker compose stop pma`

Смотрим результат в [prometheus](http://127.0.0.1/prometheus/alerts?search=)
и [alertmanager](http://127.0.0.1/alertmanager/#/alerts)    
Так же должно приходить оповещение в чат [Otus_home_works_notify](https://t.me/+5WJ6QCTIjWdiZTYy)


# Домашка 2 для Otus
## Задача
- Для выполнения данного дз воспользуйтесь наработками из предыдущего домашнего задания.
- На VM с установленным Prometheus установите Grafana последней версии доступной на момент выполнения дз
- Создайте внутри Grafana папки с названиями infra и app
- Внутри директории infra создайте дашборд который будет отображать сводную информацию по инфраструктуре (CPU, RAM, Network, etc.)
- Внутри директории app создайте дашборд который будет отображать сводную информацию о CMS (доступность компонентов, время ответа, etc.)
- Задание со звездочкой 1 - при помощи Grafana создайте alert о недоступности одного из компонентов CMS и инфраструктуры
- Задание со звездочкой 2 - создайте DrillDown dashboard который будет отображать сводную информацию по инфраструктуре, но нажав на конкретный инстанс можно получить полную информацию
- Результат: пере используйте репозиторий созданный для сдачи предыдущего ДЗ. Дополните Readme описание действий выполненных в результате выполнения данного дз. В директорию GAP-2 приложите скриншоты дашбордов которые вы создали.

## Процесс
  - grafana со всем необходимыми dashboards, собираем в образ  
    чтобы потом использовать в других системах
  - Dashboards взял из [готовых](https://grafana.com/grafana/dashboards)

## Результат 
Скрины в Observability_homework_2023_03/homework/GAP2

# Домашка 3 для Otus
## Задача
- Для успешного выполнения ДЗ вам необходимо установить ELK (elasticsearch, logstash, kibana).
- Базовая операционная система - по вашему выбору.
- После успешной установки ELK-стека вам необходимо настроить отправку логов sshd в elasticsearch через logstash.
- Для этого вам придется изменить настройку rsyslog.
- Проверьте создался ли index в elasticsearch.
- После настройки отправки логов в ELK попробуйте настроить визуализацию логов от sshd в kibana.
- В качестве результата ДЗ принимается: конфиг rsyslog, конфиг logstash и результат проверки index в elasticsearch, а также скриншот из kibana, если получилось настроить визуализацию.

## В процессе 
- пока настроена передача логов в loki
- rsyslog и efk стэк не настроены (да скорее всего буду делать через Fluentd)
