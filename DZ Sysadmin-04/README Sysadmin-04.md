# ДЗ Операционные системы. Лекция 2

1. На лекции мы познакомились с node_exporter. В демонстрации его исполняемый файл запускался в background. Этого достаточно для демо, но не для настоящей production-системы, где процессы должны находиться под внешним управлением. Используя знания из лекции по systemd, создайте самостоятельно простой unit-файл для node_exporter:

поместите его в автозагрузку,
предусмотрите возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на systemctl cat cron),
удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, а после перезагрузки автоматически поднимается.

Ответ:

Создаём файл unit-file для node_exporter.

![Alt text](../DZ Sysadmin-04/1.png)

Добавляем в автозагрузку.

![Alt text](../DZ Sysadmin-04/1.1.png)

2. Ознакомьтесь с опциями node_exporter и выводом /metrics по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.
   Ответ:
   Просмотр собираемых метрик - curl http://localhost:9100/metrics. Опции - cpu, diskstats, filesystem, meminfo.

3. Установите в свою виртуальную машину Netdata. Воспользуйтесь готовыми пакетами для установки (sudo apt install -y netdata).

После успешной установки:

в конфигурационном файле /etc/netdata/netdata.conf в секции [web] замените значение с localhost на bind to = 0.0.0.0,
добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте vagrant reload:
config.vm.network "forwarded_port", guest: 19999, host: 19999
После успешной перезагрузки в браузере на своем ПК (не в виртуальной машине) вы должны суметь зайти на localhost:19999. Ознакомьтесь с метриками, которые по умолчанию собираются Netdata и с комментариями, которые даны к этим метрикам.

Ответ:

![Alt text](../DZ Sysadmin-04/2.png)

4. Можно ли по выводу dmesg понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?
   Ответ:

Да, можно.

![Alt text](../DZ Sysadmin-04/3.png)

5. Как настроен sysctl fs.nr_open на системе по-умолчанию? Определите, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа (ulimit --help)?

Ответ:

fs.nr_open = 1048576
Данный параметр означает максимальное количество дескрипторов файлов, которые могут быть открыты.
ulimit -Sn - меняет мягкий лимит. Увеличивает количество открытых дескрипторов.
ulimit -Hn - меняет жесткий лимит. Мягкий лимит можно увеличить до значений жёсткого лимита. Максимальное значение жёсткого не может быть больше, чем параметр fs.nr_open.

6. Запустите любой долгоживущий процесс (не ls, который отработает мгновенно, а, например, sleep 1h) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через nsenter. Для простоты работайте в данном задании под root (sudo -i). Под обычным пользователем требуются дополнительные опции (--map-root-user) и т.д.
Ответ:

![Alt text](../DZ Sysadmin-04/4.png)

Цитата из man 2 uname: Part of the utsname information is also accessible via /proc/sys/kernel/{ostype, hostname, osrelease, version,
domainname}

7. Найдите информацию о том, что такое :(){ :|:& };:. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (это важно, поведение в других ОС не проверялось). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов dmesg расскажет, какой механизм помог автоматической стабилизации.
Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?

Ответ:

:(){ :|:& };: - это fork bomb. Это определяет функцию :, которая вызывает себя дважды (Код: : | :). Она делает это в фоновом режиме (&). После ; определение функции завершается и запускается функция :. Таким образом, каждый экземпляр : запускает два новых : и так далее, забивая память и вызывая зависание.
Отрабатывает механизм fork rejected by pids controller in /user.slice/user-1000.slice/session-3.scope. Он ориентируется на максимальное количество процессов в данном окружении. Параметр ulimit -u позволяет узнать и изменить это значение.
