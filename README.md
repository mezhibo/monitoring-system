**Обязательные задания**

1. Вас пригласили настроить мониторинг на проект. На онбординге вам рассказали, что проект представляет из себя платформу для вычислений с выдачей текстовых отчетов, которые сохраняются на диск. Взаимодействие с платформой осуществляется по протоколу http. Также вам отметили, что вычисления загружают ЦПУ. Какой минимальный набор метрик вы выведите в мониторинг и почему?

2. Менеджер продукта посмотрев на ваши метрики сказал, что ему непонятно что такое RAM/inodes/CPUla. Также он сказал, что хочет понимать, насколько мы выполняем свои обязанности перед клиентами и какое качество обслуживания. Что вы можете ему предложить?

3. Вашей DevOps команде в этом году не выделили финансирование на построение системы сбора логов. Разработчики в свою очередь хотят видеть все ошибки, которые выдают их приложения. Какое решение вы можете предпринять в этой ситуации, чтобы разработчики получали ошибки приложения?

4. Вы, как опытный SRE, сделали мониторинг, куда вывели отображения выполнения SLA=99% по http кодам ответов. Вычисляете этот параметр по следующей формуле: summ_2xx_requests/summ_all_requests. Данный параметр не поднимается выше 70%, но при этом в вашей системе нет кодов ответа 5xx и 4xx. Где у вас ошибка?

5. Опишите основные плюсы и минусы pull и push систем мониторинга.

6. Какие из ниже перечисленных систем относятся к push модели, а какие к pull? А может есть гибридные?

 - Prometheus
 - TICK
 - Zabbix
 - VictoriaMetrics
 - Nagios

7. Склонируйте себе репозиторий и запустите TICK-стэк, используя технологии docker и docker-compose.
В виде решения на это упражнение приведите скриншот веб-интерфейса ПО chronograf (http://localhost:8888).

P.S.: если при запуске некоторые контейнеры будут падать с ошибкой - проставьте им режим Z, например ./data:/var/lib:Z

8. Перейдите в веб-интерфейс Chronograf (http://localhost:8888) и откройте вкладку Data explorer.

- Нажмите на кнопку Add a query

- Изучите вывод интерфейса и выберите БД telegraf.autogen

 - В measurments выберите cpu->host->telegraf-getting-started, а в fields выберите usage_system. Внизу появится график утилизации cpu.

-  Вверху вы можете увидеть запрос, аналогичный SQL-синтаксису. Поэкспериментируйте с запросом, попробуйте изменить группировку и интервал наблюдений.

Для выполнения задания приведите скриншот с отображением метрик утилизации cpu из веб-интерфейса.

Изучите список telegraf inputs. Добавьте в конфигурацию telegraf следующий плагин - docker:
```
[[inputs.docker]]
  endpoint = "unix:///var/run/docker.sock"
```

Дополнительно вам может потребоваться донастройка контейнера telegraf в docker-compose.yml дополнительного volume и режима privileged:

```
  telegraf:
    image: telegraf:1.4.0
    privileged: true
    volumes:
      - ./etc/telegraf.conf:/etc/telegraf/telegraf.conf:Z
      - /var/run/docker.sock:/var/run/docker.sock:Z
    links:
      - influxdb
    ports:
      - "8092:8092/udp"
      - "8094:8094"
      - "8125:8125/udp"
```
После настройке перезапустите telegraf, обновите веб интерфейс и приведите скриншотом список measurments в веб-интерфейсе базы telegraf.autogen . Там должны появиться метрики, связанные с docker.

Факультативно можете изучить какие метрики собирает telegraf после выполнения данного задания.





**Решение 1**

*Загрузка процессора (CPU) Вычисления загружают ЦПУ, мониторинг использования процессора является приоритетным. Это поможет отслеживать, не приближается ли система к пределам своих возможностей.*

*Метрики:*

 - Средняя загрузка ЦПУ (в процентах).

 - Загрузка по ядрам.

 - Количество процессов, ожидающих ЦПУ (load average).

 - Температура процессора и датчиков системы: полезно для серверов, особенно при высоких вычислительных нагрузках.


*Использование памяти (RAM) Нагрузки в процессе вычислений могут потреблять много оперативной памяти. Недостаток памяти может привести к использованию swap'а или сбоям в работе платформы.*

*Метрики:*

 - Использование оперативной памяти (в абсолютных значениях и процентах).

 - Объем использованного swap (индикатор нехватки памяти).


*Доступность и время отклика HTTP-сервера Взаимодействие с платформой осуществляется по протоколу HTTP, поэтому важно отслеживать ее доступность и время отклика, чтобы удостовериться, что она отвечает на запросы.*

*Метрики:*

 - Время ответа на HTTP-запросы (latency).

 - Статус-коды HTTP (чтобы отслеживать ошибки 4xx и 5xx).

 - Количество активных соединений.


*Дисковое пространство и I/O операции Важно следить за достаточностью дискового пространства, отчеты созраняются на диск. Также стоит отслеживать активность дисковых операций, так как интенсивный I/O может снижать общую производительность.*

*Метрики:*

 - Доступное дисковое пространство.

 - Нагрузка на диск (операции чтения/записи в секунду).

 - Время отклика диска (disk latency).


*Ошибки в работе приложения Ошибки в работе приложения могут указывать на проблемы в логике вычислений или интеграции. Логирование ошибок поможет оперативно выявлять и исправлять неполадки.*

*Метрики:*

 - Логирование критических ошибок или исключений (по уровням log severity).

Дополнительно: Нагрузка на сеть: если результаты вычислений передаются по сети, можно отслеживать объем передаваемых данных и активность сети.





**Решение 2**


1). RAM - Память, используемая для временного хранения данных и выполнения программ. inodes - Структуры данных в файловой системе, которые хранят информацию о файлах и каталогах. Ограниченное количество inodes может помешать созданию новых файлов CPUla - Загрузка ЦПУ показывает, насколько сильно он занят обработкой задач

Для того чтобы сделать метрики более понятными и удобными для анализа с точки зрения качества обслуживания и выполнения обязательств перед клиентами, можно использовать следующие подходы, связав их с понятиями SLA (Service Level Agreement), SLO (Service Level Objective) и SLI (Service Level Indicator)

*Определение ключевых показателей:*

SLA (Service Level Agreement): это соглашение, определяющее уровень услуг, которые клиент ожидает от провайдера. Например, "Платформа должна быть доступна 99.9% времени". SLO (Service Level Objective): это конкретные цели, которые команда ставит для достижения SLA. Например, "Время отклика на HTTP-запросы должно составлять не более 200 мс для 95% запросов". SLI (Service Level Indicator): это метрики, которые используются для измерения достижения SLO. Например: Доступность: процент времени, когда сервис был доступен. Время отклика: среднее время отклика на HTTP-запросы.

*Перевод технических метрик в понятные клиенту показатели*

Доступность платформы (Uptime) SLI: Процент времени, когда платформа была доступна. SLO: "99.9% доступности". Пояснение: платформа была доступна на 99.9% времени в прошлом месяце.

Время отклика на запросы

SLI: Среднее время отклика на HTTP-запросы. SLO: "90% запросов обрабатываются за менее чем 200 мс". Пояснение: 90% запросов обрабатывались быстрее, чем за 200 мс.

Число ошибок

SLI: Количество 4xx и 5xx ошибок за определённый период. SLO: "Количество ошибок не превышает 1% всех запросов". Пояснение: Ошибки при обработке запросов составляют менее 1% от общего числа.

Что упростит:

 - Создайте регулярные отчёты о выполнении SLO и состоянии сервисов. Это поможет менеджеру продукта видеть прогресс и принимать обоснованные решения.

- Поддерживайте открытый диалог с менеджером продукта, чтобы вместе определять, какие метрики наиболее важны для клиентов, и при необходимости корректировать SLO и SLI в соответствии с изменениями в бизнесе

 - Используйте простые и интуитивно понятные термины. Например, вместо "использование RAM" можно говорить "объём памяти, доступный для обработки ваших запросов"

 - Предоставляйте данные в виде графиков, диаграмм и дашбордов, чтобы менеджер продукта мог легко визуализировать информацию
